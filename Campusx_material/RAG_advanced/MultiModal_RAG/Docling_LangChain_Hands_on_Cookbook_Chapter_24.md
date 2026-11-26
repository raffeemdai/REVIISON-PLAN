# Hands-on Cookbook: LangChain + Docling for Each PDF Element

This is a standalone copy of Chapter 24 from the updated Advanced RAG notes. It contains practical examples for text, headings, tables, images, scanned pages/OCR, multi-column layouts, and charts.

**Prerequisite:** Use the updated `LangChain_Docling_Complex_PDF_Advanced_RAG_Complete_Notes_v2.md` for the complete guide.

---

# 24. Hands-on cookbook: LangChain + Docling for each PDF element

This chapter adds the missing practical examples. It uses **LangChain's `DoclingLoader` as the entry point**, and uses the underlying Docling document only where the task requires access to individual tables or image assets.

> **Important distinction:** `DoclingLoader` returns LangChain `Document` objects. It does not mean every visual element becomes searchable text automatically. Text, headings, tables and reading order can be represented in Markdown/chunks. Images and charts usually need an additional image-export and captioning/vision step if you want their visual meaning searchable.

## 24.1 Install packages and prepare a test PDF

For local PDF conversion, install the local extra. OCR engines may require additional system packages depending on the selected backend.

```bash
pip install -U "langchain-docling[local]" langchain-core langchain-text-splitters pandas tabulate
```

Set the PDF path once:

```python
PDF_PATH = "data/complex_annual_report.pdf"
```

## 24.2 Common loader: use this for text, headings, tables and columns

`MARKDOWN` returns one LangChain `Document` for the converted input document. It is useful when you want to inspect the complete structured Markdown before splitting it.

```python
from langchain_docling import DoclingLoader
from langchain_docling.loader import ExportType

loader = DoclingLoader(
    file_path=PDF_PATH,
    export_type=ExportType.MARKDOWN,
)

markdown_docs = loader.load()

print("LangChain documents:", len(markdown_docs))
print("Object type:", type(markdown_docs[0]))
print(markdown_docs[0].page_content[:3000])
print("Metadata:", markdown_docs[0].metadata)
```

For RAG, `DOC_CHUNKS` is normally the better starting point. It uses Docling's document-aware chunking (by default, `HybridChunker`) and returns multiple LangChain documents.

```python
from langchain_docling import DoclingLoader
from langchain_docling.loader import ExportType

chunk_loader = DoclingLoader(
    file_path=PDF_PATH,
    export_type=ExportType.DOC_CHUNKS,
)

chunks = chunk_loader.load()

for number, chunk in enumerate(chunks[:5], start=1):
    print(f"\n--- CHUNK {number} ---")
    print(chunk.page_content)
    print("Metadata:", chunk.metadata)
```

**Expected result:** each chunk is a LangChain `Document` with `page_content` and `metadata`. The metadata can carry source and Docling provenance. Inspect the actual keys in your installed version rather than assuming every release returns identical metadata.

## 24.3 Example 1 — Extract normal text

**Goal:** extract paragraphs and make them available to LangChain.

```python
from langchain_docling import DoclingLoader
from langchain_docling.loader import ExportType

loader = DoclingLoader(
    file_path=PDF_PATH,
    export_type=ExportType.MARKDOWN,
)

docs = loader.load()
full_text = docs[0].page_content

print(full_text[:2000])

# Optional: save the extracted text/Markdown for manual inspection
with open("output/extracted_document.md", "w", encoding="utf-8") as file:
    file.write(full_text)
```

**What to verify:** compare a few paragraphs against the original PDF. Check symbols, hyphenated words, bullet lists, footnotes and page breaks. If the PDF is scanned, use the OCR example below; the default text extraction alone may not find text embedded in page images.

**RAG use:** use `DOC_CHUNKS` or split the Markdown at heading boundaries before creating embeddings. Avoid embedding a very large complete document as one vector.

## 24.4 Example 2 — Extract headings and preserve section context

Docling's Markdown export represents recognized heading hierarchy with Markdown heading markers. The following example loads the Markdown through `DoclingLoader`, then uses LangChain's `MarkdownHeaderTextSplitter` to produce section-aware LangChain documents.

```python
from langchain_docling import DoclingLoader
from langchain_docling.loader import ExportType
from langchain_text_splitters import MarkdownHeaderTextSplitter

markdown_doc = DoclingLoader(
    file_path=PDF_PATH,
    export_type=ExportType.MARKDOWN,
).load()[0]

headers_to_split_on = [
    ("#", "heading_1"),
    ("##", "heading_2"),
    ("###", "heading_3"),
]

splitter = MarkdownHeaderTextSplitter(
    headers_to_split_on=headers_to_split_on,
    strip_headers=False,
)

section_docs = splitter.split_text(markdown_doc.page_content)

for section in section_docs[:10]:
    print("\nSECTION METADATA:", section.metadata)
    print(section.page_content[:500])
```

**Expected result:** LangChain documents are split by Markdown headings, and the heading values are available in each section's metadata. This makes it easier to retrieve a paragraph together with its topic/section context.

**What to verify:** confirm that headings are not ordinary bold text in the PDF, and check whether the detected hierarchy (`#`, `##`, `###`) matches the original document. Heading recognition can vary with PDF quality and layout.

## 24.5 Example 3 — Extract tables as Markdown and as DataFrames

There are two useful paths:

- Use `DoclingLoader` in `MARKDOWN` mode when a Markdown table is sufficient for RAG.
- Use Docling's `TableItem` objects when you need structured rows/columns, CSV or DataFrame output. The table is then wrapped in a LangChain `Document` so it can enter the same downstream pipeline.

### A. Table in LangChain Markdown

```python
from langchain_docling import DoclingLoader
from langchain_docling.loader import ExportType

md_doc = DoclingLoader(
    file_path=PDF_PATH,
    export_type=ExportType.MARKDOWN,
).load()[0]

print(md_doc.page_content)  # Look for Markdown table syntax: | column | column |
```

### B. Extract each table as a DataFrame and create LangChain documents

```python
from pathlib import Path
import pandas as pd
from langchain_core.documents import Document
from docling.document_converter import DocumentConverter

converter = DocumentConverter()
conversion = converter.convert(PDF_PATH)
docling_doc = conversion.document

Path("output/tables").mkdir(parents=True, exist_ok=True)
table_documents = []

for index, table in enumerate(docling_doc.tables, start=1):
    dataframe = table.export_to_dataframe(doc=docling_doc)

    csv_path = Path("output/tables") / f"table_{index}.csv"
    dataframe.to_csv(csv_path, index=False)

    # Convert the table to text/Markdown for LangChain retrieval.
    table_text = dataframe.to_markdown(index=False)
    table_documents.append(
        Document(
            page_content=f"Table {index}\n{table_text}",
            metadata={
                "source": PDF_PATH,
                "element_type": "table",
                "table_number": index,
            },
        )
    )

print("Tables found:", len(table_documents))
for table_doc in table_documents:
    print(table_doc.page_content)
    print(table_doc.metadata)
```

**Expected result:** one CSV file and one LangChain `Document` per detected table. The table text includes column headers and values, which is much safer for retrieval than embedding isolated cell values.

**What to verify:** merged cells, multi-page tables, units, footnotes, column names and row alignment. Always compare financial or compliance-related values with the source PDF. The metadata above is deliberately explicit; add the actual page number/provenance from the corresponding Docling item if your application requires page-level citations.

## 24.6 Example 4 — Extract images and diagrams

`DoclingLoader` is designed to provide LangChain text documents. Do not assume an image is automatically converted into a meaningful textual description. Use Docling's `PictureItem` to export the picture asset, then create a companion LangChain document containing a verified caption/description.

```python
from pathlib import Path
from docling_core.types.doc import PictureItem
from docling.document_converter import DocumentConverter

converter = DocumentConverter()
conversion = converter.convert(PDF_PATH)
docling_doc = conversion.document

image_dir = Path("output/images")
image_dir.mkdir(parents=True, exist_ok=True)

picture_paths = []
for index, (item, level) in enumerate(docling_doc.iterate_items(), start=1):
    if isinstance(item, PictureItem):
        image_path = image_dir / f"picture_{index}.png"
        picture = item.get_image(docling_doc)
        if picture is not None:
            picture.save(image_path, format="PNG")
            picture_paths.append(image_path)
            print("Saved:", image_path)

print("Pictures exported:", len(picture_paths))
```

### Turn the picture into searchable RAG content

After exporting, send the image to a vision-language model (VLM) or have a human review it. Then create a text document with the caption. Example below uses a placeholder caption — replace it with a description actually produced by your VLM or verified by a person.

```python
from langchain_core.documents import Document

verified_caption = "Architecture diagram showing the ingestion service, queue, processing workers and vector database."

image_doc = Document(
    page_content=f"Figure description: {verified_caption}",
    metadata={
        "source": PDF_PATH,
        "element_type": "image",
        "image_path": str(picture_paths[0]),
    },
)
```

**Important:** the caption above is illustrative, not extracted from your PDF. Never invent a chart or diagram description. Preserve the original image path and page reference so users can inspect the source visual.

## 24.7 Example 5 — Process scanned pages with OCR and LangChain

A scanned PDF page is an image. OCR must recognize the text before it can be embedded as text. Configure OCR in Docling's PDF pipeline, pass that converter to `DoclingLoader`, and let the loader return LangChain documents.

The following example uses Tesseract's full-page OCR mode. Install Tesseract separately on your operating system before running it.

```python
from docling.datamodel.base_models import InputFormat
from docling.datamodel.pipeline_options import (
    OcrMode,
    PdfPipelineOptions,
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
pipeline_options.ocr_options = TesseractCliOcrOptions(
    mode=OcrMode.FULL_PAGE
)

ocr_converter = DocumentConverter(
    format_options={
        InputFormat.PDF: PdfFormatOption(
            pipeline_options=pipeline_options
        )
    }
)

ocr_loader = DoclingLoader(
    file_path=PDF_PATH,
    converter=ocr_converter,
    export_type=ExportType.DOC_CHUNKS,
)

ocr_docs = ocr_loader.load()

for doc in ocr_docs[:5]:
    print(doc.page_content)
    print(doc.metadata)
```

**Expected result:** OCR-recognized text is included in the converted content and then exposed as LangChain documents/chunks.

**What to verify:** compare names, dates, decimal points, currency values, table cells and low-resolution text with the original scanned pages. OCR is not guaranteed to be perfect. If the chosen OCR backend is unavailable, install/configure its dependency or select another supported backend from the Docling documentation.

## 24.8 Example 6 — Process multi-column PDFs

For a two-column research paper, the main challenge is reading order. A naïve text extractor can read across the page horizontally and mix the left and right columns. Docling performs layout analysis and reading-order reconstruction during conversion; `DoclingLoader` exposes the converted result to LangChain.

```python
from langchain_docling import DoclingLoader
from langchain_docling.loader import ExportType

column_loader = DoclingLoader(
    file_path="data/two_column_research_paper.pdf",
    export_type=ExportType.MARKDOWN,
)

column_doc = column_loader.load()[0]

with open("output/two_column_reading_order.md", "w", encoding="utf-8") as file:
    file.write(column_doc.page_content)

print(column_doc.page_content[:5000])
```

**Expected result:** a Markdown representation in which text is arranged in Docling's inferred reading order, then available for LangChain splitting and retrieval.

**What to verify:** inspect several pages, especially where columns contain figures, sidebars, footnotes or equations. There is no special `MultiColumnLoader` switch in this example; layout analysis is part of the Docling conversion pipeline. If the reading order is wrong, inspect the page image and conversion output rather than assuming chunking will fix it.

## 24.9 Example 7 — Extract charts and make them searchable

A chart is often represented as a picture. Text extraction may find the chart title, legend or axis labels, but that does not guarantee the actual trend or numerical values have been understood. Treat chart processing as two stages:

1. Export the chart image from the PDF.
2. Use a VLM or human validation to create a factual textual description, then wrap that description in a LangChain `Document`.

```python
from pathlib import Path
from docling_core.types.doc import PictureItem
from docling.document_converter import DocumentConverter

converter = DocumentConverter()
conversion = converter.convert(PDF_PATH)
docling_doc = conversion.document

chart_dir = Path("output/charts")
chart_dir.mkdir(parents=True, exist_ok=True)
chart_images = []

for index, (item, level) in enumerate(docling_doc.iterate_items(), start=1):
    if isinstance(item, PictureItem):
        image = item.get_image(docling_doc)
        if image is not None:
            path = chart_dir / f"visual_{index}.png"
            image.save(path, format="PNG")
            chart_images.append(path)

print("Exported visual assets:", chart_images)
```

Now add a caption produced by your chosen vision model. For example, if the chart genuinely shows the following values, a verified description could be:

```python
from langchain_core.documents import Document

verified_chart_description = (
    "Quarterly revenue chart: Q1 = 10 million, Q2 = 12 million, "
    "Q3 = 15 million, and Q4 = 18 million. Revenue increases each quarter."
)

chart_doc = Document(
    page_content=verified_chart_description,
    metadata={
        "source": PDF_PATH,
        "element_type": "chart_description",
        "image_path": str(chart_images[0]),
    },
)
```

The numbers are only an example. Use values returned by your VLM and validate them against the chart. For financial or scientific charts, preserve units, axis ranges, series names, legend labels and reporting period. Do not treat a guessed visual summary as source truth.

## 24.10 Combine extracted text, table documents and visual descriptions

Once the content has been converted to LangChain `Document` objects, you can combine the different sources into one list for embedding. In a real project, avoid running the same PDF conversion multiple times: convert once, retain the conversion result, and build the text/table/image outputs from that result.

```python
from langchain_core.documents import Document

# markdown_docs: LangChain docs from DoclingLoader
# table_documents: LangChain docs created from extracted DataFrames
# image_doc / chart_doc: docs created from verified visual descriptions

all_rag_documents = [
    *markdown_docs,
    *table_documents,
    image_doc,
    chart_doc,
]

for doc in all_rag_documents:
    print(doc.metadata.get("element_type", "document"), doc.page_content[:160])
```

For a production pipeline, deduplicate content, attach source/page identifiers, and make sure image and chart descriptions link back to the original image asset. Add OCR-derived content only once; otherwise a scanned page may be indexed twice.

## 24.11 Quick reference: which component handles what?

| PDF element | Docling step | LangChain step | Important validation |
|---|---|---|---|
| Text | PDF text/layout extraction | `DoclingLoader` → `Document.page_content` | Missing symbols and paragraphs |
| Headings | Heading/layout recognition; Markdown hierarchy | `MARKDOWN` + `MarkdownHeaderTextSplitter`, or `DOC_CHUNKS` | Correct heading level and section context |
| Tables | Table structure recognition | Markdown table or DataFrame wrapped in `Document` | Rows, columns, merged cells, units |
| Images | Detect/export `PictureItem` | VLM/human caption wrapped in `Document` | Caption matches actual image |
| Scanned pages | OCR-enabled PDF pipeline | `DoclingLoader` returns OCR text as documents | OCR accuracy and page provenance |
| Multi-column | Layout analysis and reading order | Markdown/chunks from `DoclingLoader` | Left/right column order |
| Charts | Picture export; optional VLM enrichment | Verified chart description as `Document` | Values, axes, units and trend |

## 24.12 One complete practice sequence

1. Run the common `MARKDOWN` loader and inspect the complete output.
2. Run `DOC_CHUNKS` and inspect chunk text and metadata.
3. Extract headings and split by Markdown heading levels.
4. Export tables to CSV and wrap each table in a LangChain `Document`.
5. Export pictures and charts as image files.
6. Generate verified captions with a VLM; create companion LangChain documents.
7. Enable OCR and process a scanned PDF.
8. Test a two-column paper and validate reading order.
9. Combine the resulting LangChain documents, add source/page metadata, embed and index.
10. Ask questions that require paragraph context, table values and chart interpretation; inspect the retrieved chunks before trusting the answer.

## 24.13 Official references for these examples

- [Docling: RAG with LangChain](https://docling-project.github.io/docling/_generated/examples/rag_langchain/) — `DoclingLoader`, `MARKDOWN`, `DOC_CHUNKS`, chunking and RAG.
- [Docling LangChain integration README](https://github.com/docling-project/docling-langchain) — installation and loader configuration.
- [Docling: Full-page OCR](https://docling-project.github.io/docling/_generated/examples/full_page_ocr/) — OCR pipeline configuration and supported OCR backends.
- [Docling: Export figures](https://docling-project.github.io/docling/_generated/examples/export_figures/) — saving page images, pictures and table images.
- [Docling: Export tables](https://docling-project.github.io/docling/_generated/examples/export_tables/) — exporting table content to DataFrames, CSV and HTML.
- [Docling: Picture descriptions](https://docling-project.github.io/docling/_generated/examples/pictures_description/) — using vision models to describe document pictures.

**Version note:** Docling and `langchain-docling` APIs evolve. The examples follow the current official documentation patterns at the time this appendix was prepared. If an import or option differs in your environment, check the installed package versions and the linked official examples.
