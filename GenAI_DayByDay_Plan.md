


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

# Gen AI Developer Classroom Topics Summary

## Foundation For Gen AI

| Class Date | Blog URL | Topics Covered |
|---|---|---|
| 23-Jun-2026 | https://directai.blog/2026/06/23/gen-ai-developer-classroom-notes-23-jun-2026/ | Introduction to LLMs, next-token prediction, probability distributions, temperature, prompting, transformers, SDKs (OpenAI/Gemini/Claude), LangChain overview |
| 26-Jun-2026 | https://directai.blog/2026/06/26/gen-ai-developer-classroom-notes-26-jun-2026/ | How LLMs understand prompts, tokenization, embeddings, transformers, probability-based output generation |

## Deep Agents

| Class Date | Blog URL | Topics Covered |
|---|---|---|
| 21-May-2026 | https://directai.blog/2026/05/21/gen-ai-developer-classroom-notes-21-may-2026/ | Course overview, RAG, vector databases, agents, MCP, Deep Agents, fine-tuning, governance, guardrails, grounding, deployment, GenAI ecosystem |
| 25-May-2026 | https://directai.blog/2026/05/25/gen-ai-developer-classroom-notes-25-may-2026/ | Journey into agents, model types (self-hosted/cloud/vendor), credentials, SDKs, LangChain abstractions |
| 26-May-2026 | https://directai.blog/2026/05/26/gen-ai-developer-classroom-notes-26-may-2026/ | LLM fundamentals, multimodal models, tool calling, ReAct agents, context limits, context engineering, hallucinations, guardrails |
| 27-May-2026 | https://directai.blog/2026/05/27/gen-ai-developer-classroom-notes-27-may-2026/ | Environment setup, cloud SDKs, authentication, Gemini connectivity, LangChain basics, first LLM application |
| 29-May-2026 | https://directai.blog/2026/05/29/gen-ai-developer-classroom-notes-29-may-2026/ | AWS Bedrock integration, LangChain agents, tool creation, calculator agent example, agent architecture |
| 04-Jun-2026 | https://directai.blog/2026/06/04/gen-ai-developer-classroom-notes-04-jun-2026/ | Deep Agents introduction, planning, todos, skills, subagents, filesystems, agent customization |
| 05-Jun-2026 | https://directai.blog/2026/06/05/gen-ai-developer-classroom-notes-05-jun-2026/ | Deep Agent internals, orchestration, subagents, backends, middleware, travel planner example |
| 08-Jun-2026 | https://directai.blog/2026/06/08/gen-ai-developer-classroom-notes-08-jun-2026/ | LangGraph Studio setup, LangSmith configuration, Deep Agent project creation, tools, middleware |
| 09-Jun-2026 | https://directai.blog/2026/06/09/gen-ai-developer-classroom-notes-09-jun-2026/ | Middleware concepts, AgentMiddleware, todo-list middleware, travel planning agent workflow |
| 11-Jun-2026 | https://directai.blog/2026/06/11/gen-ai-developer-classroom-notes-11-jun-2026/ | Deep Agent backends, StateBackend, FileSystemBackend, StoreBackend, CompositeBackend, article writer example |
| 12-Jun-2026 | https://directai.blog/2026/06/12/gen-ai-developer-classroom-notes-12-jun-2026/ | Skills in Deep Agents, SKILL.md, FileSystemBackend integration, skill discovery and execution |
| 15-Jun-2026 | https://directai.blog/2026/06/15/gen-ai-developer-classroom-notes-15-jun-2026/ | Advanced skills examples, subagents, subagent implementation patterns |
| 17-Jun-2026 | https://directai.blog/2026/06/17/gen-ai-developer-classroom-notes-17-jun-2026/ | Subagents continuation, dictionary-based subagents, compiled LangGraph subagents, entrance-test generator example |
| 18-Jun-2026 | https://directai.blog/2026/06/18/gen-ai-developer-classroom-notes-18-jun-2026/ | Structured output, response formats, async subagents, parallel execution, LangGraph multi-agent architecture |
| 21-Jun-2026 | https://directai.blog/2026/06/21/gen-ai-developer-classroom-notes-21-jun-2026/ | Async subagents, context engineering, context compression, context isolation, memory management, customer churn agent project |
| 19-Jul-2026 | https://directai.blog/2026/07/19/gen-ai-developer-classroom-notes-19-jul-2026/ | Customer churn signalling agent, synthetic data generation, SQLite setup, SQL agents, subagent design, retention analytics |

# MCP and Foundation for Gen AI - Topics Summary

## MCP

| Class Date | Blog URL | Topics Covered |
|---|---|---|
| 23-May-2026 | https://directai.blog/2026/05/23/gen-ai-developer-classroom-notes-23-may-2026/ | Library MCP implementation, MCP Inspector, MCP client setup, REST API to MCP design, OAuth/JWT authorization, Docker deployment |
| 20-May-2026 | https://directai.blog/2026/05/20/gen-ai-developer-classroom-notes-20-may-2026/ | MCP enhancements, library MCP exercises, issue_book implementation, top-performing books analytics |
| 19-May-2026 | https://directai.blog/2026/05/19/gen-ai-developer-classroom-notes-19-may-2026/ | College Library MCP design, librarian/student workflows, MCP tools and resources, database schema, analytics and reporting |
| 17-May-2026 | https://directai.blog/2026/05/17/gen-ai-developer-classroom-notes-17-may-2026/ | MCP client development with FastMCP, LangChain integration, AI agent using MCP servers |
| 16-May-2026 | https://directai.blog/2026/05/16/gen-ai-developer-classroom-notes-16-may-2026/ | Library MCP, MySQL connectivity, database operations, MCP integration with Claude |
| 14-May-2026 | https://directai.blog/2026/05/14/gen-ai-developer-classroom-notes-14-may-2026-2/ | Enterprise MCP deployment, Docker containers, MySQL, phpMyAdmin, environment variables, database integration |

## Foundation For Gen AI

| Class Date | Blog URL | Topics Covered |
|---|---|---|
| 21-Apr-2026 | https://directai.blog/2026/04/21/gen-ai-developer-classroom-notes-21-apr-2026/ | AI foundations, Machine Learning, Neural Networks, Deep Learning, Vector Representations, GPUs, LLM deployment approaches |
| 22-Apr-2026 | https://directai.blog/2026/04/22/gen-ai-developer-classroom-notes-22-apr-2026/ | LLM fundamentals, next-token prediction, multimodal AI (text, image, audio, video, code), token generation process |
| 23-Apr-2026 | https://directai.blog/2026/04/23/gen-ai-developer-classroom-notes-23-apr-2026/ | LLM training pipeline, supervised fine-tuning (SFT), RLHF, distillation, reasoning models |
| 26-Apr-2026 | https://directai.blog/2026/04/26/gen-ai-developer-classroom-notes-26-apr-2026/ | Tool calling, interaction with external systems, MCP fundamentals, MCP client-server architecture, MCP ecosystem and use cases |
