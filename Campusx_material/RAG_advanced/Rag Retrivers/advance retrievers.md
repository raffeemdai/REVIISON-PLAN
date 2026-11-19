Contextual Compression Retriever
Parent Document Retriever
Self-Query Retrieve
Multi-Query Retrieve


Contextual Compression Retriever :
https://academy.forgestackk.com/advanced-rag/07-advanced-retrievers/#industry-usage
https://searchministry.au/guides/what-is-contextual-compression

https://enterpriseai.handbook.mihirkjha.com/05-advanced-retrieval-augmented-generation/02-enterprise-retrieval-engineering/01-contextual-compression-retriever/#6-contextual-compression-components

# Contextual Compression Retriever
<img width="1376" height="768" alt="image" src="https://github.com/user-attachments/assets/aa5310e7-8f73-42dd-924c-0298abb6537e" />

<img width="1376" height="768" alt="image" src="https://github.com/user-attachments/assets/f285d170-942a-4d85-b708-85dbca080609" />
<img width="1376" height="768" alt="image" src="https://github.com/user-attachments/assets/33e74b69-6e6c-4fa1-b1cf-bb6805e525b9" />
<img width="1376" height="768" alt="image" src="https://github.com/user-attachments/assets/a4e920e9-8ab1-48bf-b957-cdc97ca369f4" />


<img width="1149" height="545" alt="image" src="https://github.com/user-attachments/assets/2cdd343f-855b-491f-8852-8b6817785036" />



A Contextual Compression Retriever improves RAG retrieval by reducing irrelevant information from documents returned by a retriever.

A traditional retriever may successfully identify relevant documents, but those documents can still contain a large amount of information that is unrelated to the user's question.

For example:

```text
User Query

"What authentication mechanism is required
for production APIs?"
```

The retriever may return:

```text
Production API Security Guide
├── Authentication
├── Authorization
├── Network Security
├── Rate Limiting
├── Logging
├── Monitoring
├── Deployment
└── Incident Response
```

The document is relevant, but the LLM may only need:

```text
Authentication
Authorization
```

Contextual compression introduces an additional processing stage:

```text
Query
  ↓
Retriever
  ↓
Candidate Documents
  ↓
Contextual Compression
  ↓
Relevant Content
  ↓
LLM
```

The goal is not simply to retrieve fewer documents.

The goal is to provide the LLM with more relevant context with less unnecessary information.

### Technical overview

A Contextual Compression Retriever is a post-retrieval optimization layer designed to improve Retrieval-Augmented Generation (RAG) systems by eliminating irrelevant text from documents returned by a base retriever before context assembly.

Standard vector retrievers search by document-level or chunk-level similarity. While candidate chunks often contain correct answers, they frequently introduce substantial amounts of off-topic background text. Contextual compression wraps around a base retriever and applies an automated post-retrieval compression stage. This stage evaluates retrieved chunks against the specific user query, extracting only query-relevant sentences verbatim while discarding off-topic noise.

```text
User Query ──► Base Retriever ──► Candidate Documents ──► Contextual Compression ──► Relevant Content ──► LLM
```

The objective of contextual compression is not simply to retrieve fewer documents, but to provide the LLM with higher signal density and significantly reduced context noise.

---

## 1. The Context Problem in RAG

A basic RAG pipeline typically looks like:

```text
User Query
    ↓
Retriever
    ↓
Retrieved Documents
    ↓
Prompt
    ↓
LLM
    ↓
Response
```

Suppose the retriever returns:

```text
Document A → 1,800 tokens
Document B → 1,500 tokens
Document C → 1,700 tokens
Document D → 1,200 tokens
```

Total:

```text
6,200 tokens
```

But only:

```text
900 tokens
```

may actually be useful for answering the question.

The LLM still receives:

```text
6,200 tokens
```

unless the application introduces another mechanism to reduce the context.

This can lead to:

- Higher token usage
- Higher cost
- Higher latency
- More irrelevant context
- More context competition
- Lower answer quality

### Operational Consequences of Uncompressed Context

When documents are retrieved at full chunk size, substantial token overhead is introduced. Without a post-retrieval refinement mechanism, the entire 6,200 tokens are injected directly into the prompt.

* **Higher Token Usage:** Inflates context window consumption unnecessarily.
* **Elevated API Costs:** Pay-per-token API billing scales directly with uncompressed prompt length.
* **Increased Latency:** LLMs require more processing time to compute attention over large input contexts.
* **Context Pollution:** Off-topic paragraphs dilute query focus.
* **Attention Degradation:** LLMs suffer from "lost-in-the-middle" phenomena when core evidence is buried inside long text blocks.
* **Elevated Hallucination Risk:** Excess noise increases the probability that the LLM will mix in off-topic information or generate unsupported claims.

---

## 2. What Is Contextual Compression?

Contextual compression takes retrieved documents and keeps only the information relevant to the current query.

The basic architecture is:

```text
User Query
     ↓
Base Retriever
     ↓
Candidate Documents
     ↓
Contextual Compressor
     ↓
Compressed Context
     ↓
Prompt Assembly
     ↓
LLM
     ↓
Response
```

The important distinction is:

```text
Retriever
    ↓
Find potentially relevant information

Compressor
    ↓
Remove information that is not useful for this query
```

### Functional Component Division

Contextual compression introduces an evidence-refinement stage positioned between candidate search and prompt assembly. It decouples retrieval into two distinct responsibilities:

1. **Base Retriever:** Responsible for broad candidate discovery ("Find potentially relevant information").
2. **Base Compressor:** Responsible for fine-grained evidence refinement ("Remove information that is not useful for this specific query").

### Distinguishing Embeddings from Contextual Compression

* **Vector Embedding Models:** Compress *semantic representation* by transforming raw text into dense numerical vectors for vector-distance matching.
* **Contextual Compression:** Trims *text content* post-search by removing unneeded sentences and filtering out non-matching document chunks.

---

## 3. Retrieval vs Compression

Retrieval asks:

> "Which documents might contain the answer?"

Compression asks:

> "Which parts of those documents are useful for answering this specific query?"

Therefore:

```text
Retrieval
    ↓
Candidate Evidence

Compression
    ↓
Focused Evidence
```

This creates a two-stage retrieval architecture:

```text
Broad Retrieval
      ↓
Context Refinement
```

Retrieval and compression operate at different stages of the information discovery pipeline, addressing complementary objectives:

| Pipeline Stage | Primary Question Asked | System Objective |
| :--- | :--- | :--- |
| **Retrieval Stage** | "Which documents might contain the answer?" | **Candidate Evidence** (High Recall / Broad Search) |
| **Compression Stage** | "Which parts of those documents are useful for answering this specific query?" | **Focused Evidence** (High Precision / Noise Reduction) |

By implementing a two-stage retrieval architecture:

```text
Broad Candidate Discovery (e.g., Top-K = 10) ──► Context Refinement (Compress & Filter) ──► Focused Prompt Evidence
```

Engineers can configure base retrievers with high top-k parameters to ensure maximum recall without risking prompt window saturation or attention degradation.

---

## 4. Why Do We Need Contextual Compression?

Consider an enterprise policy document:

```text
Remote Work Policy

1. Introduction
2. Eligibility
3. Working Arrangements
4. Approval Process
5. Exceptions
6. Security Requirements
7. Manager Responsibilities
8. Compliance
9. Reporting
```

User query:

```text
"Who approves exceptions to remote work?"
```

A retriever may return the entire policy.

But the answer may primarily exist in:

```text
Section 4 — Approval Process
Section 5 — Exceptions
Section 7 — Manager Responsibilities
```

Contextual compression can reduce:

```text
Entire Policy
```

to:

```text
Relevant Sections
```

This allows the generation layer to work with a smaller and more focused context.

### Structural Motivations for Contextual Compression

Standard vector similarity search operates at fixed chunk boundaries, creating several structural challenges in real-world enterprise RAG deployments:

#### A. Coarse Chunk Boundaries & Mixed Topics

Text splitters divide documents based on character counts or token lengths rather than semantic topic shifts. Consequently, a 400-word chunk retrieved for a query such as "What is the capital of France?" may contain only one relevant sentence about Paris surrounded by 399 words of general historical background. Contextual compression isolates that single relevant sentence.

#### B. Enterprise Policy & Manual Structure

In the Remote Work Policy example above, for the query "Who approves exceptions to remote work?", a standard retriever returns the entire policy document due to high overall document similarity. Contextual compression condenses the text down strictly to:

* Section 4: Approval Process
* Section 5: Exceptions
* Section 7: Manager Responsibilities

#### C. Verbatim Extractive Precision vs. Summarization Risks

Unlike text summarization, which rewrites context and introduces risks of paraphrase distortion or hallucination, contextual compression performs **strict verbatim extraction ("as is")**. Original wording, numbers, dates, and technical specifications are preserved without modification, guaranteeing complete factual traceability to source material.

#### D. Preservation of Qualifications and Conditions

In policy, legal, and compliance contexts, compression algorithms maintain crucial conditional clauses (such as "subject to manager approval" or "excluding production environments"), ensuring policy intent remains intact during context trimming.

#### E. AI Citation Probability

Search architectures powering AI search overviews (such as Perplexity and Google AI Overviews) utilize contextual compression pipelines. Content that front-loads key claims in clear, direct sentences is significantly more likely to survive compression filters and be selected as cited evidence.

---

## 5. Types of Compressors in LangChain

LangChain provides three primary compressor types to accommodate varying trade-offs between processing speed, execution cost, and extraction precision:

| Compressor Type | Operational Mechanism | Key Strengths | Practical Limitations |
| :--- | :--- | :--- | :--- |
| **`LLMChainExtractor`** | An LLM evaluates each chunk alongside the query, extracting relevant text *verbatim* or outputting `NO_OUTPUT`. | Trims text *within* chunks and discards non-matching chunks entirely. | Increases latency (~200ms–800ms) and token cost due to 1 LLM call per chunk. |
| **`EmbeddingsFilter`** | Re-embeds query and chunks, retaining documents whose similarity meets a `similarity_threshold`. | Fast and cost-effective with zero extra LLM calls; attaches similarity scores to metadata. | Operates only at whole-chunk granularity (cannot trim text within a chunk). |
| **`DocumentCompressorPipeline`** | Sequences multiple transformers and filters sequentially in a defined pipeline. | Combines the computational efficiency of embedding filters with the precision of LLM extraction. | Pipeline step order must be configured carefully. |

### Architectural Best Practice: Cheap-to-Expensive Pipeline Sequencing

When configuring a `DocumentCompressorPipeline`, always sequence transformers from lowest computational cost to highest:

1. **Step 1 (`EmbeddingsFilter`):** Rapidly filter out obvious non-matching document chunks without incurring LLM call costs.
2. **Step 2 (`LLMChainExtractor`):** Perform fine-grained verbatim sentence extraction strictly on the surviving candidate documents.

This pattern minimizes downstream token consumption and drastically cuts total API execution costs.

---

## 6. Production Implementation in Python (LangChain Classic)

The following Python implementation demonstrates `LLMChainExtractor`, `EmbeddingsFilter`, and `DocumentCompressorPipeline` using standard `langchain_classic` components:

```python
import os
from dotenv import load_dotenv
from langchain_core.documents import Document
from langchain_core.vectorstores import InMemoryVectorStore
from langchain_openai import OpenAIEmbeddings, ChatOpenAI
from langchain_classic.retrievers import ContextualCompressionRetriever
from langchain_classic.retrievers.document_compressors import (
    LLMChainExtractor,
    EmbeddingsFilter,
    DocumentCompressorPipeline,
)

# 1. Environment & Model Initialization
load_dotenv()

# Initialize embeddings and deterministic LLM (temperature=0 for strict extraction)
embeddings = OpenAIEmbeddings(model="text-embedding-3-small")
llm = ChatOpenAI(model="gpt-4o-mini", temperature=0)

# 2. Multi-Topic Benchmark Documents
docs = [
    Document(
        page_content=(
            "Artificial intelligence has made remarkable strides in natural language processing... "
            "However, the rapid advancement of AI has raised significant ethical concerns."
        ),
        metadata={"topic": "artificial_intelligence"},
    ),
    Document(
        page_content=(
            "CRISPR gene editing technology has revolutionized medical genomics, enabling precise "
            "modifications to DNA sequences that were previously impossible. Researchers are using "
            "genomic data to develop personalized medicine approaches, tailoring treatments based on "
            "an individual's genetic profile. Recent breakthroughs in mRNA technology, accelerated by "
            "COVID-19 vaccine development, are now being applied to cancer immunotherapy. "
            "Hospital information systems are increasingly integrating genomic data."
        ),
        metadata={"topic": "medicine"},
    ),
    Document(
        page_content=(
            "Quantum computing has reached a critical milestone... Error correction remains the biggest challenge."
        ),
        metadata={"topic": "quantum_computing"},
    ),
]

# 3. Base Retriever Setup
vector_store = InMemoryVectorStore.from_documents(docs, embeddings)
base_retriever = vector_store.as_retriever(search_kwargs={"k": 3})

query = "How is CRISPR acting as a big enabler in creating personalized medicine?"


# =====================================================================
# METHOD 1: LLM-Based Compression (LLMChainExtractor)
# =====================================================================
llm_compressor = LLMChainExtractor.from_llm(llm)

cc_retriever_llm = ContextualCompressionRetriever(
    base_compressor=llm_compressor,
    base_retriever=base_retriever,
)

# Execution: Drops non-medical docs and trims the medicine doc strictly to CRISPR sentences
compressed_docs = cc_retriever_llm.invoke(query)

for i, doc in enumerate(compressed_docs):
    print(f"--- LLM Compressed Result {i+1} [{doc.metadata.get('topic')}] ---")
    print(doc.page_content)
    print()


# =====================================================================
# METHOD 2: Embedding-Based Filtering (EmbeddingsFilter)
# =====================================================================
embeddings_filter = EmbeddingsFilter(
    embeddings=embeddings,
    similarity_threshold=0.732,  # Retains documents meeting similarity threshold
)

cc_retriever_filter = ContextualCompressionRetriever(
    base_compressor=embeddings_filter,
    base_retriever=base_retriever,
)

# Execution: Drops non-matching docs cheaply, but returns the medicine doc whole
filtered_docs = cc_retriever_filter.invoke(query)

for i, doc in enumerate(filtered_docs):
    print(f"--- Embeddings Filtered Result {i+1} [{doc.metadata.get('topic')}] ---")
    print(doc.page_content)
    print()


# =====================================================================
# METHOD 3: Hybrid Pipeline (DocumentCompressorPipeline)
# =====================================================================
# Rule: Sequence cheap EmbeddingsFilter first -> expensive LLMChainExtractor second
pipeline_compressor = DocumentCompressorPipeline(
    transformers=[embeddings_filter, llm_compressor]
)

cc_retriever_pipeline = ContextualCompressionRetriever(
    base_compressor=pipeline_compressor,
    base_retriever=base_retriever,
)

# Execution: Drops non-matching docs via embeddings first, then uses LLM to trim surviving docs
pipeline_docs = cc_retriever_pipeline.invoke(query)

for i, doc in enumerate(pipeline_docs):
    print(f"--- Pipeline Result {i+1} [{doc.metadata.get('topic')}] ---")
    print(doc.page_content)
    print()
```

---

## 7. Key Points to Remember

1. A normal retriever finds documents that may contain the answer.
2. A retrieved document can contain both useful and unrelated information.
3. Contextual Compression examines retrieved content in relation to the user's specific query.
4. It keeps or extracts only the information that is useful for that query.
5. The LLM receives a more focused context window.
6. This reduces unnecessary tokens, API costs, and processing latency.
7. Compression does not replace retrieval; it works directly on the results returned by a base retriever.

> **Memory Trick:**  
> **Retriever** = *Find the right document.*  
> **Compressor** = *Keep the useful parts of that document.*

---

## 8. Interview-Ready Technical Guide

### Q1: What is a Contextual Compression Retriever?

**Answer:** A Contextual Compression Retriever is a post-retrieval optimization technique in RAG pipelines that combines a base retriever (for candidate document search) with a context compressor (for noise filtering). It evaluates fetched chunks relative to the user's query, extracting or filtering relevant passages verbatim so that downstream LLMs receive high-precision, signal-dense prompts.

### Q2: What is the difference between retrieval and compression?

**Answer:** Retrieval answers *"Which documents might contain the answer?"* by prioritizing high recall during candidate search. Compression answers *"Which specific parts of those documents are useful for this query?"* by prioritizing high precision and noise removal post-search.

### Q3: Does Contextual Compression mean only reducing the number of documents?

**Answer:** No. While it can drop non-matching documents entirely, its primary goal is to eliminate irrelevant content within documents. Depending on the compressor used (e.g., `LLMChainExtractor`), it can perform fine-grained sentence extraction to shorten chunks while preserving original wording.

