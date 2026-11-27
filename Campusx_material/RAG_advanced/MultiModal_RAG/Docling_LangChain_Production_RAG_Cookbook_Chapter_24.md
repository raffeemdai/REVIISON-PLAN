# Chapter 24: Production Cookbook — LangChain + Docling for Multimodal RAG

This chapter shows a production-oriented pattern for processing PDFs with text, headings, tables, images, scanned pages, multi-column layouts and charts, then indexing searchable content with LangChain and a vector database.

**Primary RAG mode:** `ExportType.DOC_CHUNKS` with Docling's `HybridChunker`.

**One-time inspection/export mode:** `ExportType.MARKDOWN` is retained in Section 24.2 only. It is useful for debugging and reviewing the complete converted document, but it is not the primary ingestion mode for a production RAG index.

> **Architecture principle:** Docling understands and structures the source document. HybridChunker creates document-aware chunks. LangChain standardizes the chunks as `Document` objects and connects embeddings/retrievers/vector stores. S3 stores original visual assets when the application needs to retrieve or display them later. Pinecone stores searchable vectors and metadata/pointers.

---

## 24.1 Production setup and configuration

Install the local Docling integration and the LangChain/Pinecone dependencies. OCR engines and their system dependencies must be installed separately when required by the selected OCR backend.

```bash
pip install -U "langchain-docling[local]" langchain-core langchain-pinecone langchain-openai boto3 python-dotenv pandas tabulate
```

Example project layout:

```text
rag-app/
├── app/
│   ├── config.py
│   ├── ingestion.py
│   ├── asset_storage.py
│   ├── indexing.py
│   └── retrieval.py
├── .env
└── requirements.txt
```

Use environment variables or a managed secret store. Do not hard-code API keys, AWS credentials, bucket names or Pinecone credentials in source code.

```python
# app/config.py
import os
from pathlib import Path

PDF_PATH = Path(r"E:/Euron_Super_30_project/practice/source_docs_rag/doclaynet.pdf")
AWS_S3_BUCKET = os.environ["AWS_S3_BUCKET"]
PINECONE_INDEX_NAME = os.environ["PINECONE_INDEX_NAME"]
EMBEDDING_MODEL = os.getenv("EMBEDDING_MODEL", "text-embedding-3-small")
```

For production, pin and test compatible versions of `docling`, `langchain-docling`, `langchain-core`, the embedding integration and vector-store integration. The APIs evolve; validate the installed versions in CI before deploying.

## 24.2 Common loader: production chunking + one Markdown inspection example

For a RAG index, use `ExportType.DOC_CHUNKS`. It returns multiple LangChain `Document` objects, one for each Docling chunk. **In the LangChain Docling integration, `ExportType.DOC_CHUNKS` is the default export mode and `HybridChunker()` is the default chunker for this mode. You do not need to pass `chunker=HybridChunker()` in every `DoclingLoader` call.** Omit it for the default behavior; provide a custom chunker only when you need to tune it (for example, to align tokenization with your embedding model).

```python

> **Default behavior to remember:** `DOC_CHUNKS` + `HybridChunker()` are the defaults in `DoclingLoader`. Keep the code simple by omitting `chunker=HybridChunker()` unless you are customizing chunking.
from langchain_docling import DoclingLoader
from langchain_docling.loader import ExportType

loader = DoclingLoader(
    file_path=PDF_PATH,
    export_type=ExportType.DOC_CHUNKS,
)

rag_chunks = loader.load()

print("Chunks:", len(rag_chunks))
for chunk in rag_chunks[:5]:
    print("\n--- CHUNK ---")
    print(chunk.page_content)
    print("Metadata:", chunk.metadata)
```

**Production note:** choose a tokenizer compatible with the embedding model and configure `HybridChunker` accordingly. Check the actual chunk token lengths against the embedding model's input limit. Keep Docling's provenance metadata (including page/item information where supplied) instead of replacing it with only a filename.

### One Markdown example — inspect the full converted document

Use this only when you need a human-readable complete Markdown rendering for debugging, QA or archival. A complete document is usually too large to embed as one vector; do not use this as the normal RAG indexing path.

```python
from langchain_docling import DoclingLoader
from langchain_docling.loader import ExportType

markdown_loader = DoclingLoader(
    file_path=PDF_PATH,
    export_type=ExportType.MARKDOWN,
)
markdown_docs = markdown_loader.load()

full_markdown = markdown_docs[0].page_content
print(full_markdown[:5000])

with open("output/converted_document.md", "w", encoding="utf-8") as f:
    f.write(full_markdown)
```

## 24.3 Example 1 — Ingest normal text paragraphs

**Goal:** index normal paragraphs as document-aware chunks, rather than embedding the entire PDF as one large string.

```python
from langchain_docling import DoclingLoader
from langchain_docling.loader import ExportType

text_loader = DoclingLoader(
    file_path=PDF_PATH,
    export_type=ExportType.DOC_CHUNKS,
)
text_chunks = text_loader.load()

for chunk in text_chunks[:5]:
    print(chunk.page_content)
    print(chunk.metadata)
```

**Production checks:**
- Compare representative paragraphs against the original PDF.
- Check symbols, hyphenated words, bullet lists, footnotes and page boundaries.
- Preserve the source identifier and Docling provenance metadata for citations.
- Deduplicate repeated content before indexing, especially when the same source is reprocessed.

**RAG use:** these chunk documents can be embedded and added to a vector store. Do not strip useful heading/caption context from `page_content` or metadata before embedding.

## 24.4 Example 2 — Preserve headings and section context

With `DOC_CHUNKS` and `HybridChunker`, Docling uses the document structure and includes contextual information such as headings in the chunk serialization. This is generally preferable to converting the whole PDF to Markdown and then applying a second, unrelated splitter.

```python
from langchain_docling import DoclingLoader
from langchain_docling.loader import ExportType

section_loader = DoclingLoader(
    file_path=PDF_PATH,
    export_type=ExportType.DOC_CHUNKS,
)
section_chunks = section_loader.load()

for chunk in section_chunks[:10]:
    print("CONTENT:", chunk.page_content[:800])
    print("SOURCE:", chunk.metadata.get("source"))
    print("DOCLING METADATA:", chunk.metadata.get("dl_meta"))
```

**Expected result:** chunks retain relevant document context and Docling metadata. The exact metadata keys can vary by integration version; inspect the installed output rather than assuming every release has identical keys.

**Production checks:** verify heading hierarchy and chunk boundaries on representative pages. If you need normalized fields such as `document_id`, `page_number`, `section_path` or `tenant_id`, add them to metadata in a controlled normalization step while preserving the original Docling metadata.

## 24.5 Example 3 — Extract tables for retrieval and preserve originals

A table has two useful representations:

1. **Search representation:** a textual table or factual table summary that can be embedded and retrieved.
2. **Original representation:** the complete table export (for example Markdown/HTML/CSV) retained in S3 so the application can display or reprocess the source content.

`DOC_CHUNKS` may include table content in its chunk output. For applications that need exact structured rows/columns and a separately stored original table, use Docling's native table objects from the same conversion result.

```python
from pathlib import Path
import boto3
from docling.document_converter import DocumentConverter
from langchain_core.documents import Document

s3 = boto3.client("s3")
BUCKET = AWS_S3_BUCKET

converter = DocumentConverter()
conversion = converter.convert(PDF_PATH)
docling_doc = conversion.document

table_documents = []
for table_number, table in enumerate(docling_doc.tables, start=1):
    dataframe = table.export_to_dataframe(doc=docling_doc)
    table_markdown = dataframe.to_markdown(index=False)
    table_html = table.export_to_html(doc=docling_doc)

    # Stable object keys make reprocessing/idempotent writes easier.
    markdown_key = f"documents/{PDF_PATH.stem}/tables/table_{table_number}.md"
    html_key = f"documents/{PDF_PATH.stem}/tables/table_{table_number}.html"

    s3.put_object(
        Bucket=BUCKET,
        Key=markdown_key,
        Body=table_markdown.encode("utf-8"),
        ContentType="text/markdown; charset=utf-8",
    )
    s3.put_object(
        Bucket=BUCKET,
        Key=html_key,
        Body=table_html.encode("utf-8"),
        ContentType="text/html; charset=utf-8",
    )

    table_documents.append(
        Document(
            page_content=f"Table {table_number}\n{table_markdown}",
            metadata={
                "source": str(PDF_PATH),
                "element_type": "table",
                "table_number": table_number,
                "s3_markdown_uri": f"s3://{BUCKET}/{markdown_key}",
                "s3_html_uri": f"s3://{BUCKET}/{html_key}",
            },
        )
    )

print("Tables extracted:", len(table_documents))
```

**Important implementation note:** the example above shows the table-asset branch using a `DocumentConverter` result. In a production worker, avoid converting the same PDF twice. Prefer a single conversion orchestration that retains the `DoclingDocument`, then derives text chunks and table/image assets from that same conversion result. If you use `DoclingLoader` as the conversion entry point, plan an integration layer that exposes/reuses the underlying conversion result rather than silently running a second conversion.

**Production checks:** validate merged cells, multi-page tables, units, footnotes, column names and row alignment. Add page provenance from the corresponding Docling item when available. For numeric or regulated content, validate extracted values against the source PDF before indexing or answering.

## 24.6 Example 4 — Extract images and make them searchable

`DOC_CHUNKS` is the primary text/chunk export. It does not mean every picture has automatically become a reliable textual description. Export original picture assets from Docling, store them in S3, and generate a searchable caption using a vision-language model (VLM) or human review.

```python
from pathlib import Path
import boto3
from io import BytesIO
from docling_core.types.doc import PictureItem
from docling.document_converter import DocumentConverter
from langchain_core.documents import Document

s3 = boto3.client("s3")
BUCKET = AWS_S3_BUCKET

converter = DocumentConverter()
conversion = converter.convert(PDF_PATH)
docling_doc = conversion.document

image_documents = []
for picture_number, (item, _level) in enumerate(docling_doc.iterate_items(), start=1):
    if not isinstance(item, PictureItem):
        continue

    picture = item.get_image(docling_doc)
    if picture is None:
        continue

    image_key = f"documents/{PDF_PATH.stem}/images/picture_{picture_number}.png"
    image_buffer = BytesIO()
    picture.save(image_buffer, format="PNG")
    image_bytes = image_buffer.getvalue()
    s3.put_object(
        Bucket=BUCKET,
        Key=image_key,
        Body=image_bytes,
        ContentType="image/png",
    )

    # Replace this call with your selected VLM integration.
    # The VLM must inspect image_bytes (or a secure S3 object) and return a factual caption.
    verified_caption = describe_image_with_vlm(image_bytes)

    image_documents.append(
        Document(
            page_content=f"Figure description: {verified_caption}",
            metadata={
                "source": str(PDF_PATH),
                "element_type": "image",
                "asset_number": picture_number,
                "s3_uri": f"s3://{BUCKET}/{image_key}",
            },
        )
    )
```

`describe_image_with_vlm()` is an application-specific function, not a built-in Docling or LangChain function. Implement it with the vision model/provider selected for your project, including timeout, retry, output validation and cost controls.

**Why store both?** Pinecone can search the caption embedding, while S3 preserves the original PNG. At question-answer time, the retriever can use the S3 pointer to fetch the original image and pass it to a vision-capable model when visual details are required.

**Production checks:** verify the caption against the actual image; retain document/page/figure identifiers; use private S3 access and short-lived pre-signed URLs only when a client must view an asset. Never place public S3 URLs or sensitive image contents in vector metadata.

## 24.7 Example 5 — Process scanned PDFs with OCR

A scanned PDF page is image-based. Enable OCR in Docling's PDF pipeline before chunking. The OCR result then flows through the same `DOC_CHUNKS` ingestion path.

```python
from docling.datamodel.base_models import InputFormat
from docling.datamodel.pipeline_options import (
    PdfPipelineOptions,
    OcrMode,
    TableStructureOptions,
    TesseractCliOcrOptions,
)
from docling.document_converter import DocumentConverter, PdfFormatOption
from langchain_docling import DoclingLoader
from langchain_docling.loader import ExportType

pipeline_options = PdfPipelineOptions()
pipeline_options.do_ocr = True
pipeline_options.do_table_structure = True
pipeline_options.table_structure_options = TableStructureOptions(
    do_cell_matching=True
)
pipeline_options.ocr_options = TesseractCliOcrOptions(mode=OcrMode.FULL_PAGE)

ocr_converter = DocumentConverter(
    format_options={
        InputFormat.PDF: PdfFormatOption(pipeline_options=pipeline_options)
    }
)

ocr_loader = DoclingLoader(
    file_path=PDF_PATH,
    converter=ocr_converter,
    export_type=ExportType.DOC_CHUNKS,
)
ocr_chunks = ocr_loader.load()

for chunk in ocr_chunks[:5]:
    print(chunk.page_content)
    print(chunk.metadata)
```

Install Tesseract separately on the host/container and ensure the executable is available to the worker. Select/configure an OCR backend appropriate to the languages and quality of your documents.

**Production checks:** compare names, dates, decimal points, currency values and table cells against page images. OCR is not guaranteed to be perfect. Track OCR failures and low-confidence cases for review; do not silently treat uncertain OCR text as verified source truth.

## 24.8 Example 6 — Process multi-column PDFs

Multi-column documents require layout analysis and reading-order reconstruction. Docling performs this during conversion; use the resulting `DOC_CHUNKS` for indexing rather than applying a generic character splitter to a flattened text stream.

```python
from langchain_docling import DoclingLoader
from langchain_docling.loader import ExportType

column_loader = DoclingLoader(
    file_path=PDF_PATH,
    export_type=ExportType.DOC_CHUNKS,
)
column_chunks = column_loader.load()

for chunk in column_chunks[:10]:
    print(chunk.page_content)
    print(chunk.metadata)
```

**Production checks:** inspect pages where columns contain figures, sidebars, footnotes, equations or tables. If reading order is incorrect, inspect the original page and Docling conversion output. Chunking cannot repair an incorrect reading order produced during document conversion.

## 24.9 Example 7 — Extract charts and make them searchable

Charts are often represented as pictures. Text extraction may find a title, legend or axis labels without understanding the trend or exact values. Treat chart ingestion as two linked operations:

1. Store the original chart image in S3.
2. Generate a factual chart description with a VLM, validate it, then index that description as a LangChain `Document`.

```python
from docling_core.types.doc import PictureItem
from docling.document_converter import DocumentConverter
from langchain_core.documents import Document
import boto3

s3 = boto3.client("s3")
BUCKET = AWS_S3_BUCKET

converter = DocumentConverter()
conversion = converter.convert(PDF_PATH)
docling_doc = conversion.document
chart_documents = []

for visual_number, (item, _level) in enumerate(docling_doc.iterate_items(), start=1):
    if not isinstance(item, PictureItem):
        continue

    image = item.get_image(docling_doc)
    if image is None:
        continue

    image_buffer = BytesIO()
    image.save(image_buffer, format="PNG")
    image_bytes = image_buffer.getvalue()
    chart_key = f"documents/{PDF_PATH.stem}/charts/visual_{visual_number}.png"
    s3.put_object(
        Bucket=BUCKET,
        Key=chart_key,
        Body=image_bytes,
        ContentType="image/png",
    )

    # Implement VLM classification/description and only index verified chart outputs.
    chart_description = describe_chart_with_vlm(image_bytes)
    chart_documents.append(
        Document(
            page_content=f"Chart description: {chart_description}",
            metadata={
                "source": str(PDF_PATH),
                "element_type": "chart",
                "asset_number": visual_number,
                "s3_uri": f"s3://{BUCKET}/{chart_key}",
            },
        )
    )
```

`describe_chart_with_vlm()` is a placeholder for your provider-specific vision call. A robust implementation should ask for chart title, series/legend, axes, units, date range, trend and values only when they can be read confidently. For scientific or financial charts, validate extracted values against the source.

**Important:** not every `PictureItem` is a chart. In production, classify pictures (chart, diagram, photograph, logo, decorative image) and apply the appropriate captioning/validation policy before indexing.

## 24.10 Combine text chunks, table records and visual descriptions

After extraction, normalize all searchable content into LangChain `Document` objects. Keep the original S3 asset references in metadata so retrieval can locate the source asset later.

```python
# rag_chunks: output from DoclingLoader with DOC_CHUNKS
# table_documents: table Documents created in Section 24.5
# image_documents: verified image-caption Documents from Section 24.6
# chart_documents: verified chart-description Documents from Section 24.9

all_rag_documents = [
    *rag_chunks,
    *table_documents,
    *image_documents,
    *chart_documents,
]

for document in all_rag_documents:
    document.metadata.setdefault("document_id", "annual-report-2026")
    document.metadata.setdefault("tenant_id", "tenant-example")
    print(
        document.metadata.get("element_type", "text"),
        document.page_content[:160],
        document.metadata,
    )
```

**Avoid duplicate conversion and duplicate indexing.** In a production implementation, convert each source once, retain the conversion result, and derive text chunks plus table/image/chart assets from that same result. The examples above isolate element-specific operations for clarity; consolidate them into one ingestion worker in the deployed application.

Use a stable document/version identifier and deterministic chunk/asset IDs. On re-ingestion, update or delete prior vectors for the same document version so stale chunks do not remain searchable.

## 24.11 Embed and index searchable documents in Pinecone

Use a LangChain embedding integration and Pinecone vector store. The following example assumes the API keys and Pinecone index are already configured, and that the index dimension and metric match the selected embedding model.

```python
from langchain_openai import OpenAIEmbeddings
from langchain_pinecone import PineconeVectorStore

embeddings = OpenAIEmbeddings(model="text-embedding-3-small")

vector_store = PineconeVectorStore.from_documents(
    documents=all_rag_documents,
    embedding=embeddings,
    index_name=PINECONE_INDEX_NAME,
    # For production, supply stable IDs and use a namespace per tenant/environment.
    namespace="production",
)
```

**Production considerations:**
- Confirm vector dimensions match the Pinecone index configuration.
- Use tenant/document namespaces or metadata filters appropriate to your security model.
- Use deterministic IDs for idempotent upserts and implement cleanup for replaced document versions.
- Keep metadata compact; do not store image binaries or entire large tables in Pinecone metadata.
- Store S3 object keys/URIs and source provenance in metadata. Restrict access to those objects through IAM and application authorization.
- Batch embedding/upsert work and use bounded concurrency, retries and backoff.

## 24.12 Retrieve chunks and fetch original assets when needed

Vector search finds the relevant text, table description or image caption. If the retrieved record contains an S3 pointer, the application can fetch the original asset for display or multimodal reasoning.

```python
from langchain_core.documents import Document

retriever = vector_store.as_retriever(
    search_type="similarity",
    search_kwargs={"k": 5},
)

question = "What does the chart say about quarterly revenue?"
retrieved_docs = retriever.invoke(question)

for doc in retrieved_docs:
    print(doc.page_content)
    print("Element:", doc.metadata.get("element_type"))
    print("Original asset:", doc.metadata.get("s3_uri"))
```

For a retrieved image/table, use the S3 object key to fetch the original asset only after checking that the requesting user is authorized to access the source document. Pass the original image to a vision-capable model when the question requires visual interpretation. For tables, retrieve the original Markdown/HTML/CSV and provide the structured content to the answer model when exact rows or values matter.

**Important:** vector retrieval plus S3 storage does not automatically implement multimodal retrieval. Your query pipeline must explicitly fetch the asset, construct the model's multimodal input, and preserve source/page citations in the final answer.

## 24.13 Quick reference: which component handles what?

| PDF element | Docling step | LangChain / application step | Production validation |
|---|---|---|---|
| Text | Layout-aware extraction and `HybridChunker` | `DOC_CHUNKS` → LangChain `Document` → embeddings | Symbols, paragraph integrity, provenance |
| Headings | Heading recognition and document hierarchy | `DOC_CHUNKS` contextualizes relevant section information | Correct heading levels and section context |
| Tables | Table structure recognition and `TableItem` export | Text/table `Document`; S3 original; Pinecone vector + pointer | Rows, columns, merged cells, units, footnotes |
| Images | `PictureItem` extraction | S3 original + VLM caption `Document` | Caption accuracy and page/asset reference |
| Scanned pages | OCR-enabled PDF pipeline | OCR text → `DOC_CHUNKS` → embeddings | OCR accuracy and low-confidence handling |
| Multi-column | Layout analysis and reading-order reconstruction | `DOC_CHUNKS` → LangChain `Document` | Left/right column order and figure placement |
| Charts | Picture extraction and classification | S3 original + validated VLM description `Document` | Axes, units, series, values and trend |

## 24.14 Production ingestion checklist

1. Validate file type, size, checksum and access permissions before processing.
2. Assign a stable document ID and version.
3. Convert the document once with Docling and retain the conversion result.
4. Configure OCR and table extraction based on the document corpus.
5. Create RAG chunks with `HybridChunker` and `ExportType.DOC_CHUNKS` (or directly from the retained Docling document when a single-conversion pipeline needs both chunks and assets).
6. Extract original tables and pictures from the same conversion result; store required assets in private S3.
7. Generate factual table/image/chart descriptions and validate them before indexing.
8. Normalize LangChain metadata while preserving Docling provenance.
9. Embed and upsert in batches with stable IDs, tenant isolation and retry handling.
10. Record ingestion status, conversion/OCR errors, processing duration, model usage and asset locations.
11. At query time, retrieve relevant chunks, rerank if appropriate, fetch original S3 assets when needed, and send them to a capable model.
12. Evaluate retrieval quality and answer grounding against a representative test set before production release.

## 24.15 Official references

- [Docling: RAG with LangChain](https://docling-project.github.io/docling/_generated/examples/rag_langchain/) — `DoclingLoader`, `DOC_CHUNKS`, `MARKDOWN` and RAG workflow.
- [Docling LangChain integration README](https://github.com/docling-project/docling-langchain) — loader configuration, local/remote converter options and advanced usage.
- [Docling: Chunking concepts](https://docling-project.github.io/docling/concepts/chunking/) — `HybridChunker`, token-aware refinement and contextualization.
- [Docling: Full-page OCR](https://docling-project.github.io/docling/_generated/examples/full_page_ocr/) — OCR configuration and supported backends.
- [Docling: Export figures](https://docling-project.github.io/docling/_generated/examples/export_figures/) — extracting pictures and image assets.
- [Docling: Export tables](https://docling-project.github.io/docling/_generated/examples/export_tables/) — exporting tables to DataFrames, CSV and HTML.
- [Docling: Picture descriptions](https://docling-project.github.io/docling/_generated/examples/pictures_description/) — enriching pictures with vision-language descriptions.
- [LangChain Pinecone integration](https://python.langchain.com/docs/integrations/vectorstores/pinecone/) — vector-store integration.

**Version note:** The Docling and LangChain integration APIs evolve. Pin package versions, run the examples in a clean environment, and verify OCR, table export, image serialization and metadata behavior against your installed versions. Treat the code in this chapter as production-oriented patterns that still require application-specific error handling, security, observability, testing and deployment configuration.
