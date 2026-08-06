


# RAG — Date-wise Topics Covered

## Date-wise Blog Summary

| Date | Main topics covered | Blog URL |
|---|---|---|
| **25 July 2026** | **Handling document updates in RAG:** full re-indexing, content hashing with incremental upsert, LangChain Indexing API, document versioning, soft deletion, and the **NCERT Book RAG project**. | https://directai.blog/2026/07/25/gen-ai-developer-classroom-notes-25-jul-2026/ |
| **23 July 2026** | **LLM and RAG evaluation:** why exact-text comparison does not work for nondeterministic LLM outputs; introduction to **DeepEval**; test cases, evaluation metrics, evaluation runners, and integration with pytest. | https://directai.blog/2026/07/23/gen-ai-developer-classroom-notes-23-jul-2026/ |
| **22 July 2026** | **RAG user interface development:** invoking RAG through a JavaScript chat interface; Streamlit and Gradio for prototypes; Streamlit application setup, session state, widget keys, callbacks, `cache_data`, and `cache_resource`. | https://directai.blog/2026/07/22/gen-ai-developer-classroom-notes-22-jul-2026-2/ |
| **21 July 2026** | **Vector-database retrieval techniques:** dense semantic retrieval, sparse keyword retrieval, and hybrid or fusion retrieval; combining dense and BM25 results with weights; ChromaDB and `rank_bm25`. | https://directai.blog/2026/07/21/gen-ai-developer-classroom-notes-21-jul-2026/ |
| **16 July 2026** | **Advanced chunking and vector storage:** recursive character splitting; NCERT document chunking fixes; vector storage contents; Flat, IVF, and HNSW indexes; cosine similarity, dot product, and Euclidean distance; hybrid search, metadata filtering, and CRUD operations. | https://directai.blog/2026/07/16/gen-ai-developer-classroom-notes-16-jul-2026/ |
| **15 July 2026** | **Section-aware chunking:** combining chapter pages, detecting sections using regular expressions, recursive splitting by section and length, chunk overlap, metadata organization, and enforcing maximum chunk size. | https://directai.blog/2026/07/15/gen-ai-developer-classroom-notes-15-jul-2026/ |
| **13 July 2026** | **Handling images in RAG:** extracting an image’s meaning through captions, directly embedding images, multimodal RAG, and generating useful image captions for retrieval. | https://directai.blog/2026/07/13/gen-ai-developer-classroom-notes-13-jul-2026-2/ |
| **11 July 2026** | **RAG productionization:** production chunking, vector stores, RAG scoring, guardrails, text and image data; CBSE or NCERT Teacher RAG idea; extracting and organizing PDF text and images by chapters and sections; filtering blank or incorrectly extracted images, tables, and flowcharts. | https://directai.blog/2026/07/11/gen-ai-developer-classroom-notes-11-jul-2026/ |
| **9 July 2026** | **RAG with relational databases:** building RAG applications over structured data, understanding database schemas, and using database information to answer natural-language questions. | https://directai.blog/2026/07/09/gen-ai-developer-classroom-notes-09-jul-2026/ |
| **8 July 2026** | **Building the first RAG application:** implementing a simple end-to-end RAG flow that retrieves relevant information and passes the retrieved context to an LLM. | https://directai.blog/2026/07/08/gen-ai-developer-classroom-notes-08-jul-2026/ |
| **7 July 2026** | **Embeddings and vector databases:** meaning represented as vectors; Word2Vec and embedding evolution; open-source and cloud embedding models; LangChain `embed_query` and `embed_documents`; Chroma, FAISS, pgvector, MongoDB, Pinecone, and Weaviate; building an indexing pipeline with metadata. | https://directai.blog/2026/07/07/gen-ai-developer-classroom-notes-07-jul-2026/ |
| **6 July 2026** | **Document chunking and splitting:** relationship between LLMs and embedding models; why documents are split for retrieval; chunk size and overlap; LangChain text splitters; loading and splitting together; adding ingestion date and project metadata. | https://directai.blog/2026/07/06/gen-ai-developer-classroom-notes-06-jul-2026/ |
| **4 July 2026** | **Documents and document loaders:** LangChain `Document` objects; loading documents instead of manually creating them; TextLoader, PDF loader, CSV loader, and directory loader; required external packages and loader experiments. | https://directai.blog/2026/07/04/gen-ai-developer-classroom-notes-04-jul-2026/ |
| **2 July 2026** | **Setting up Google Cloud with LangChain:** installing the Google Cloud SDK, selecting the GCP project, configuring authentication, and preparing the GCP or Vertex AI environment for LangChain model integration. | https://directai.blog/2026/07/02/gen-ai-developer-classroom-notes-02-jul-2026/ |
| **30 June 2026** | **LangChain Runnable and RAG introduction:** `invoke`, `ainvoke`, `batch`, and `stream`; LCEL chains; BaseChatModel and BaseMessage; RAG indexing and retrieval phases; vector databases; chunking, embeddings, retrievers, document loaders, and document types. | https://directai.blog/2026/06/30/gen-ai-developer-classroom-notes-30-jun-2026/ |
| **29 June 2026** | **LangChain chaining:** combining prompts, LLMs, and output-processing components into chains using the pipe operator; introduction to composable LLM workflows. | https://directai.blog/2026/06/29/gen-ai-developer-classroom-notes-29-jun-2026/ |
| **27 June 2026** | **Vector fundamentals:** vectors as multidimensional mathematical points, semantic relationships between words, and vector arithmetic such as the commonly used “king − man + woman” example. | https://directai.blog/2026/06/27/gen-ai-developer-classroom-notes-27-jun-2026/ |

## Overall RAG Learning Sequence

### 1. Foundations — 27 June to 2 July

Vectors → LangChain chaining → Runnable interface → RAG architecture → GCP and LangChain setup.

### 2. Data Ingestion — 4 to 6 July

Document loaders → loading files → document chunking → chunk overlap → metadata.

### 3. Embeddings and Retrieval — 7 to 9 July

Embedding models → vector databases → indexing pipeline → first RAG application → structured-data RAG.

### 4. Production Document Processing — 11 to 16 July

PDF text and image extraction → multimodal RAG → section-aware chunking → recursive splitting → vector indexes and distance metrics.

### 5. Application and Quality — 21 to 25 July

Dense, sparse, and hybrid retrieval → Streamlit UI → DeepEval testing → document update and re-indexing strategies.


RAG

25th/July/2026 – [Click Here](https://directai.blog/2026/07/25/gen-ai-developer-classroom-notes-25-jul-2026/)
23rd/July/2026 – [Click Here](https://directai.blog/2026/07/23/gen-ai-developer-classroom-notes-23-jul-2026/)
22nd /July/2026 – [Click Here](https://directai.blog/2026/07/22/gen-ai-developer-classroom-notes-22-jul-2026-2/)
21st/July/2026 – [Click Here](https://directai.blog/2026/07/21/gen-ai-developer-classroom-notes-21-jul-2026/)
16th/July/2026 – [Click Here](https://directai.blog/2026/07/16/gen-ai-developer-classroom-notes-16-jul-2026/)
15th/July/2026 – [Click Here](https://directai.blog/2026/07/15/gen-ai-developer-classroom-notes-15-jul-2026/)
13th/July/2026 – [Click Here](https://directai.blog/2026/07/13/gen-ai-developer-classroom-notes-13-jul-2026-2/)
11th/July/2026 – [Click Here](https://directai.blog/2026/07/11/gen-ai-developer-classroom-notes-11-jul-2026/)
9th/July/2026 – [Click Here](https://directai.blog/2026/07/09/gen-ai-developer-classroom-notes-09-jul-2026/)
8th/July/2026 – [Click Here](https://directai.blog/2026/07/08/gen-ai-developer-classroom-notes-08-jul-2026/)
7th/july/2026 – [Click Here](https://directai.blog/2026/07/07/gen-ai-developer-classroom-notes-07-jul-2026/)
6th/July/2026 – [Click Here](https://directai.blog/2026/07/06/gen-ai-developer-classroom-notes-06-jul-2026/)
4th/July/2026 – [Click Here](https://directai.blog/2026/07/04/gen-ai-developer-classroom-notes-04-jul-2026/)
2nd July/2026 – [Click Here](https://directai.blog/2026/07/02/gen-ai-developer-classroom-notes-02-jul-2026/)
30th/June/2026 – [Click Here](https://directai.blog/2026/06/30/gen-ai-developer-classroom-notes-30-jun-2026/)
29th/June/2026 – [Click Here](https://directai.blog/2026/06/29/gen-ai-developer-classroom-notes-29-jun-2026/)
27th/June/2026 – [Click Here](https://directai.blog/2026/06/27/gen-ai-developer-classroom-notes-27-jun-2026/)
Foundation For Gen Ai

26th/June/2026 – [Click Here](https://directai.blog/2026/06/26/gen-ai-developer-classroom-notes-26-jun-2026/)
23rd/June/2026 – [Click Here](https://directai.blog/2026/06/23/gen-ai-developer-classroom-notes-23-jun-2026/)
Deep Agents

18th/July/2026 – [Click Here](https://directai.blog/2026/07/19/gen-ai-developer-classroom-notes-19-jul-2026/)
21st/June/2026 – [Click Here](https://directai.blog/2026/06/21/gen-ai-developer-classroom-notes-21-jun-2026/)
18th/June/2026 – [Click Here](https://directai.blog/2026/06/18/gen-ai-developer-classroom-notes-18-jun-2026/)
17th/June/2026 – [Click Here](https://directai.blog/2026/06/17/gen-ai-developer-classroom-notes-17-jun-2026/)
15th/June/2026 – [Click Here](https://directai.blog/2026/06/15/gen-ai-developer-classroom-notes-15-jun-2026/)
12th/June/2026 – [Click Here](https://directai.blog/2026/06/12/gen-ai-developer-classroom-notes-12-jun-2026/)
11th/June/2026 – [Click Here](https://directai.blog/2026/06/11/gen-ai-developer-classroom-notes-11-jun-2026/)
9th/June/2026 – [Click Here](https://directai.blog/2026/06/09/gen-ai-developer-classroom-notes-09-jun-2026/)
8th/June/2026 – [Click Here](https://directai.blog/2026/06/08/gen-ai-developer-classroom-notes-08-jun-2026/)
5th/June/2026 – [Click Here](https://directai.blog/2026/06/05/gen-ai-developer-classroom-notes-05-jun-2026/)
4th/June/2026 – [Click Here](https://directai.blog/2026/06/04/gen-ai-developer-classroom-notes-04-jun-2026/)
29th/May/2026 – [Click Here](https://directai.blog/2026/05/29/gen-ai-developer-classroom-notes-29-may-2026/)
27th/May/2026 – [Click Here](https://directai.blog/2026/05/27/gen-ai-developer-classroom-notes-27-may-2026/)
26th/May/2026 – [Click Here](https://directai.blog/2026/05/26/gen-ai-developer-classroom-notes-26-may-2026/)
25th/May/2026 – [Click Here](https://directai.blog/2026/05/25/gen-ai-developer-classroom-notes-25-may-2026/)
21st/May/2026 – [Click Here](https://directai.blog/2026/05/21/gen-ai-developer-classroom-notes-21-may-2026/)
MCP

23rd/May/2026 – [Click Here](https://directai.blog/2026/05/23/gen-ai-developer-classroom-notes-23-may-2026/)
20th/May/2026 – [Click Here](https://directai.blog/2026/05/20/gen-ai-developer-classroom-notes-20-may-2026/)
19th/May/2026 – [Click Here](https://directai.blog/2026/05/19/gen-ai-developer-classroom-notes-19-may-2026/)
17th/May/2026 – [Click Here](https://directai.blog/2026/05/17/gen-ai-developer-classroom-notes-17-may-2026/)
16th/May/2026 – [Click Here](https://directai.blog/2026/05/16/gen-ai-developer-classroom-notes-16-may-2026/)
14th/May/2026 – [Click Here](https://directai.blog/2026/05/14/gen-ai-developer-classroom-notes-14-may-2026-2/)
13th/May/2026 – [Click Here](https://directai.blog/2026/05/13/gen-ai-developer-classroom-notes-13-may-2026/)
12th/May/2026 – [Click Here](https://directai.blog/2026/05/12/gen-ai-developer-classroom-notes-12-may-2026/)
10th/May/2026 – [Click Here](https://directai.blog/2026/05/10/gen-ai-developer-classroom-notes-10-may-2026/)
9th/May2026 – [Click Here](https://directai.blog/2026/05/09/gen-ai-developer-classroom-notes-09-may-2026/)
6th/May/2026 – [Click Here](https://directai.blog/2026/05/06/gen-ai-developer-classroom-notes-06-may-2026/)
5th/May/2026 – [Click Here](https://directai.blog/2026/05/05/gen-ai-developer-classroom-notes-05-may-2026-2/)
4th/May/2026 – [Click Here](https://directai.blog/2026/05/04/gen-ai-developer-classroom-notes-04-may-2026/)
30th/April/2026 – [Click Here](https://directai.blog/2026/04/30/gen-ai-developer-classroom-notes-30-apr-2026/)
Foundation For Gen Ai

26th/April/2026 – [Click Here](https://directai.blog/2026/04/26/gen-ai-developer-classroom-notes-26-apr-2026/)
23rd/April/2026 – [Click Here](https://directai.blog/2026/04/23/gen-ai-developer-classroom-notes-23-apr-2026/)
22nd/April/2026 – [Click Here](https://directai.blog/2026/04/22/gen-ai-developer-classroom-notes-22-apr-2026/)
21st/April/2026 – [Click Here](https://directai.blog/2026/04/21/gen-ai-developer-classroom-notes-21-apr-2026/)
Agentic AI

7th/June/2026 – [Click Here](https://directai.blog/2026/06/07/gen-ai-developer-classroom-notes-07-jun-2026/)
27th/April/2026 – [Click Here](https://directai.blog/2026/04/27/gen-ai-developer-classroom-notes-27-apr-2026/)
22nd/April/2026 – [Click Here](https://directai.blog/2026/04/22/gen-ai-developer-classroom-notes-22-apr-2026-3/)
19th/April/2026 – [Click Here](https://directai.blog/2026/04/19/gen-ai-developer-classroom-notes-19-apr-2026-2/)
18th/April/2026 – [Click Here](https://directai.blog/2026/04/18/gen-ai-developer-classroom-notes-18-apr-2026/)
15th/April/2026 – [Click Here](https://directai.blog/2026/04/15/gen-ai-developer-classroom-notes-15-apr-2026-2/)
14th/April/2026 – [Click Here](https://directai.blog/2026/04/14/gen-ai-developer-classroom-notes-14-apr-2026/)
11th/April/2026 – [Click Here](https://directai.blog/2026/04/11/gen-ai-developer-classroom-notes-11-apr-2026/)
9th/April/2026 – [Click Here](https://directai.blog/2026/04/09/gen-ai-developer-classroom-notes-09-apr-2026/)
7th/April/2026 – [Click Here](https://directai.blog/2026/04/07/gen-ai-developer-classroom-notes-07-apr-2026/)
6th/April/2026 – [Click Here](https://directai.blog/2026/04/06/gen-ai-developer-classroom-notes-06-apr-2026/)
4th/April/2026 – [Click Here](https://directai.blog/2026/04/04/gen-ai-developer-classroom-notes-04-apr-2026-2/)
2nd/April/2026 – [Click Here](https://directai.blog/2026/04/02/gen-ai-developer-classroom-notes-02-apr-2026/)
1st/April/2026 – [Click Here](https://directai.blog/2026/04/01/gen-ai-developer-classroom-notes-01-apr-2026/)
31st/March/2026 – [Click Here](https://directai.blog/2026/03/31/gen-ai-developer-classroom-notes-31-mar-2026/)
29th/March/2026 – [Click Here](https://directai.blog/2026/03/29/gen-ai-developer-classroom-notes-29-mar-2026/)
28th/March/2026 – [Click Here](https://directai.blog/2026/03/28/gen-ai-developer-classroom-notes-28-mar-2026/)
25th/March/2026 – [Click Here](https://directai.blog/2026/03/25/gen-ai-developer-classroom-notes-25-mar-2026/)
24th/March/2026 – [Click Here](https://directai.blog/2026/03/24/gen-ai-developer-classroom-notes-24-mar-2026/)
23rd/March/2026 – [Click Here](https://directai.blog/2026/03/23/gen-ai-developer-classroom-notes-23-mar-2026/)
20th/March/2026 – [Click Here](https://directai.blog/2026/03/20/gen-ai-developer-classroom-notes-20-mar-2026/)
18th/March/2026 – [Click Here](https://directai.blog/2026/03/18/gen-ai-developer-classroom-notes-18-mar-2026-2/)
17th/March/2026 – [Click Here](https://directai.blog/2026/03/17/gen-ai-developer-classroom-notes-17-mar-2026/)
16th/March/2026 – [Click Here](https://directai.blog/2026/03/16/gen-ai-developer-classroom-notes-16-mar-2026/)
14th/March/2026 – [Click Here](https://directai.blog/2025/02/14/gen-ai-classroom-notes-14-02-2025/)
13th/March/2026 – [Click Here](https://directai.blog/2026/03/13/gen-ai-developer-classroom-notes-13-mar-2026/)
Fine tuning

11th/March/2026 – [Click Here](https://directai.blog/2026/03/11/gen-ai-developer-classroom-notes-11-mar-2026/)
10th/March/2026 – [Click Here](https://directai.blog/2026/03/10/gen-ai-developer-classroom-notes-10-mar-2026/)
9th/March/2026 – [Click Here](https://directai.blog/2026/03/09/gen-ai-developer-classroom-notes-09-mar-2026/)
7th/March/2026 – [Click Here](https://directai.blog/2026/03/07/gen-ai-developer-classroom-notes-07-mar-2026-2/)
5th/March/2026 – [Click Here](https://directai.blog/2026/03/05/gen-ai-developer-classroom-notes-05-mar-2026/)
2nd /Mar/2026 – [Click Here](https://directai.blog/2026/03/02/gen-ai-developer-classroom-notes-02-mar-2026/)
28th/Feb/2026 – [Click Here](https://directai.blog/2026/02/28/gen-ai-developer-classroom-notes-28-feb-2026/)
26th/Feb/2026 – [Click Here](https://directai.blog/2026/02/26/gen-ai-developer-classroom-notes-26-feb-2026/)
25th/Feb/2026 – [Click Here](https://directai.blog/2026/02/25/gen-ai-developer-classroom-notes-25-feb-2026-2/)
24th/Feb/2026 – [Click Here](https://directai.blog/2026/02/24/gen-ai-developer-classroom-notes-24-feb-2026/)
22nd/Feb/2026 – [Click Here](https://directai.blog/2026/02/22/gen-ai-developer-classroom-notes-22-feb-2026/)
21st/Feb/2026 – [Click Here](https://directai.blog/2026/02/21/gen-ai-developer-classroom-notes-21-feb-2026/)
19th/Feb/2026 – [Click Here](https://directai.blog/2026/02/19/gen-ai-developer-classroom-notes-19-feb-2026/)
18th/Feb/2026 – [Click Here](https://directai.blog/2026/02/18/gen-ai-developer-classroom-notes-18-feb-2026/)
17th/Feb/2026 – [Click Here](https://directai.blog/2026/02/17/gen-ai-developer-classroom-notes-17-feb-2026/)
https://github.com/raffeemdai/LangChain_LangGraph
https://github.com/raffeemdai/RAG_BY_ME
https://github.com/raffeemdai/RAG_BY_ME/blob/main/AI%20Engineering%20Guidebook%20Full-compressed.pdf   this book is from dailydose datascience author
https://github.com/raffeemdai/ai-engineering-interview-questions
https://github.com/raffeemdai/LangChain_LangGraph
https://github.com/raffeemdai/LangChain_LangGraph/blob/main/prompt_engineering_notes_euri.md
