<img width="981" height="702" alt="image" src="https://github.com/user-attachments/assets/029e06bc-ee9a-40bf-932e-b0e78773f10a" />

<img width="1105" height="618" alt="image" src="https://github.com/user-attachments/assets/4ac2b625-f174-49fb-8c65-8c3078fdbc30" />

https://www.youtube.com/watch?v=CC2fQhBQrTM


# RAG RETRIEVAL

```text
RAG RETRIEVAL
|
+-- BEFORE SEARCH (Improve the query)
|     |
|     +-- Multi-Query
|     +-- Query Rewriting
|     +-- Self-Query
|
+-- DURING SEARCH
|     |
|     +-- A. RETRIEVAL MECHANISMS
|     |      |
|     |      +-- Dense Retrieval (Semantic)
|     |      +-- Sparse Retrieval (Keyword-Based / BM25)
|     |      +-- Hybrid Retrieval (Dense + Sparse)
|     |
|     +-- B. RESULT SELECTION
|     |      |
|     |      +-- Top-k Similarity Search
|     |      +-- Score Threshold
|     |      +-- MMR (Maximal Marginal Relevance)
|     |
|     +-- C. RESULT FUSION
|            |
|            +-- Ensemble Retriever
|                   |
|                   +-- Reciprocal Rank Fusion (RRF)
|                   +-- Custom / Hand-Built Fusion
|
+-- INDEX STRUCTURE
|     |
|     +-- Parent Document Retrieval
|            |
|            +-- Search using Child Chunks
|            +-- Return Parent Documents
|
+-- AFTER SEARCH (Clean and Improve Results)
      |
      +-- Contextual Compression
      +-- Reranking
```



#### belkow is good
https://github.com/KowshiqKatta/Generative-AI/tree/main/RAG     
https://mayankchugh-learning.github.io/pages/ai-learning/retriever-guide.html


# RAG Retrievers: A Complete Guide to Document Retrieval in Retrieval-Augmented Generation

## 📋 Table of Contents

1. [Repository Overview](#-repository-overview)
2. [What is a Retriever?](#-what-is-a-retriever)
3. [Retriever Types & Concepts](#-retriever-types--concepts)
4. [Project Structure](#-project-structure)
5. [Technology Stack](#-technology-stack)
6. [Detailed Notebook Guide](#-detailed-notebook-guide)
7. [Key Takeaways](#-key-takeaways)
8. [Interview Questions](#-interview-questions)

---

## 🎯 Repository Overview

This repository is a **complete educational blueprint** for understanding and implementing document retrievers in Retrieval-Augmented Generation (RAG) systems. It demonstrates six progressive retriever strategies—from basic similarity search to advanced hybrid ensemble approaches—with hands-on Jupyter notebooks and working code examples.

### 🎓 Who Should Use This?
- **Beginners in RAG/NLP**: Learn retriever fundamentals from scratch
- **ML Engineers**: Understand different retrieval strategies and their trade-offs
- **Developers**: Implement production-ready retriever systems
- **Researchers**: Compare semantic vs. keyword-based approaches

### 💡 Central Goal
To provide a **crystal-clear roadmap** showing **why** different retriever strategies exist, **how** they work, and **when** to use each one. This repo proves that effective RAG depends as much on smart document retrieval as it does on language models.

---

## 🧠 What is a Retriever?

### The Problem
When an LLM needs to answer a question, it only "knows" what's in its training data. If you want it to answer questions about your proprietary documents, you need to:
1. Store your documents somewhere searchable
2. Find the most relevant documents for the user's question
3. Feed those documents to the LLM as context

A **retriever** is the component that solves step 2—finding relevant documents quickly and accurately.

### The Basic Flow
```
User Query → [RETRIEVER] → Retrieved Documents → LLM → Answer
```

### Why It Matters
- **Quality of retrieval directly impacts answer quality** — If the retriever misses relevant documents, the LLM can't use them
- **Speed matters** — Searching through millions of documents must be fast
- **Different queries need different strategies** — Keyword searches work differently than semantic searches

---

## 🔍 Retriever Types & Concepts

### 1. **Semantic (Dense) Retrievers**
**Concept**: Convert queries and documents into dense vectors (embeddings), then measure similarity.

**How it works**:
- Documents and queries are embedded into fixed-size vectors (e.g., 1536 dimensions)
- Similarity is measured using cosine similarity, L2 distance, or inner product
- Returns documents semantically closest to the query

**Pros**:
- ✅ Understands meaning, not just keywords
- ✅ Works across synonyms ("vehicle" matches "car")
- ✅ Works with vague or conceptual queries

**Cons**:
- ❌ Requires embedding model (computational cost)
- ❌ May miss exact keyword matches
- ❌ Embedding quality depends on training data

**Example Query**: *"How do cells generate energy?"* → Finds documents about mitochondria, ATP, respiration even if they don't contain "energy"

---

### 2. **Keyword-Based (Sparse) Retrievers (BM25)**
**Concept**: Match query terms against document terms using TF-IDF variants.

**How it works**:
- Builds an inverted index of terms → documents
- Scores documents based on term frequency and inverse document frequency
- No embeddings needed—pure token matching

**Pros**:
- ✅ Fast and lightweight (no embedding model)
- ✅ Deterministic—same query always returns same results
- ✅ Excellent for exact keyword matches
- ✅ Works well for domain-specific terminology

**Cons**:
- ❌ No semantic understanding
- ❌ Fails on paraphrased queries
- ❌ Struggles with short documents

**Example Query**: *"antibiotic bacterial infection"* → Finds documents containing these exact or similar terms

---

### 3. **Similarity Score Threshold**
**Concept**: Filter semantic search results by a minimum similarity score.

**Why it matters**:
- Prevents low-quality results from being returned
- Trade-off: strict threshold may return no results, loose threshold may include noise
- Useful when you need **precision over recall**

**Use case**: Medical information retrieval where low-confidence matches are risky

---

### 4. **Maximal Marginal Relevance (MMR)**
**Concept**: Balance relevance (semantic similarity) with diversity (dissimilarity between results).

**The Problem It Solves**: 
Imagine searching for "deep learning training" and getting three near-identical documents about gradient descent. They're all relevant but redundant. MMR penalizes similar results and picks diverse ones.

**How it works**:
- Fetches top-k candidate documents by similarity
- Iteratively selects documents that are:
  - **Relevant** to the query (high similarity)
  - **Diverse** from already-selected documents (low similarity to each other)
- Controlled by `lambda_mult` parameter:
  - `lambda_mult=1.0`: Pure relevance (like similarity search)
  - `lambda_mult=0.5`: Balanced relevance + diversity
  - `lambda_mult=0.0`: Pure diversity

**Result**: Returned documents cover different aspects of the topic instead of repeating the same information.

---

### 5. **Hybrid Search (Semantic + Keyword)**
**Concept**: Combine dense and sparse retrievers to get the best of both worlds.

**Why it works**:
- **Dense search** finds semantically related documents (even with paraphrased queries)
- **Sparse search** finds exact keyword matches (even if semantically unrelated)
- Together they cover more ground

**Example**:
Query: *"How do vaccines work?"*
- **BM25 alone**: Returns documents containing "vaccine" (docs 1-2)
- **Semantic alone**: Returns documents about immune system (docs 3-5)
- **Hybrid**: Returns both sets, providing comprehensive coverage

**Merging Strategy**: Reciprocal Rank Fusion (RRF)
- Combines rankings from multiple retrievers
- Formula: `score(doc) = sum(weight_i / (rank_i + k))`
- Weights allow prioritizing one retriever over another
- `k` (typically 60) smooths the contribution of lower-ranked results

---

### 6. **Custom Ensemble Retriever**
**Concept**: Implement your own retriever logic by extending `BaseRetriever`.

**Why it matters**:
- You can implement custom scoring logic
- Combine retrievers in custom ways
- Add business logic (e.g., document recency, source reliability)
- Educational value—understand exactly how fusion works

**Key Implementation Details**:
- Inherit from `langchain_core.retrievers.BaseRetriever`
- Implement `_get_relevant_documents()` method
- Manually implement RRF or other fusion algorithms
- Fully customizable and transparent

---

## 📁 Project Structure

```
Retrievers/
├── main.py                          # Entry point (placeholder)
├── pyproject.toml                   # Project metadata and dependencies
├── requirements.txt                 # Python package dependencies
├── README.md                        # This file
└── notebooks/
    ├── 01_similarity_search.ipynb              # Baseline: semantic search
    ├── 02_similarity_score_threshold.ipynb     # Add confidence filtering
    ├── 03_mmr.ipynb                           # Diversity-aware ranking
    ├── 04_bm25.ipynb                          # Keyword-based retrieval
    ├── 05_hybrid_search.ipynb                 # Combine semantic + keyword
    └── 06_custom_ensemble_retriever.ipynb     # Custom RRF implementation
```

---

## 🛠 Technology Stack

### Core Libraries

| Library | Version | Purpose |
|---------|---------|---------|
| **LangChain** | ≥1.2.12 | Retriever interfaces, composition, and abstractions |
| **LangChain-Community** | ≥0.4.1 | BM25Retriever and other community retrievers |
| **LangChain-OpenAI** | ≥1.1.11 | OpenAI embedding model integration |
| **LangChain-Chroma** | ≥1.1.0 | ChromaDB vector store integration |
| **ChromaDB** | ≥1.5.5 | Vector database for storing and searching embeddings |
| **rank-bm25** | ≥0.2.2 | BM25 algorithm implementation |
| **python-dotenv** | ≥1.2.2 | Load API keys from .env files |
| **ipykernel** | ≥7.2.0 | Jupyter kernel for running notebooks |

### Embedding Model
- **text-embedding-3-small** (OpenAI): 1536-dimensional embeddings, fast and efficient

### Vector Store
- **ChromaDB**: In-memory vector database with HNSW indexing, cosine similarity support

### Environment
- **Python**: 3.12+
- **Jupyter**: For interactive notebook execution

---

## 📚 Detailed Notebook Guide

### **Notebook 1: Similarity Search (01_similarity_search.ipynb)**

**What You'll Learn**:
- How to create vector embeddings using OpenAI
- Store documents in a vector store (ChromaDB)
- Perform semantic similarity search
- Use retrievers in LangChain chains

**Key Concepts**:
- **Vector embeddings**: Dense representations capturing document meaning
- **Cosine similarity**: Dot product divided by vector magnitudes—ranges from -1 to 1
- **Retriever interface**: LangChain abstraction for composable retrieval

**Code Walkthrough**:
1. Load environment variables (OpenAI API key)
2. Create 12 sample documents across topics: space, biology, history, geography
3. Initialize OpenAI embeddings model
4. Store documents in ChromaDB vector store
5. Create a retriever with `search_type="similarity"` and `k=2`
6. Execute queries and retrieve top-2 most similar documents

**Expected Results**:
```
Query: "How do rockets work?"
Result 1: "Rockets work by expelling gas..."  [topic: space]
Result 2: "Spacecraft use gravitational slingshots..."  [topic: space]
```

**Why This Matters**: Foundation for all semantic search—understanding how meaning is captured and searched.

---

### **Notebook 2: Similarity Score Threshold (02_similarity_score_threshold.ipynb)**

**What You'll Learn**:
- Filter results by confidence threshold
- Understand similarity score distribution
- Adjust precision vs. recall trade-off

**Key Concepts**:
- **Similarity score**: Distance between query and document embeddings (0-1 range)
- **Precision**: Of results returned, how many are relevant
- **Recall**: Of all relevant documents, how many did we find
- **Threshold tuning**: Higher threshold = higher precision, lower recall

**Code Walkthrough**:
1. Create 12 documents: 5 ML docs (highly relevant), 3 economics, 2 cooking, 2 sports (off-topic)
2. Query: "How does ML model training work?"
3. Retrieve top-3 results with similarity scores
4. Convert scores to actual cosine similarity: `actual_similarity = 1 - score`
5. Apply threshold filter (e.g., threshold=0.43)
6. Only documents exceeding threshold are returned

**Expected Results**:
```
Threshold=0.43 → Returns 3 ML documents
Threshold=0.50 → Returns 1-2 top ML documents
Threshold=0.60 → Likely returns 0 documents (too strict)
```

**Practical Use**: Use in production when you need high confidence answers—better to return nothing than wrong information.

---

### **Notebook 3: Maximal Marginal Relevance (03_mmr.ipynb)**

**What You'll Learn**:
- Problem with redundant results from similarity search
- How MMR balances relevance and diversity
- Tuning the relevance-diversity trade-off

**Key Concepts**:
- **Redundancy problem**: Semantic search returns similar documents
- **Diversity**: Covering different aspects of a topic
- **lambda_mult**: Parameter controlling trade-off (0 = pure diversity, 1 = pure relevance)
- **fetch_k vs. k**: Fetch more candidates before re-ranking

**Code Walkthrough**:
1. Create documents with intentional redundancy:
   - 3 near-identical gradient descent docs (deep learning)
   - 3 diverse deep learning concepts (dropout, batch norm, learning rate scheduler)
   - Off-topic docs (climate, art, law)
2. Query: "deep learning model training and optimization"
3. Compare similarity search vs. MMR with different lambda values:
   - lambda_mult=1.0: Returns 3 redundant gradient descent docs
   - lambda_mult=0.5: Returns gradient descent + dropout + batch norm (diverse!)
   - lambda_mult=0.0: Returns most dissimilar docs (loses relevance)

**Expected Results**:
```
λ=1.0 (relevance): [GD doc 1, GD doc 2, GD doc 3] — Redundant
λ=0.5 (balanced): [GD doc 1, Dropout, Batch Norm] — Diverse yet relevant
λ=0.0 (diversity): [Random diverse docs] — No relevance
```

**When to Use**: When you want coverage of multiple aspects (e.g., "Tell me about COVID-19" should cover vaccines, symptoms, variants, etc.)

---

### **Notebook 4: BM25 Keyword-Based Retrieval (04_bm25.ipynb)**

**What You'll Learn**:
- Keyword-based retrieval without embeddings
- When semantic search fails vs. when keyword search excels
- TF-IDF and BM25 algorithms

**Key Concepts**:
- **BM25 (Best Matching 25)**: Probabilistic ranking function based on term frequency
- **TF (Term Frequency)**: How often a term appears in a document
- **IDF (Inverse Document Frequency)**: How rare the term is across all documents
- **Sparse retrieval**: No dense vectors—only term indices

**Code Walkthrough**:
1. Create 12 documents: medicine, architecture, finance, literature
2. Create BM25 retriever from raw documents (no embeddings)
3. Test three queries:
   - **Query 1 (keyword match)**: "antibiotic bacterial infection treatment" 
     - BM25 ✅ Finds medicine docs with exact terms
   - **Query 2 (keyword match)**: "compound interest returns stakes in company"
     - BM25 ✅ Finds finance docs with matching terms
   - **Query 3 (semantic)**: "a structure that feels light and with windows"
     - BM25 ❌ Misses Gothic cathedral doc (no keyword overlap)

**Expected Results**:
```
Query 1 ✅ BM25: Returns antibiotic & vaccine docs
Query 2 ✅ BM25: Returns compound interest & stock docs
Query 3 ❌ BM25: Might return architecture docs but not the right one
```

**Key Insight**: BM25 is great for domains with specific terminology (medical, legal, technical) but struggles with natural language variations.

---

### **Notebook 5: Hybrid Search / Ensemble Retrieval (05_hybrid_search.ipynb)**

**What You'll Learn**:
- Combine semantic and keyword retrievers
- Use EnsembleRetriever with Reciprocal Rank Fusion
- Leverage strengths of both approaches
- Weight different retrievers

**Key Concepts**:
- **Reciprocal Rank Fusion (RRF)**: Combines rankings from multiple systems
  - RRF formula: `score = Σ(weight_i × 1/(rank_i + k))`
  - `k=60` smoothing constant prevents rank-1 bias
  - Weights allow prioritizing certain retrievers
- **Hybrid retrieval**: Semantic + keyword = comprehensive coverage

**Code Walkthrough**:
1. Create 12 documents: health (vaccine docs), programming, history, nature
2. Initialize two retrievers:
   - **Dense**: ChromaDB semantic search (k=4)
   - **Sparse**: BM25Plus (k=2, bm25_variant="plus")
3. Create EnsembleRetriever with weights [0.8, 0.2] (80% semantic, 20% keyword)
4. Query: "How do vaccines work to protect against diseases?"

**Retriever Comparison**:
```
BM25 Results:
  [1] "Vaccines work by introducing weakened pathogen..." (exact match)
  [2] "The flu vaccine is reformulated each year..." (exact match)

ChromaDB Results:
  [1] "The immune system produces antibodies..." (semantic match)
  [2] "Herd immunity occurs when..." (semantic match)
  [3] "B-lymphocytes produce proteins..." (semantic match)
  [4] "Vaccines work by introducing..." (semantic match)

Ensemble Results (RRF):
  [1] "Vaccines work..." (rank 1 from both)
  [2] "The immune system produces antibodies..." (from semantic)
  [3] "Herd immunity occurs..." (from semantic)
  [4] "B-lymphocytes produce proteins..." (from semantic)
```

**Practical Benefit**: Ensemble catches documents from both strategies—exact keyword matches AND semantically similar documents.

---

### **Notebook 6: Custom Ensemble Retriever Implementation (06_custom_ensemble_retriever.ipynb)**

**What You'll Learn**:
- Extend LangChain's BaseRetriever class
- Implement Reciprocal Rank Fusion algorithm manually
- Build production-ready custom retrievers
- Understand RRF scoring in detail

**Key Concepts**:
- **BaseRetriever**: Abstract class for all LangChain retrievers
- **_get_relevant_documents()**: Core method to implement
- **RRF scoring**: Manual implementation showing exact calculations
- **Document deduplication**: Handle same document from multiple retrievers

**Code Walkthrough**:

1. **Custom Retriever Class**: `MyEnsembleRetriever`
   ```python
   class MyEnsembleRetriever(BaseRetriever):
       retrievers: List[BaseRetriever]
       weights: List[float]
       rrf_k: int = 60
   ```

2. **Core Algorithm**:
   ```python
   doc_scores = {}
   for retriever_idx, results in enumerate(all_results):
       weight = weights[retriever_idx]
       for rank, doc in enumerate(results):
           # RRF formula: weight × 1/(rank + k)
           rrf_score = weight * (1.0 / (rank + rrf_k))
           # Accumulate scores for duplicate docs
           if doc in doc_scores:
               doc_scores[doc] += rrf_score
           else:
               doc_scores[doc] = rrf_score
   ```

3. **Return sorted results**:
   ```python
   sorted_docs = sorted(doc_scores.items(), 
                       key=lambda x: x[1], 
                       reverse=True)
   ```

**Why Implement It Yourself?**:
- ✅ Understand the algorithm deeply
- ✅ Add custom scoring logic
- ✅ Debug specific retrieval issues
- ✅ Implement domain-specific fusion strategies

**Results**: Same as notebook 5, but with full transparency of how scoring works.

---

## 🎓 Key Takeaways

### 1. **Retrieval Quality Directly Impacts LLM Answers**
The retriever is the foundation. A perfect LLM with poor retrieval = poor answers.

### 2. **No One-Size-Fits-All Solution**
- **Semantic search**: Paraphrased queries, conceptual matching
- **Keyword search**: Specific terminology, domain-specific docs
- **Hybrid**: Coverage of both patterns

### 3. **Trade-offs Are Central**
| Aspect | Semantic | Keyword | Hybrid |
|--------|----------|---------|--------|
| Speed | ⚠️ Slower | ✅ Fast | ⚠️ Moderate |
| Semantic understanding | ✅ Excellent | ❌ Poor | ✅ Good |
| Exact matches | ⚠️ Approximate | ✅ Perfect | ✅ Perfect |
| Embedding cost | ❌ Required | ✅ None | ⚠️ Required |
| Result diversity | ❌ Redundant | ⚠️ Limited | ✅ Diverse (with MMR) |

### 4. **Diversity Matters**
Different documents covering different aspects > Multiple redundant documents

### 5. **LangChain Abstractions Enable Composition**
Retrievers are composable building blocks—stack, weight, and combine them flexibly.

### 6. **Always Benchmark with Your Data**
Optimal retriever choice depends on:
- Document length and style
- Query type (keyword vs. conceptual)
- Domain (medical, legal, technical requires keywords; creative/conceptual requires semantic)
- Latency requirements

### 7. **Monitoring & Thresholds Are Production-Ready**
- Monitor retrieval accuracy (are retrieved docs actually relevant?)
- Use thresholds for high-stakes applications
- Adjust weights in ensemble based on performance

---

## 🧑‍💼 Probable Interview Questions

### **Beginner Level**

**Q1: What is a retriever and why do we need it in RAG?**
**A**: A retriever finds relevant documents from a knowledge base to provide context to an LLM. Without a good retriever, the LLM can only use what's in its training data—retrievers let it answer questions about new, proprietary, or real-time documents.

**Q2: What's the difference between semantic and keyword-based retrieval?**
**A**: 
- Semantic: Understands meaning, finds paraphrased matches, but slower and requires embeddings
- Keyword: Fast, deterministic, perfect for exact matches, but can't understand synonyms or concepts

**Q3: When would you use BM25 instead of embeddings?**
**A**: When you have domain-specific terminology where exact keywords matter (medical, legal), when latency is critical and you can't embed documents, or when you have very short documents where embeddings are unreliable.

---

### **Intermediate Level**

**Q4: Explain Maximal Marginal Relevance (MMR) and give an example.**
**A**: MMR balances relevance (similarity to query) with diversity (dissimilarity between results). Example: Searching "COVID-19" should return docs about vaccines AND symptoms AND variants—not three docs all about vaccines. The `lambda_mult` parameter tunes this trade-off.

**Q5: What's Reciprocal Rank Fusion and how does it work?**
**A**: RRF merges rankings from multiple retrievers by scoring each document based on its rank in each retriever: `score = Σ(weight_i / (rank_i + k))`. The `k` constant (usually 60) ensures lower-ranked results still contribute. It's order-aware—a document ranked #1 in one retriever gets credit even if completely absent from another.

**Q6: Why would you use a hybrid (semantic + keyword) retriever?**
**A**: Because they solve different problems:
- Semantic finds conceptually similar but differently-worded docs
- Keyword finds exact terminology matches
Together they maximize recall. You pay 2x retrieval cost but get more comprehensive results.

---

### **Advanced Level**

**Q7: How would you implement a custom retriever that boosts recent documents or from trusted sources?**
**A**: Extend `BaseRetriever` and implement `_get_relevant_documents()`. After getting results from base retrievers:
1. Apply semantic/BM25 ranking
2. Apply custom scoring: `score *= (recency_boost × source_credibility_score)`
3. Re-sort by adjusted scores
4. Return top-k

This lets you inject domain-specific logic into retrieval.

**Q8: Compare the three retrieval strategies at scale (millions of documents).**
**A**:
- **Semantic**: O(1) lookup via HNSW indexing, but embedding millions of docs is expensive upfront. Query latency: ~100ms
- **Keyword**: O(log n) inverted index lookup, extremely fast. Query latency: ~10ms
- **Hybrid**: Runs both, then merges—~110ms but comprehensive

For millions of docs, hybrid with optimized weights is often best.

**Q9: What happens if your embedding model and documents were trained on different distributions?**
**A**: Embedding quality suffers. Example: If embeddings trained on news but documents are medical, semantic similarity is meaningless. Solutions:
1. Fine-tune embeddings on your domain
2. Use keyword retrieval for domain-specific terms
3. Use hybrid retrieval to compensate
4. Validate with human judges

**Q10: How do you evaluate retrieval quality?**
**A**: Metrics include:
- **Recall@k**: Of truly relevant docs, what % appear in top-k results?
- **Precision@k**: Of returned top-k, what % are actually relevant?
- **MRR (Mean Reciprocal Rank)**: Average rank of first relevant document
- **NDCG (Normalized Discounted Cumulative Gain)**: Ranking quality considering position
- **Human evaluation**: Most reliable but expensive

---

### **Scenario-Based Questions**

**Q11: You're building a RAG system for a law firm. What retriever strategy would you use?**
**A**: **Primarily BM25 with semantic as backup**. Reasoning:
- Legal language is precise; keyword matches are critical ("statute of limitations" must match exactly)
- Domain-specific terminology matters more than semantic similarity
- Speed is important (lawyers work under time pressure)
- Hybrid as fallback: if BM25 returns few results, semantic search provides options

**Q12: A user complains that your RAG system returns three very similar documents instead of covering different angles on their question.**
**A**: **Use MMR instead of pure similarity search**. Set `lambda_mult=0.6` to balance relevance with diversity. This tells the system: "Give me relevant results, but avoid redundancy." The ranking will pick diverse documents.

**Q13: Your retriever works perfectly on small test data but fails on production data (1M documents). What could be wrong?**
**A**: Common issues:
1. **Embedding quality at scale**: Dense index (HNSW) may have insufficient node connections—adjust `efConstruction` parameter
2. **Memory/latency**: Retrieving semantically on 1M docs takes time—add caching or use hybrid (BM25 is fast)
3. **Data drift**: Test docs are clean; production docs are messier—add preprocessing
4. **Embedding model limitations**: "text-embedding-3-small" may struggle on your domain—fine-tune or use larger model

---

## 🚀 How to Use This Repository

### Setup
```bash
# Install dependencies
pip install -r requirements.txt

# Create .env file with your OpenAI API key
echo "OPENAI_API_KEY=your_key_here" > .env
```

### Run Notebooks
```bash
# Start Jupyter
jupyter notebook

# Open notebooks in order: 01 → 06
# Each builds on concepts from previous notebooks
```

### Experiment
- Modify queries to see how retrievers respond
- Adjust parameters (`k`, `lambda_mult`, threshold, weights)
- Add your own documents and test
- Compare retriever outputs

---

## 📖 Further Reading

- [LangChain Retrievers Documentation](https://python.langchain.com/docs/modules/data_connection/retrievers/)
- [Reciprocal Rank Fusion Paper](https://dl.acm.org/doi/10.1145/1571941.1572114)
- [Maximal Marginal Relevance](https://arxiv.org/abs/2107.03810)
- [BM25 Algorithm](https://en.wikipedia.org/wiki/Okapi_BM25)
- [Vector Embeddings & Similarity](https://huggingface.co/course/en/chapter5/1)

---

## 📝 Summary Sheet (Quick Reference)

| Retriever | When to Use | Speed | Cost | Strength |
|-----------|------------|-------|------|----------|
| **Similarity Search** | General semantic matching | ⚠️ Moderate | ⚠️ Embedding cost | Semantic understanding |
| **Similarity + Threshold** | High-confidence retrieval | ⚠️ Moderate | ⚠️ Embedding cost | Precision-focused |
| **MMR** | Diverse results wanted | ⚠️ Moderate | ⚠️ Embedding cost | Coverage of topics |
| **BM25** | Exact keywords matter | ✅ Fast | ✅ Free | Speed + determinism |
| **Hybrid** | Comprehensive coverage | ⚠️ Slower | ⚠️ 2x cost | Best of both |
| **Custom Ensemble** | Domain-specific logic | ⚠️ Custom | ⚠️ Custom | Flexibility |

---

**Created**: 2026
**Purpose**: Educational blueprint for understanding document retrievers in RAG systems  
**Audience**: Beginners to advanced practitioners in AI/ML  

*This README is a complete reference guide. Start with notebook 01, progress sequentially, and refer back to this guide for concepts.*


# 🟡📌 RAG Retrievers & Advanced Retrieval — Taxonomy-Based Study Notes

## 🟡📌 Source and scope

These revised notes are based on the five uploaded Python notebooks:

1. `02_similarity_score_threshold.ipynb`
2. `03_mmr.ipynb`
3. `04_bm25.ipynb`
4. `05_hybrid_search.ipynb`
5. `06_custom_ensemble_retriever.ipynb`

The guide is reorganized by **retriever taxonomy** rather than by notebook order. The code examples and demonstrated behaviours are grounded in those notebooks. Additional taxonomy concepts that are not implemented in the uploaded notebooks are clearly marked as **supplementary concepts**.

## 🟡📌 🎨 Reading guide

- 🔵📌 **Blue** — core retrieval mechanisms
- 🟢📌 **Green** — result selection and ranking
- 🟣📌 **Purple** — retriever combination and fusion
- 🟠📌 **Orange** — query-oriented advanced retrieval
- 🔴📌 **Red** — hierarchical and context-aware retrieval
- 🟡📌 **Yellow** — comparisons, practical guidance, and revision

---

> **Scope warning:** These five notebooks demonstrate similarity-score threshold retrieval, MMR, BM25, hybrid dense+sparse retrieval, built-in ensemble retrieval, and a custom RRF ensemble. They do not implement Multi-Query, Query Rewriting, Self-Query, Parent Document, or Contextual Compression. Those topics are included later only to help you understand where they fit in a broader RAG taxonomy.

---

# 🔵📌 1. Big picture: what does a retriever do?

Imagine a company has 100,000 documents. A user asks:

> “How do vaccines protect people from diseases?”

The LLM should not read all 100,000 documents. A **retriever** searches the knowledge base and returns a smaller set of potentially useful documents. The RAG pipeline passes those documents to the LLM as context.

```text
User question
     |
     v
Retriever searches the knowledge base
     |
     v
Relevant documents
     |
     v
LLM receives question + retrieved context
     |
     v
Answer
```

A retriever's job is to find useful evidence. It does not write the final answer.

## 🔵📌 1.1 A practical conceptual taxonomy

There is no single universally mandated taxonomy used by every framework or textbook. The following is a useful way to organize the concepts by **what each technique does**.

```text
RAG RETRIEVAL
|
+-- A. Retrieval mechanisms: HOW documents are found
|   +-- Dense / semantic retrieval
|   +-- Sparse / lexical retrieval (BM25)
|   +-- Hybrid retrieval (dense + sparse)
|
+-- B. Candidate selection and ranking: WHICH results are kept
|   +-- Similarity search / top-k
|   +-- Similarity-score threshold
|   +-- MMR (relevance + diversity)
|
+-- C. Retriever combination and fusion: HOW result lists are combined
|   +-- Ensemble retriever
|   +-- Reciprocal Rank Fusion (RRF)
|   +-- Custom ensemble retriever
|
+-- D. Query-oriented advanced retrieval: HOW THE QUERY is transformed
|   +-- Multi-Query
|   +-- Query rewriting / expansion
|   +-- Self-Query
|
+-- E. Hierarchical and context-aware retrieval
    +-- Parent Document retrieval
    +-- Child-to-parent retrieval
    +-- Contextual compression
```

**Important distinction:** A–C are the areas directly demonstrated by the five uploaded notebooks. D–E are supplementary concepts, not implementations found in those files. Contextual compression is often better described as a post-retrieval processing step than as a separate first-stage search algorithm.

## 🔵📌 1.2 Do not mix up these three questions

| Question | Relevant family | Examples |
|---|---|---|
| How do we find candidate documents? | Retrieval mechanism | Dense, BM25, Hybrid |
| Which candidates should we keep or prefer? | Selection / ranking | Top-k, Threshold, MMR |
| How do we combine results from multiple searches? | Fusion | Ensemble, RRF |

This simple separation prevents most taxonomy confusion.

---

# 🔵📌 2. Category A — Retrieval mechanisms: how documents are found

## 🔵📌 2.1 Dense / semantic retrieval

**Meaning:** Convert the query and documents into dense embedding vectors, then retrieve documents whose vectors are close to the query vector.

### 🟡📌 Story example

A document says:

> “The immune system produces antibodies that neutralise pathogens.”

The user asks:

> “How do vaccines help the body fight illness?”

The exact wording is different, but the meaning is related. Dense retrieval can find the document because the embeddings represent semantic relationships, not just exact word overlap.

### 🟡📌 Basic flow

```text
Documents -----------------> Embedding model
                                  |
                                  v
                           Document vectors
                                  |
                                  v
                             Vector store

User query ----------------> Embedding model
                                  |
                                  v
                              Query vector
                                  |
                                  v
                     Vector similarity search
                                  |
                                  v
                         Top matching documents
```

### 🟡📌 Code pattern used in the notebooks

```python
from langchain_openai import OpenAIEmbeddings
from langchain_chroma import Chroma

embeddings = OpenAIEmbeddings(
    model="text-embedding-3-small"
)

vectorstore = Chroma.from_documents(
    documents=docs,
    embedding=embeddings,
    collection_name="hybrid_search"
)

dense_retriever = vectorstore.as_retriever(
    search_type="similarity",
    search_kwargs={"k": 4}
)

results = dense_retriever.invoke(
    "How do vaccines work to protect against diseases?"
)
```

The threshold notebook also constructs Chroma explicitly with a cosine HNSW space:

```python
vectorstore = Chroma(
    embedding_function=embeddings,
    collection_name="demo",
    collection_configuration={
        "hnsw": {
            "space": "cosine"  # cosine, L2, or ip
        }
    }
)
```

### 🟡📌 What does `k=4` mean?

Return up to four nearest documents. It does not guarantee all four are useful.

### 🟢📌 Similarity versus distance: an important notebook detail

The threshold notebook uses:

```python
similarity_scores = vectorstore.similarity_search_with_score(
    query,
    k=3
)
```

In the Chroma configuration shown in that notebook, the returned `score` is a **distance**. For cosine distance, lower distance means closer vectors. The notebook displays `1 - distance` as a cosine-similarity-style value:

```python
for doc, distance in similarity_scores:
    cosine_similarity = 1 - distance
    print(cosine_similarity)
```

Do not assume every vector database or API uses the same score direction or scale. Always check whether an API returns a distance or a relevance score.

### 🟡📌 Strengths and limitations

| Strengths | Limitations |
|---|---|
| Finds paraphrases and semantically related content | May miss exact rare terms, codes, or identifiers |
| Works when query and document use different wording | Depends on embedding model and chunk quality |
| Useful for natural-language questions | Requires embedding generation and vector indexing |

**Use dense retrieval** when users ask questions in natural language and relevant documents may express the same idea using different words.

---

## 🔵📌 2.2 Sparse / lexical retrieval — BM25

**Meaning:** Retrieve documents using query-term matching and term statistics. BM25 is a widely used sparse lexical retrieval algorithm.

### 🟡📌 Story example

User searches for:

> `ERR_CONN_RESET`

They need the document containing that exact error identifier. A semantically related document that discusses “network connection problems” may not be enough.

BM25 is useful because it focuses on the words and terms in the query.

### 🟡📌 How BM25 works conceptually

BM25 is related to TF-IDF, with additional refinements:

- **Term Frequency (TF):** How often does a query term occur in a document?
- **Inverse Document Frequency (IDF):** How informative is the term across the whole collection? Rare terms are often more informative.
- **Document-length normalization:** Adjusts for document length so long documents do not win simply because they contain more words.

BM25 does not need an embedding model or a vector database.

### 🟡📌 Notebook implementation

```python
from langchain_core.documents import Document
from langchain_community.retrievers import BM25Retriever

retriever = BM25Retriever.from_documents(
    docs,
    k=2
)

query = "antibiotic bacterial infection treatment"
results = retriever.invoke(query)

for doc in results:
    print(doc.page_content)
```

`docs` is the collection of LangChain `Document` objects, and `k=2` asks for the top two results.

### 🟡📌 BM25 strengths and limitations

| Strengths | Limitations |
|---|---|
| No embedding model required | Does not inherently understand meaning |
| Strong for exact terms, names, codes, and identifiers | Can miss synonyms and paraphrases |
| Simple lexical retrieval approach | Tokenization and preprocessing affect results |

The BM25 notebook demonstrates why lexical overlap matters. A query such as:

> “a structure that feels light and with windows”

may fail to retrieve a document about Gothic cathedrals and flying buttresses if the important query terms do not overlap with the document wording—even when the document is conceptually relevant.

### 🟡📌 BM25Plus in the hybrid notebook

The hybrid and custom-ensemble notebooks use:

```python
bm25_retriever = BM25Retriever.from_documents(
    docs,
    k=2,
    bm25_variant="plus"
)
```

The notebook selects BM25Plus and comments that it can improve recall for short documents by ensuring matched terms contribute positively. The standalone BM25 notebook uses the default call without explicitly selecting this variant.

### 🟡📌 When should you use BM25?

- Error codes and logs
- Product IDs, SKUs, and identifiers
- Exact names and technical terms
- Legal clauses and section numbers
- Queries where exact lexical matches are important

---

## 🔵📌 2.3 Hybrid retrieval — dense + sparse

**Meaning:** Combine dense semantic retrieval and sparse lexical retrieval so that the strengths of one can compensate for the weaknesses of the other.

### 🟡📌 Story example from the uploaded notebook

The user asks:

> “How do vaccines work to protect against diseases?”

The sample collection contains:
- Documents with the exact word “vaccine”
- Documents about antibodies, immune response, and B-lymphocytes that are semantically related but do not contain the word “vaccine”

BM25 can find exact “vaccine” matches. Dense retrieval can find semantically related immune-system documents even when the exact word is absent.

### 🟡📌 Architecture

```text
                     User query
                         |
               +---------+---------+
               |                   |
               v                   v
          Dense search         BM25 search
          (Chroma)             (Sparse)
               |                   |
               v                   v
        Ranked result list   Ranked result list
               |                   |
               +---------+---------+
                         |
                         v
                   Result fusion
                         |
                         v
                Combined ranked results
```

### 🟡📌 Notebook implementation: dense retriever

```python
embeddings = OpenAIEmbeddings(
    model="text-embedding-3-small"
)

vectorstore = Chroma.from_documents(
    documents=docs,
    embedding=embeddings,
    collection_name="hybrid_search"
)

chroma_retriever = vectorstore.as_retriever(
    search_type="similarity",
    search_kwargs={"k": 4}
)
```

### 🟡📌 Notebook implementation: sparse retriever

```python
bm25_retriever = BM25Retriever.from_documents(
    docs,
    k=2,
    bm25_variant="plus"
)
```

### 🟡📌 Notebook implementation: combine using EnsembleRetriever

```python
from langchain_classic.retrievers import EnsembleRetriever

ensemble_retriever = EnsembleRetriever(
    retrievers=[
        chroma_retriever,
        bm25_retriever
    ],
    weights=[0.8, 0.2]
)

query = "How do vaccines work to protect against diseases?"
results = ensemble_retriever.invoke(query)
```

`weights=[0.8, 0.2]` assigns relative influence to the dense and BM25 retrievers respectively. These are example weights, not universal best values. Tune them using evaluation queries.

### 🔵📌 Hybrid retrieval versus EnsembleRetriever versus RRF

- **Hybrid retrieval** describes the strategy: combine different retrieval mechanisms, commonly dense + sparse.
- **EnsembleRetriever** is a framework component that combines outputs from multiple retrievers.
- **RRF** is a rank-fusion algorithm that can be used to merge their ranked lists.

The uploaded notebook implements hybrid retrieval using Chroma dense search, BM25, and `EnsembleRetriever`.

### 🟡📌 When should you use hybrid retrieval?

Use it when:
- Exact terms and semantic meaning both matter.
- Users ask natural-language questions but also include IDs, names, or technical terms.
- Dense-only search misses exact matches.
- BM25-only search misses paraphrases.

Hybrid retrieval adds complexity and may increase latency because multiple retrieval methods run for one query. Evaluate whether the recall improvement is worth the cost.

---

# 🟢📌 3. Category B — Candidate selection and ranking

These techniques do not all represent new ways to build an index. They determine how candidate results are selected, filtered, or ordered.

## 🟢📌 3.1 Similarity search / top-k selection

Similarity search retrieves the nearest vectors to the query. In LangChain, the notebook uses:

```python
sim_retriever = vectorstore.as_retriever(
    search_type="similarity",
    search_kwargs={"k": 3}
)

results = sim_retriever.invoke(query)
```

Think of it as asking:

> “Give me the three closest documents.”

The limitation is that top-k search tries to fill the requested number of slots even when some results are weak matches.

## 🟢📌 3.2 Similarity-score threshold

### 🟡📌 Problem it solves

Suppose top-k returns five documents, but only two are genuinely relevant. A threshold lets the retriever reject documents that do not meet a minimum relevance requirement.

**Similarity search asks:** Which documents are closest?

**Threshold retrieval asks:** Which documents are relevant enough to keep?

### 🟡📌 Notebook example

The notebook creates 12 sample documents across machine learning, economics, cooking, and sports, then searches for:

> “How does ML model training work?”

It inspects the top results and uses a threshold retriever:

```python
threshold = 0.43

retriever = vectorstore.as_retriever(
    search_type="similarity_score_threshold",
    search_kwargs={
        "score_threshold": threshold
    }
)

results = retriever.invoke(
    "How does ML model training work?"
)
```

The number of returned documents can vary. It may return fewer than `k`, or no documents if nothing passes.

### 🟡📌 Important: what does `0.43` mean?

It is the threshold value used by the retriever interface in this example. It is **not a universal value** for all embedding models, vector stores, distance metrics, or datasets.

The notebook's raw Chroma score and the retriever's relevance-score threshold should not be compared blindly. The API may convert distance into relevance score.

### 🟡📌 How to choose a threshold

1. Run representative queries.
2. Inspect retrieved documents and scores.
3. Decide what counts as a useful match.
4. Tune the threshold on a validation set.
5. Check false positives (irrelevant results included) and false negatives (useful results excluded).

A threshold that is too high may remove useful evidence. A threshold that is too low may allow irrelevant context through.

### 🟡📌 When to use it

- You prefer fewer high-confidence documents over a fixed number.
- Irrelevant context can confuse the LLM.
- Your application should be able to return no documents when nothing is sufficiently relevant.

---

## 🟢📌 3.3 MMR — Maximum Marginal Relevance

### 🟡📌 Problem it solves: duplicate context

The MMR notebook intentionally creates three near-identical deep-learning documents about gradient descent. Ordinary similarity search returns those similar documents together.

But a useful context set might contain:
- One gradient-descent explanation
- One dropout explanation
- One batch-normalisation explanation
- One learning-rate-scheduler explanation

MMR balances:
1. **Relevance:** Is this document related to the query?
2. **Diversity:** Is this document different from the documents already selected?

### 🟡📌 Story explanation

Imagine asking three people to explain deep-learning training. All three repeat the same gradient-descent explanation. You would rather have one explanation of gradient descent, one of dropout, and one of batch normalisation.

MMR tries to avoid wasting context space on near-duplicates while retaining relevance.

### 🟢📌 Similarity search versus MMR

| Similarity search | MMR |
|---|---|
| Focuses on query-document similarity | Balances query relevance and diversity |
| Can return near-duplicates | Tries to reduce redundancy |
| Good when top matches are already distinct | Useful when many documents repeat similar information |

### 🟡📌 Notebook implementation

Ordinary similarity retrieval:

```python
sim_retriever = vectorstore.as_retriever(
    search_type="similarity",
    search_kwargs={"k": 3}
)

results = sim_retriever.invoke(query)
```

MMR retrieval:

```python
retriever = vectorstore.as_retriever(
    search_type="mmr",
    search_kwargs={
        "k": 3,
        "fetch_k": 10,
        "lambda_mult": 0.5
    }
)

results = retriever.invoke(query)
```

### 🟡📌 Understand the parameters

| Parameter | Meaning |
|---|---|
| `k` | Number of final documents to return |
| `fetch_k` | Number of candidate documents fetched before MMR selects the final results |
| `lambda_mult` | Controls relevance-versus-diversity balance |

```text
Query
  |
  v
Fetch fetch_k candidate documents
  |
  v
MMR selects a relevant, diverse subset
  |
  v
Return k documents
```

### 🟡📌 Understanding `lambda_mult`

The notebook tests:

```python
lambda_values = [1.0, 0.7, 0.5, 0.0]

for lm in lambda_values:
    retriever = vectorstore.as_retriever(
        search_type="mmr",
        search_kwargs={
            "k": 3,
            "fetch_k": 10,
            "lambda_mult": lm
        }
    )

    results = retriever.invoke(query)
```

| `lambda_mult` | Behaviour |
|---:|---|
| `1.0` | Pure relevance; equivalent to similarity-based selection |
| `0.7` | Strong relevance preference, some diversity |
| `0.5` | Balances relevance and diversity |
| `0.0` | Pure diversity objective; relevance is not directly rewarded |

**Memory trick:** Lambda high = relevance high. Lambda low = diversity has more influence.

Do not automatically choose `0.0`: the results may become diverse but not useful if relevance is ignored.

### 🟡📌 When should you use MMR?

- Your knowledge base contains repeated or overlapping information.
- Similarity search returns near-duplicate chunks.
- You want broader topic coverage within a limited context window.
- The LLM needs different aspects of a subject rather than repeated versions of one point.

---

# 🟣📌 4. Category C — Retriever combination and fusion

This category is about combining multiple ranked lists. Hybrid retrieval is one common reason to combine retrievers, but ensembles can combine other retriever types too.

## 🟣📌 4.1 Ensemble retriever

An **ensemble retriever** combines outputs from multiple retrievers into one result list.

In the hybrid notebook:

```python
ensemble_retriever = EnsembleRetriever(
    retrievers=[
        chroma_retriever,
        bm25_retriever
    ],
    weights=[0.8, 0.2]
)
```

The dense retriever and BM25 retriever each produce their own results. The ensemble combines them using the configured weights and the library's fusion behaviour.

### 🟡📌 What do the weights mean?

```python
weights=[0.8, 0.2]
```

The first retriever receives weight `0.8`; the second receives `0.2`. The weights influence the relative contribution of each retriever. They should be tuned on representative data rather than copied blindly.

## 🟣📌 4.2 RRF — Reciprocal Rank Fusion

### 🟡📌 Why do we need rank fusion?

Imagine two retrievers return:

```text
Dense:
1. Document A
2. Document B
3. Document C

BM25:
1. Document B
2. Document D
3. Document A
```

A document may appear in both lists at different ranks. RRF combines the lists by adding a contribution based on each document's rank. Documents ranked highly by multiple retrievers can rise in the fused ranking.

A common RRF formula is:

\[
RRF(d) = \sum_{r \in R} \frac{1}{k + rank_r(d)}
\]

Weighted form:

\[
RRF(d) = \sum_{r \in R} \frac{w_r}{k + rank_r(d)}
\]

Where:
- `d` = document
- `r` = a retriever
- `rank_r(d)` = document's rank in that retriever's list
- `w_r` = retriever weight
- `k` = smoothing constant

A retriever that does not return a document contributes zero for that document.

### 🟡📌 Why use RRF instead of adding raw scores?

Dense similarity scores and BM25 scores may have different scales and meanings. Directly adding them can be misleading. RRF uses rank positions, so it does not require raw scores to be directly comparable.

## 🟣📌 4.3 Custom Ensemble Retriever — implement RRF yourself

The fifth notebook rebuilds ensemble retrieval manually. This is useful for understanding what happens behind the framework abstraction.

### 🟡📌 Notebook code

```python
from typing import List
from langchain_core.documents import Document
from langchain_core.retrievers import BaseRetriever
from langchain_core.callbacks.manager import CallbackManagerForRetrieverRun

class MyEnsembleRetriever(BaseRetriever):
    retrievers: List[BaseRetriever]
    weights: List[float]
    rrf_k: int = 60

    def _get_relevant_documents(
        self,
        query: str,
        run_manager: CallbackManagerForRetrieverRun
    ) -> List[Document]:

        # 1. Collect ranked results from every retriever
        all_results: List[List[Document]] = [
            retriever.invoke(query)
            for retriever in self.retrievers
        ]

        # 2. Accumulate RRF scores; use page_content as the key
        doc_scores: dict[str, tuple[float, Document]] = {}

        for retriever_idx, results in enumerate(all_results):
            weight = self.weights[retriever_idx]

            for rank, doc in enumerate(results):
                rrf_score = weight * (
                    1.0 / (rank + self.rrf_k)
                )

                key = doc.page_content

                if key in doc_scores:
                    prev_score, prev_doc = doc_scores[key]
                    doc_scores[key] = (
                        prev_score + rrf_score,
                        prev_doc
                    )
                else:
                    doc_scores[key] = (rrf_score, doc)

        # 3. Sort by accumulated score, highest first
        sorted_docs = sorted(
            doc_scores.values(),
            key=lambda x: x[0],
            reverse=True
        )

        return [doc for _, doc in sorted_docs]
```

### 🟡📌 Code explained step by step

**Step 1 — Accept multiple retrievers**

```python
retrievers: List[BaseRetriever]
```

Example:

```python
[chroma_retriever, bm25_retriever]
```

**Step 2 — Configure weights**

```python
weights: List[float]
```

The notebook uses `[0.8, 0.2]`, giving dense retrieval more influence in that example.

**Step 3 — Run every retriever**

```python
all_results = [
    retriever.invoke(query)
    for retriever in self.retrievers
]
```

Each retriever produces its own ranked document list.

**Step 4 — Calculate each rank contribution**

```python
rrf_score = weight * (
    1.0 / (rank + self.rrf_k)
)
```

A better rank produces a larger contribution. The weight scales the contribution.

**Implementation detail:** Python's `enumerate()` starts at rank `0` here. Some RRF descriptions use ranks starting at `1`. The notebook's formula is internally consistent with its zero-based loop, but exact numerical scores differ slightly from a one-based implementation.

**Step 5 — Deduplicate and accumulate**

```python
key = doc.page_content
```

The notebook uses the document text as its deduplication key. If the same text appears in multiple result lists, the RRF contributions are added.

**Step 6 — Sort**

```python
sorted_docs = sorted(
    doc_scores.values(),
    key=lambda x: x[0],
    reverse=True
)
```

The highest accumulated RRF scores appear first.

### 🟡📌 Instantiate and call the custom retriever

```python
my_ensemble = MyEnsembleRetriever(
    retrievers=[
        chroma_retriever,
        bm25_retriever
    ],
    weights=[0.8, 0.2],
    rrf_k=60
)

results = my_ensemble.invoke(
    "How do vaccines work to protect against diseases?"
)
```

### 🟡📌 Production cautions

The uploaded implementation is a teaching example. For production:

1. Validate that the number of weights equals the number of retrievers.
2. Consider stable document IDs instead of using only `page_content` for deduplication.
3. Decide whether ranks start at zero or one and document the convention.
4. Add a configurable final result limit if required; the notebook returns every unique document in the merged lists.
5. Decide how to handle a retriever that fails or responds slowly.
6. Check LangChain versions because imports and APIs can change.

### 🟡📌 Built-in versus custom ensemble

| Built-in `EnsembleRetriever` | Custom ensemble |
|---|---|
| Less code to maintain | You control the fusion logic |
| Convenient for common cases | Useful for special requirements |
| Library owns the implementation | You own correctness and edge cases |
| Good default starting point | Useful for custom deduplication, scoring, logging, or filtering |

**Practical advice:** Start with the built-in ensemble. Implement your own only when you have a clear requirement that the built-in version does not satisfy.

---

# 🟠📌 5. Category D — Query-oriented advanced retrieval (supplementary)

> **Not implemented in the five uploaded notebooks.** These concepts are included to complete the conceptual map, not as claims about the code you uploaded.

These techniques change or expand the user's query before or during retrieval.

## 🟠📌 5.1 Multi-Query Retriever

**Problem:** A user may phrase a question in one way, while relevant documents use different terminology.

**Idea:** Ask an LLM to generate several alternative versions of the original query. Run retrieval for each query, then combine or deduplicate the results.

```text
Original question
       |
       v
LLM generates query variations
   /       |        \
  v        v         v
Query 1  Query 2   Query 3
  |        |         |
  v        v         v
Retriever Retriever Retriever
   \       |        /
    \      |       /
     Merge / deduplicate
             |
             v
       Retrieved context
```

Example original query:

> “How can I reduce LLM hallucinations?”

Possible query variations:
- “Methods for improving factual accuracy in large language models”
- “Techniques to ground LLM answers in trusted sources”
- “How retrieval-augmented generation reduces hallucination”

**Use when:** Query wording is ambiguous or relevant documents may use different vocabulary.

## 🟠📌 5.2 Query rewriting / query expansion

**Idea:** Rewrite the user's original query into a clearer, more searchable query, or expand it with useful terms.

Example:

Original:
> “RAG not finding docs for latest policy”

Rewritten:
> “Retrieve the most recently updated policy documents and identify the current policy version.”

**Use when:** User questions are vague, conversational, misspelled, or incomplete.

**Difference from Multi-Query:** Query rewriting often produces one improved query; Multi-Query deliberately generates multiple alternative queries. Implementations can overlap.

## 🟠📌 5.3 Self-Query Retriever

**Idea:** Use an LLM to interpret a natural-language question and separate:
1. The semantic search query
2. Structured metadata filters

Example user question:

> “Find engineering documents about Kubernetes published after 2024.”

The retriever may derive:
- Semantic query: “Kubernetes”
- Metadata filter: `department = engineering`
- Metadata filter: `publication_year > 2024`

**Use when:** Your documents contain useful metadata such as department, author, date, product, region, or document type.

**Important:** Metadata filtering requires the relevant metadata to be present and correctly indexed. The LLM does not magically create reliable metadata that your data store does not have.

---

# 🔴📌 6. Category E — Hierarchical and context-aware retrieval (supplementary)

> **Not implemented in the five uploaded notebooks.** These concepts are included for taxonomy understanding only.

## 🔴📌 6.1 Parent Document Retriever

**Problem:** Small chunks can be precise for matching, but may not contain enough surrounding context for the LLM to answer well.

**Idea:** Split documents into:
- Small child chunks for searching
- Larger parent chunks or documents for context

Search using child chunks. When a child matches, return its parent document or larger parent section.

```text
Full document
     |
     +-------------------------------+
     |                               |
     v                               v
Large parent sections          Small child chunks
                                      |
                                      v
                                Search child chunks
                                      |
                                      v
                              Matching child found
                                      |
                                      v
                              Return parent section
```

**Use when:** Precise matching is important, but the LLM needs surrounding paragraphs or a larger section to understand the evidence.

## 🔴📌 6.2 Child-to-parent retrieval

This describes the retrieval pattern used by parent-document approaches:
- Search smaller child chunks.
- Use the child's stored parent reference.
- Return the larger parent content.

It is a relationship between indexing granularity and retrieval granularity, not necessarily a separate algorithm.

## 🔴📌 6.3 Contextual Compression

**Problem:** Retrieved documents may contain useful information mixed with irrelevant paragraphs.

**Idea:** After retrieval, compress the retrieved content by extracting only the parts relevant to the query. Depending on the implementation, this may use an LLM or another filtering method.

```text
User query
    |
    v
Retriever returns documents
    |
    v
Compression / filtering step
    |
    v
Only query-relevant passages
    |
    v
LLM answer generation
```

**Use when:** Retrieved documents are long, noisy, or contain only a small amount of relevant information.

**Taxonomy note:** Contextual compression is usually best understood as a post-retrieval processing layer around a retriever, rather than a standalone dense or sparse retrieval mechanism.

---

# 🟡📌 7. The complete comparison table

| Technique | Taxonomy role | Needs embeddings? | Main purpose |
|---|---|---:|---|
| Dense similarity | Retrieval mechanism | Yes | Find semantically close documents |
| BM25 | Retrieval mechanism | No | Find lexical / keyword matches |
| Hybrid search | Retrieval combination strategy | Dense side: yes; sparse side: no | Combine semantic and lexical signals |
| Similarity top-k | Candidate selection | For vector similarity, yes | Return a fixed number of nearest results |
| Similarity threshold | Candidate filtering | For vector similarity, yes | Reject weak matches |
| MMR | Candidate selection / re-ranking | Usually yes | Balance relevance and diversity |
| Ensemble retriever | Result combination | Depends on component retrievers | Combine multiple retriever outputs |
| RRF | Rank-fusion algorithm | No, not by itself | Fuse ranked lists using rank positions |
| Multi-Query | Query transformation | Not inherently | Search multiple query formulations |
| Query rewriting | Query transformation | Not inherently | Improve the search query |
| Self-Query | Query + metadata filter generation | Not inherently | Derive semantic query and structured filters |
| Parent Document | Hierarchical retrieval | Commonly used with vector search | Search small chunks, return larger context |
| Contextual Compression | Post-retrieval processing | Not inherently | Remove irrelevant parts of retrieved context |

---

# 🟡📌 8. How to choose in a real RAG application

| Requirement | Good starting point | Reason |
|---|---|---|
| Search by meaning or paraphrase | Dense similarity | Semantic matching |
| Search exact IDs, codes, names | BM25 | Lexical matching |
| Need both semantic and exact matching | Hybrid retrieval | Complementary retrieval signals |
| Top-k contains weak matches | Similarity threshold | Reject low-relevance results |
| Top-k contains near-duplicates | MMR | Improve diversity |
| Need to combine several retrievers | Ensemble + RRF | Merge ranked result lists |
| Query is vague or uses different terminology | Query rewriting / Multi-Query | Improve query coverage |
| User asks for metadata constraints | Self-Query | Convert natural language to filters |
| Small chunks lack surrounding context | Parent Document | Return a larger parent section |
| Retrieved documents are long and noisy | Contextual Compression | Keep relevant passages |

## 🟡📌 A sensible experimentation sequence

For many enterprise knowledge-base projects:

1. Build a dense similarity baseline.
2. Evaluate whether exact terms and identifiers are being missed.
3. Add BM25 and compare against dense-only retrieval.
4. Try hybrid retrieval and tune weights.
5. Inspect whether the returned context contains duplicates; if so, test MMR.
6. If weak matches are reaching the LLM, evaluate a score threshold.
7. If queries are vague or metadata filters matter, evaluate query-oriented techniques.
8. If chunks lack enough context or retrieved documents are too long, evaluate parent retrieval or contextual compression.

This is a practical sequence for experimentation, not a universal recipe. Measure retrieval quality and end-to-end answer quality on representative queries.

---

# 🟡📌 9. Quick memory tricks

- **Dense = meaning.** It searches by semantic closeness.
- **Sparse / BM25 = words.** It searches by lexical overlap and term statistics.
- **Hybrid = both.** Dense + sparse retrieval.
- **Top-k = fixed count.** Return the nearest `k` candidates.
- **Threshold = minimum acceptable relevance.** It can return fewer than `k`.
- **MMR = relevant + diverse.** Avoid repeated context.
- **Ensemble = multiple retrievers together.**
- **RRF = combine ranks.** It avoids directly adding incompatible raw scores.
- **Multi-Query = many versions of the question.**
- **Self-Query = question + metadata filters.**
- **Parent Document = search small, return big.**
- **Contextual Compression = retrieve, then trim.**

---

# 🟡📌 10. Interview questions and answers

### 🟡📌 Q1. What is the difference between dense and sparse retrieval?

**Answer:** Dense retrieval represents text as embedding vectors and retrieves based on semantic similarity. Sparse retrieval, such as BM25, relies on lexical term matching and term statistics. Dense retrieval is useful for paraphrases; sparse retrieval is useful for exact terms.

### 🟡📌 Q2. What is similarity search?

**Answer:** It embeds a query and compares it with stored document vectors, returning the nearest documents according to a configured distance or similarity metric.

### 🟡📌 Q3. Why can top-k search return irrelevant documents?

**Answer:** Top-k tries to return the requested number of results, even if some are weak matches. A threshold can filter out results below a relevance cutoff.

### 🟡📌 Q4. What is a similarity-score threshold retriever?

**Answer:** It returns documents only when their relevance score passes a configured threshold. The number returned can vary and may be zero.

### 🟡📌 Q5. Is `score_threshold=0.43` suitable for every project?

**Answer:** No. The correct threshold depends on the embedding model, vector store, score conversion, dataset, and query distribution. It must be calibrated.

### 🟡📌 Q6. What is the difference between distance and similarity?

**Answer:** Distance measures how far apart vectors are; smaller is usually closer. Similarity measures how alike vectors are; larger is usually more similar. Check the API's score semantics.

### 🟡📌 Q7. What is MMR?

**Answer:** Maximum Marginal Relevance selects documents by balancing relevance to the query with diversity relative to documents already selected.

### 🟡📌 Q8. Why use MMR instead of ordinary similarity search?

**Answer:** Similarity search can return near-duplicate documents. MMR tries to provide a more diverse set of relevant documents, improving context coverage.

### 🟡📌 Q9. What does `fetch_k` mean in MMR?

**Answer:** It is the number of candidate documents fetched before MMR selects the final `k` documents.

### 🟡📌 Q10. What does `lambda_mult` control?

**Answer:** It controls the relevance-diversity trade-off. A value near 1 emphasizes relevance; a value near 0 emphasizes diversity.

### 🟡📌 Q11. What is BM25?

**Answer:** BM25 is a sparse lexical retrieval algorithm that ranks documents using query-term statistics, including term frequency, inverse document frequency, and document-length normalization.

### 🟡📌 Q12. Does BM25 need embeddings?

**Answer:** No. BM25 works with text and term statistics; it does not need an embedding model or vector database.

### 🟡📌 Q13. Why can BM25 fail on paraphrases?

**Answer:** The query may use different words from the relevant document. BM25 relies on lexical overlap and does not inherently understand semantic equivalence.

### 🟡📌 Q14. What is hybrid search?

**Answer:** Hybrid search combines different retrieval approaches, commonly dense semantic retrieval and sparse lexical retrieval, to improve retrieval coverage.

### 🟡📌 Q15. Why combine BM25 and dense retrieval?

**Answer:** BM25 can find exact words and identifiers, while dense retrieval can find semantically related content. Their strengths are complementary.

### 🟡📌 Q16. What is an ensemble retriever?

**Answer:** It combines the outputs of multiple retrievers into one result list. The uploaded notebook combines dense Chroma retrieval and BM25.

### 🟡📌 Q17. What is Reciprocal Rank Fusion?

**Answer:** RRF combines ranked lists by giving each document a contribution based on its rank in each list. Contributions are added to create a fused ranking.

### 🟡📌 Q18. Why use RRF instead of adding raw scores?

**Answer:** Different retrievers can produce scores on different scales. RRF uses ranks, avoiding the need to directly compare incompatible raw score values.

### 🟡📌 Q19. What is the purpose of weights in an ensemble retriever?

**Answer:** Weights control the relative influence of each retriever in the fusion process. They should be tuned using retrieval evaluation.

### 🟡📌 Q20. What is the difference between hybrid search and RRF?

**Answer:** Hybrid search is the overall strategy of combining retrieval methods. RRF is one algorithm for merging their ranked results.

### 🟡📌 Q21. What is the difference between built-in and custom ensemble retrieval?

**Answer:** A built-in ensemble provides a ready-made implementation. A custom ensemble gives more control but requires you to handle validation, deduplication, ranking, failures, and maintenance.

### 🟡📌 Q22. What is Multi-Query retrieval?

**Answer:** It generates multiple alternative query formulations, retrieves results for each, and combines the results to improve query coverage.

### 🟡📌 Q23. What is the difference between Multi-Query and query rewriting?

**Answer:** Query rewriting commonly produces one improved query. Multi-Query generates multiple alternatives. They can overlap in some implementations.

### 🟡📌 Q24. What is Self-Query retrieval?

**Answer:** It uses a language model to derive a semantic search query and structured metadata filters from a natural-language request.

### 🟡📌 Q25. What is Parent Document retrieval?

**Answer:** It searches small child chunks for precise matching and returns the larger parent document or section to provide sufficient context.

### 🟡📌 Q26. What is Contextual Compression?

**Answer:** It filters or compresses retrieved documents after retrieval so the LLM receives only the parts relevant to the query.

### 🟡📌 Q27. Is Contextual Compression a dense or sparse retrieval algorithm?

**Answer:** No. It is better understood as a post-retrieval processing layer that operates on retrieved content.

---

# 🟡📌 11. Final revision checklist

Use this checklist to test whether you understand the taxonomy:

- [ ] I can explain dense retrieval and why it uses embeddings.
- [ ] I can explain BM25 without mentioning embeddings.
- [ ] I can explain why dense and sparse retrieval complement each other.
- [ ] I can distinguish hybrid retrieval from an ensemble implementation.
- [ ] I understand top-k versus score threshold.
- [ ] I can explain MMR, `k`, `fetch_k`, and `lambda_mult`.
- [ ] I can explain why RRF uses rank positions instead of directly adding raw scores.
- [ ] I can walk through the custom ensemble code.
- [ ] I know that Multi-Query, Self-Query, Parent Document, and Contextual Compression were not implemented in the five uploaded notebooks.
- [ ] I can identify whether a technique changes the retrieval mechanism, filters candidates, fuses results, transforms the query, or processes retrieved context.

---

## 🟡📌 One-line summary

**Dense/BM25 find candidates; Hybrid combines retrieval signals; Top-k/Threshold/MMR select results; Ensemble/RRF fuse ranked lists; Query-based methods improve the search query; Parent retrieval and Contextual Compression improve the context passed to the LLM.**



########################33

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
