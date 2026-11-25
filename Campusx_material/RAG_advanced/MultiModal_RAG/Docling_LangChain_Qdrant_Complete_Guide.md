# Docling + LangChain + Qdrant Complete Guide
# What is Docling?

Docling is an open-source document understanding framework developed by IBM that converts complex documents into structured representations suitable for AI applications and Retrieval-Augmented Generation (RAG).

It supports:

- PDF
- DOCX
- PPTX
- HTML
- Images
- Scanned Documents

Unlike traditional PDF loaders, Docling understands:

- Document hierarchy
- Headings
- Tables
- Lists
- Images
- Layout information
- Reading order

This makes it ideal for enterprise RAG systems.

---

# Why Do We Need Docling?

## Traditional PDF Loaders

Common loaders:

```python
PyPDFLoader
PDFPlumber
UnstructuredPDFLoader
```

Traditional Approach:

```text
PDF
 ↓
Extract Raw Text
 ↓
Character Splitter
 ↓
Embeddings
 ↓
Vector DB
```

### Problems

1. Broken tables
2. Lost headings
3. Wrong reading order
4. No layout understanding
5. Poor chunk quality

---

# How Docling Solves This

```text
PDF
 ↓
Docling
 ↓
Document Object
 ↓
Structured Elements
 ├── Headings
 ├── Paragraphs
 ├── Tables
 ├── Lists
 └── Images
```

---
## 1. What is Docling?

Docling is an open-source document understanding framework that converts PDFs, DOCX, PPTX, HTML, images, and text documents into structured representations for AI and RAG applications.

### Why not use traditional PDF loaders?

Traditional loaders:
- PyPDFLoader
- PDFPlumber
- Unstructured

Problems:
- Broken tables
- Lost headings
- No layout awareness
- Poor chunk quality

Docling preserves:
- Headings
- Sections
- Tables
- Lists
- Reading order
- Metadata

---

## 2. Docling Architecture

PDF/DOCX/PPTX/TXT
        |
        v
     Docling
        |
        v
 DoclingDocument
        |
        v
    Chunker
        |
        v
LangChain Documents
        |
        v
   Embeddings
        |
        v
 Qdrant / Chroma
        |
        v
   Retriever
        |
        v
      LLM

---

# 🔄 Does Docling Create LangChain Document Objects?

## 🎯 Short Answer

✅ **Yes, when Docling is used through LangChain.**

⚠️ **No, when using core Docling directly.**

The key is understanding the difference between:

1. 📘 **DoclingDocument**
2. 📄 **LangChain Document**

---

# 🏗️ Understanding the Flow

```text
PDF
 │
 ▼
DoclingDocument
 │
 ▼
LangChain Document
 │
 ▼
Embeddings
 │
 ▼
Vector Database
```

---

# 📌 Case 1: Traditional LangChain Loader

Example using `PyPDFLoader`

```python
from langchain_community.document_loaders import PyPDFLoader

loader = PyPDFLoader("report.pdf")

docs = loader.load()
```

### Output

```python
[
    Document(
        page_content="...",
        metadata={}
    )
]
```

✅ Directly creates **LangChain Document** objects.

---

# 📌 Case 2: Pure Docling

Example:

```python
from docling.document_converter import DocumentConverter

converter = DocumentConverter()

result = converter.convert("report.pdf")

doc = result.document
```

### Output

```python
DoclingDocument
```

✅ Creates:

```text
DoclingDocument
```

❌ Does NOT create:

```text
LangChain Document
```

---

# 🧠 What is Inside a DoclingDocument?

```text
DoclingDocument
├── Headings
├── Sections
├── Paragraphs
├── Tables
├── Lists
├── Images
├── Layout Information
└── Metadata
```

Docling understands the structure of a document before converting it into chunks.

---

# 📌 Case 3: Docling + LangChain

Example:

```python
from langchain_docling.loader import DoclingLoader

loader = DoclingLoader(
    file_path="report.pdf"
)

docs = loader.load()
```

### Output

```python
[
    Document(
        page_content="...",
        metadata={}
    )
]
```

✅ Returns:

```python
List[Document]
```

where `Document` is a **LangChain Document**.

---

# 🔄 What Happens Internally?

```text
report.pdf
      │
      ▼
DocumentConverter
      │
      ▼
📘 DoclingDocument
      │
      ▼
HybridChunker
      │
      ▼
📄 LangChain Documents
      │
      ▼
Embeddings
      │
      ▼
Qdrant / Chroma
```

---

# 📋 Actual Processing Pipeline

### Step 1

📄 Input Document

```text
annual_report.pdf
```

---

### Step 2

📘 Docling Parses Document

Creates:

```python
DoclingDocument
```

which contains:

✅ Headings

✅ Tables

✅ Images

✅ Metadata

✅ Layout Information

---

### Step 3

🔪 HybridChunker Creates Chunks

```python
HybridChunker
```

Output:

```text
Chunk 1
Chunk 2
Chunk 3
...
```

---

### Step 4

📄 Chunks Become LangChain Documents

```python
[
    Document(
        page_content="Revenue increased...",
        metadata={...}
    )
]
```

These are standard LangChain objects.

---

# ⚖️ PyPDFLoader vs DoclingLoader

| Feature | PyPDFLoader | DoclingLoader |
|----------|------------|---------------|
| Returns LangChain Document | ✅ | ✅ |
| Creates DoclingDocument | ❌ | ✅ |
| Table Understanding | ❌ | ✅ |
| Layout Awareness | ❌ | ✅ |
| OCR Support | ❌ | ✅ |
| Structure-Aware Chunking | ❌ | ✅ |
| Metadata Richness | ⚠️ Limited | ✅ Rich |

---

# 🔍 Why Use Docling If Both Return Document Objects?

Great question!

Both eventually produce:

```python
Document(
    page_content="...",
    metadata={}
)
```

The difference is **how the content is generated**.

---

## 🚫 Traditional Loader

May extract:

```text
Revenue Product Sales 2024
10000 ProductA
20000 ProductB
```

❌ Table structure lost

❌ Context lost

❌ Poor chunk quality

---

## ✅ Docling Loader

Produces:

```markdown
# Revenue

| Product | Sales |
|----------|--------|
| ProductA | 10000 |
| ProductB | 20000 |
```

✅ Better structure

✅ Better retrieval

✅ Better embeddings

✅ Better RAG responses

---

# 🎯 Interview Answer

> When using the core Docling library, the output is a **DoclingDocument**, which is a rich document representation containing structure, layout, tables, images, and metadata.
>
> When using **Docling through LangChain's `DoclingLoader`**, the DoclingDocument is internally converted into standard **LangChain Document objects**.
>
> Therefore, the final output is still a list of LangChain Documents, but with much richer content and metadata than traditional loaders like PyPDFLoader.

---

# ⭐ One-Line Summary

> 📄 **PyPDFLoader:** PDF → LangChain Document
>
> 📘 **DoclingLoader:** PDF → DoclingDocument → LangChain Document

The extra **DoclingDocument layer** is where all the advanced document understanding happens.

# 🌍 Is Docling a Universal Loader?

## ⚠️ Not Truly Universal

**Docling** is primarily designed for **document ingestion, document understanding, and RAG pipelines**.

Think of it as a **Universal Document Loader**, not a Universal Data Loader.

---

# ✅ Supported File Types

Docling works exceptionally well with enterprise document formats:

📄 **PDF**

📝 **DOCX (Microsoft Word)**

📊 **PPTX (PowerPoint)**

🌐 **HTML**

📃 **TXT Files**

📘 **Markdown Files**

🖼️ **Images (PNG, JPG, TIFF)**

🔍 **Scanned Documents (OCR)**

📚 **Research Papers**

📋 **Policies & Manuals**

📑 **Reports & Documentation**

---

# ❌ Not Intended For

Docling is **not optimized** for structured data sources or multimedia content.

🚫 CSV Files

🚫 Parquet Files

🚫 JSON APIs

🚫 SQL Databases

🚫 Excel Analytics Workflows

🚫 Audio Files

🚫 Video Files

🚫 Streaming Data

🚫 Time-Series Databases

---

# 🎯 Recommended Alternatives

| 📂 Data Source | ✅ Recommended Loader |
|---------------|----------------------|
| PDF | DoclingLoader |
| DOCX | DoclingLoader |
| PPTX | DoclingLoader |
| HTML | DoclingLoader |
| TXT | DoclingLoader |
| CSV | CSVLoader |
| Excel | DataFrameLoader |
| SQL Database | SQLDatabaseLoader |
| REST API | Custom API Loader |
| YouTube | YoutubeLoader |
| Audio | Whisper / STT Pipeline |
| Video | Video Processing Pipeline |

---

# 🧠 Quick Rule of Thumb

## 📄 If it's a document

Use:

```python
DoclingLoader
```

Examples:

✅ PDF

✅ DOCX

✅ PPTX

✅ HTML

✅ TXT

✅ Images

---

## 📊 If it's structured data

Use specialized loaders.

Examples:

✅ CSVLoader

✅ DataFrameLoader

✅ SQLDatabaseLoader

✅ API Loader

---

# 🏗️ Architecture View

```text
📄 PDF
📝 DOCX
📊 PPTX
🌐 HTML
📃 TXT
🖼️ Images
🔍 Scanned PDFs
        |
        v
┌─────────────────┐
│   DoclingLoader │
└─────────────────┘
        |
        v
📘 DoclingDocument
        |
        v
📄 LangChain Documents
        |
        v
🧮 Embeddings
        |
        v
🗄️ Vector Database
        |
        v
🔎 Retriever
        |
        v
🤖 LLM
```

---

# 💡 Interview Answer

> 🎯 **Docling is not a fully universal loader.**
>
> It is best described as a **document-centric ingestion framework** or a **universal document loader** because it supports most enterprise document formats including PDF, DOCX, PPTX, HTML, TXT, Markdown, images, and scanned documents.
>
> For non-document sources such as CSV, databases, APIs, audio, and video, specialized loaders are typically more suitable.

---

# ⭐ One-Line Summary

> **Docling = Universal Document Loader ✅**
>
> **Docling ≠ Universal Data Loader ❌**
>

# 🔄 Does Docling Create LangChain Document Objects?

## 🎯 Short Answer

✅ **Yes, when Docling is used through LangChain.**

⚠️ **No, when using core Docling directly.**

The key is understanding the difference between:

1. 📘 **DoclingDocument**
2. 📄 **LangChain Document**

---

# 🏗️ Understanding the Flow

```text
PDF
 │
 ▼
DoclingDocument
 │
 ▼
LangChain Document
 │
 ▼
Embeddings
 │
 ▼
Vector Database
```

---

# 📌 Case 1: Traditional LangChain Loader

Example using `PyPDFLoader`

```python
from langchain_community.document_loaders import PyPDFLoader

loader = PyPDFLoader("report.pdf")

docs = loader.load()
```

### Output

```python
[
    Document(
        page_content="...",
        metadata={}
    )
]
```

✅ Directly creates **LangChain Document** objects.

---

# 📌 Case 2: Pure Docling

Example:

```python
from docling.document_converter import DocumentConverter

converter = DocumentConverter()

result = converter.convert("report.pdf")

doc = result.document
```

### Output

```python
DoclingDocument
```

✅ Creates:

```text
DoclingDocument
```

❌ Does NOT create:

```text
LangChain Document
```

---

# 🧠 What is Inside a DoclingDocument?

```text
DoclingDocument
├── Headings
├── Sections
├── Paragraphs
├── Tables
├── Lists
├── Images
├── Layout Information
└── Metadata
```

Docling understands the structure of a document before converting it into chunks.

---

# 📌 Case 3: Docling + LangChain

Example:

```python
from langchain_docling.loader import DoclingLoader

loader = DoclingLoader(
    file_path="report.pdf"
)

docs = loader.load()
```

### Output

```python
[
    Document(
        page_content="...",
        metadata={}
    )
]
```

✅ Returns:

```python
List[Document]
```

where `Document` is a **LangChain Document**.

---

# 🔄 What Happens Internally?

```text
report.pdf
      │
      ▼
DocumentConverter
      │
      ▼
📘 DoclingDocument
      │
      ▼
HybridChunker
      │
      ▼
📄 LangChain Documents
      │
      ▼
Embeddings
      │
      ▼
Qdrant / Chroma
```

---

# 📋 Actual Processing Pipeline

### Step 1

📄 Input Document

```text
annual_report.pdf
```

---

### Step 2

📘 Docling Parses Document

Creates:

```python
DoclingDocument
```

which contains:

✅ Headings

✅ Tables

✅ Images

✅ Metadata

✅ Layout Information

---

### Step 3

🔪 HybridChunker Creates Chunks

```python
HybridChunker
```

Output:

```text
Chunk 1
Chunk 2
Chunk 3
...
```

---

### Step 4

📄 Chunks Become LangChain Documents

```python
[
    Document(
        page_content="Revenue increased...",
        metadata={...}
    )
]
```

These are standard LangChain objects.

---

# ⚖️ PyPDFLoader vs DoclingLoader

| Feature | PyPDFLoader | DoclingLoader |
|----------|------------|---------------|
| Returns LangChain Document | ✅ | ✅ |
| Creates DoclingDocument | ❌ | ✅ |
| Table Understanding | ❌ | ✅ |
| Layout Awareness | ❌ | ✅ |
| OCR Support | ❌ | ✅ |
| Structure-Aware Chunking | ❌ | ✅ |
| Metadata Richness | ⚠️ Limited | ✅ Rich |

---

# 🔍 Why Use Docling If Both Return Document Objects?

Great question!

Both eventually produce:

```python
Document(
    page_content="...",
    metadata={}
)
```

The difference is **how the content is generated**.

---

## 🚫 Traditional Loader

May extract:

```text
Revenue Product Sales 2024
10000 ProductA
20000 ProductB
```

❌ Table structure lost

❌ Context lost

❌ Poor chunk quality

---

## ✅ Docling Loader

Produces:

```markdown
# Revenue

| Product | Sales |
|----------|--------|
| ProductA | 10000 |
| ProductB | 20000 |
```

✅ Better structure

✅ Better retrieval

✅ Better embeddings

✅ Better RAG responses

---

# 🎯 Interview Answer

> When using the core Docling library, the output is a **DoclingDocument**, which is a rich document representation containing structure, layout, tables, images, and metadata.
>
> When using **Docling through LangChain's `DoclingLoader`**, the DoclingDocument is internally converted into standard **LangChain Document objects**.
>
> Therefore, the final output is still a list of LangChain Documents, but with much richer content and metadata than traditional loaders like PyPDFLoader.

---

# ⭐ One-Line Summary

> 📄 **PyPDFLoader:** PDF → LangChain Document
>
> 📘 **DoclingLoader:** PDF → DoclingDocument → LangChain Document

The extra **DoclingDocument layer** is where all the advanced document understanding happens.

## 3. Installation

```bash
pip install docling
pip install langchain
pip install langchain-docling
pip install langchain-huggingface
pip install langchain-qdrant
pip install qdrant-client
pip install sentence-transformers
```

---

## 4. Core Docling Example

```python
from docling.document_converter import DocumentConverter

converter = DocumentConverter()
result = converter.convert('sample.pdf')

doc = result.document

print(doc.export_to_markdown())
```

---

## 5. Export Formats

### Markdown

```python
markdown = doc.export_to_markdown()
```

### Dictionary

```python
data = doc.export_to_dict()
```

---

## 6. LangChain Integration

```python
from langchain_docling.loader import DoclingLoader

loader = DoclingLoader(
    file_path='sample.pdf'
)

docs = loader.load()
```

---

## 7. Loading Different File Types

### PDF

```python
loader = DoclingLoader(file_path='annual_report.pdf')
docs = loader.load()
```

### DOCX

```python
loader = DoclingLoader(file_path='policy.docx')
docs = loader.load()
```

### PPTX

```python
loader = DoclingLoader(file_path='training.pptx')
docs = loader.load()
```

### TXT

```python
loader = DoclingLoader(file_path='knowledge.txt')
docs = loader.load()
```

### Multiple Files

```python
from pathlib import Path
from langchain_docling.loader import DoclingLoader

all_docs = []

for f in Path('documents').glob('*'):
    all_docs.extend(
        DoclingLoader(file_path=str(f)).load()
    )
```

---

## 8. Export Types

### MARKDOWN

```python
from langchain_docling.loader import ExportType

loader = DoclingLoader(
    file_path='report.pdf',
    export_type=ExportType.MARKDOWN
)
```

Returns entire document.

### DOC_CHUNKS

```python
loader = DoclingLoader(
    file_path='report.pdf',
    export_type=ExportType.DOC_CHUNKS
)
```

Recommended for RAG.

---

## 9. HybridChunker

```python
from docling.chunking import HybridChunker

chunker = HybridChunker(
    max_tokens=256
)

chunks = list(chunker.chunk(doc))
```

Advantages:
- Structure aware
- Heading preservation
- Better retrieval
- Better embeddings

---

## 10. Metadata Example

```python
for d in docs:
    print(d.metadata)
```

Example:

```python
{
 'source':'report.pdf',
 'page_no':5,
 'section':'Financial Results'
}
```

---

## 11. Embeddings

```python
from langchain_huggingface import HuggingFaceEmbeddings

embeddings = HuggingFaceEmbeddings(
 model_name='sentence-transformers/all-MiniLM-L6-v2'
)
```

---

## 12. Qdrant Integration

### Start Qdrant

```bash
docker run -p 6333:6333 qdrant/qdrant
```

### Create Vector Store

```python
from langchain_qdrant import QdrantVectorStore
from qdrant_client import QdrantClient

client = QdrantClient(
    url='http://localhost:6333'
)

vectorstore = QdrantVectorStore.from_documents(
    docs,
    embeddings,
    url='http://localhost:6333',
    collection_name='docling_docs'
)
```

---

## 13. Create Retriever

```python
retriever = vectorstore.as_retriever(
    search_kwargs={'k':5}
)
```

---

## 14. Query Documents

```python
results = retriever.invoke(
  'What is quarterly revenue?'
)

for r in results:
    print(r.page_content)
```

---

## 15. Complete RAG Pipeline

```python
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(model='gpt-4o')

context = retriever.invoke(
    'What is quarterly revenue?'
)

response = llm.invoke(str(context))
```

---

## 16. Contextual Compression Retriever

```python
from langchain.retrievers import ContextualCompressionRetriever
```

Flow:

Retriever -> Compression -> Important Passages -> LLM

Benefits:
- Reduced tokens
- Better relevance
- Lower cost

---

## 17. Parent Document Retriever

Useful when chunks are small.

Flow:

Child Chunk Match -> Parent Section Returned

Benefits:
- Better context
- Better answers

---

## 18. Production Architecture

Documents
  |
Docling
  |
HybridChunker
  |
Embeddings
  |
Qdrant
  |
Retriever
  |
Reranker
  |
Contextual Compression
  |
GPT-4o / Claude / Llama

---

## Best Practices

1. Use ExportType.DOC_CHUNKS.
2. Use HybridChunker.
3. Preserve metadata.
4. Use Qdrant for production.
5. Add reranking.
6. Add ContextualCompressionRetriever.
7. Monitor chunk sizes (200-500 tokens).
8. Store source references.
