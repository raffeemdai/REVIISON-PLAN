# Advanced RAG Notes (Chapter 1 & 2)
Source: CampusX Advanced RAG Complete Course

> IMPORTANT
>
> 🟢 Core Definitions
> 🔵 Exam / Interview Focus
> 🟡 Frequently Asked Concept
> 🔴 High Priority RAG Concept

# Chapter 1: Introduction to RAG

## Why RAG is Important

Large Language Models suffer from four major limitations:

1. Knowledge Cutoff
   - Model knows only information available during training.
   - Cannot automatically know recent events.

2. Hallucinations
   - Can produce confident but incorrect answers.

3. No Source Attribution
   - Cannot naturally cite where information came from.

4. No Access to Private Data
   - Cannot answer questions about company documents unless data is provided.

🔴 RAG solves these issues by supplying external knowledge during inference.

---

## Traditional Solutions Before RAG

### Option 1: Pretraining
- Retrain model on new data.
- Extremely expensive.
- Requires huge compute resources.

### Option 2: Fine Tuning
- Adjust model behavior.
- Useful for style, formatting, domain adaptation.
- Not ideal for continuously updating knowledge.

🔵 Interview Question:
Why is fine-tuning not considered the best solution for knowledge updates?

Answer:
Because fine-tuning changes model behavior more than knowledge and is costly compared with RAG.

---

## In-Context Learning

LLMs can use additional information placed inside the prompt.

Example:

Question + Relevant Context → Better Answer

Problem:
Putting the entire knowledge base inside a prompt is impossible.

Reasons:
- Context window limitation
- Lost in the middle problem

---

## Lost in the Middle Problem

When too much context is supplied:

- Beginning information remembered well
- End information remembered well
- Middle information often ignored

🟡 Common interview topic.

---

## Birth of RAG

RAG combines:

1. In-context learning
2. Smart filtering of knowledge
3. External knowledge storage

Definition:

Retrieval Augmented Generation is a framework that retrieves relevant information from an external knowledge source and augments the prompt before sending it to an LLM.

---

## How RAG Works

### Step 1: Retrieval
Retrieve relevant information.

### Step 2: Augmentation
Add retrieved information to prompt.

### Step 3: Generation
LLM produces final answer.

Flow:

User Query
→ Retrieval
→ Relevant Context
→ Prompt Augmentation
→ LLM
→ Final Answer

🔴 RAG itself does NOT generate answers.
The LLM performs generation.

---

## Retrieval

Purpose:
Fetch semantically relevant documents.

Operations:

1. Understand user query.
2. Semantic search.
3. Rank documents.
4. Return top-k documents.

### Similarity Ranking

Every retrieved document gets a similarity score.

Example:

Retrieved = 15 documents
Need = 4 documents

Sort by similarity score.
Keep top 4.

---

## Knowledge Base

A central repository containing all information.

Examples:

- PDFs
- CSV files
- Websites
- APIs
- Databases
- Company documents

---

## Important Concepts

### Query
User question.

### Context
Retrieved supporting information.

### Retriever
Component that searches knowledge base.

### Generator
LLM creating final response.

---

## Chapter 1 Quick Revision

✅ LLM Problems
- Knowledge cutoff
- Hallucination
- No citations
- No private data access

✅ RAG Pipeline
- Retrieval
- Augmentation
- Generation

✅ Key Benefits
- Current information
- Reduced hallucinations
- Source-backed answers
- Private knowledge support

---

## Chapter 1 Interview Questions

1. What is RAG?
2. Why was RAG introduced?
3. Difference between Fine-Tuning and RAG?
4. What is In-Context Learning?
5. Explain Lost-in-the-Middle problem.
6. Explain Retrieval, Augmentation and Generation.
7. Who performs generation in a RAG system?
8. What is a Retriever?
9. How does similarity ranking work?
10. What are the advantages of RAG?

---

## Practice Code (Chapter 1)

```python
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(model="gpt-4o-mini")

response = llm.invoke(
    "What are the three stages of RAG?"
)

print(response.content)
```

---

# Chapter 2: Document Loaders

## RAG Refresher

RAG needs relevant context.

The retriever can only retrieve data already stored in the knowledge base.

Question:
How does data enter the knowledge base?

Answer:
Through Document Loaders.

---

## Why Document Loaders are Needed

Knowledge can exist in many formats:

- TXT
- PDF
- CSV
- HTML
- JSON
- Database
- APIs
- Websites

Each format stores data differently.

Document Loaders convert all of them into a common format.

---

## Knowledge Source vs Knowledge Base

### Knowledge Source
Original location of information.

Examples:
- PDF
- Website
- Database

### Knowledge Base
Processed storage used by RAG.

Examples:
- Vector Database
- Chroma
- Pinecone
- Weaviate

🔴 Frequently asked interview question.

---

## What is a Document Loader?

Definition:

A document loader extracts information from a source and converts it into LangChain Document objects.

Standard Document Object:

```python
Document(
    page_content="text",
    metadata={}
)
```

Document has:

1. page_content
2. metadata

---

## Document Loading Workflow

Knowledge Source
→ Loader
→ Document Objects
→ Chunking
→ Embeddings
→ Vector Database

---

## Need for Chunking

Large documents create multiple problems.

### Problem 1
Context window limits.

### Problem 2
Lost in the middle.

### Problem 3
Poor retrieval performance.

---

## Why Chunking Improves Retrieval

Without Chunking:

5 documents
→ 5 embeddings
→ Retriever often returns everything.

With Chunking:

5 documents
→ 50 chunks
→ 50 embeddings
→ Better filtering.

🔴 Key Insight:
Chunking makes retrieval selective.

---

## Text Loader

Used for:

```text
.txt
```

Example:

```python
from langchain_community.document_loaders import TextLoader

loader = TextLoader("sample.txt")

docs = loader.load()

print(docs[0].page_content)
```

---

## CSV Loader

Used for:

```text
.csv
```

Example:

```python
from langchain_community.document_loaders import CSVLoader

loader = CSVLoader(file_path="employees.csv")
docs = loader.load()

print(docs[0])
```

---

## PDF Loader

Popular choices:

- PyPDFLoader
- PDFPlumberLoader
- UnstructuredPDFLoader

Example:

```python
from langchain_community.document_loaders import PyPDFLoader

loader = PyPDFLoader("rag.pdf")
docs = loader.load()

print(len(docs))
```

---

## JSON Loader

Used for reading structured data.

```python
from langchain_community.document_loaders import JSONLoader
```

Common in APIs and application logs.

---

## Web Loader

Used to scrape webpages.

```python
from langchain_community.document_loaders import WebBaseLoader

loader = WebBaseLoader(
    "https://example.com"
)

docs = loader.load()
```

---

## Directory Loader

Load multiple files at once.

```python
from langchain_community.document_loaders import DirectoryLoader
```

Useful for enterprise projects.

---

## Important Metadata Examples

```python
{
  "source": "report.pdf",
  "page": 5,
  "author": "CampusX"
}
```

Metadata enables:
- Citation
- Traceability
- Filtering
- Source attribution

---

## Chapter 2 Quick Revision

✅ Document Loader
Converts data source into Document objects.

✅ Document Object
- page_content
- metadata

✅ Workflow
Source → Loader → Chunking → Embedding → Vector DB

✅ Chunking Benefit
Better retrieval quality.

✅ Common Loaders
- TextLoader
- CSVLoader
- PyPDFLoader
- JSONLoader
- WebBaseLoader
- DirectoryLoader

---

## Chapter 2 Interview Questions

1. What is a Document Loader?
2. Why do we need Document Loaders?
3. Knowledge Source vs Knowledge Base?
4. Explain LangChain Document object.
5. Why is chunking necessary?
6. How does chunking improve retrieval?
7. Explain metadata.
8. What does PyPDFLoader do?
9. Difference between CSVLoader and TextLoader?
10. Where are Document Loaders used in the RAG pipeline?

---

## End-to-End Practice Project

```python
from langchain_community.document_loaders import PyPDFLoader
from langchain.text_splitter import RecursiveCharacterTextSplitter

loader = PyPDFLoader("sample.pdf")
docs = loader.load()

splitter = RecursiveCharacterTextSplitter(
    chunk_size=500,
    chunk_overlap=50
)

chunks = splitter.split_documents(docs)

print("Pages:", len(docs))
print("Chunks:", len(chunks))
```

# Study Tip

Focus heavily on:
- Why RAG is needed
- Retrieval process
- Knowledge Base vs Knowledge Source
- Document Object structure
- Chunking strategies
- Common LangChain Loaders

These topics appear most frequently in RAG interviews.
