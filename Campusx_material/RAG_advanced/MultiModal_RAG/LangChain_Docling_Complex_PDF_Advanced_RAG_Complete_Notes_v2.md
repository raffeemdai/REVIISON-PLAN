# LangChain + Docling: Complete Guide to Extracting Every Element from Complex PDFs for Advanced RAG

> **Merged study notes**
>
> This guide brings together the two topics:
> 1. **LangChain + Docling: Extracting Every Element from a Complex PDF**
> 2. **Docling – Loading Complex PDFs for Advanced RAG**
>
> It follows the complete journey from a PDF file to structured document content, LangChain documents, chunks, embeddings, a vector database, retrieval, and an LLM answer. It covers text, headings, tables, images, scanned pages, OCR, multi-column layouts, charts, metadata, export formats, and production considerations.

---

## Table of contents

1. [The big picture: why Docling?](#1-the-big-picture-why-docling)
2. [What makes a PDF complex?](#2-what-makes-a-pdf-complex)
3. [Traditional PDF loaders vs Docling](#3-traditional-pdf-loaders-vs-docling)
4. [What Docling extracts from a PDF](#4-what-docling-extracts-from-a-pdf)
5. [Docling architecture and processing pipeline](#5-docling-architecture-and-processing-pipeline)
6. [DoclingDocument vs LangChain Document](#6-doclingdocument-vs-langchain-document)
7. [Installation and project setup](#7-installation-and-project-setup)
8. [Load and convert a complex PDF with core Docling](#8-load-and-convert-a-complex-pdf-with-core-docling)
9. [Export the converted document](#9-export-the-converted-document)
10. [Extracting each PDF element](#10-extracting-each-pdf-element)
11. [Images, OCR, scanned pages, and charts: important limitations](#11-images-ocr-scanned-pages-and-charts-important-limitations)
12. [Use Docling through LangChain](#12-use-docling-through-langchain)
13. [MARKDOWN vs DOC_CHUNKS](#13-markdown-vs-doc_chunks)
14. [Chunking with HybridChunker](#14-chunking-with-hybridchunker)
15. [Metadata and provenance](#15-metadata-and-provenance)
16. [End-to-end Docling + LangChain + Qdrant example](#16-end-to-end-docling--langchain--qdrant-example)
17. [Advanced RAG retrieval after ingestion](#17-advanced-rag-retrieval-after-ingestion)
18. [Production ingestion architecture](#18-production-ingestion-architecture)
19. [Best practices and troubleshooting](#19-best-practices-and-troubleshooting)
20. [Interview explanation](#20-interview-explanation)
21. [Quick revision and memory tricks](#21-quick-revision-and-memory-tricks)
22. [Practice exercises](#22-practice-exercises)
23. [Reference notes and version caveats](#23-reference-notes-and-version-caveats)

---

# 1. The big picture: why Docling?

Imagine an enterprise has a 300-page annual report. It contains:

- Normal paragraphs
- Main headings and subheadings
- Tables with financial figures
- Images and diagrams
- Charts
- Scanned pages
- Two-column layouts
- Headers, footers, page numbers, and footnotes

A basic PDF text loader may extract words, but it may not preserve how the words relate to each other. A table can become a sequence of disconnected values. A two-column page can be read in the wrong order. A scanned page may contain no selectable text at all.

That creates a problem for Retrieval-Augmented Generation (RAG): if ingestion loses structure, the retriever may return incomplete or confusing chunks. The LLM cannot reliably answer from information that was not extracted correctly.

**Docling is a document-understanding framework** designed to convert complex documents into structured representations that are useful for AI and RAG applications.

It helps identify and preserve:

- Document hierarchy and headings
- Paragraphs and lists
- Tables and their structure
- Images and picture regions
- Reading order and layout information
- Page and source information
- Text obtained through OCR when configured for scanned content

The important idea is:

> **Do not treat a complex PDF as one long string. First understand its structure, then prepare it for retrieval.**

## Overall RAG flow

```text
Complex PDF
    |
    v
Docling document conversion
    |
    v
Structured DoclingDocument
    |
    v
Export / structure-aware chunking
    |
    v
LangChain Documents
    |
    v
Embedding model
    |
    v
Vector database (Qdrant / Chroma / etc.)
    |
    v
Retriever (+ optional reranker / compression)
    |
    v
LLM
    |
    v
Grounded answer
```

---

# 2. What makes a PDF complex?

A PDF is complex when important information is represented through more than simple paragraphs.

| PDF content | What can go wrong with simple text extraction | What document understanding should preserve |
|---|---|---|
| Headings | Heading and paragraph may be mixed together | Section hierarchy and context |
| Tables | Rows and columns may become disconnected text | Row/column relationships |
| Images | Image content may not appear in extracted text | Picture region and associated information |
| Scanned pages | There may be no selectable text | OCR-recognized text |
| Multi-column pages | Left and right columns may be interleaved | Correct reading order |
| Charts | Text labels may be separated from visual meaning | Labels, values, and chart context where extractable |
| Lists | Bullets and indentation may be lost | List structure |
| Footnotes | Footnote text may be separated from the statement | Page and positional context |

For enterprise RAG, these details matter. For example, retrieving the number `25.4` without its table heading, row label, unit, or reporting period may produce a misleading answer.

---

# 3. Traditional PDF loaders vs Docling

## Traditional loader approach

Typical loaders include:

- `PyPDFLoader`
- `PDFPlumber`
- `UnstructuredPDFLoader`

A simple pipeline often looks like this:

```text
PDF
 |
 v
Extract text
 |
 v
Split text by characters / tokens
 |
 v
Create embeddings
 |
 v
Vector database
```

### Common limitations

1. **Broken tables** – values may be extracted without clear row/column relationships.
2. **Lost headings** – a paragraph may be separated from the heading that explains it.
3. **Wrong reading order** – text from multiple columns may be mixed.
4. **Limited layout understanding** – the loader may not understand where a block appears on the page.
5. **Poor chunk quality** – arbitrary splitting can break a table, list, or explanation across chunks.
6. **Scanned content** – OCR may need a separate configuration or pipeline.

These loaders remain useful for simple, text-based PDFs. The point is not that every traditional loader is bad; the point is that complex documents may need more document-aware processing.

## Docling approach

```text
PDF
 |
 v
Docling conversion
 |
 v
Document structure and layout
 |
 +-- Headings
 +-- Paragraphs
 +-- Lists
 +-- Tables
 +-- Pictures / image regions
 +-- Page and provenance information
 |
 v
Structured export / chunking
 |
 v
LangChain Documents
```

## Comparison

| Feature | Basic text-oriented PDF loading | Docling-based ingestion |
|---|---|---|
| Extract text | Yes | Yes |
| Preserve headings and hierarchy | Depends on loader and PDF | Document-structure aware |
| Understand tables | Often limited | Table structure extraction |
| Layout awareness | Usually limited | Layout analysis |
| Reading order | May be imperfect | Designed to analyze reading order |
| Scanned PDF support | Requires OCR support/configuration | OCR pipeline can be configured |
| Structure-aware chunking | Usually requires additional logic | Docling chunkers are available |
| Structured document representation | Usually a list of extracted text documents | `DoclingDocument` |
| LangChain integration | Loader-dependent | `DoclingLoader` |

---

# 4. What Docling extracts from a PDF

The goal is to understand what is present in the PDF and what happens to each element.

## 4.1 Text and paragraphs

**Input example**

```text
The company reported revenue growth during the financial year.
Operating costs increased moderately.
```

Docling identifies text content and its location/order within the document.

Why it matters for RAG:

- The paragraph can be converted into a useful chunk.
- Page/source information can be retained as metadata.
- It can remain associated with the correct section heading.

## 4.2 Headings and document hierarchy

Example PDF structure:

```text
1. Financial Results
   1.1 Revenue
       Revenue increased by 12%.
   1.2 Operating Costs
       Operating costs increased by 5%.
2. Risk Factors
   2.1 Market Risk
       ...
```

A structure-aware representation can preserve the relationship between headings and the content below them.

Why it matters:

If the chunk contains only `Revenue increased by 12%`, the retrieval result may lack context. A chunk that also carries the heading `Financial Results > Revenue` is easier to interpret.

## 4.3 Tables

Consider this table in a PDF:

| Product | Sales | Year |
|---|---:|---:|
| Product A | 10,000 | 2024 |
| Product B | 20,000 | 2024 |

A weak extraction might produce:

```text
Product Sales Year Product A 10000 2024 Product B 20000 2024
```

The relationships between values are harder to infer.

A structured or Markdown-like representation is clearer:

```markdown
| Product | Sales | Year |
|---|---:|---:|
| Product A | 10,000 | 2024 |
| Product B | 20,000 | 2024 |
```

Docling includes table-understanding capabilities (including TableFormer in its table processing pipeline). The exact result depends on the source PDF, table layout, configuration, and software version.

**Why table structure matters for RAG**

Question: “What were Product B sales in 2024?”

A retriever needs to return the value together with the correct product, column, and year. A chunk containing disconnected numbers is not sufficient.

## 4.4 Lists

Example:

```text
The system supports:
- PDF ingestion
- OCR
- Table extraction
- Structure-aware chunking
```

List markers and item relationships help keep related points together. This is useful for policy documents, technical manuals, and procedure documents.

## 4.5 Images and picture regions

A PDF can contain:

- Photographs
- Architecture diagrams
- Screenshots
- Flowcharts
- Scanned figures
- Embedded illustrations

Docling can identify picture regions and preserve document structure around them. Depending on the conversion configuration and export path, picture data may be represented as image items, references, or placeholders.

**Important distinction:** identifying or extracting an image region is not the same as understanding every word or concept inside the image. If the image contains meaningful text or a diagram, an OCR or vision-language processing step may be required.

For example, a network architecture diagram may contain:

```text
Client -> API Gateway -> Service -> Database
```

If the diagram is only stored as an image without a textual description, ordinary text embeddings cannot automatically understand all of its visual meaning. A production multimodal RAG system may need to create a caption or structured description and index it with a link to the original image.

## 4.6 Multi-column layouts and reading order

A PDF page may have two columns:

```text
+----------------------+----------------------+
| LEFT COLUMN          | RIGHT COLUMN         |
| Introduction text... | Related discussion.. |
| More left text...    | More right text...    |
+----------------------+----------------------+
```

A naive extraction can read across the page instead of finishing the left column before moving to the right. That can mix unrelated sentences.

Docling performs layout analysis and reading-order reconstruction to improve the sequence of extracted elements. Complex or low-quality PDFs should still be checked because no parser can guarantee perfect results for every layout.

## 4.7 Charts and graphs

A chart may contain:

- A title
- Axis labels
- Legends
- Categories
- Numeric values
- Bars, lines, or other visual marks

Text extraction may capture the chart title but miss the relationship between the visual marks and their values.

For a chart such as “Revenue by Quarter,” a useful RAG representation should ideally include a textual description such as:

```text
Chart: Revenue by Quarter
Q1: 10 million
Q2: 12 million
Q3: 15 million
Q4: 18 million
```

Do not assume that every chart is converted into a complete table automatically. Depending on the chart, PDF quality, and Docling configuration, additional image understanding, OCR, or custom processing may be necessary.

---

# 5. Docling architecture and processing pipeline

## 5.1 High-level architecture

```text
PDF / DOCX / PPTX / HTML / Images
                 |
                 v
          DocumentConverter
                 |
                 v
           DoclingDocument
                 |
                 v
       Export or Docling Chunker
                 |
                 v
        LangChain Document objects
                 |
                 v
              Embeddings
                 |
                 v
           Vector database
                 |
                 v
              Retriever
                 |
                 v
                 LLM
```

## 5.2 Main stages

1. **Input** – provide a PDF or another supported document.
2. **Conversion** – Docling analyzes the source and converts it.
3. **Document model** – the result is represented as a `DoclingDocument`.
4. **Export or chunking** – export as Markdown/JSON or create structure-aware chunks.
5. **LangChain integration** – `DoclingLoader` can provide LangChain `Document` objects.
6. **Embedding** – convert chunk text into vectors.
7. **Storage** – store vectors and metadata in Qdrant, Chroma, or another vector store.
8. **Retrieval** – find relevant chunks for a user query.
9. **Generation** – pass the retrieved context to the LLM.

---

# 6. DoclingDocument vs LangChain Document

This distinction is important in interviews and when writing ingestion code.

## 6.1 `DoclingDocument`

When using the core Docling library directly:

```python
from docling.document_converter import DocumentConverter

converter = DocumentConverter()
result = converter.convert("report.pdf")

doc = result.document
```

The object is a **`DoclingDocument`**.

It is Docling's structured representation of the converted document. It can contain document items and relationships such as:

```text
DoclingDocument
├── Text / paragraphs
├── Headings
├── Lists
├── Tables
├── Pictures
├── Layout / reading-order information
└── Document and provenance metadata
```

It is richer than a plain string because it represents document structure.

## 6.2 LangChain `Document`

A LangChain document is a simpler standard object commonly shaped like:

```python
Document(
    page_content="Extracted text or chunk text...",
    metadata={
        "source": "report.pdf",
        "page": 5
    }
)
```

The key fields are:

- `page_content` – the text content of the document or chunk.
- `metadata` – source, page, section, and other useful attributes.

## 6.3 Three cases

### Case 1: Traditional LangChain loader

```python
from langchain_community.document_loaders import PyPDFLoader

loader = PyPDFLoader("report.pdf")
docs = loader.load()
```

Output: a list of LangChain `Document` objects.

### Case 2: Core Docling

```python
from docling.document_converter import DocumentConverter

converter = DocumentConverter()
result = converter.convert("report.pdf")
doc = result.document
```

Output: a `DoclingDocument`, not automatically a LangChain `Document`.

### Case 3: Docling through LangChain

```python
from langchain_docling.loader import DoclingLoader

loader = DoclingLoader(file_path="report.pdf")
docs = loader.load()
```

Output: a list of LangChain `Document` objects.

The Docling conversion and chunking work happens inside the integration path.

## 6.4 Internal flow

```text
report.pdf
    |
    v
DocumentConverter
    |
    v
DoclingDocument
    |
    v
Chunking / export conversion
    |
    v
LangChain Documents
    |
    v
Embeddings
    |
    v
Qdrant / Chroma
```

## 6.5 Why use Docling if both loaders return LangChain Documents?

Both a traditional loader and `DoclingLoader` may eventually return:

```python
Document(
    page_content="...",
    metadata={}
)
```

The difference is the processing that happened before the object was created.

A traditional text extraction may flatten a table:

```text
Revenue Product Sales 2024 10000 ProductA 20000 ProductB
```

A Docling-based path may preserve a clearer structure:

```markdown
# Revenue

| Product | Sales |
|---|---:|
| Product A | 10000 |
| Product B | 20000 |
```

The LangChain object is the common output format. Docling's additional document-understanding stage can make the content and metadata more useful for retrieval.

---

# 7. Installation and project setup

Create and activate a virtual environment first.

```bash
# Create a project folder
mkdir docling-rag
cd docling-rag

# Create a virtual environment
python -m venv .venv
```

Activate it:

```bash
# Windows PowerShell
.venv\Scripts\Activate.ps1

# macOS / Linux
source .venv/bin/activate
```

Install the core packages:

```bash
pip install docling
pip install langchain
pip install langchain-docling
pip install langchain-community
pip install langchain-huggingface
pip install sentence-transformers
pip install langchain-qdrant
pip install qdrant-client
pip install langchain-openai
```

You do not need every integration package for every project. Install the packages required by the path you use.

A simple project layout:

```text
docling-rag/
|
├── .venv/
├── documents/
│   └── annual_report.pdf
├── output/
├── ingest.py
├── query.py
└── requirements.txt
```

Save installed dependencies:

```bash
pip freeze > requirements.txt
```

**Version note:** Python package APIs and import paths can change. If an import in an older tutorial does not work, check the installed package version and the current Docling/LangChain integration documentation.

---

# 8. Load and convert a complex PDF with core Docling

The core Docling route gives direct access to the converted document model.

## 8.1 Basic conversion

```python
from docling.document_converter import DocumentConverter

# Create the converter
converter = DocumentConverter()

# Convert the PDF
result = converter.convert("documents/annual_report.pdf")

# Get the structured Docling document
doc = result.document

print(type(doc))
```

The conversion result contains the converted document. `result.document` is the `DoclingDocument`.

## 8.2 Export a quick Markdown preview

```python
markdown_text = doc.export_to_markdown()

print(markdown_text)
```

This is a convenient way to inspect the extracted content and structure.

## 8.3 Save the Markdown output

```python
from pathlib import Path

output_path = Path("output/annual_report.md")
output_path.parent.mkdir(parents=True, exist_ok=True)

output_path.write_text(
    doc.export_to_markdown(),
    encoding="utf-8"
)

print(f"Saved to: {output_path}")
```

## 8.4 Inspect the converted document

```python
print(doc)
```

For deeper inspection, use the document's supported export methods and the Docling document-model APIs available in the installed version.

---

# 9. Export the converted document

Docling can export a converted document in different representations. Choose based on what the next stage needs.

## 9.1 Markdown export

```python
markdown = doc.export_to_markdown()
```

Useful for:

- Reading the extracted content
- Inspecting heading hierarchy
- Reviewing tables in Markdown form
- Debugging conversion quality
- Feeding a Markdown representation into a downstream pipeline

## 9.2 Dictionary / JSON-like export

```python
data = doc.export_to_dict()
```

Useful when you need a structured representation for:

- Custom processing
- Inspection of document elements
- Metadata extraction
- Building a custom ingestion pipeline

You can save the dictionary as JSON:

```python
import json

with open("output/annual_report.json", "w", encoding="utf-8") as f:
    json.dump(data, f, ensure_ascii=False, indent=2)
```

The exact fields in the exported representation depend on the Docling version and document content.

## 9.3 Choosing an export

| Export | Best use |
|---|---|
| Markdown | Human-readable content and a convenient text representation |
| Dictionary / JSON | Programmatic inspection and custom processing |
| Document chunks | Retrieval-oriented chunk output through the LangChain integration |

**Important:** exporting Markdown does not mean that every visual detail of every image or chart has been converted into a complete textual explanation. Inspect the output and add image understanding where the application requires it.

---

# 10. Extracting each PDF element

This section connects the PDF elements to practical ingestion decisions.

## 10.1 Text extraction

Use the converted document's Markdown export to inspect text:

```python
markdown = doc.export_to_markdown()

print(markdown)
```

Check that:

- Paragraphs are present.
- Sentences are in the correct order.
- Page sections are not mixed.
- Important text is not missing.

## 10.2 Heading extraction and context

A heading should ideally stay connected to its content during chunking.

Example:

```markdown
# Financial Results

## Revenue

Revenue increased by 12% during the year.

## Operating Costs

Operating costs increased by 5%.
```

A good chunking strategy should avoid separating `Revenue increased by 12%` from the `Revenue` heading if that heading is needed to understand the statement.

## 10.3 Table extraction

Inspect Markdown output for table formatting:

```python
markdown = doc.export_to_markdown()

if "|" in markdown:
    print("Possible Markdown table content found")
```

This is only a simple inspection example, not a complete table-detection algorithm.

For production use, inspect the structured table items in the Docling document model and validate important tables against the original PDF. For critical numeric data, consider storing a normalized table representation with its title, page, column names, units, and source reference.

## 10.4 Picture and image handling

A picture in the source document may be represented as a picture item in the document model. Depending on the pipeline, you may need to inspect picture references or export image assets separately.

Conceptual flow:

```text
PDF page
   |
   v
Picture region detected
   |
   +--> Preserve image reference / asset
   |
   +--> OCR if image contains text
   |
   +--> Vision model if visual interpretation is required
   |
   v
Text description + source/page reference
   |
   v
Index for retrieval
```

Do not assume that `export_to_markdown()` alone gives you a complete semantic description of every image.

## 10.5 Multi-column pages

Review extracted Markdown around pages with columns. Verify:

- The first column is not mixed with the second.
- Paragraph order makes sense.
- Headings appear before the section they introduce.
- Footnotes are not inserted into unrelated paragraphs.

## 10.6 Charts and diagrams

For charts, diagrams, and screenshots, decide whether text extraction is enough. If not, create a companion textual description using OCR or a vision-language model, and retain the original image's source/page reference.

Example of a useful companion description:

```text
Source: annual_report.pdf
Page: 18
Figure: Revenue by Quarter
Description: The chart shows quarterly revenue increasing from Q1 to Q4.
Extracted values: Q1 10M, Q2 12M, Q3 15M, Q4 18M.
```

The example is illustrative. Do not create values that are not actually present in the source.

---

# 11. Images, OCR, scanned pages, and charts: important limitations

## 11.1 Scanned PDFs

A scanned PDF page is often a picture of a page rather than selectable text.

```text
Scanned PDF page
       |
       v
Image / page processing
       |
       v
OCR
       |
       v
Recognized text
       |
       v
Document structure and chunking
```

Docling supports OCR-oriented processing, but the OCR options and configuration depend on the installed version and selected pipeline.

OCR quality depends on:

- Image resolution
- Skew and rotation
- Blur
- Handwriting
- Language
- Table complexity
- Background noise

Always inspect OCR output for important documents.

## 11.2 Image extraction vs image understanding

These are different tasks:

| Task | Meaning |
|---|---|
| Image detection | Identify that an image or picture region exists |
| Image extraction | Obtain or preserve the image asset |
| OCR | Read text inside an image |
| Image captioning | Produce a natural-language description |
| Visual reasoning | Interpret relationships, diagrams, charts, or visual meaning |

A complete multimodal RAG pipeline may combine these methods.

## 11.3 Tables inside scanned pages

A scanned table requires OCR and table-structure recognition. Plain OCR can read the words but may not correctly associate each value with its row and column.

For critical tables:

1. Inspect the original page.
2. Validate row and column boundaries.
3. Check units and headers.
4. Confirm values.
5. Preserve the page reference.
6. Store a readable table representation.

## 11.4 Charts are not automatically understood

A chart's labels may be extracted while the actual visual relationship is missed. If chart interpretation is important, use a suitable visual processing step and validate the resulting description.

**Practical rule:** Docling is the document-ingestion foundation; OCR and vision models may be additional components for image-heavy or multimodal use cases.

---

# 12. Use Docling through LangChain

When you want the result as LangChain documents, use the Docling integration.

## 12.1 Basic `DoclingLoader`

```python
from langchain_docling.loader import DoclingLoader

loader = DoclingLoader(
    file_path="documents/annual_report.pdf"
)

docs = loader.load()

print(type(docs))
print(len(docs))
print(docs[0].page_content[:500])
print(docs[0].metadata)
```

The loader returns LangChain `Document` objects.

## 12.2 Loading different file types

Examples:

```python
from langchain_docling.loader import DoclingLoader

# PDF
pdf_docs = DoclingLoader(
    file_path="documents/annual_report.pdf"
).load()

# Word document
docx_docs = DoclingLoader(
    file_path="documents/policy.docx"
).load()

# PowerPoint
pptx_docs = DoclingLoader(
    file_path="documents/training.pptx"
).load()

# Text file
txt_docs = DoclingLoader(
    file_path="documents/knowledge.txt"
).load()
```

The exact supported formats depend on the Docling version and configured document converters. Confirm support for the file type you plan to ingest.

## 12.3 Loading multiple files

```python
from pathlib import Path
from langchain_docling.loader import DoclingLoader

all_docs = []

for file_path in Path("documents").glob("*"):
    if file_path.is_file():
        loader = DoclingLoader(file_path=str(file_path))
        all_docs.extend(loader.load())

print(f"Loaded {len(all_docs)} LangChain documents")
```

For a real project, filter the file extensions you intend to support. Do not send every file in a directory to one loader without checking its type.

Example filter:

```python
from pathlib import Path

SUPPORTED_EXTENSIONS = {".pdf", ".docx", ".pptx", ".txt"}

for file_path in Path("documents").iterdir():
    if file_path.suffix.lower() not in SUPPORTED_EXTENSIONS:
        continue

    # Process the supported file here.
```

---

# 13. MARKDOWN vs DOC_CHUNKS

The LangChain Docling integration provides export choices. Two commonly used choices are `MARKDOWN` and `DOC_CHUNKS`.

## 13.1 MARKDOWN

```python
from langchain_docling.loader import DoclingLoader, ExportType

loader = DoclingLoader(
    file_path="documents/report.pdf",
    export_type=ExportType.MARKDOWN
)

docs = loader.load()
```

Conceptually, this mode returns a Markdown-oriented representation of the document.

Useful when:

- You want to inspect the full converted content.
- You want a readable representation.
- You want to apply your own chunking logic afterward.
- You need to debug extraction quality.

## 13.2 DOC_CHUNKS

```python
from langchain_docling.loader import DoclingLoader, ExportType

loader = DoclingLoader(
    file_path="documents/report.pdf",
    export_type=ExportType.DOC_CHUNKS
)

docs = loader.load()
```

This mode uses document-oriented chunking and is commonly the more direct choice for RAG ingestion.

Useful when:

- You want structure-aware chunks.
- You want to preserve document context.
- You want to send chunks toward embeddings and a vector database.

## 13.3 Comparison

| Question | MARKDOWN | DOC_CHUNKS |
|---|---|---|
| Main output idea | Markdown-oriented document representation | Document chunks |
| Best for | Inspection and custom processing | Retrieval ingestion |
| Chunking | You may apply your own chunking | Chunking is part of the export path |
| RAG use | Possible with a later chunking step | Directly suited to chunk-based ingestion |

**Practical recommendation:** inspect the Markdown output during development; use `DOC_CHUNKS` when you want the Docling integration to produce document-aware chunks for RAG. Verify the behavior against your installed integration version.

---

# 14. Chunking with HybridChunker

## 14.1 Why chunking is needed

Embedding models and vector databases usually work with smaller text units rather than an entire 300-page report.

Bad chunking can:

- Separate a heading from its explanation.
- Split a table across unrelated chunks.
- Break a numbered procedure.
- Remove context needed to interpret a value.
- Create chunks that are too large or too small.

## 14.2 What is `HybridChunker`?

Docling's `HybridChunker` is designed to create chunks using document structure and token-size considerations. It can use document hierarchy and context when forming chunks.

The word **hybrid** refers to combining structural awareness with chunk-size/token constraints.

## 14.3 Basic example

```python
from docling.document_converter import DocumentConverter
from docling.chunking import HybridChunker

converter = DocumentConverter()
result = converter.convert("documents/report.pdf")
doc = result.document

chunker = HybridChunker(
    max_tokens=256
)

chunks = list(chunker.chunk(doc))

print(f"Number of chunks: {len(chunks)}")

for index, chunk in enumerate(chunks, start=1):
    print(f"\n--- Chunk {index} ---")
    print(chunk)
```

The exact constructor options and chunk object fields can vary by Docling version. Check the installed version's API before relying on optional arguments.

## 14.4 What does HybridChunker help preserve?

- Section and heading context
- Document-aware boundaries
- Better grouping of related content
- Chunk sizes constrained by token limits
- More meaningful units for embedding and retrieval

## 14.5 Conceptual example

Source document:

```text
Financial Results
    Revenue
        Revenue increased by 12%.
    Operating Costs
        Operating costs increased by 5%.
```

A structure-aware chunk could retain the path:

```text
Financial Results > Revenue

Revenue increased by 12%.
```

Another chunk could retain:

```text
Financial Results > Operating Costs

Operating costs increased by 5%.
```

The heading path helps the retriever return content with context.

## 14.6 Chunk-size guidance

The uploaded study guide uses **200–500 tokens** as a practical range to monitor, and shows a `max_tokens=256` example. These are starting points, not universal rules.

Choose chunk size based on:

- Embedding model limits
- LLM context window
- Average document structure
- Query type
- Whether tables or procedures need larger context
- Retrieval quality measured on representative questions

Do not optimize only for chunk count. Evaluate whether retrieved chunks answer real user questions.

---

# 15. Metadata and provenance

Metadata explains where a chunk came from and how it relates to the original document.

Example:

```python
for item in docs:
    print(item.metadata)
```

Illustrative metadata:

```python
{
    "source": "report.pdf",
    "page_no": 5,
    "section": "Financial Results"
}
```

The exact metadata keys depend on the loader, exporter, and version.

## Why metadata matters

Metadata can support:

- Source citations in answers
- Page-level references
- Filtering by document or department
- Version tracking
- Debugging extraction errors
- Replacing or deleting chunks during re-ingestion
- Access-control filtering

## Recommended metadata to preserve

Where available and relevant, preserve:

| Metadata | Why it is useful |
|---|---|
| Source file name or URI | Identify the original document |
| Page number | Link answer back to a page |
| Section / heading path | Preserve topic context |
| Chunk identifier | Track an individual chunk |
| Document identifier | Group chunks from one source |
| Version / modified timestamp | Support document updates |
| Content type | Distinguish text, table, image description, etc. |
| Image/table reference | Link back to the original element |

Do not assume every field is automatically supplied. Add application-specific metadata during ingestion where required.

---

# 16. End-to-end Docling + LangChain + Qdrant example

This example shows the main RAG ingestion path. It is a starting template; check package versions and connection settings in your environment.

## 16.1 Start Qdrant locally

```bash
docker run -p 6333:6333 qdrant/qdrant
```

This starts a local Qdrant service using the default local configuration.

## 16.2 Load the PDF as document chunks

```python
from langchain_docling.loader import DoclingLoader, ExportType

loader = DoclingLoader(
    file_path="documents/annual_report.pdf",
    export_type=ExportType.DOC_CHUNKS
)

docs = loader.load()

print(f"Loaded {len(docs)} chunks")
```

## 16.3 Inspect the chunks and metadata

```python
for index, item in enumerate(docs[:3], start=1):
    print(f"\n--- Chunk {index} ---")
    print(item.page_content[:1000])
    print(item.metadata)
```

Check the extracted content before embedding it. This is where you catch broken tables, missing pages, or unexpected reading order.

## 16.4 Create embeddings

Example using a Hugging Face sentence-transformer model:

```python
from langchain_huggingface import HuggingFaceEmbeddings

embeddings = HuggingFaceEmbeddings(
    model_name="sentence-transformers/all-MiniLM-L6-v2"
)
```

The embedding model converts text into numerical vectors that can be compared by semantic similarity.

## 16.5 Store chunks in Qdrant

```python
from langchain_qdrant import QdrantVectorStore

vectorstore = QdrantVectorStore.from_documents(
    documents=docs,
    embedding=embeddings,
    url="http://localhost:6333",
    collection_name="docling_docs"
)
```

Depending on the installed LangChain Qdrant version, parameter names may differ. Follow the API signature for your installed package.

An alternative explicit client setup:

```python
from qdrant_client import QdrantClient

client = QdrantClient(
    url="http://localhost:6333"
)
```

## 16.6 Create a retriever

```python
retriever = vectorstore.as_retriever(
    search_kwargs={"k": 5}
)
```

`k=5` asks for five candidate chunks. Tune this value using evaluation rather than assuming five is always optimal.

## 16.7 Retrieve relevant chunks

```python
question = "What was the company's quarterly revenue?"

results = retriever.invoke(question)

for index, item in enumerate(results, start=1):
    print(f"\n--- Result {index} ---")
    print(item.page_content)
    print(item.metadata)
```

## 16.8 Pass retrieved context to an LLM

A simple illustrative example:

```python
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(model="gpt-4o")

question = "What was the company's quarterly revenue?"
retrieved_docs = retriever.invoke(question)

context = "\n\n".join(
    item.page_content for item in retrieved_docs
)

prompt = f"""
Answer the question using only the context below.
If the context does not contain the answer, say that you cannot find it.

Context:
{context}

Question:
{question}
"""

response = llm.invoke(prompt)

print(response.content)
```

For production, use a proper prompt template or a LangChain retrieval chain, include source metadata in the context, and return citations to the original document/page.

## 16.9 Full pipeline summary

```text
annual_report.pdf
       |
       v
   DoclingLoader
       |
       v
   DOC_CHUNKS
       |
       v
LangChain Documents
       |
       v
HuggingFace Embeddings
       |
       v
      Qdrant
       |
       v
    Retriever
       |
       v
Retrieved context
       |
       v
      LLM
       |
       v
      Answer
```

---

# 17. Advanced RAG retrieval after ingestion

Docling improves the document ingestion stage. It does not replace retrieval strategies. Once chunks are indexed, you can add advanced retrievers and reranking.

## 17.1 Contextual Compression Retriever

Purpose: reduce retrieved content to the portions most relevant to the query.

```text
User query
    |
    v
Base retriever
    |
    v
Candidate documents
    |
    v
Compression / relevance filtering
    |
    v
Important passages
    |
    v
LLM
```

Benefits:

- Less irrelevant context
- Lower prompt-token usage
- Potentially better answer focus
- Reduced context passed to the LLM

LangChain import example:

```python
from langchain.retrievers import ContextualCompressionRetriever
```

A complete implementation also needs a compressor, such as an LLM-based or extraction-based document compressor, configured for the chosen LangChain version.

## 17.2 Parent Document Retriever

Purpose: search using smaller child chunks but return a larger parent section when more context is needed.

```text
Full document
     |
     v
Parent sections
     |
     v
Smaller child chunks
     |
     v
Embed child chunks
     |
     v
Find matching child chunk
     |
     v
Return parent section
     |
     v
LLM receives broader context
```

Benefits:

- Precise matching through small chunks
- More surrounding context for answer generation
- Less chance of returning a tiny fragment without explanation

## 17.3 Reranking

Purpose: reorder the initial retrieved candidates according to how well each candidate answers the actual query.

```text
User query
    |
    v
Vector retriever
    |
    v
Top candidate chunks
    |
    v
Reranker
    |
    v
Best-ranked chunks
    |
    v
LLM
```

A common design uses a bi-encoder embedding model for first-stage retrieval and a cross-encoder reranker for deeper query-document relevance scoring.

## 17.4 Production retrieval chain

```text
Question
   |
   v
Query processing
   |
   v
Vector / hybrid retrieval
   |
   v
Candidate chunks
   |
   v
Reranking
   |
   v
Contextual compression (optional)
   |
   v
LLM
   |
   v
Answer + source references
```

Docling prepares high-quality source content; retrieval components decide which content is most useful for a particular question.

---

# 18. Production ingestion architecture

A real-time or frequently updated enterprise ingestion pipeline needs more than a one-time PDF conversion.

## 18.1 Recommended architecture

```text
Enterprise document sources
(PDF / DOCX / PPTX / HTML / images)
              |
              v
       File discovery / sync
              |
              v
     File validation and routing
              |
              v
       Docling conversion
              |
              v
   OCR / image understanding when needed
              |
              v
    Structured document representation
              |
              v
       Structure-aware chunking
              |
              v
     Metadata + provenance tagging
              |
              v
          Embeddings
              |
              v
       Vector database (Qdrant)
              |
              v
     Retriever / hybrid search
              |
              v
       Reranker / compression
              |
              v
             LLM
```

## 18.2 Handling document updates

For frequently changing documents:

1. Detect new or modified files.
2. Compute a stable document identifier and version/fingerprint.
3. Convert the document.
4. Generate chunks and metadata.
5. Upsert the new chunks.
6. Remove or retire stale chunks from the previous version.
7. Keep source references for traceability.

Avoid blindly appending every re-ingested version. Otherwise, the vector database can contain duplicate or stale content.

## 18.3 Preserve original source references

For every chunk, retain enough metadata to locate the source:

- Document ID
- File name or URI
- Page number, where available
- Section or heading path
- Version or timestamp
- Chunk ID

This makes it possible to show users where an answer came from and to debug incorrect retrieval.

## 18.4 Security and governance

For enterprise systems:

- Apply document-level access controls.
- Filter retrieval by the user's authorization.
- Do not rely on the LLM to enforce access permissions.
- Track document versions.
- Log ingestion failures.
- Protect sensitive source files and embeddings.
- Validate OCR and extracted content for high-impact use cases.

---

# 19. Best practices and troubleshooting

## 19.1 Best practices

1. Use Docling for complex, document-centric sources where layout and structure matter.
2. Inspect Markdown output during development.
3. Use `DOC_CHUNKS` when you want the Docling LangChain integration to provide document-aware chunks.
4. Evaluate `HybridChunker` settings against real questions and source documents.
5. Preserve metadata and provenance.
6. Validate important tables against the original PDF.
7. Use OCR for scanned pages when required.
8. Add image or chart understanding when visual content is important.
9. Use a vector database such as Qdrant for semantic retrieval.
10. Add reranking when first-stage retrieval returns too many weak candidates.
11. Consider contextual compression when retrieved context contains irrelevant material.
12. Monitor chunk sizes; the study example uses 200–500 tokens as a starting range.
13. Keep the original document and page references.
14. Test with representative questions before calling ingestion complete.

## 19.2 Troubleshooting checklist

| Symptom | Possible cause | What to check |
|---|---|---|
| Missing text | Scanned page, unusual fonts, damaged PDF | OCR settings and source page |
| Broken table | Complex table layout or low-quality scan | Table extraction and page image |
| Wrong paragraph order | Multi-column or unusual layout | Reading-order output |
| Image has no useful description | Image was detected but not semantically interpreted | OCR or vision processing |
| Chart values missing | Chart information is visual rather than textual | Chart-specific extraction/validation |
| Chunks lack context | Heading not included or chunk boundary unsuitable | Chunker output and metadata |
| Duplicate retrieval results | Re-ingestion created duplicate vectors | Document IDs and update/delete strategy |
| Import error | Package API/version mismatch | Installed versions and current API |
| Empty or poor OCR | Low resolution, skew, handwriting, unsupported language | Image quality and OCR configuration |

## 19.3 Quality validation before embedding

Before creating embeddings, inspect a sample of chunks:

```python
for index, item in enumerate(docs[:10], start=1):
    print(f"\n===== CHUNK {index} =====")
    print(item.page_content)
    print("METADATA:", item.metadata)
```

Ask:

- Does each chunk make sense by itself?
- Is its heading or section context present?
- Are table headers and values connected?
- Is the source page available?
- Is there duplicated or irrelevant content?
- Are scanned pages represented?
- Are images/charts important to the use case, and if so, have they been handled?

---

# 20. Interview explanation

## 20.1 Story format

**Scenario:** Imagine we are building an enterprise chatbot over annual reports, policies, and technical manuals. These PDFs contain paragraphs, tables, charts, images, scanned pages, and multiple columns.

**Problem:** If we use a basic text loader, it may extract text but lose table relationships, headings, or reading order. Scanned pages may not contain selectable text. When the content is split into chunks, important context can be lost. The vector database then stores poor chunks, and retrieval quality suffers.

**Solution:** We use Docling to understand the document structure before indexing it. Docling converts the source into a structured `DoclingDocument`. We can inspect or export it as Markdown or a structured representation. With the LangChain integration, `DoclingLoader` can produce LangChain `Document` objects. We can use document-aware chunking such as `HybridChunker`, preserve metadata, create embeddings, and store the chunks in Qdrant.

**Retrieval:** At query time, a retriever finds relevant chunks. We can add reranking or contextual compression before sending the selected context to the LLM.

## 20.2 Interview-ready answer

> “For complex PDF ingestion in a RAG application, I use Docling because it is designed for document understanding rather than only extracting a flat string. It analyzes document structure and layout, including headings, paragraphs, tables, lists, and picture regions. It can also be configured for OCR-oriented processing of scanned documents.
>
> When I use core Docling, the converted output is a `DoclingDocument`. When I use the LangChain integration through `DoclingLoader`, the content is converted into LangChain `Document` objects. I can use Markdown export for inspection or document chunks for retrieval. I then preserve source and page metadata, generate embeddings, and store the chunks in a vector database such as Qdrant. At query time, I use retrieval and, when needed, reranking or contextual compression before passing the context to the LLM.
>
> For image-heavy PDFs and charts, I validate the extraction and add OCR or vision processing when the visual content must be understood.”

## 20.3 Common interview questions

### Q1. What is Docling?

Docling is an open-source document-understanding framework that converts complex documents into structured representations for AI and RAG applications.

### Q2. Why use Docling instead of only PyPDFLoader?

A basic PDF loader is often sufficient for simple text PDFs. Docling is useful when document hierarchy, layout, tables, reading order, and scanned content matter.

### Q3. Does Docling return LangChain documents?

Core Docling returns a `DoclingDocument`. When using the LangChain integration through `DoclingLoader`, the result is a list of LangChain `Document` objects.

### Q4. What is the difference between `DoclingDocument` and LangChain `Document`?

`DoclingDocument` is Docling's richer structured document model. LangChain `Document` is a standard object containing `page_content` and `metadata`, commonly used in retrieval pipelines.

### Q5. What is `DoclingLoader`?

It is the LangChain integration loader used to load documents through Docling and return LangChain document objects.

### Q6. What is the difference between `MARKDOWN` and `DOC_CHUNKS`?

`MARKDOWN` provides a Markdown-oriented representation. `DOC_CHUNKS` provides document-aware chunks that are more directly useful for RAG ingestion.

### Q7. What is `HybridChunker`?

It is a Docling chunker that combines document structure awareness with chunk-size/token constraints to create meaningful chunks.

### Q8. How do you handle scanned PDFs?

Use an OCR-capable conversion configuration, inspect OCR quality, and validate important content against the original pages.

### Q9. Does Docling understand every image and chart automatically?

No. Detecting or extracting an image is different from interpreting its meaning. OCR or a vision-language processing stage may be required.

### Q10. Why preserve metadata?

Metadata supports source citations, page references, filtering, versioning, debugging, and traceability.

### Q11. Where does Qdrant fit?

Qdrant stores vector embeddings and associated payload/metadata so the retriever can find relevant chunks for a query.

### Q12. Does Docling replace advanced retrievers?

No. Docling improves ingestion. Retrievers, rerankers, and compression techniques operate later in the RAG pipeline.

---

# 21. Quick revision and memory tricks

## Remember the responsibilities

| Component | Responsibility | Memory phrase |
|---|---|---|
| Docling | Understand and convert the document | **Understand** |
| `DoclingDocument` | Structured representation | **Structure** |
| `DoclingLoader` | Connect Docling to LangChain | **Bridge** |
| `HybridChunker` | Create meaningful chunks | **Chunk** |
| LangChain `Document` | Carry text and metadata | **Package** |
| Embedding model | Convert text into vectors | **Vectorize** |
| Qdrant | Store and search vectors | **Store** |
| Retriever | Find relevant chunks | **Retrieve** |
| Reranker | Reorder candidates by relevance | **Reorder** |
| LLM | Generate the answer from context | **Answer** |

## One-line flow to memorize

> **PDF → Docling → DoclingDocument → Chunking → LangChain Documents → Embeddings → Qdrant → Retriever → LLM**

## The most important distinction

> **Core Docling:** PDF → `DoclingDocument`
>
> **Docling + LangChain:** PDF → `DoclingDocument` internally → LangChain `Document` objects

## Easy memory trick for complex PDF processing

**T-H-T-I-O-C**

- **T**ext
- **H**eadings
- **T**ables
- **I**mages
- **O**CR
- **C**harts / columns

When reviewing a complex PDF ingestion pipeline, check each of these elements.

---

# 22. Practice exercises

Use a real PDF that contains a mixture of text, tables, images, and multiple columns.

## Exercise 1 – Basic conversion

- [ ] Install Docling.
- [ ] Convert a PDF with `DocumentConverter`.
- [ ] Print the type of `result.document`.
- [ ] Export to Markdown.
- [ ] Save the Markdown to a file.

## Exercise 2 – Inspect extraction quality

- [ ] Find a page with headings.
- [ ] Find a page with a table.
- [ ] Find a page with an image or diagram.
- [ ] Find a multi-column page.
- [ ] Compare the exported content with the original PDF.

## Exercise 3 – LangChain loading

- [ ] Load the same PDF with `DoclingLoader`.
- [ ] Print the number of returned documents.
- [ ] Print `page_content` for a few items.
- [ ] Inspect metadata.
- [ ] Compare `MARKDOWN` and `DOC_CHUNKS`.

## Exercise 4 – Chunking and retrieval

- [ ] Try `HybridChunker`.
- [ ] Inspect chunk boundaries.
- [ ] Create embeddings.
- [ ] Store chunks in Qdrant.
- [ ] Ask five questions that require different sections of the PDF.
- [ ] Check whether retrieved chunks contain enough context.

## Exercise 5 – Multimodal checks

- [ ] Test a scanned page.
- [ ] Test a table inside a scanned page.
- [ ] Test a chart.
- [ ] Decide where OCR or a vision model is required.
- [ ] Preserve a source/page reference for each extracted description.

---

# 23. Reference notes and version caveats

- Docling, `langchain-docling`, LangChain, and vector-store integrations evolve. Check the versions installed in your environment when an import, constructor, or parameter differs from an example.
- The code examples are learning templates. Some APIs may require small changes for a particular package release.
- OCR, image extraction, chart understanding, and table extraction are related but distinct tasks. Do not assume that one setting solves every multimodal requirement.
- Validate important facts, financial values, tables, and compliance-related content against the original PDF.
- The `200–500` token chunk range is a starting point from the study material, not a fixed rule. Evaluate retrieval quality with your own documents and questions.
- Docling is best thought of as a **document-centric ingestion and understanding framework**, not a universal loader for every data source.

## Final takeaway

Docling helps convert a complex PDF into a structured representation. LangChain integration makes that content convenient to use in a RAG pipeline. Structure-aware chunking and metadata improve the quality and traceability of indexed content. Qdrant stores the embeddings, and retrieval components select relevant information for the LLM.

**The quality of a RAG answer starts with the quality of document ingestion.**

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
