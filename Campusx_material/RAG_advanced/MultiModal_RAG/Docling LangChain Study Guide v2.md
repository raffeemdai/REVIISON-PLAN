# Docling with LangChain - Complete Study Guide

## Supported File Types

Docling can process:
- PDF
- DOCX
- PPTX
- HTML
- Images (PNG/JPG)
- Markdown
- Text files

---

## PDF Example

```python
from langchain_docling.loader import DoclingLoader

loader = DoclingLoader(
    file_path="annual_report.pdf"
)

docs = loader.load()
print(docs[0].page_content[:500])
```

Use for:
- Annual reports
- Research papers
- Financial statements
- Multi-column PDFs

---

## TXT File Example

```python
loader = DoclingLoader(
    file_path="application_logs.txt"
)

docs = loader.load()
```

Sample Output:

```text
User login successful
Request processed
Payment completed
```

Best for:
- Log analysis
- Knowledge bases
- Notes

---

## DOCX Example

```python
loader = DoclingLoader(
    file_path="company_policy.docx"
)

docs = loader.load()
```

Docling preserves:
- Headings
- Sections
- Bullets
- Tables

---

## PPTX Example

```python
loader = DoclingLoader(
    file_path="rag_architecture.pptx"
)

docs = loader.load()
```

Useful for:
- Training material
- Internal documentation
- Presentations

---

## Multiple Files Example

```python
from pathlib import Path
from langchain_docling.loader import DoclingLoader

all_docs=[]

for file in Path("documents").glob("*"):
    loader = DoclingLoader(file_path=str(file))
    all_docs.extend(loader.load())

print(len(all_docs))
```

---

## Export Modes

### MARKDOWN

```python
from langchain_docling.loader import ExportType

loader = DoclingLoader(
    file_path="report.pdf",
    export_type=ExportType.MARKDOWN
)
```

Returns entire document as markdown.

### DOC_CHUNKS

```python
loader = DoclingLoader(
    file_path="report.pdf",
    export_type=ExportType.DOC_CHUNKS
)
```

Returns chunked documents for RAG.

---

## Metadata Example

```python
for doc in docs:
    print(doc.metadata)
```

Typical metadata:

```python
{
  "source":"report.pdf",
  "page_no":4,
  "section":"Revenue"
}
```

---

## Chroma Integration

```python
from langchain_huggingface import HuggingFaceEmbeddings
from langchain_community.vectorstores import Chroma

embeddings = HuggingFaceEmbeddings(
 model_name="sentence-transformers/all-MiniLM-L6-v2"
)

vectorstore = Chroma.from_documents(
 docs,
 embeddings
)
```

---

## Qdrant Integration

```python
from langchain_qdrant import QdrantVectorStore
```

Recommended for production workloads.

---

## RAG Pipeline

```text
PDF/DOCX/TXT/PPTX
        |
        v
     Docling
        |
        v
  DoclingLoader
        |
        v
  HybridChunker
        |
        v
    Embeddings
        |
        v
 Vector Database
        |
        v
    Retriever
        |
        v
        LLM
```

---

## Contextual Compression Retriever

```python
from langchain.retrievers import ContextualCompressionRetriever
```

Flow:

Retriever -> Compression -> Important Sentences -> LLM

---

## Parent Document Retriever

Useful when chunk size is small but large context is required.

```text
Child Chunk Match
       |
       v
Return Parent Section
```

---

## Best Practices

1. Use ExportType.DOC_CHUNKS for RAG.
2. Use HybridChunker.
3. Store metadata.
4. Use Qdrant/Chroma.
5. Add rerankers.
6. Combine with Contextual Compression Retriever.
