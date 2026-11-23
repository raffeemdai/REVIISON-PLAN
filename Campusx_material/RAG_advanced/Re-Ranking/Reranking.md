https://mlpills.substack.com/p/issue-115-reranking-in-your-rag-pipeline

https://dataaspirant.com/blog/reranking/

https://searchministry.au/guides/what-is-cross-encoder-reranking


<img width="870" height="549" alt="image" src="https://github.com/user-attachments/assets/e807d25e-98cf-49ba-a327-0432fb8b9459" />

<img width="640" height="427" alt="image" src="https://github.com/user-attachments/assets/74217873-ad90-40c3-b1b4-7a1c63e313cb" />
<img width="668" height="504" alt="image" src="https://github.com/user-attachments/assets/99759bb7-6aa0-4ea7-8e89-7bb16c8dfd72" />
<img width="1456" height="813" alt="image" src="https://github.com/user-attachments/assets/459e516e-7334-41c7-b6c0-cd03ed4d1c6e" />

<img width="848" height="473" alt="image" src="https://github.com/user-attachments/assets/e97a0fc5-3e9d-44f2-881f-466a7a5c34be" />
<img width="1730" height="909" alt="image" src="https://github.com/user-attachments/assets/14fc7174-3d5f-427c-a350-643f9852c0ce" />
<img width="968" height="437" alt="image" src="https://github.com/user-attachments/assets/eb770156-9b70-48e7-a008-6b754f49a3e9" />
<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/69943baa-aff0-44c1-9a14-61b4dd1282d5" />
<img width="1109" height="555" alt="image" src="https://github.com/user-attachments/assets/85fb705d-4204-40ca-9df3-40a8d74247c4" />
<img width="1147" height="573" alt="image" src="https://github.com/user-attachments/assets/d770e063-95de-4a4b-828d-1ba0498cde18" />
<img width="1048" height="583" alt="image" src="https://github.com/user-attachments/assets/bfc872a6-0ed0-4686-a7c9-cc49570530e7" />
<img width="1140" height="569" alt="image" src="https://github.com/user-attachments/assets/8ca234a6-b3c9-4ece-aa53-b43d3dcca25d" />
<img width="650" height="579" alt="image" src="https://github.com/user-attachments/assets/ad84f49d-29c7-4ced-b70a-52a28860e0c5" />
<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/a42e7631-8f8b-4b33-b9e8-a840ff3466d5" />
<img width="1376" height="768" alt="image" src="https://github.com/user-attachments/assets/9d0f987f-3a5b-444c-ab94-2f44b8be5ef1" />
<img width="1376" height="768" alt="image" src="https://github.com/user-attachments/assets/a4745b10-ae12-4685-8c31-c3242bdd49ec" />
<img width="1376" height="768" alt="image" src="https://github.com/user-attachments/assets/6e6f0643-261f-4d5b-95a6-9c70680cd921" />
<img width="1376" height="768" alt="image" src="https://github.com/user-attachments/assets/a0a62617-733c-4059-a247-7181a04bc532" />
<img width="1376" height="768" alt="image" src="https://github.com/user-attachments/assets/d5548906-2ce3-4722-9e0e-f2e7be558bc2" />
<img width="1376" height="768" alt="image" src="https://github.com/user-attachments/assets/4bd74802-50e3-44d9-a4f3-8a35a37fb9de" />
<img width="1376" height="768" alt="image" src="https://github.com/user-attachments/assets/97814a6c-c54b-4e31-a527-602dd58a765f" />
<img width="1376" height="768" alt="image" src="https://github.com/user-attachments/assets/fa28f831-6ac6-4bad-8a53-6d9faeaad4d6" />

<img width="1376" height="768" alt="image" src="https://github.com/user-attachments/assets/6811ba88-ae42-45d8-b8df-138a7092ce6b" />

<img width="1376" height="768" alt="image" src="https://github.com/user-attachments/assets/d0056ae3-1b80-480c-b892-83b5cf7aafca" />
<img width="1376" height="768" alt="image" src="https://github.com/user-attachments/assets/6b567b94-293a-4fe1-880a-0d35089bfce6" />
<img width="1376" height="768" alt="image" src="https://github.com/user-attachments/assets/974af2c3-342a-4500-bd36-975a03861c86" />

# RAG Re-Ranking --- Interview Guide

## Story-first explanation, core concepts, workflow, and Python implementation

> **Goal:** Explain why reranking is needed, how a cross-encoder
> improves retrieved-chunk ordering, and how to add it to a LangChain
> RAG pipeline.



------------------------------------------------------------------------

# Part 1 --- Explain Reranking in an Interview: Story Format

## 1. The story

Imagine a company has a knowledge base containing **one million document
chunks**: product manuals, technical documents, support tickets, and
security policies.

An employee asks:

> "What are the security requirements for API access?"

The RAG system searches its vector database and quickly finds 20 chunks
that look semantically related. The first result may discuss APIs
generally, another may discuss network security, and a chunk ranked
fourth may contain the exact API access requirements.

Vector search found useful candidates, but its ordering is not always
perfect.

Now imagine a subject-matter expert reads the question and each of those
20 chunks, one by one, and asks: **"How directly does this chunk answer
this exact question?"** The expert moves the most relevant chunks to the
top and pushes less useful ones down.

That second review is the idea behind **re-ranking**.

## 2. Problem

🔵 Normal vector search compares separately generated embeddings of the
query and document. The embedding model compresses text into a
fixed-size vector. This makes semantic search fast, but fine-grained
details can be lost.

As a result: - A generally related chunk may receive a high similarity
score. - A highly relevant chunk may receive a lower score. - With a
small top-k, the best chunk may not make it into the final context. -
The LLM may receive noisy or less useful context.

🟠 **The issue is not always that retrieval finds nothing relevant. The
issue can be that the retrieved candidates are in the wrong order.**

## 3. Solution

🔵 **Re-ranking adds a second ranking pass after initial retrieval.** A
cross-encoder examines the query and each candidate chunk together and
produces a relevance score.

1.  Retrieve a larger candidate set using the fast vector retriever.
2.  Score each candidate again using the reranker.
3.  Sort candidates by the new relevance scores.
4.  Keep only the best few chunks for the LLM.

🟢 Example: retrieve 20 candidates, rerank those 20, and pass the best 5
to the LLM.

## 4. How it works --- Step-by-step workflow

``` text
                 USER QUESTION
                       |
                       v
              QUERY EMBEDDING
                       |
                       v
             VECTOR DATABASE
          (1 million stored chunks)
                       |
                       v
        BI-ENCODER / SIMILARITY SEARCH
              Retrieve top 20
                       |
                       v
             CANDIDATE CHUNKS
          Chunk 1, Chunk 2, ... Chunk 20
                       |
                       v
                CROSS-ENCODER
        Query + each candidate chunk together
                       |
                       v
             RELEVANCE SCORES
           Score every candidate chunk
                       |
                       v
              SORT BY NEW SCORE
                       |
                       v
             KEEP FINAL TOP 5
                       |
                       v
              LLM + CONTEXT
                       |
                       v
                 FINAL ANSWER
```

🟠 The vector database does the initial candidate retrieval. The
reranker reorders that candidate list; it does not normally search the
entire knowledge base again.

## 5. Purpose and use cases

The purpose is to improve the **quality and ordering of context**
supplied to the LLM.

Useful situations: - Enterprise knowledge assistants searching manuals
and policies. - Customer-support RAG where several documents discuss
similar issues. - Technical documentation search where exact details
matter. - Legal, finance, or compliance retrieval where the best passage
should be prioritized. - Large knowledge bases where initial similarity
ranking contains near-matches and noise.

🔴 Re-ranking improves the relevance estimate; it does not guarantee
correctness and cannot recover a document that was never retrieved into
the candidate set.

## 6. It is useful when

-   The retriever finds relevant documents, but the ordering is not good
    enough.
-   Top results are broadly related rather than directly answering the
    question.
-   Important chunks appear lower in the initial ranking.
-   The LLM receives too much irrelevant context.
-   You can afford an additional scoring step on a limited candidate
    set.

It may be less useful when the corpus is tiny, initial retrieval is
already strong, or added latency is unacceptable.

## 7. Easy memory trick

🟣 **Bi-Encoder = Prelims. Cross-Encoder = Mains.**

-   **Prelims:** Quickly filters a very large group.
-   **Mains:** Carefully evaluates the shortlisted candidates.

The bi-encoder is fast and selects candidates. The cross-encoder is
slower but judges the shortlist more carefully.

> **Retrieve broadly. Rerank carefully. Send selectively.**

## 8. Interview-ready summary

**Question: What is reranking in RAG, and why do we need it?**

"Reranking is a second-stage retrieval technique used to improve the
relevance order of documents before they are passed to the LLM. In a
typical RAG pipeline, a bi-encoder embedding model quickly retrieves an
initial set of candidate chunks from the vector database. However,
embeddings compress the query and documents separately, so similarity
scores are approximate and may not place the most relevant chunk at the
top. A cross-encoder then evaluates the query and each candidate chunk
together, assigns a relevance score, and reorders the candidates. We
keep only the top few chunks as context for the LLM. This combines the
speed of bi-encoder retrieval with the more precise relevance judgment
of cross-encoder reranking."

------------------------------------------------------------------------

# Part 2 --- Core Concepts Explained Simply

## 9. Why Re-Ranking Is Needed

🔵 Vector search finds semantically similar content quickly. But
**semantic similarity is not always the same as answering the user's
exact question**.

For example, a query about "API access security requirements" might
retrieve a general API overview, a network security guide, an
authentication guide, and a policy listing the exact access
requirements. All are related, but the policy should rank highest.

🟠 The final goal of RAG is to give the LLM the most useful context with
as little irrelevant information as possible.

**Key point:** Re-ranking improves the ordering of candidates already
retrieved.

## 10. Where Ranking Happens in the RAG Pipeline

``` text
User Query
    |
    v
Embedding Model
    |
    v
Vector Database
    |
    v
Similarity Search + Initial Ranking
    |
    v
Top-N Candidate Chunks
    |
    v
Re-Ranker (Second Ranking Pass)
    |
    v
Final Top-K Chunks
    |
    v
Prompt / Context
    |
    v
LLM
    |
    v
Answer
```

-   **Initial ranking:** The vector retriever orders candidates using
    embedding similarity.
-   **Second ranking:** The reranker scores query-document pairs and
    changes their order.
-   **Generation:** The LLM receives the final selected context.

🟠 Re-ranking belongs in the **retrieval stage**, after initial
retrieval and before context assembly and generation.

## 11. The Root Problem: Embeddings Are Lossy

🔵 An embedding model converts text into a numeric vector so a computer
can compare meaning mathematically. A long chunk may be represented by a
fixed-size vector. This is compression.

Compression is useful, but it is **lossy**: the vector does not preserve
every fine detail of the original text.

Both sides are compressed: - The document chunk is converted into a
document vector. - The user query is converted into a query vector.

The vector database compares those vectors. The similarity score is
therefore an approximation of relevance.

### What can go wrong?

1.  A less relevant chunk may get a higher similarity score.
2.  A highly relevant chunk may get a lower score because an important
    detail was not preserved strongly.
3.  If the system retrieves only a small number of chunks, the highly
    relevant chunk may be left out.

🟣 **Analogy:** A book summary helps you identify a book's topic
quickly, but it cannot preserve every detail from the original book.
Embeddings are useful compressed representations, not perfect copies.

🔴 Do not say embeddings are useless or that reranking makes retrieval
perfect. Embeddings make large-scale search practical; reranking refines
candidate ordering.

## 12. What Re-Ranking Does

🔵 Re-ranking takes the original query and each retrieved candidate
chunk, evaluates them together, and assigns a new relevance score.

It can move a genuinely useful chunk higher, push a loosely related
chunk lower, and remove low-ranked candidates when only the final top-k
are retained.

The score is a model's relevance judgment, not a guarantee of truth.

### Simple example

Initial vector search:

    Initial rank Candidate                    Initial assessment
  -------------- ---------------------------- ------------------------
               1 General API overview         Related
               2 Network security guide       Related
               3 API authentication guide     Relevant
               4 API access security policy   Most directly relevant
               5 General software security    Weakly related

After reranking:

    New rank Candidate
  ---------- ----------------------------
           1 API access security policy
           2 API authentication guide
           3 Network security guide
           4 General API overview
           5 General software security

🟢 The reranker changes the order based on the query-document
relationship.

## 13. What Re-Ranking Does Differently: Bi-Encoder vs Cross-Encoder

This is the most important distinction to understand for interviews.

### Bi-Encoder --- Initial retrieval

A bi-encoder processes the query and document separately.

``` text
Query ----------------> Query Embedding ----\
                                             \
                                              Compare vectors
                                             /
Document -------------> Document Embedding -/
```

-   Document chunks are embedded during ingestion.
-   Their vectors are stored in the vector database.
-   At query time, only the new query needs to be embedded.
-   The vector database compares the query vector with stored document
    vectors.

**Strength:** Fast and scalable.\
**Weakness:** Query and document are not processed together, so the
model does not directly examine their detailed relationship during
encoding.

### Cross-Encoder --- Re-ranking

A cross-encoder takes the query and one document chunk together.

``` text
Query + Document Chunk
          |
          v
    Cross-Encoder
          |
          v
   Relevance Score
```

-   Query and candidate chunk are processed as a pair.
-   The model can examine fine-grained interactions between query and
    document tokens.
-   It produces a relevance score for that pair.
-   Candidates are sorted by their new scores.

**Strength:** More accurate relevance estimation in the described
retrieval setup.\
**Weakness:** It must run again for every candidate and every new query,
so it is slower.

### Comparison table

  -----------------------------------------------------------------------
  Point                   Bi-Encoder              Cross-Encoder
  ----------------------- ----------------------- -----------------------
  Main job                Initial retrieval       Re-ranking

  Input                   Query and document      Query + document
                          separately              together

  What it represents      Meaning of each text    Relationship between
                                                  the two texts

  Document processing     Can be precomputed at   Repeated for each
                          ingestion               query-candidate pair

  Query-time work         Encode query and search Score each candidate
                          vectors                 pair

  Speed                   Fast                    Slower

  Typical role            Search a large          Refine a small
                          collection              shortlist
  -----------------------------------------------------------------------

🟠 **Interview line:** A bi-encoder is optimized for scalable retrieval;
a cross-encoder is used for more detailed query-document relevance
scoring.

## 14. Why Bi-Encoders Are Fast

Documents are relatively static. They can be loaded and chunked,
embedded once, stored in the vector database, and reused for many user
queries.

A new query arrives at runtime, so the system only needs to encode that
query and compare it against stored vectors.

🟢 This is why bi-encoders work well for searching very large
collections.

## 15. Why Cross-Encoders Are Slow

A cross-encoder needs both the query and candidate document together.

-   The query changes every time a user asks a new question.
-   Each candidate document must be paired with that query.
-   The model must score each pair.
-   The work cannot be fully precomputed for future queries.

If the system ran a cross-encoder against one million chunks for every
query, it would require enormous computation.

🟠 This is why cross-encoders are used after a fast retriever has
narrowed the search to a small candidate set.

## 16. Why Not Use a Cross-Encoder From the Start?

Because it is too expensive and slow to score every document in a huge
knowledge base.

The practical answer is a **two-stage retrieval pipeline**:

  Stage                   Model                    Example Goal
  ----------------------- --------------- ---------------- ------------------------
  1\. Initial retrieval   Bi-encoder        1,000,000 → 20 Fast filtering
  2\. Re-ranking          Cross-encoder             20 → 5 More accurate ordering

🟣 **Prelims and Mains analogy:** Do not conduct a detailed interview
with every applicant. First shortlist candidates quickly, then evaluate
the shortlist carefully.

## 17. Worked Example

Suppose the knowledge base has 1 million chunks and the final
requirement is 5 chunks.

### Stage 1 --- Initial retrieval

The bi-encoder searches the vector database and retrieves 20 candidate
chunks. It removes most irrelevant material, but the order is
approximate.

### Stage 2 --- Re-ranking

The cross-encoder receives the original query paired with each of the 20
candidate chunks. It scores every query-chunk pair.

### Stage 3 --- Final selection

The system sorts the 20 candidates by their new scores and returns the
best 5.

  Stage               Method            Number of chunks Speed / quality
  ------------------- --------------- ------------------ ----------------------
  Initial retrieval   Bi-encoder          1,000,000 → 20 Fast, approximate
  Re-ranking          Cross-encoder               20 → 5 Slower, more precise

🟢 **Remember: 1 million → 20 → 5.**

🔴 If the initial retriever does not retrieve a relevant chunk at all,
the reranker cannot promote it. A reranker only reorders the candidates
it receives.

## 18. How Re-Ranking Works: The Cross-Encoder

The CampusX guide explains a typical BERT-based cross-encoder at an
overview level.

### Input format

The query and document are packed into one sequence:

``` text
[CLS]  Query tokens  [SEP]  Document chunk tokens  [SEP]
```

-   `[CLS]` --- classification token at the beginning.
-   First `[SEP]` --- marks the end of the query.
-   Second `[SEP]` --- marks the end of the document chunk.

### Main components

  -----------------------------------------------------------------------
  Component                           Simple explanation
  ----------------------------------- -----------------------------------
  Embedding layer                     Converts input tokens into vectors

  Stacked encoder layers              Allow query and document tokens to
                                      interact through self-attention

  Fully connected layer               Converts the final `[CLS]`
                                      representation into a score value

  Sigmoid                             Maps that value into a
                                      probability-like score between 0
                                      and 1
  -----------------------------------------------------------------------

### Step-by-step model flow

1.  **Token embeddings:** Each token, including `[CLS]` and `[SEP]`, is
    converted into an embedding.
2.  **Bidirectional self-attention:** Query tokens can attend to
    document tokens, and document tokens can attend to query tokens.
3.  **Progressive refinement:** Encoder layers build increasingly
    detailed query-document relationship information.
4.  **Use the `[CLS]` vector:** The final `[CLS]` representation is
    passed to the scoring layer.
5.  **Produce a score:** A fully connected layer produces a value, and
    sigmoid maps it to a score between 0 and 1.
6.  **Reorder:** The system sorts candidate chunks by their scores.

### Small example

**Query:** What is photosynthesis?\
**Document:** Photosynthesis is how plants make food using sunlight.

The cross-encoder processes both together and estimates how relevant
that document is to the question.

🟠 A higher score means the model considers the query-document pair more
relevant. It is a relevance score, not necessarily a calibrated
probability that the answer is factually correct.

### Why only the `[CLS]` vector?

The `[CLS]` vector is updated through the encoder layers as it attends
to the other tokens. It becomes an aggregate representation of the
query-document relationship and is used by the scoring head.

🔴 The `[CLS]` vector is not selected simply because it is first. The
model is trained to use it as the sequence-level representation for
classification/scoring.

------------------------------------------------------------------------

# Part 3 --- Implementing Re-Ranking in a RAG Pipeline

## 19. Overall implementation idea

The CampusX guide demonstrates two rerankers:

1.  **Cohere Rerank** --- hosted/proprietary reranking API.
2.  **FlashRank** --- open-source reranking that runs locally.

Both are connected through the same LangChain component:

`ContextualCompressionRetriever`

### How the LangChain components fit together

  -----------------------------------------------------------------------
  Component                           Responsibility
  ----------------------------------- -----------------------------------
  Base retriever                      Fetch initial candidate documents
                                      from the vector store

  Base compressor                     Rerank candidates using the
                                      selected reranker

  ContextualCompressionRetriever      Connect retriever and reranker,
                                      then return selected documents
  -----------------------------------------------------------------------

``` text
User Query
    |
    v
Base Retriever (Chroma)
    |
    v
Initial Candidate Documents
    |
    v
Base Compressor (Cohere or FlashRank)
    |
    v
Query + Candidate Document Scoring
    |
    v
Reordered Documents
    |
    v
Final top_n documents
```

## 20. Core LangChain code pattern

The following pattern follows the CampusX guide. It assumes
`vectorstore` has already been created and populated.

``` python
# Cohere reranker
from langchain_cohere import CohereRerank

# FlashRank local reranker
from langchain_community.document_compressors import FlashrankRerank

# Contextual compression retriever
from langchain_classic.retrievers.contextual_compression import (
    ContextualCompressionRetriever
)

# Stage 1: normal vector-store retriever
retriever = vectorstore.as_retriever(
    search_kwargs={"k": 5}
)

# Stage 2A: choose Cohere reranker
compressor = CohereRerank(
    model="rerank-english-v3.0"
)

# OR Stage 2B: choose FlashRank (local)
# compressor = FlashrankRerank(
#     model="ms-marco-MiniLM-L-12-v2"
# )

# Stage 3: connect retriever and reranker
compression_retriever = ContextualCompressionRetriever(
    base_compressor=compressor,
    base_retriever=retriever,
)

# Stage 4: retrieve and rerank
results = compression_retriever.invoke(
    "How do LLMs handle factual errors in their output?"
)

# Results are returned in reranked order
for doc in results:
    print(doc.page_content)
```

🟠 In the source example, the base retriever fetches 5 candidates and
the reranker returns the default top 3. Set the compressor's `top_n`
according to the installed integration version when you need a different
final count.

🔴 **Important:** This is a source-based integration pattern, not a
guarantee that every import path or model identifier will remain
unchanged across LangChain package versions. Check the installed package
documentation before running it.

## 21. Cohere vs FlashRank

  -----------------------------------------------------------------------
  Point                   Cohere Rerank           FlashRank
  ----------------------- ----------------------- -----------------------
  Type                    Hosted proprietary API  Open-source local model
                                                  wrapper

  Execution               Remote API              Runs on your machine

  API key                 Required                Not required

  Cost                    Trial/paid plan         No reranking API charge

  Accuracy                Described in the guide  Described as lower than
                          as higher               Cohere

  Best fit in the guide   Production settings     Small-scale and local
                                                  use
  -----------------------------------------------------------------------

🟠 The guide's comparison reflects the course material and its example
setup. Current pricing, model availability, API limits, and performance
can change.

## 22. Understand `k` vs `top_n`

-   `k` = how many candidate documents the base retriever fetches.
-   `top_n` = how many documents the reranker returns after scoring and
    sorting.

Example:

``` python
retriever = vectorstore.as_retriever(
    search_kwargs={"k": 20}
)
```

Conceptually, the reranker then keeps the final 5:

``` text
k = 20       -> retrieve 20 candidates
top_n = 5    -> return 5 reranked documents
```

🟠 A larger candidate pool gives the reranker more opportunities to find
a relevant chunk that was initially ranked lower.

🔴 If `k` equals `top_n`, the reranker can change the order, but it
cannot reduce the number of returned documents.

## 23. What the CampusX implementation demonstrates

The guide describes a demo setup with: - 30 short documents across
machine learning, generative AI, and cloud. - Recursive character
splitting with chunk size 500 and overlap 100. - OpenAI embeddings. -
Chroma vector store. - Similarity retriever returning 5 candidates. -
Reranker returning 3 documents.

### Example result from the guide

Query:

> How do LLMs handle factual errors in their output?

The initial retriever placed the reinforcement-learning chunk at rank 4.
After Cohere reranking, it moved to rank 2, while a weakly related
regularization chunk was dropped from the final top 3.

🟢 This demonstrates the main value of reranking: the most useful
candidate can move up even when the initial vector search did not rank
it highly.

## 24. Golden Rules

-   🔵 **Bi-encoders retrieve fast; cross-encoders judge more
    carefully.**
-   🟠 Retrieve more candidates than you finally need, then rerank down.
-   🔴 Do not run a cross-encoder against the entire knowledge base.
-   🟠 Keep the candidate set small enough to control latency and cost.
-   🔵 Embedding similarity is approximate; it can misorder relevant
    chunks.
-   🟢 Choose hosted or local reranking based on accuracy, latency,
    cost, privacy, and deployment requirements.
-   🔴 A reranker cannot recover a relevant chunk that the initial
    retriever did not include.

------------------------------------------------------------------------

# Part 4 --- Interview Revision

## 25. Common interview questions and short answers

### Q1. What is reranking in RAG?

Reranking is a second-stage retrieval process that scores and reorders
candidate documents returned by the initial retriever, so the most
relevant chunks reach the LLM.

### Q2. Why is reranking needed if we already use vector search?

Vector search uses compressed embeddings and gives approximate
similarity scores. It can rank broadly related chunks above chunks that
answer the exact question. Reranking evaluates query and document
together to refine the order.

### Q3. What is the difference between a bi-encoder and a cross-encoder?

A bi-encoder encodes query and document separately, making large-scale
retrieval fast. A cross-encoder processes the query and candidate
document together, giving a more detailed relevance score but requiring
more computation per candidate.

### Q4. Why don't we use cross-encoders for the whole vector database?

They must score each query-document pair at runtime. Scoring millions of
chunks for every query would be too slow and expensive. We use a
bi-encoder to shortlist candidates first.

### Q5. Does reranking retrieve new documents?

In the standard two-stage flow described here, no. It reorders the
candidate documents returned by the initial retriever. If the relevant
chunk is absent from that candidate set, reranking cannot promote it.

### Q6. What is `ContextualCompressionRetriever`?

In the demonstrated LangChain setup, it combines a base retriever with a
base compressor. The retriever fetches candidates, and the reranker
scores and selects the most relevant documents.

### Q7. What is the difference between `k` and `top_n`?

`k` controls the number of initial candidates retrieved. `top_n`
controls the number retained after reranking.

### Q8. Name two rerankers demonstrated in the CampusX guide.

Cohere Rerank and FlashRank.

### Q9. Does a higher reranker score guarantee factual correctness?

No. It indicates the model's estimate of query-document relevance. The
wider RAG system still needs to ground the answer in reliable evidence.

### Q10. What is the biggest trade-off in reranking?

Improved relevance ordering versus extra inference time, latency, and
potentially API cost.

------------------------------------------------------------------------

# One-Page Final Revision

  -----------------------------------------------------------------------
  Question                            Remember this
  ----------------------------------- -----------------------------------
  Why rerank?                         Initial vector similarity is
                                      approximate

  First-stage model?                  Bi-encoder

  Second-stage model?                 Cross-encoder

  Bi-encoder input?                   Query and document separately

  Cross-encoder input?                Query + document together

  Why bi-encoder is fast?             Document vectors are precomputed

  Why cross-encoder is slow?          Every candidate pair is scored at
                                      query time

  Where does reranking happen?        After retrieval, before LLM context

  Example flow?                       1 million → 20 → 5

  LangChain component?                `ContextualCompressionRetriever`

  Course examples?                    Cohere Rerank and FlashRank

  `k` means?                          Initial candidate count

  `top_n` means?                      Final reranked count

  Main limitation?                    Cannot recover candidates never
                                      retrieved
  -----------------------------------------------------------------------

## The 20-second memory answer

> **Vector search finds candidates quickly. Reranking checks which
> candidates best match the exact question.** A bi-encoder retrieves a
> shortlist using separately encoded query and document vectors. A
> cross-encoder then reads each query-document pair together, assigns a
> relevance score, and reorders the shortlist. The RAG pipeline sends
> only the final top-ranked chunks to the LLM.

------------------------------------------------------------------------






