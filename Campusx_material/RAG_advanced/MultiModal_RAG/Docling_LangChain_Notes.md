# Docling with LangChain
## Complete Notes for RAG Applications

---

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

# Installation

```bash
pip install docling
pip install langchain-docling
pip install langchain chromadb sentence-transformers
```

---

# Core Docling Example

```python
from docling.document_converter import DocumentConverter

converter = DocumentConverter()
result = converter.convert("sample.pdf")
doc = result.document

markdown = doc.export_to_markdown()
print(markdown)
```

---

# LangChain Integration

```python
from langchain_docling.loader import DoclingLoader

loader = DoclingLoader(file_path="sample.pdf")
docs = loader.load()
```

---

# Export Types

## ExportType.MARKDOWN

```python
from langchain_docling.loader import ExportType
```

Use for small documents and summarization.

## ExportType.DOC_CHUNKS

Use for RAG and semantic search.

---

# Hybrid Chunker

```python
from docling.chunking import HybridChunker

chunker = HybridChunker(max_tokens=256)
chunks = list(chunker.chunk(doc))
```

Benefits:
- Structure-aware chunking
- Better retrieval quality
- Preserves context

---

# Complete RAG Flow

```text
PDF
 ↓
Docling
 ↓
HybridChunker
 ↓
LangChain Documents
 ↓
Embeddings
 ↓
Chroma / Qdrant / Milvus
 ↓
Retriever
 ↓
LLM
```

---

# Vector Store Example

```python
from langchain_huggingface import HuggingFaceEmbeddings
from langchain_community.vectorstores import Chroma

embeddings = HuggingFaceEmbeddings(
    model_name="sentence-transformers/all-MiniLM-L6-v2"
)

db = Chroma.from_documents(docs, embeddings)
retriever = db.as_retriever()
```

---

# Advanced RAG Integration

- ContextualCompressionRetriever
- ParentDocumentRetriever
- Hybrid Search (BM25 + Vector)
- Rerankers (BGE, Cohere)
- LangGraph Agentic RAG

---

# Best Practices

1. Use HybridChunker.
2. Preserve metadata.
3. Use DOC_CHUNKS for RAG.
4. Combine with reranking.
5. Use Qdrant/Chroma/Milvus in production.

---

# References

- https://docling-project.github.io/docling/
- https://github.com/docling-project/docling-langchain
- https://docs.langchain.com/oss/python/integrations/document_loaders/docling
- https://docling-project.github.io/docling/_generated/examples/rag_langchain/
