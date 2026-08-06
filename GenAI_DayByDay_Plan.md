


# RAG — Date-wise Topics Covered

# Gen AI Developer Classroom Notes

| Class Date | Topics Covered | Blog URL |
|---|---|---|
| 25 Jul 2026 | Handling document updates in RAG (full re-index, content hashing + incremental upsert, LangChain's built-in indexing API, document-level versioning + soft delete); NCERT Book RAG project | https://directai.blog/2026/07/25/gen-ai-developer-classroom-notes-25-jul-2026/ |
| 23 Jul 2026 | Evaluation frameworks for LLM outputs; DeepEval setup and its three building blocks (TestCase, Metric, Evaluation Runner) | https://directai.blog/2026/07/23/gen-ai-developer-classroom-notes-23-jul-2026/ |
| 22 Jul 2026 | User interface options for RAG (chat API, Streamlit, Gradio); Streamlit basics — session state, widget keys vs plain variables, and caching (cache_data, cache_resource) | https://directai.blog/2026/07/22/gen-ai-developer-classroom-notes-22-jul-2026-2/ |
| 21 Jul 2026 | Vector database retrieval techniques — Dense, Sparse (keyword/BM25), and Hybrid/Fuse search | https://directai.blog/2026/07/21/gen-ai-developer-classroom-notes-21-jul-2026/ |
| 16 Jul 2026 | Recursive character splitting; vector storage challenges (source doc updates); embedding storage layer internals; indexing algorithms (Flat, IVF, HNSW); distance metrics (cosine, dot product, Euclidean); hybrid search; metadata filtering and CRUD updates | https://directai.blog/2026/07/16/gen-ai-developer-classroom-notes-16-jul-2026/ |
| 15 Jul 2026 | Section-aware chunking for the NCERT book (splitting by section then by length with overlap) | https://directai.blog/2026/07/15/gen-ai-developer-classroom-notes-15-jul-2026/ |
| 13 Jul 2026 | Dealing with images in RAG (image captioning vs image embedding) — Multi-modal RAG | https://directai.blog/2026/07/13/gen-ai-developer-classroom-notes-13-jul-2026-2/ |
| 11 Jul 2026 | Productionizing RAG (chunking, vector stores, RAG scoring, guardrails, multi-modal data); CBSE Teacher idea — extracting text/images from PDFs, handling flowcharts/tables | https://directai.blog/2026/07/11/gen-ai-developer-classroom-notes-11-jul-2026/ |
| 9 Jul 2026 | Building RAG with relational databases (schema understanding, NL-to-query); SQLDatabaseToolkit; NoSQL (MongoDB) | https://directai.blog/2026/07/09/gen-ai-developer-classroom-notes-09-jul-2026/ |
| 8 Jul 2026 | Building the first simple RAG pipeline | https://directai.blog/2026/07/08/gen-ai-developer-classroom-notes-08-jul-2026/ |
| 7 Jul 2026 | Embeddings (Word2Vec origins, embedding models); Vector databases (Chroma, FAISS, pgvector, MongoDB, Pinecone, Weaviate, cloud providers); indexing pipeline exercise | https://directai.blog/2026/07/07/gen-ai-developer-classroom-notes-07-jul-2026/ |
| 6 Jul 2026 | Chunking/splitting strategies in RAG; chunk overlap; metadata during load-and-split | https://directai.blog/2026/07/06/gen-ai-developer-classroom-notes-06-jul-2026/ |
| 4 Jul 2026 | Document and Document Loaders (text, PDF, CSV, directory loaders) | https://directai.blog/2026/07/04/gen-ai-developer-classroom-notes-04-jul-2026/ |
| 2 Jul 2026 | Setting up GCP with LangChain (gcloud SDK, auth, Vertex AI project setup); ChatPromptTemplate; Output Parsers | https://directai.blog/2026/07/02/gen-ai-developer-classroom-notes-02-jul-2026/ |
| 30 Jun 2026 | Runnable base class and its 4 methods (invoke, ainvoke, batch, stream); LCEL chains; RAG phases (indexing vs retrieval); vector databases and key RAG terms | https://directai.blog/2026/06/30/gen-ai-developer-classroom-notes-30-jun-2026/ |
| 29 Jun 2026 | Chaining with LCEL; prompts (system/user); LangChain message types (SystemMessage, HumanMessage, AIMessage, ToolMessage); first LLM interaction setup with Gemini | https://directai.blog/2026/06/29/gen-ai-developer-classroom-notes-29-jun-2026/ |
| 27 Jun 2026 | Vectors and vector arithmetic (king − man + woman = queen analogy); intro to vector databases; RAG trailer/overview | https://directai.blog/2026/06/27/gen-ai-developer-classroom-notes-27-jun-2026/ |

*Topics above are taken directly from each post's actual heading/content, not just the source list's group label.*

---

## Agentic AI Topic Group

| Class Date | Topics Covered | Blog URL |
|---|---|---|
| 7 Jun 2026 | Customer Support Tickets and grievance project — adding utils, state, SQL agent service, and a LangGraph graph; visualizing/fixing the graph in `langgraph dev` | https://directai.blog/2026/06/07/gen-ai-developer-classroom-notes-07-jun-2026/ |
| 27 Apr 2026 | Customer Support Tickets and grievance — setting up the customer care Postgres DB with Docker/Docker Compose; SQLDatabaseToolkit + LangGraph natural-language DB queries | https://directai.blog/2026/04/27/gen-ai-developer-classroom-notes-27-apr-2026/ |
| 22 Apr 2026 | Sub graphs in LangGraph | https://directai.blog/2026/04/22/gen-ai-developer-classroom-notes-22-apr-2026-3/ |
| 19 Apr 2026 | Customer Support Tickets and grievance — case types (known/new/serious issues), candidate domains, MVP scope; things to learn (pause/resume workflows, human-in-the-loop, sub graphs, supervisor, langgraph-client, REST API exposure) | https://directai.blog/2026/04/19/gen-ai-developer-classroom-notes-19-apr-2026-2/ |
| 18 Apr 2026 | Long-term memory in LangGraph (Runtime, store, Namespace) vs short-term memory; small capstone ideas (customer support, application failure analysis, proposal processing) | https://directai.blog/2026/04/18/gen-ai-developer-classroom-notes-18-apr-2026/ |
| 15 Apr 2026 | LangGraph primitives — threads, config/thread_id, checkpointer, InMemorySaver, multi-session scenarios | https://directai.blog/2026/04/15/gen-ai-developer-classroom-notes-15-apr-2026-2/ |
| 14 Apr 2026 | MessagesState shortcut and built-in add_messages reducer; making stateless graph invocations stateful (memory, checkpoints, threads) | https://directai.blog/2026/04/14/gen-ai-developer-classroom-notes-14-apr-2026/ |
| 11 Apr 2026 | ReAct (Reasoning + Acting) technique; using LLMs in LangGraph with ToolNode; building and fixing a basic ReAct loop | https://directai.blog/2026/04/11/gen-ai-developer-classroom-notes-11-apr-2026/ |
| 9 Apr 2026 | Conditional edges and loops in LangGraph | https://directai.blog/2026/04/09/gen-ai-developer-classroom-notes-09-apr-2026/ |
| 7 Apr 2026 | LangGraph partial state updates at nodes; parallel graphs; handling state-update conflicts with reducers (operator.add, custom reducer) | https://directai.blog/2026/04/07/gen-ai-developer-classroom-notes-07-apr-2026/ |
| 6 Apr 2026 | Intro to LangGraph (StateGraph, state, nodes, edges); building the first LangGraph app; running with `langgraph dev` / LangGraph Studio | https://directai.blog/2026/04/06/gen-ai-developer-classroom-notes-06-apr-2026/ |
| 4 Apr 2026 | Tools in LangChain continued (@tool decorator, docstrings); third-party tools/toolkits; Tavily internet search; Yahoo Finance stock tool | https://directai.blog/2026/04/04/gen-ai-developer-classroom-notes-04-apr-2026-2/ |
| 2 Apr 2026 | Tool calling fundamentals (bind_tools, ToolMessage); defining tools with @tool, type hints, docstrings; LangChain's create_agent (built on LangGraph) | https://directai.blog/2026/04/02/gen-ai-developer-classroom-notes-02-apr-2026/ |
| 1 Apr 2026 | LangChain Prompt Templates (BasePromptTemplate); exercise building a simple chain for "top n facts about a topic" | https://directai.blog/2026/04/01/gen-ai-developer-classroom-notes-01-apr-2026/ |
| 31 Mar 2026 | LangChain framework overview (Python/JS support, Runnable base class, BaseChatModel, message types, BasePromptTemplate); LangChain package structure (core, community, vendor packages); first project using GCP Vertex models | https://directai.blog/2026/03/31/gen-ai-developer-classroom-notes-31-mar-2026/ |
| 29 Mar 2026 | Configuring models from AWS Bedrock (IAM, region, ChatBedrockConverse) and Azure AI Foundry (az login, serverless models, LangChain/LangGraph integration) | https://directai.blog/2026/03/29/gen-ai-developer-classroom-notes-29-mar-2026/ |
| 28 Mar 2026 | Configuring the dev system for LLMs (git, python, uv, VS Code, GCP/AWS/Azure CLIs); enabling and calling Vertex AI (gemini-2.5-flash-lite) via LangChain | https://directai.blog/2026/03/28/gen-ai-developer-classroom-notes-28-mar-2026/ |
| 25 Mar 2026 | AI Agents vs simple rule-based chatbots; autonomous multi-step agentic workflows and multi-agent systems; deep agents (plans/skills, todos); customer support agent example; agent use cases in a university setting | https://directai.blog/2026/03/25/gen-ai-developer-classroom-notes-25-mar-2026/ |
| 24 Mar 2026 | Tool calling by LLMs — why it's needed (real-time data like stock price, weather, news); simulated agent–LLM tool-call exercises | https://directai.blog/2026/03/24/gen-ai-developer-classroom-notes-24-mar-2026/ |
| 23 Mar 2026 | Prompt engineering continued — Chain of Thought prompting, multiple-answer/plan-evaluation prompts, anatomy of a good prompt (role, task, audience, examples, constraints, output format); context engineering and memory (short/long term) | https://directai.blog/2026/03/23/gen-ai-developer-classroom-notes-23-mar-2026/ |
| 20 Mar 2026 | Writing effective prompts — clear/explicit prompts, structured prompting, few-shot prompting, role-based prompting exercise | https://directai.blog/2026/03/20/gen-ai-developer-classroom-notes-20-mar-2026/ |
| 18 Mar 2026 | Writing effective prompts — clear/explicit prompts and structured prompting (partial repeat of the 20-Mar session) | https://directai.blog/2026/03/18/gen-ai-developer-classroom-notes-18-mar-2026-2/ |
| 17 Mar 2026 | LLM terms (temperature, context length, tokens); LLM cost models (open-source vs proprietary); cloud LLM hosting (GCP Vertex, AWS Bedrock, Azure AI Foundry) and available models; lab credits plan; prompt engineering resources | https://directai.blog/2026/03/17/gen-ai-developer-classroom-notes-17-mar-2026/ |
| 16 Mar 2026 | LLM evolution (AI → ML → Neural Networks → Deep Learning → NLP → LLMs); transformers and next-word prediction; embeddings and semantic search | https://directai.blog/2026/03/16/gen-ai-developer-classroom-notes-16-mar-2026/ |
| 14 Mar 2026 | ⚠️ Source link is mismatched — it points to an older "How LLMs Work" post (Transformers, Attention, Temperature, Word Embeddings, Foundational Models like GPT/BERT) dated 14 Feb 2025, not a Mar 2026 session | https://directai.blog/2025/02/14/gen-ai-classroom-notes-14-02-2025/ |
| 13 Mar 2026 | Using AI to learn (ChatGPT, Claude, Gemini, NotebookLM); anatomy of a basic prompt (role, understanding level, question, constraints); example study-plan prompt | https://directai.blog/2026/03/13/gen-ai-developer-classroom-notes-13-mar-2026/ |
