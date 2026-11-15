<img width="981" height="702" alt="image" src="https://github.com/user-attachments/assets/029e06bc-ee9a-40bf-932e-b0e78773f10a" />


#### belkow is good
https://github.com/KowshiqKatta/Generative-AI/tree/main/RAG     
https://mayankchugh-learning.github.io/pages/ai-learning/retriever-guide.html



# CampusX Advanced RAG — Retrievers & Advanced Retrievers
## A story-based, ordered study guide
**Theory • simple explanations • Python patterns • decision guide • interview questions**

> **Purpose:** These notes reorganize CampusX Advanced RAG Chapters 6 and 7 into a learning sequence. The goal is to understand *why* each retriever exists, what problem it solves, how it works, and when to choose it—not to memorize disconnected class names.

> **Course alignment:** CampusX Advanced RAG, Chapter 6 (Retrievers, printed pages 124–146) and Chapter 7 (Advanced Retrievers, printed pages 147–177). Examples are learning patterns based on the course concepts. LangChain APIs and package locations change; check the installed version before running them.

---

# 0. First understand the big picture

Imagine a company has a digital library containing thousands of documents: HR policies, technical manuals, support tickets, product documents, research papers and PDFs.

A user asks:

> “What is the process to get a refund for an annual subscription?”

The RAG system must find the right passages before the LLM can answer. The component responsible for finding those passages is the **retriever**.

## The complete RAG flow

```text
OFFLINE / DOCUMENT INGESTION
PDFs, Word, web, CSV
       |
       v
Document loaders
       |
       v
Text splitters -> chunks
       |
       v
Embedding model -> vectors
       |
       v
Vector store (vectors + original text + metadata)


ONLINE / EVERY USER QUESTION
User question
       |
       v
Retriever
       |
       v
Relevant Document objects
       |
       v
Build prompt with question + retrieved context
       |
       v
LLM
       |
       v
Answer
```

**Important separation**

- Loaders, splitters, embeddings and vector stores prepare and store the knowledge.
- Retrievers search that knowledge when a question arrives.
- The LLM generates the answer using the retrieved context.

A retriever does not itself write the final answer. It finds evidence.

## What exactly goes into and comes out of a retriever?

```text
Input:  "How can I cancel my subscription?"
Output: [
  Document(page_content="A customer can cancel ...",
           metadata={"source": "billing.pdf", "page": 8}),
  Document(page_content="Cancellation takes effect ...",
           metadata={"source": "terms.pdf", "page": 12})
]
```

A LangChain `Document` normally contains:

- `page_content`: the text of the chunk.
- `metadata`: source, page, document ID, date, category, parent ID, etc.

**Interview point:** the retriever takes a text query and returns documents, not raw embedding vectors.

---

# PART A — CHAPTER 6: BASIC RETRIEVERS

# 1. What is a retriever?

Think of a librarian. You ask a question; the librarian searches the shelves and gives you the most useful pages. The librarian does not write your report—that is the LLM's job.

A retriever performs that search programmatically.

## 1.1 Why does retrieval quality matter?

The LLM can only use the evidence that reaches its prompt. If the correct paragraph is missing, even a powerful LLM may give an incomplete answer or guess.

A good retriever should:

1. Understand the intent of the query well enough to search.
2. Find useful passages in the knowledge base.
3. Avoid returning too much irrelevant material.
4. Preserve metadata so the application can cite the source.

Relevance is **query-dependent**. A chunk about employee leave may be relevant to “How many vacation days do I get?” but irrelevant to “How do I reset my password?”

## 1.2 How a dense/vector retriever works

Suppose the document says:

> “Customers may terminate their yearly plan before the renewal date.”

The user asks:

> “How can I cancel my annual subscription?”

The words are different, but the meaning is similar. Dense retrieval uses embeddings to match meaning.

```text
Document chunks
   |
   v
Document embedding model
   |
   v
Stored document vectors in vector database

User question
   |
   v
SAME compatible embedding model
   |
   v
Query vector
   |
   v
Vector similarity search
   |
   v
Ranked document chunks
```

Step by step:

1. The query text is sent to the embedding model.
2. The model creates a query vector.
3. The vector store compares it with stored document vectors.
4. A similarity or distance score is calculated.
5. Results are ranked.
6. The top `k` documents are returned as `Document` objects.

**Critical rule:** the query embedding model must be compatible with the model/configuration used to embed the documents. Changing the embedding model usually means re-embedding the collection.

## 1.3 Similarity score versus distance

Do not assume every score follows the same direction.

| Measure | Usually better result |
|---|---|
| Cosine similarity | Higher is better |
| Dot-product similarity | Often higher is better, depending on setup |
| Euclidean/L2 distance | Lower is better |

Always check the vector store's score convention. A score of `0.2` could be excellent in a distance-based system and poor in a similarity-based system.

---

# 2. Vanilla Similarity Retriever — “Give me the top K closest chunks”

This is the simplest baseline. It returns the `k` chunks that are most similar to the query.

### Example

Question: “How do I reset my account password?”

The vector store ranks chunks:

| Rank | Chunk | Similarity (illustrative) |
|---:|---|---:|
| 1 | Password reset instructions | 0.91 |
| 2 | Account login troubleshooting | 0.86 |
| 3 | Two-factor authentication | 0.79 |
| 4 | Billing and refunds | 0.31 |

If `k=3`, the first three are returned.

## 2.1 LangChain code

```python
# Assume vectorstore has already been created and populated.
retriever = vectorstore.as_retriever(
    search_type="similarity",
    search_kwargs={"k": 3}
)

docs = retriever.invoke("How do I reset my account password?")

for doc in docs:
    print(doc.page_content)
    print(doc.metadata)
```

### Understand the two parameters

- `search_type`: which retrieval strategy to use.
- `search_kwargs`: parameters for that strategy. For basic similarity, `k` controls the number of returned documents.

## 2.2 When should you use it?

Use vanilla similarity when:

- You need a simple, fast baseline.
- The corpus and questions are reasonably well-behaved.
- You want to establish retrieval quality before adding complexity.
- You do not have a demonstrated need for diversity, keyword matching or query rewriting.

**Limitation:** top-k similarity may return several near-duplicate chunks. It also always tries to return `k` results, even if all matches are weak (subject to the vector store's behavior).

---

# 3. Similarity Score Threshold Retriever — “Only return results good enough”

Imagine asking the librarian for three documents, but the library only has one document that is genuinely relevant. A fixed top-3 retriever may still return two weak matches.

A threshold retriever adds a minimum relevance requirement.

```text
Query
  |
  v
Similarity search
  |
  v
Calculate score for each result
  |
  v
Keep only results meeting the threshold
  |
  v
Could return 0, 1, 5, or more documents
```

## 3.1 Example

Suppose the minimum accepted similarity is `0.80`.

| Chunk | Score | Keep? |
|---|---:|---|
| Password reset | 0.92 | Yes |
| Login troubleshooting | 0.84 | Yes |
| Account profile | 0.66 | No |
| Refund policy | 0.32 | No |

Unlike vanilla top-k, the number of results is not necessarily fixed.

## 3.2 LangChain code

```python
threshold_retriever = vectorstore.as_retriever(
    search_type="similarity_score_threshold",
    search_kwargs={
        "score_threshold": 0.80,
        "k": 10,  # candidate/result limit supported by the store
    }
)

docs = threshold_retriever.invoke(
    "How do I reset my account password?"
)
```

**Important:** threshold values are not universal. They depend on the embedding model, distance metric, vector store and score conversion. Calibrate the threshold on your own questions and documents. Do not blindly copy `0.80`.

## 3.3 When should you use it?

Use a threshold when:

- Returning irrelevant documents is worse than returning no documents.
- You want a system to say “I don't have enough evidence” when confidence is low.
- You need to prevent weak context from reaching the LLM.

**Trade-off:** a threshold that is too high can reject useful evidence (false negatives). A threshold that is too low lets noise through.

---

# 4. MMR Retriever — “Give me relevant results, but not five copies of the same idea”

MMR means **Maximal Marginal Relevance**.

Imagine asking for five useful articles about cloud security. A normal similarity retriever might return five chunks all explaining the same firewall rule. They are relevant, but they do not give broad coverage.

MMR tries to balance two goals:

1. **Relevance:** each result should match the query.
2. **Diversity:** selected results should not be too similar to one another.

## 4.1 The story

A user asks:

> “What should I know about securing a cloud application?”

Possible relevant topics include identity management, network security, encryption, monitoring and backups.

- Similarity search may select several chunks about identity management.
- MMR may select one identity chunk, one network chunk, one encryption chunk and one monitoring chunk.

MMR does not mean “choose unrelated documents.” It means choose relevant documents that add different information.

## 4.2 MMR parameters

| Parameter | Meaning |
|---|---|
| `k` | Number of final documents to return |
| `fetch_k` | Number of initial candidates to consider before selecting the final `k` |
| `lambda_mult` | Balance between relevance and diversity |

`lambda_mult` is commonly interpreted as:

- `1.0`: prioritize relevance; behaves closer to ordinary similarity selection.
- `0.0`: prioritize diversity more strongly.
- `0.5`: balance the two.

The exact implementation details can vary by vector-store integration.

## 4.3 LangChain code

```python
mmr_retriever = vectorstore.as_retriever(
    search_type="mmr",
    search_kwargs={
        "k": 4,
        "fetch_k": 15,
        "lambda_mult": 0.5,
    }
)

docs = mmr_retriever.invoke(
    "What should I know about securing a cloud application?"
)
```

Why `fetch_k` is larger than `k`:

1. First find a broader candidate pool.
2. Then select a smaller set that balances relevance and diversity.

If `fetch_k` is too small, MMR has little choice. If it is too large, selection can take more computation.

## 4.4 When should you use MMR?

Use it when:

- The top results are repetitive.
- A question needs coverage across multiple aspects.
- You are building summaries or broad research answers.
- The prompt is being filled with near-duplicate chunks.

Avoid overusing diversity for questions that need one exact fact. For “What is the invoice number?”, the most relevant exact match matters more than topic diversity.

---

# 5. BM25 Retriever — “Find the exact words”

Dense search focuses on meaning. BM25 focuses on **keyword overlap**.

Imagine the user asks:

> “What does error code `E1047` mean?”

A semantic model may understand the general topic but fail to prioritize the exact code. Keyword search is excellent at finding rare literal terms such as:

- Error codes
- Product IDs
- API names
- Legal clause numbers
- Exact technical terms
- Names and acronyms

## 5.1 BM25 in simple language

BM25 is a ranking method built on ideas from TF-IDF, with improvements.

It considers:

1. **Term frequency (TF):** how often a query term appears in a document. Repetition helps, but the benefit saturates; repeating a word 100 times should not make the document infinitely more relevant.
2. **Inverse document frequency (IDF):** rare terms are more informative than common terms. A rare product code is more useful than a common word such as “the”.
3. **Document-length normalization:** a very long document should not automatically win just because it contains more words.

BM25 uses an inverted index and raw text. It does **not** require an embedding model or vector database.

## 5.2 Example

Query: `E1047 payment gateway timeout`

- Document A contains the exact code `E1047` and the phrase “payment gateway timeout”.
- Document B discusses payment processing generally but never mentions the code.
- BM25 is likely to rank A higher because of the literal term matches.

## 5.3 LangChain code

```python
from langchain_community.retrievers import BM25Retriever

# docs is a list of LangChain Document objects.
bm25_retriever = BM25Retriever.from_documents(
    docs,
    k=3
)

results = bm25_retriever.invoke(
    "E1047 payment gateway timeout"
)

for doc in results:
    print(doc.page_content)
```

## 5.4 BM25 limitation

BM25 does not understand synonyms or paraphrases.

A document says:

> “The treatment neutralizes pathogens.”

A user asks:

> “How does it kill harmful microorganisms?”

The concepts are related, but the literal words differ. BM25 may miss the document.

Dense search may catch the meaning even without the exact words.

**Course takeaway:** BM25 is especially useful as the keyword half of hybrid retrieval; using it alone is often not enough for general-purpose semantic Q&A.

## 5.5 When should you use BM25?

Use it when exact terminology matters:

- Error codes and stack traces.
- Product names, IDs and model numbers.
- Legal references and policy identifiers.
- Exact phrases, acronyms and technical vocabulary.

---

# 6. Hybrid Search — “Combine meaning and exact words”

Now we have two search specialists:

- Dense retriever: understands semantic similarity.
- BM25: matches exact words and terms.

Why choose one when they have complementary strengths?

Hybrid search combines their results/signals.

```text
                      User query
                          |
             +------------+------------+
             |                         |
             v                         v
       Dense retriever             BM25 retriever
       meaning search              keyword search
             |                         |
             v                         v
       Dense result list          BM25 result list
             |                         |
             +------------+------------+
                          |
                          v
                Combine / fuse rankings
                          |
                          v
                 Final ranked documents
```

## 6.1 Example

Query:

> “How do vaccines protect people from disease?”

- Dense search can retrieve passages about antibodies, immune response and flu vaccines even if the word “vaccine” is absent.
- BM25 can prioritize passages containing the exact term “vaccine”.
- Hybrid search combines the complementary signals.

## 6.2 Why not just append the two lists?

The two retrievers may use different score scales. A dense score of `0.82` and a BM25 score of `8.2` are not directly comparable.

A fusion method or a vector-store-native hybrid scoring mechanism is needed. One common option is rank-based fusion such as Reciprocal Rank Fusion (RRF); some systems use normalized or weighted scores.

## 6.3 LangChain pattern: EnsembleRetriever

```python
from langchain_classic.retrievers import EnsembleRetriever
from langchain_community.retrievers import BM25Retriever

# Dense retriever
dense_retriever = vectorstore.as_retriever(
    search_kwargs={"k": 5}
)

# Keyword retriever
bm25_retriever = BM25Retriever.from_documents(
    docs,
    k=5
)

# Combine both
ensemble_retriever = EnsembleRetriever(
    retrievers=[bm25_retriever, dense_retriever],
    weights=[0.4, 0.6],
)

results = ensemble_retriever.invoke(
    "How do vaccines protect people from disease?"
)
```

The weights above are illustrative. They express the relative contribution of each retriever in the ensemble implementation; tune them against a test set.

**Version note:** the CampusX material uses LangChain Classic APIs for several advanced retrievers. Package names and deprecation status can change. Treat this as a course-aligned API pattern, not a promise that every import works unchanged in every LangChain release.

## 6.4 When should you use hybrid search?

Use it when your knowledge base contains both:

- Natural-language explanations and paraphrases.
- Exact identifiers, acronyms, product codes or domain terms.

This is a strong general-purpose choice for enterprise search, technical support and policy/document search.

---

# 7. Ensemble Retriever — “Let multiple search experts vote”

Hybrid search is one example of combining retrievers. **Ensemble retrieval** is the broader idea: run more than one retriever and combine their ranked results.

For example:

- Dense retriever searches meaning.
- BM25 searches keywords.
- A domain-specific retriever searches a specialized index.

Each produces a ranked list. The ensemble combines them.

## 7.1 Why use rank fusion?

Different retrievers produce scores that may not be comparable. Instead of comparing raw scores, rank-based fusion can use the position of a document in each list.

### Reciprocal Rank Fusion (RRF)

A common RRF formula is:

\[
RRF(d) = \sum_{r \in R} \frac{w_r}{c + rank_r(d)}
\]

Where:

- `d` = a document.
- `R` = the set of retrievers/result lists.
- `rank_r(d)` = the document's rank in retriever `r` (rank 1 is best).
- `w_r` = optional weight for that retriever.
- `c` = a smoothing constant, often set around 60 in common implementations.

A document ranked highly by multiple retrievers accumulates a strong fused score.

### Tiny example

| Document | BM25 rank | Dense rank |
|---|---:|---:|
| A | 1 | 4 |
| B | 2 | 1 |
| C | 8 | 2 |

Document B performs strongly in both systems. RRF can reward that consistency even though the raw score scales differ.

## 7.2 Code pattern

```python
from langchain_classic.retrievers import EnsembleRetriever

ensemble = EnsembleRetriever(
    retrievers=[bm25_retriever, dense_retriever],
    weights=[0.5, 0.5],
)

docs = ensemble.invoke("How do vaccines protect against disease?")
```

## 7.3 When should you use ensemble retrieval?

Use it when:

- One retrieval method misses important documents.
- You have multiple useful indexes or search strategies.
- You want broader coverage and more robust candidate retrieval.

Trade-off: more retrieval calls, more candidate merging and potentially higher latency. Do not add retrievers without measuring whether they improve results.

---

# 8. Chapter 6: Put the basic retrievers in the right order

A useful learning sequence is:

1. **Similarity:** understand the baseline semantic search.
2. **Threshold:** understand when to reject weak matches.
3. **MMR:** understand how to reduce duplicate results.
4. **BM25:** understand exact keyword search.
5. **Hybrid:** combine dense meaning + keyword matching.
6. **Ensemble:** combine multiple retrievers and fuse their rankings.

These are not six unrelated products. Each adds a capability to solve a particular weakness.

## 8.1 Comparison table

| Retriever | Main signal | Solves | Main limitation |
|---|---|---|---|
| Similarity | Dense embedding similarity | Finds semantically related chunks | Can return duplicates or weak matches |
| Threshold | Similarity + minimum score | Rejects weak matches | Threshold needs calibration |
| MMR | Relevance + diversity | Reduces repetitive context | May trade away a little relevance |
| BM25 | Keyword overlap | Exact terms, IDs, codes | Misses synonyms/paraphrases |
| Hybrid | Dense + sparse/keyword | Meaning and exact-term coverage | More complexity and latency |
| Ensemble | Multiple ranked result lists | Combines complementary retrievers | More calls and fusion logic |

---

# PART B — CHAPTER 7: ADVANCED RETRIEVERS

# 9. Why do we need advanced retrievers?

Basic retrievers are good at finding candidate chunks. But real documents and real user questions create four additional problems:

1. A retrieved chunk contains useful information mixed with irrelevant sentences.
2. Small chunks retrieve precisely but do not contain enough surrounding context.
3. A query contains metadata constraints such as date, author, category or department.
4. A user asks a vague question using wording that does not match the documents.

Advanced retrievers are designed to address these problems.

**Mental model:** basic retrievers mainly decide *which chunks look relevant*. Advanced retrievers improve *what is retrieved, how much context is returned, which metadata filters apply, or how the query is expressed*.

The four Chapter 7 techniques in this course are:

1. Contextual Compression Retriever
2. Parent Document Retriever
3. Self-Query Retriever
4. Multi-Query Retriever

---

# 10. Contextual Compression Retriever — “Find the useful sentence, not the whole noisy page”

## 10.1 The problem

Suppose a chunk contains five topics:

- AI
- quantum computing
- medicine
- hospital management
- personalized medicine

The user asks:

> “How does CRISPR help create personalized medicine?”

The base retriever finds the chunk because it contains relevant medicine information. But the entire chunk is sent to the LLM, including unrelated topics.

This creates **context noise**:

- More tokens enter the prompt.
- The LLM must search through irrelevant sentences.
- Useful information can be diluted.
- Prompt cost may increase.

## 10.2 The solution

Contextual compression adds a step after normal retrieval:

```text
User question
     |
     v
Base retriever
     |
     v
Full candidate documents
     |
     v
Compressor receives query + documents
     |
     v
Keep relevant documents / passages
     |
     v
Smaller, focused context
     |
     v
LLM generates answer
```

The base retriever can be similarity, MMR, threshold, ensemble or another retriever.

**Important distinction:** embedding a document into a vector is not contextual compression. Embeddings represent text numerically; contextual compression deliberately filters or extracts text after retrieval.

## 10.3 Types of compressors

### A. LLMChainExtractor — remove irrelevant text inside a document

The LLM reads the query and each retrieved document, then extracts the relevant portions.

Example:

Original retrieved chunk:

```text
Quantum computers use qubits.
CRISPR can edit DNA at targeted locations.
Hospitals use software to manage patient appointments.
CRISPR research may support personalized medicine.
```

Query: “How can CRISPR support personalized medicine?”

Compressed result:

```text
CRISPR can edit DNA at targeted locations.
CRISPR research may support personalized medicine.
```

The extractor should preserve source meaning and avoid inventing or paraphrasing facts. Validate outputs because LLMs can omit qualifiers or make mistakes.

### B. EmbeddingsFilter — keep or drop whole documents

This filter embeds the query and each candidate document, calculates similarity, and keeps documents above a threshold.

- Cheap compared with an LLM extractor.
- Can remove irrelevant documents.
- Cannot remove irrelevant sentences from inside a document.
- A surviving document is returned whole.

### C. DocumentCompressorPipeline — combine steps

You can apply a cheap filter first, then an LLM extractor:

```text
Retrieved documents
       |
       v
EmbeddingsFilter (drop clearly irrelevant documents)
       |
       v
LLMChainExtractor (trim irrelevant sentences)
       |
       v
Compressed documents
```

Order matters. Filtering first can reduce the number of documents sent to the more expensive LLM step.

## 10.4 LangChain code pattern

```python
from langchain_classic.retrievers import ContextualCompressionRetriever
from langchain_classic.retrievers.document_compressors import (
    LLMChainExtractor,
)

# llm is your configured chat model.
# base_retriever is an existing retriever, such as vectorstore.as_retriever().
compressor = LLMChainExtractor.from_llm(llm)

compression_retriever = ContextualCompressionRetriever(
    base_compressor=compressor,
    base_retriever=base_retriever,
)

compressed_docs = compression_retriever.invoke(
    "How does CRISPR support personalized medicine?"
)
```

Embedding filter pattern:

```python
from langchain_classic.retrievers.document_compressors import EmbeddingsFilter

embedding_filter = EmbeddingsFilter(
    embeddings=embeddings,
    similarity_threshold=0.73,  # illustrative; calibrate for your model
)
```

Pipeline pattern:

```python
from langchain_classic.retrievers.document_compressors import (
    DocumentCompressorPipeline,
)

pipeline = DocumentCompressorPipeline(
    transformers=[
        embedding_filter,  # inexpensive document-level filtering
        compressor,        # query-aware text extraction
    ]
)
```

The exact constructor and import paths can differ by LangChain version.

## 10.5 When should you use contextual compression?

Use it when:

- Chunks are large or contain several topics.
- Retrieved documents are relevant overall but contain substantial noise.
- Prompt token usage is high.
- You want concise evidence before generation.

Avoid it when every word and qualifier matters and the compressor has not been validated. Compression can accidentally remove a condition, exception, number or important caveat.

**Memory trick:** *Retrieve first; clean second.*

---

# 11. Parent Document Retriever — “Search small, return big”

## 11.1 The chunk-size problem

Imagine a technical manual with a section:

> “Installing and configuring the database”

The section is 2,000 words long. It contains installation steps, prerequisites, troubleshooting and configuration.

There are two possible chunking choices:

- **Large chunks:** preserve context but may be difficult to retrieve precisely.
- **Small chunks:** retrieve precise details but may omit surrounding explanation.

This is the precision-versus-context trade-off.

## 11.2 The parent-child solution

Parent Document Retriever uses two levels:

- **Parent document:** a larger section containing broader context.
- **Child document:** a smaller chunk used for accurate retrieval.

The system indexes/searches the child chunks. Each child stores a reference to its parent. When a child matches, the retriever fetches the parent and returns the larger context.

```text
Parent section: Database installation
|
+-- Child A: system prerequisites       (indexed)
+-- Child B: install database package   (indexed)
+-- Child C: configure connection       (indexed)
+-- Child D: troubleshoot connection    (indexed)

Question: "How do I configure the database connection?"
                         |
                         v
              Search child chunks
                         |
                         v
              Child C is a strong match
                         |
                         v
              Look up Child C's parent ID
                         |
                         v
              Return parent section as context
```

## 11.3 Why is this useful?

- Child chunks provide precise matching.
- Parent documents restore surrounding context.
- The LLM receives a more complete explanation.
- You avoid searching only large, broad chunks.

The parent need not be the entire original document. It can be a section or a larger chunk.

## 11.4 LangChain code pattern

```python
from langchain_classic.retrievers import ParentDocumentRetriever
from langchain_classic.storage import InMemoryStore
from langchain_text_splitters import RecursiveCharacterTextSplitter

# vectorstore: child chunks are indexed here.
# docs: original parent Document objects.

child_splitter = RecursiveCharacterTextSplitter(
    chunk_size=400,
    chunk_overlap=60,
)

parent_splitter = RecursiveCharacterTextSplitter(
    chunk_size=2000,
    chunk_overlap=200,
)

store = InMemoryStore()

parent_retriever = ParentDocumentRetriever(
    vectorstore=vectorstore,
    docstore=store,
    child_splitter=child_splitter,
    parent_splitter=parent_splitter,
)

parent_retriever.add_documents(docs)

results = parent_retriever.invoke(
    "How do I configure the database connection?"
)
```

This is a representative pattern. Some implementations use only a child splitter and store the original input documents as parents; others use a parent splitter as well. Check the installed class signature.

## 11.5 What is stored where?

| Component | Stores |
|---|---|
| Vector store | Child chunk embeddings, text and metadata (including parent ID) |
| Document/key-value store | Parent documents keyed by their IDs |
| Retriever | Logic to search children and fetch their parents |

A persistent production system should use a persistent document store rather than relying on an in-memory store that disappears when the process stops.

## 11.6 When should you use Parent Document Retriever?

Use it when:

- Precise child-level matching is important.
- The answer needs surrounding explanation or steps.
- Documents have meaningful sections.
- Small chunks alone produce incomplete answers.

Avoid returning huge parent documents without limits. Large parents can reintroduce noise and consume the context window.

**Memory trick:** *Child finds; parent explains.*

---

# 12. Self-Query Retriever — “Understand the meaning and the filters”

## 12.1 The problem

A user asks:

> “Find recent research papers about Graph RAG written after 2024 by CampusX instructors.”

This question contains two kinds of information:

### Semantic meaning
- Research papers about Graph RAG.

### Metadata constraints
- Date after 2024.
- Author/organization = CampusX instructors.

A normal semantic retriever may find papers about Graph RAG but ignore the date or author requirement unless those details are represented and filtered correctly.

## 12.2 The solution

Self-Query Retriever uses an LLM to translate the natural-language query into:

1. A **semantic search query**.
2. A **structured metadata filter**.

```text
Natural-language question
          |
          v
        LLM
          |
          v
+----------------------------+
| Semantic query:            |
| "Graph RAG research paper" |
|                            |
| Metadata filter:           |
| year > 2024                |
| author/organization = ...  |
+----------------------------+
          |
          v
Vector store search with filter
          |
          v
Relevant documents satisfying constraints
```

The LLM is not searching the database itself. It is converting the user's request into a structured query that the retriever can execute.

## 12.3 Metadata must exist

A filter only works if the indexed documents actually contain the relevant metadata.

Example:

```python
from langchain_core.documents import Document

docs = [
    Document(
        page_content="A paper discussing Graph RAG...",
        metadata={
            "title": "Graph Retrieval Study",
            "year": 2025,
            "author": "A. Kumar",
            "category": "research",
        },
    )
]
```

If `year` or `author` is missing or stored in an inconsistent format, the filter may fail or return incomplete results.

## 12.4 Schema and translator

Self-query systems need a description of:

- What each metadata field means.
- Its data type (string, integer, date, etc.).
- Which comparison operators are valid.
- Which fields are allowed to be filtered.

The query constructor creates a structured representation. A translator converts that representation into the filter syntax supported by the vector store.

## 12.5 LangChain code pattern

```python
from langchain_classic.retrievers.self_query.base import (
    SelfQueryRetriever,
)
from langchain_classic.chains.query_constructor.base import (
    AttributeInfo,
)
from langchain_core.documents import Document

metadata_field_info = [
    AttributeInfo(
        name="year",
        description="Publication year of the paper",
        type="integer",
    ),
    AttributeInfo(
        name="author",
        description="Author of the paper",
        type="string",
    ),
    AttributeInfo(
        name="category",
        description="Document category, such as research or tutorial",
        type="string",
    ),
]

document_content_description = (
    "Research papers and technical documents about retrieval "
    "augmented generation and related AI topics."
)

self_query_retriever = SelfQueryRetriever.from_llm(
    llm,
    vectorstore,
    document_content_description,
    metadata_field_info,
    verbose=True,
)

results = self_query_retriever.invoke(
    "Find research papers about Graph RAG published after 2024"
)
```

This is a course-style example. The exact import path and vector-store translator support vary across LangChain versions and vector databases.

## 12.6 Security and correctness warning

Never treat LLM-generated metadata filters as authorization.

For example, a user must not be able to retrieve confidential HR documents simply by asking for `department = "HR"`.

- Apply access-control filters in trusted application code.
- Allowlist filterable fields and operators.
- Validate generated filters.
- Enforce user/tenant permissions independently of the LLM.

## 12.7 When should you use Self-Query?

Use it when:

- Users naturally mention dates, authors, departments, categories, locations or document types.
- The vector store contains reliable metadata.
- You need semantic matching and structured filtering together.

Do not use it just to add complexity to a simple semantic question. If the filter is known by the application, a normal metadata filter is often simpler and safer.

**Memory trick:** *Self-query = LLM understands the request, then creates the filter.*

---

# 13. Multi-Query Retriever — “Ask the same question in several ways”

## 13.1 The problem

Users do not know the exact vocabulary used in your documents.

Document wording:

> “Retrieval-augmented generation grounds model output in external evidence.”

User wording:

> “How does RAG stop AI from making things up?”

The query may be understandable to a person but may not align perfectly with the indexed wording.

One query produces one search. If that search misses a relevant passage, the evidence is lost.

## 13.2 The solution

An LLM generates multiple alternative phrasings or subtopic-focused versions of the original query. The retriever searches for each one, then merges and deduplicates the results.

```text
Original question:
"How does RAG reduce hallucinations?"
             |
             v
     LLM creates variants
       /       |        \
      v        v         v
"RAG grounding" "reduce LLM errors" "factual accuracy with retrieval"
      |        |         |
      v        v         v
  Search 1   Search 2   Search 3
      \        |         /
       \       |        /
        Merge + deduplicate
                 |
                 v
         Combined documents
```

## 13.3 Example

Original question:

> “How can I improve my physical health?”

Generated queries might be:

1. “How can I improve cardiovascular fitness?”
2. “What habits support mental well-being?”
3. “How can nutrition and exercise improve immunity?”

The system can retrieve different useful documents for each angle.

**Important:** the generated queries should remain faithful to the user's intent. If they drift into unrelated topics, they introduce noise.

## 13.4 LangChain code pattern

```python
from langchain_classic.retrievers.multi_query import MultiQueryRetriever

multi_query_retriever = MultiQueryRetriever.from_llm(
    retriever=base_retriever,
    llm=llm,
)

results = multi_query_retriever.invoke(
    "How does RAG reduce hallucinations?"
)
```

A custom prompt can control how many variants are generated and what kind of variation is useful. Some versions expose an `include_original` option; confirm the installed version's signature.

## 13.5 What happens after each search?

Suppose the searches return:

```text
Query 1 -> chunks A, C, F
Query 2 -> chunks C, D, F
Query 3 -> chunks A, F, H
```

After merging and deduplication:

```text
A, C, D, F, H
```

A document returned by several variants appears once in the final list.

The course distinguishes this from RAG Fusion: basic Multi-Query merges/deduplicates, while RAG Fusion explicitly uses rank fusion (RRF) to preserve and combine rank information. RAG Fusion is covered separately in Chapter 9.

## 13.6 When should you use Multi-Query?

Use it when:

- User queries are vague or use inconsistent terminology.
- The same concept is described in multiple ways across documents.
- A single query does not provide enough coverage.
- The cost and latency of multiple searches are acceptable.

Trade-offs:

- An additional LLM call is needed to generate variants.
- More retrieval calls increase latency and load.
- The merged list may contain duplicates or off-topic results.
- More retrieved documents do not automatically mean better answers.

**Memory trick:** *One question → many phrasings → many searches → one deduplicated result set.*

---

# 14. Chapter 7: Compare the four advanced retrievers

| Advanced retriever | Problem it solves | What it changes | Use it when |
|---|---|---|---|
| Contextual Compression | Retrieved chunks contain noise | Filters documents and/or extracts relevant passages | Chunks are long or mixed-topic |
| Parent Document | Small chunks lack context | Searches child chunks, returns parent context | Precise match needs surrounding explanation |
| Self-Query | Query includes metadata conditions | Converts natural language into semantic query + filter | Users mention date, author, category, etc. |
| Multi-Query | One wording misses relevant evidence | Generates several query variants and merges results | Queries are vague or terminology varies |

### The easiest way to remember them

- **Compression:** clean the retrieved content.
- **Parent Document:** restore the surrounding context.
- **Self-Query:** understand metadata constraints.
- **Multi-Query:** search using different phrasings.

---

# PART C — WHICH ONE SHOULD I USE?

# 15. Practical decision guide

Start with the simplest retriever and add a technique only when you can identify a problem.

```text
Start with a user question
          |
          v
Do normal semantic results look good?
       /          \
     Yes           No
      |             |
  Keep simple       v
             Are exact IDs / codes / keywords
             being missed?
                 /       \
               Yes        No
                |          |
          Add BM25;        v
          consider       Are results repetitive?
          hybrid           /       \
                          Yes       No
                           |         |
                         Use MMR    v
                               Is retrieved text noisy?
                                  /       \
                                Yes        No
                                 |          |
                         Compression       v
                                   Does a small match
                                   need larger context?
                                      /       \
                                    Yes        No
                                     |          |
                              Parent Document  v
                                      Are metadata filters
                                      needed from natural language?
                                         /       \
                                       Yes        No
                                        |          |
                                   Self-Query    v
                                          Is query wording
                                          ambiguous / narrow?
                                             /       \
                                           Yes        No
                                            |          |
                                       Multi-Query  Keep baseline
```

## 15.1 Use-case examples

| Real-world requirement | Recommended starting point | Why |
|---|---|---|
| General company knowledge assistant | Dense similarity | Simple semantic baseline |
| Search exact error code `ORA-00942` | BM25 or hybrid | Exact token matters |
| Search product name plus natural-language issue | Hybrid | Exact name + semantic intent |
| “Give me different approaches to…” | MMR | Reduce repetitive chunks |
| Do not answer if evidence is weak | Threshold | Reject low-score matches |
| Retrieved paragraphs contain unrelated material | Contextual Compression | Remove noise |
| Exact detail is in a small chunk, explanation is in its section | Parent Document | Precise child search + parent context |
| “Papers after 2024 by author X” | Self-Query or explicit metadata filter | Structured constraints |
| User asks vague questions with varied wording | Multi-Query | Broaden query coverage |
| Different retrievers each find useful results | Ensemble / RRF | Combine rankings |

## 15.2 A sensible enterprise starting architecture

For many enterprise knowledge assistants, a reasonable starting point is:

```text
User query
   |
   v
Hybrid retrieval (dense + BM25)
   |
   v
Optional MMR / deduplication
   |
   v
Optional reranking (Chapter 8)
   |
   v
Optional contextual compression
   |
   v
Prompt + evidence
   |
   v
LLM answer with citations
```

This is not a universal architecture. Do not enable every technique by default. Each additional stage can increase latency, cost, operational complexity and failure opportunities.

---

# PART D — IMPORTANT CONCEPTS & INTERVIEW REVISION

# 16. Concepts interviewers commonly test

## 16.1 Retriever vs vector store

| Vector store | Retriever |
|---|---|
| Stores vectors, text and metadata (depending on integration) | Searches and returns relevant documents |
| Provides operations such as add/search/delete | Provides a common retrieval interface |
| Can expose native similarity search | Can configure search type and parameters |
| Is a storage/search backend | Is a retrieval component used in a RAG pipeline |

A vector store can perform similarity search directly. In LangChain, `as_retriever()` wraps the store in a retriever interface that can be configured and composed with other runnables.

## 16.2 Retriever vs re-ranker

- **Retriever:** finds a candidate set from a large corpus.
- **Re-ranker:** reorders a smaller candidate set using a more query-aware relevance model.

A common design is: retrieve 20–50 candidates quickly, then rerank them to the final top 3–5. The exact counts depend on latency, cost and corpus size.

## 16.3 Dense vs sparse retrieval

| Dense | Sparse / keyword |
|---|---|
| Uses dense embedding vectors | Uses token/term-weight representations |
| Captures semantic similarity | Captures lexical overlap |
| Can match paraphrases | Strong for exact terms |
| May miss rare exact tokens | May miss synonyms |
| Example: vector similarity | Example: BM25 |

## 16.4 Similarity vs MMR

- Similarity focuses on how close each result is to the query.
- MMR also considers whether a selected result adds new information compared with results already selected.

## 16.5 Multi-Query vs RAG Fusion

- Multi-Query generates alternate queries, retrieves results and merges/deduplicates them.
- RAG Fusion also generates multiple queries but uses rank fusion such as RRF to combine result rankings.

## 16.6 Contextual Compression vs Parent Document

- Compression makes retrieved content **smaller and more focused**.
- Parent Document makes retrieved context **larger and more complete** by returning the parent of a matching child.

They solve opposite-looking problems and can be combined carefully: retrieve a child, fetch an appropriate parent, then compress that parent for the final query.

## 16.7 Self-Query vs ordinary metadata filtering

- Ordinary metadata filtering: application code already knows the filter and passes it directly.
- Self-Query: an LLM extracts the filter from a natural-language request and combines it with semantic search.

If the filter is deterministic and known, ordinary filtering is usually simpler.

---

# 17. Interview questions and answers

### Q1. What is a retriever in RAG?
A retriever accepts a user query and returns relevant documents from an external knowledge source. The documents are then supplied as context to the LLM.

### Q2. Does a retriever return embeddings?
No. Embeddings are used internally for similarity calculation. The retriever normally returns `Document` objects containing text and metadata.

### Q3. What is the difference between a vector store and a retriever?
A vector store is the storage/search backend for vectors and associated records. A retriever is the component/interface that performs retrieval and returns documents; it may wrap a vector store or use another search method.

### Q4. What does `k` mean?
`k` is the number of results requested from a retrieval operation. It controls result count, not relevance quality.

### Q5. What is a similarity threshold retriever?
It returns only documents whose similarity score meets a configured minimum. The number of results can vary, including zero.

### Q6. Why can a threshold be dangerous?
A threshold that is too high can reject useful documents. Score scales vary across models and stores, so the threshold must be calibrated on representative queries.

### Q7. What is MMR?
Maximal Marginal Relevance selects documents that are relevant to the query while reducing redundancy among selected results.

### Q8. Explain `k`, `fetch_k` and `lambda_mult` in MMR.
`k` is the final number of documents; `fetch_k` is the initial candidate pool; `lambda_mult` controls the relevance-versus-diversity balance.

### Q9. What is BM25?
BM25 is a keyword-based ranking function. It considers term frequency, inverse document frequency and document-length normalization. It does not require an embedding model.

### Q10. Why is BM25 weak for paraphrases?
It relies on lexical overlap. If the query and document use different words for the same idea, BM25 may not recognize their semantic relationship.

### Q11. Why combine BM25 and dense retrieval?
They have complementary strengths. Dense search captures semantic similarity; BM25 captures exact terms. Hybrid retrieval can improve coverage.

### Q12. Why should raw scores from BM25 and vector search not be naively compared?
They may have different scales and meanings. Use an appropriate fusion method, score normalization or a vector-store-native hybrid strategy.

### Q13. What is an ensemble retriever?
It runs multiple retrievers and combines their result lists, often using weighted rank fusion such as RRF.

### Q14. What is Reciprocal Rank Fusion?
RRF gives each document a contribution based on its rank in each result list. Documents ranked highly by multiple retrievers accumulate a higher combined score.

### Q15. What problem does Contextual Compression solve?
It removes irrelevant documents or irrelevant portions of retrieved documents so that the LLM receives a smaller, more focused context.

### Q16. What is the difference between `LLMChainExtractor` and `EmbeddingsFilter`?
An LLM extractor can select relevant text within a document. An embeddings filter generally keeps or drops the whole document based on similarity; it does not trim individual sentences.

### Q17. Why put an embedding filter before an LLM extractor?
The embedding filter can cheaply remove clearly irrelevant documents first, reducing the number of documents sent to the more expensive LLM compressor.

### Q18. What problem does Parent Document Retriever solve?
Small chunks retrieve precisely but may lack context. It searches small child chunks and returns their larger parent documents for better context.

### Q19. Where are parent documents and child embeddings stored?
Child embeddings and their metadata are stored in the vector store. Parent documents are stored in a document/key-value store and fetched using the parent ID.

### Q20. What does Self-Query Retriever do?
It uses an LLM to convert a natural-language request into a semantic search query plus structured metadata filters.

### Q21. Does Self-Query create metadata?
No. It interprets the request and builds filters over metadata that must already exist in the indexed documents.

### Q22. Why is Self-Query security-sensitive?
LLM-generated filters must not control authorization. Trusted application logic must enforce tenant and user permissions independently.

### Q23. What does Multi-Query Retriever do?
It asks an LLM to create multiple query variants, runs retrieval for each, and merges/deduplicates the returned documents.

### Q24. What is the disadvantage of Multi-Query?
It adds an LLM call and multiple retrieval calls, increasing latency, cost and the possibility of query drift or irrelevant results.

### Q25. How is Multi-Query different from RAG Fusion?
Multi-Query typically merges and deduplicates results. RAG Fusion uses rank-based fusion such as RRF to combine the ranking evidence from multiple query result lists.

### Q26. Can I use more than one advanced retriever together?
Yes, if each stage solves a real problem. For example, retrieve child chunks using hybrid search, fetch parents, and compress the resulting context. Evaluate the combined pipeline because each stage adds cost and possible failure modes.

### Q27. Which retriever should I choose first in a new RAG project?
Start with dense similarity retrieval as a baseline. Evaluate it on real questions. Add BM25/hybrid for exact terms, MMR for redundancy, thresholding for weak matches, and advanced retrievers only when a measured problem calls for them.

### Q28. Does better retrieval guarantee a correct answer?
No. It improves the evidence available to the LLM, but the LLM can still misunderstand, omit conditions or generate unsupported claims. Evaluate retrieval and generation separately.

---

# 18. Practical revision checklist

Use this to check whether you can explain the concepts without reading the notes.

- [ ] I can explain the RAG retrieval stage in the complete pipeline.
- [ ] I know what a LangChain `Document` contains.
- [ ] I can explain how a dense retriever creates and searches a query vector.
- [ ] I know why query and document embeddings must be compatible.
- [ ] I can explain `k` and configure a similarity retriever.
- [ ] I can explain why a threshold retriever may return zero documents.
- [ ] I can explain MMR, `fetch_k` and `lambda_mult`.
- [ ] I can explain BM25 without relying on its mathematical formula.
- [ ] I can explain why BM25 and dense retrieval complement each other.
- [ ] I understand hybrid and ensemble retrieval.
- [ ] I can explain why RRF uses ranks rather than blindly comparing raw scores.
- [ ] I can explain contextual compression and its three compressor patterns.
- [ ] I can explain the child-search / parent-return mechanism.
- [ ] I can explain how Self-Query creates a semantic query and metadata filter.
- [ ] I know that Self-Query cannot invent missing metadata or enforce authorization.
- [ ] I can explain Multi-Query and its latency trade-off.
- [ ] I can distinguish Multi-Query from RAG Fusion.
- [ ] I can choose a retriever based on a real use case.

---

# 19. Final memory map

```text
RETRIEVERS
|
+-- BASIC
|   |
|   +-- Similarity       -> find semantically close chunks
|   +-- Threshold        -> reject weak matches
|   +-- MMR              -> reduce repetitive results
|   +-- BM25             -> find exact keywords
|   +-- Hybrid            -> combine dense meaning + keywords
|   +-- Ensemble          -> combine multiple ranked lists
|
+-- ADVANCED
    |
    +-- Contextual Compression -> remove irrelevant content
    +-- Parent Document        -> search child, return parent
    +-- Self-Query              -> semantic query + metadata filter
    +-- Multi-Query             -> rewrite question, search many ways
```

**One-line recall:**

- Similarity finds.
- Threshold rejects weak matches.
- MMR diversifies.
- BM25 matches words.
- Hybrid combines meaning and words.
- Ensemble combines retrievers.
- Compression cleans.
- Parent restores context.
- Self-Query filters.
- Multi-Query rephrases.

**Best practical rule:** Do not choose a retriever because it sounds advanced. Identify the retrieval failure, choose the lightest technique that fixes it, and verify the improvement with a representative evaluation set.
