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
