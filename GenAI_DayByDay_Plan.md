




# Gen AI Developer Classroom Notes
# GenAI Revision Plan — Topic-wise, Basics → Advanced

Source: [GenAI_DayByDay_Plan.md](https://github.com/raffeemdai/REVIISON-PLAN/blob/main/GenAI_DayByDay_Plan.md) (raffeemdai/REVIISON-PLAN)

This plan covers five source groupings, per your request — **Fine Tuning**,
**Agentic AI**, **RAG**, and the two **Foundation For Gen Ai** blocks — all
found across the repo's main table and its separate `BLOG URL` file. The
original notes are ordered by class date; this version regroups them
**topic-wise**, starting from fundamentals and moving through fine-tuning,
agentic systems, and RAG, so you can study it as a structured course rather
than a chronological log. Each topic lists the original blog post(s) to
refer to, plus suggested external material to go deeper.

---

## PART A — GenAI & LLM Foundations

### 0.1 Introduction to Large Language Models
- **What to cover:** What LLMs are; content types they generate (text, code, images, audio, video); hosting options (self-hosted, cloud-hosted via AWS/Azure/GCP, direct provider hosting); open-source vs proprietary LLMs.
- **Refer to:** [17 Feb 2026 notes](https://directai.blog/2026/02/17/gen-ai-developer-classroom-notes-17-feb-2026/)

### 0.2 How LLMs work — from ELIZA to transformers
- **What to cover:** ELIZA (1966) as an early example; an ML model as a mathematical function; text-to-number conversion; word2vec (2013) and vector embeddings; semantic search; tokenization; LLMs as next-token-prediction neural networks; role of transformers.
- **Refer to:** [18 Feb 2026 notes](https://directai.blog/2026/02/18/gen-ai-developer-classroom-notes-18-feb-2026/)

### 0.3 How LLMs work — deep learning and neural network basics
- **What to cover:** LLM as a deep learning model; supervised vs unsupervised ML; neural networks modeled on the human brain; neurons arranged in layers; activation functions.
- **Refer to:** [19 Feb 2026 notes](https://directai.blog/2026/02/19/gen-ai-developer-classroom-notes-19-feb-2026/)

### 0.4 Transformers and Attention (deep dive)
- **What to cover:** Transformer architecture and components; prompting exercises to explain transformer components in order and trace what each does to a sample sentence; transformer visualizer tool.
- **Refer to:** [21 Feb 2026 notes](https://directai.blog/2026/02/21/gen-ai-developer-classroom-notes-21-feb-2026/)
- **Extra material:** Jay Alammar's "The Illustrated Transformer"; the in-class transformer visualizer tool mentioned in the notes

> Topics 0.1–0.4 are the true starting point — cover these before Topic 1.
> Topics 2 and 3 later on (16 Mar / 14 Mar sessions) revisit transformers,
> attention, and the AI→ML→DL→NLP→LLM evolution — treat those as reinforcement,
> not new material, once you've done 0.1–0.4.

### 1. How to use AI tools to learn + prompt basics
- **What to cover:** Using ChatGPT/Claude/Gemini/NotebookLM as learning aids; anatomy of a basic prompt (role, understanding level, question, constraints); example study-plan prompt.
- **Refer to:** [13 Mar 2026 notes](https://directai.blog/2026/03/13/gen-ai-developer-classroom-notes-13-mar-2026/)
- **Extra material:** Anthropic's [prompt engineering overview](https://docs.claude.com/en/docs/build-with-claude/prompt-engineering/overview)

### 2. How LLMs actually work
- **What to cover:** Transformers, attention mechanism, temperature, word embeddings, foundational models (GPT/BERT).
- **Refer to:** [14 Mar 2026 notes (linked source is mismatched — points to a 14 Feb 2025 post, but content is correct for this topic)](https://directai.blog/2025/02/14/gen-ai-classroom-notes-14-02-2025/)
- **Extra material:** 3Blue1Brown's "Attention in transformers" video series; Jay Alammar's blog "The Illustrated Transformer"

### 3. Evolution of LLMs
- **What to cover:** AI → ML → Neural Networks → Deep Learning → NLP → LLMs; transformers and next-word prediction; embeddings and semantic search.
- **Refer to:** [16 Mar 2026 notes](https://directai.blog/2026/03/16/gen-ai-developer-classroom-notes-16-mar-2026/)

### 4. Core LLM terminology & hosting landscape
- **What to cover:** Temperature, context length, tokens; open-source vs proprietary cost models; cloud LLM hosting (GCP Vertex AI, AWS Bedrock, Azure AI Foundry) and available models.
- **Refer to:** [17 Mar 2026 notes](https://directai.blog/2026/03/17/gen-ai-developer-classroom-notes-17-mar-2026/)
- **Extra material:** Vertex AI, Bedrock, and Azure AI Foundry "getting started" docs (check current docs, as model/pricing pages change often)

---

## PART A2 — Fine-Tuning Topic Group (ascending date order)

*This group is listed separately in the source repo's `BLOG URL` file (not in the main day-by-day table) and runs chronologically right after the foundations above and before the Agentic AI group.*

### 0.5 How LLMs are trained
- **What to cover:** The three major training phases — Pretraining, Supervised Fine-Tuning, Alignment (RLHF/preference tuning); the "guess the next word" intuition; tokenization → embeddings → transformer forward pass → loss (cross-entropy) → backpropagation; what pretraining vs SFT vs RLHF each actually teach the model; example pretraining data mix (CommonCrawl, books, code, Wikipedia, ArXiv, StackExchange).
- **Refer to:** [22 Feb 2026 notes](https://directai.blog/2026/02/22/gen-ai-developer-classroom-notes-22-feb-2026/)

### 0.6 Prompting vs RAG vs Fine-Tuning
- **What to cover:** A case study (fictional "Acme Airline" support bot) comparing three approaches to customizing LLM behavior — prompting, RAG, and fine-tuning; when fine-tuning is justified (e.g., needing a specific brand tone); Full Fine-Tuning vs PEFT at a glance.
- **Refer to:** [24 Feb 2026 notes](https://directai.blog/2026/02/24/gen-ai-developer-classroom-notes-24-feb-2026/)

### 0.7 Model types and fine-tuning techniques
- **What to cover:** Distillation; pre-trained vs instruct models and when to choose each (data volume matters — instruct models suit smaller datasets); Full Fine-Tuning vs PEFT; LoRA (adapters) and QLoRA.
- **Refer to:** [25 Feb 2026 notes](https://directai.blog/2026/02/25/gen-ai-developer-classroom-notes-25-feb-2026-2/)

### 0.8 Hardware for fine-tuning — GPU/TPU fundamentals
- **What to cover:** What a GPU is and why it's used for deep learning/fine-tuning; FLOPS; CUDA and CUDA cores; Tensor cores; VRAM and what it stores during fine-tuning (weights, activations, gradients, optimizer states); TPUs.
- **Refer to:** [26 Feb 2026 notes](https://directai.blog/2026/02/26/gen-ai-developer-classroom-notes-26-feb-2026/)

### 0.9 Fine-tuning tooling: Unsloth, dataset formats, and the end-to-end pipeline
- **What to cover:** Using Unsloth (VM/GPU or Google Colab); dataset formats (Alpaca, ChatML, ShareGPT, plain text, JSONL, HuggingFace Dataset); the full fine-tuning workflow from defining an objective through packaging, deployment, and monitoring.
- **Refer to:** [28 Feb 2026 notes](https://directai.blog/2026/02/28/gen-ai-developer-classroom-notes-28-feb-2026/)
- **Extra material:** [Unsloth dataset docs](https://unsloth.ai/docs/get-started/fine-tuning-llms-guide/datasets-guide), [Unsloth LoRA hyperparameters guide](https://unsloth.ai/docs/get-started/fine-tuning-llms-guide/lora-hyperparameters-guide)

### 0.10 Applied case study — ACME Bharat Airlines dataset design
- **What to cover:** Choosing a base model (Qwen 8B instruct, alternatives Llama/Mistral); designing SOP-driven scenario buckets (delays, cancellations, baggage, refunds, escalations); building a brand-specific response template; ShareGPT format for single-turn and multi-turn conversations; train/test split ratio (90/10).
- **Refer to:** [2 Mar 2026 notes](https://directai.blog/2026/03/02/gen-ai-developer-classroom-notes-02-mar-2026/)

### 0.11 Synthetic data generation for fine-tuning
- **What to cover:** Generating training data via few-shot prompting (giving the LLM sample complaints and asking it to generate hundreds more in the same style); a scenario generator → dataset generator → quality filter → final dataset → train/test split pipeline.
- **Refer to:** [5 Mar 2026 notes](https://directai.blog/2026/03/05/gen-ai-developer-classroom-notes-05-mar-2026/)

### 0.12 Building the JSONL dataset and Colab notebook setup
- **What to cover:** Converting a raw complaints spreadsheet into ShareGPT-style JSONL conversations via prompting; setting up the Google Colab notebook used for the actual fine-tuning run.
- **Refer to:** [7 Mar 2026 notes](https://directai.blog/2026/03/07/gen-ai-developer-classroom-notes-07-mar-2026-2/)

### 0.13 Understanding fine-tuning parameters
- **What to cover:** CPU vs GPU vs TPU; Unsloth model naming conventions; `max_seq_length`; `dtype` (auto, bfloat16, float16, float32) and hardware trade-offs; `load_in_4bit` quantization; LoRA parameters — rank, `lora_alpha`, `target_modules` (attention + feedforward projections), `lora_dropout`.
- **Refer to:** [9 Mar 2026 notes](https://directai.blog/2026/03/09/gen-ai-developer-classroom-notes-09-mar-2026/)
- **Extra material:** [CPU vs GPU vs TPU explainer](https://blog.bytebytego.com/p/ep205-cpu-vs-gpu-vs-tpu)

### 0.14 Tuning hyperparameters and reading training loss
- **What to cover:** Core QLoRA hyperparameters (`batch_size`, `gradient_accumulation_steps`, `warmup_steps`, `learning_rate`, `optim`, `weight_decay`, `lr_scheduler_type`); interpreting training loss ranges (poor → excellent → overfitting); response-only training (masking non-assistant tokens) and the danger of incorrect masking; a recommended starter LoRA/QLoRA config; a troubleshooting flow for loss not decreasing, exploding, or hitting zero.
- **Refer to:** [10 Mar 2026 notes](https://directai.blog/2026/03/10/gen-ai-developer-classroom-notes-10-mar-2026/)

### 0.15 Deploying the fine-tuned model
- **What to cover:** Loading an Unsloth-tuned model into Ollama for local inference; other deployment/inference options.
- **Refer to:** [11 Mar 2026 notes](https://directai.blog/2026/03/11/gen-ai-developer-classroom-notes-11-mar-2026/)
- **Extra material:** [Unsloth: saving to Ollama](https://unsloth.ai/docs/basics/inference-and-deployment/saving-to-ollama), [Unsloth inference & deployment overview](https://unsloth.ai/docs/basics/inference-and-deployment)

---

## PART A3 — Foundations Recap, Block 1 (ascending date order)

*Also listed separately under "Foundation For Gen Ai" in the source `BLOG URL` file. These sessions ran interleaved with the Agentic AI group and recap/extend core basics — good to revisit here since they sit right before deeper agent work.*

### 0.16 Core terms recap + how to access an LLM
- **What to cover:** Quick recap of core definitions — Artificial Intelligence, Machine Learning, Neural Network, Deep Learning, Vector Representation, GPU; three ways to use an LLM — directly from the model provider, via cloud services, or self-hosted on your own hardware.
- **Refer to:** [21 Apr 2026 notes](https://directai.blog/2026/04/21/gen-ai-developer-classroom-notes-21-apr-2026/)

### 0.17 What an LLM actually produces
- **What to cover:** LLMs fundamentally predict the next token; the same mechanism generates multiple modalities (text, image, audio, video, code); the token-by-token generation loop and when it stops.
- **Refer to:** [22 Apr 2026 notes](https://directai.blog/2026/04/22/gen-ai-developer-classroom-notes-22-apr-2026/)

### 0.18 LLM training methods, recap
- **What to cover:** A condensed recap of the training pipeline — Training Phase (pretraining), Supervised Fine-Tuning, RLHF, Distillation — plus Reasoning Models as a named category.
- **Refer to:** [23 Apr 2026 notes](https://directai.blog/2026/04/23/gen-ai-developer-classroom-notes-23-apr-2026/)

### 0.19 How models act on the world: tool calling and MCP
- **What to cover:** Why models need tool calling to interact with the outside world; how a tool-enabled model picks a tool and returns structured output; introduction to Model Context Protocol (MCP) — server (the tool) vs client (the agent) roles — and why MCP acts as a provider-agnostic standard ("like a type-C charger" analogy).
- **Refer to:** [26 Apr 2026 notes](https://directai.blog/2026/04/26/gen-ai-developer-classroom-notes-26-apr-2026/)
- **Note:** This session introduces MCP as a concept only. The full, separately-dated MCP topic group (30 Apr–23 May 2026) is not included in this plan, per your request to cover only Fine Tuning, Agentic AI, RAG, and the two Foundation blocks.

---

## PART B — Prompt & Context Engineering

### 5. Writing effective prompts (intro)
- **What to cover:** Clear/explicit prompting, structured prompting.
- **Refer to:** [18 Mar 2026 notes](https://directai.blog/2026/03/18/gen-ai-developer-classroom-notes-18-mar-2026-2/)

### 6. Writing effective prompts (full)
- **What to cover:** Clear/explicit prompts, structured prompting, few-shot prompting, role-based prompting exercise.
- **Refer to:** [20 Mar 2026 notes](https://directai.blog/2026/03/20/gen-ai-developer-classroom-notes-20-mar-2026/)

### 7. Advanced prompting & context engineering
- **What to cover:** Chain-of-Thought prompting, multi-answer/plan-evaluation prompts, anatomy of a good prompt (role, task, audience, examples, constraints, output format); context engineering; short-term vs long-term memory.
- **Refer to:** [23 Mar 2026 notes](https://directai.blog/2026/03/23/gen-ai-developer-classroom-notes-23-mar-2026/)

---

## PART C — Tool Calling & Agent Concepts

### 8. Why LLMs need tool calling
- **What to cover:** Limits of pure LLM knowledge (no real-time data); simulated agent–LLM tool-call exercises (stock price, weather, news).
- **Refer to:** [24 Mar 2026 notes](https://directai.blog/2026/03/24/gen-ai-developer-classroom-notes-24-mar-2026/)

### 9. AI Agents — concepts
- **What to cover:** Agents vs rule-based chatbots; autonomous multi-step agentic workflows; multi-agent systems; "deep agents" (plans/skills/todos); customer support agent example; use cases.
- **Refer to:** [25 Mar 2026 notes](https://directai.blog/2026/03/25/gen-ai-developer-classroom-notes-25-mar-2026/)

---

## PART D — Dev Environment & Cloud Model Setup

### 10. Setting up the dev system
- **What to cover:** git, python, uv, VS Code, GCP/AWS/Azure CLIs; enabling and calling Vertex AI (gemini-2.5-flash-lite) via LangChain.
- **Refer to:** [28 Mar 2026 notes](https://directai.blog/2026/03/28/gen-ai-developer-classroom-notes-28-mar-2026/)

### 11. Connecting AWS Bedrock and Azure AI Foundry
- **What to cover:** AWS IAM/region setup, `ChatBedrockConverse`; Azure `az login`, serverless models, LangChain/LangGraph integration.
- **Refer to:** [29 Mar 2026 notes](https://directai.blog/2026/03/29/gen-ai-developer-classroom-notes-29-mar-2026/)

---

## PART E — LangChain Fundamentals

### 12. LangChain framework overview
- **What to cover:** Python/JS support, `Runnable` base class, `BaseChatModel`, message types, `BasePromptTemplate`; package structure (core, community, vendor packages).
- **Refer to:** [31 Mar 2026 notes](https://directai.blog/2026/03/31/gen-ai-developer-classroom-notes-31-mar-2026/)
- **Extra material:** [LangChain official docs](https://python.langchain.com/docs/introduction/)

### 13. Prompt templates & first chain
- **What to cover:** `BasePromptTemplate`; exercise — build a simple chain for "top n facts about a topic."
- **Refer to:** [1 Apr 2026 notes](https://directai.blog/2026/04/01/gen-ai-developer-classroom-notes-01-apr-2026/)

### 14. Tool calling in LangChain
- **What to cover:** `bind_tools`, `ToolMessage`; defining tools with `@tool`, type hints, docstrings; `create_agent` (built on LangGraph).
- **Refer to:** [2 Apr 2026 notes](https://directai.blog/2026/04/02/gen-ai-developer-classroom-notes-02-apr-2026/)

### 15. Third-party tools & toolkits
- **What to cover:** `@tool` decorator patterns continued; Tavily internet search tool; Yahoo Finance stock tool.
- **Refer to:** [4 Apr 2026 notes](https://directai.blog/2026/04/04/gen-ai-developer-classroom-notes-04-apr-2026-2/)

---

## PART F — LangGraph: Building Agentic Workflows

### 16. Intro to LangGraph
- **What to cover:** `StateGraph`, state, nodes, edges; first LangGraph app; running with `langgraph dev` / LangGraph Studio.
- **Refer to:** [6 Apr 2026 notes](https://directai.blog/2026/04/06/gen-ai-developer-classroom-notes-06-apr-2026/)
- **Extra material:** [LangGraph official docs](https://langchain-ai.github.io/langgraph/)

### 17. State management in LangGraph
- **What to cover:** Partial state updates at nodes; parallel graphs; handling state-update conflicts with reducers (`operator.add`, custom reducers).
- **Refer to:** [7 Apr 2026 notes](https://directai.blog/2026/04/07/gen-ai-developer-classroom-notes-07-apr-2026/)

### 18. Conditional edges and loops
- **What to cover:** Branching logic and cyclical graphs in LangGraph.
- **Refer to:** [9 Apr 2026 notes](https://directai.blog/2026/04/09/gen-ai-developer-classroom-notes-09-apr-2026/)

### 19. ReAct pattern
- **What to cover:** Reasoning + Acting technique; using LLMs in LangGraph with `ToolNode`; building and debugging a basic ReAct loop.
- **Refer to:** [11 Apr 2026 notes](https://directai.blog/2026/04/11/gen-ai-developer-classroom-notes-11-apr-2026/)

### 20. Making graphs stateful (short-term memory)
- **What to cover:** `MessagesState` shortcut, built-in `add_messages` reducer; converting stateless invocations to stateful ones (memory, checkpoints, threads).
- **Refer to:** [14 Apr 2026 notes](https://directai.blog/2026/04/14/gen-ai-developer-classroom-notes-14-apr-2026/)

### 21. LangGraph memory primitives
- **What to cover:** Threads, `config`/`thread_id`, checkpointer, `InMemorySaver`, multi-session scenarios.
- **Refer to:** [15 Apr 2026 notes](https://directai.blog/2026/04/15/gen-ai-developer-classroom-notes-15-apr-2026-2/)

### 22. Long-term memory
- **What to cover:** `Runtime`, `store`, `Namespace` for long-term memory vs short-term memory; capstone brainstorm (customer support, application failure analysis, proposal processing).
- **Refer to:** [18 Apr 2026 notes](https://directai.blog/2026/04/18/gen-ai-developer-classroom-notes-18-apr-2026/)

### 23. Sub-graphs
- **What to cover:** Composing graphs from sub-graphs.
- **Refer to:** [22 Apr 2026 notes](https://directai.blog/2026/04/22/gen-ai-developer-classroom-notes-22-apr-2026-3/)

---

## PART G — Capstone Project: Customer Support Agent

### 24. Project scoping
- **What to cover:** Case types (known/new/serious issues), candidate domains, MVP scope; things to learn next (pause/resume workflows, human-in-the-loop, sub-graphs, supervisor pattern, `langgraph-client`, REST API exposure).
- **Refer to:** [19 Apr 2026 notes](https://directai.blog/2026/04/19/gen-ai-developer-classroom-notes-19-apr-2026-2/)

### 25. Database setup for the agent
- **What to cover:** Setting up Postgres via Docker/Docker Compose; `SQLDatabaseToolkit` + LangGraph natural-language DB queries.
- **Refer to:** [27 Apr 2026 notes](https://directai.blog/2026/04/27/gen-ai-developer-classroom-notes-27-apr-2026/)

### 26. Assembling the full agent
- **What to cover:** Adding utils, state, SQL agent service, and a LangGraph graph; visualizing/debugging the graph in `langgraph dev`.
- **Refer to:** [7 Jun 2026 notes](https://directai.blog/2026/06/07/gen-ai-developer-classroom-notes-07-jun-2026/)

---

## PART G2 — Foundations Recap, Block 2 (ascending date order)

*Also listed separately under "Foundation For Gen Ai" (second block) in the source `BLOG URL` file. These two sessions sit right before the RAG group begins and serve as a bridge/recap.*

### 0.20 LLM fundamentals recap — token prediction and provider tooling
- **What to cover:** Next-token prediction from a probability distribution, revisited via `<EOS>` and temperature; a visual tool for how an LLM generates tokens; provider SDKs (OpenAI, Gemini, Claude); why a unified interaction layer (LangChain) is needed for consistent tool use and memory handling across providers.
- **Refer to:** [23 Jun 2026 notes](https://directai.blog/2026/06/23/gen-ai-developer-classroom-notes-23-jun-2026/)
- **Extra material:** [Transformer Explainer (visualization tool)](https://poloclub.github.io/transformer-explainer/)

### 0.21 How LLMs process a prompt
- **What to cover:** LLMs don't understand raw text; the pipeline from tokenization → embeddings → transformer processing → output token probabilities.
- **Refer to:** [26 Jun 2026 notes](https://directai.blog/2026/06/26/gen-ai-developer-classroom-notes-26-jun-2026/)

---

## PART H — RAG Foundations

### 27. Vectors and vector databases (intro)
- **What to cover:** Vector arithmetic (king − man + woman = queen analogy); intro to vector databases; RAG overview.
- **Refer to:** [27 Jun 2026 notes](https://directai.blog/2026/06/27/gen-ai-developer-classroom-notes-27-jun-2026/)

### 28. LCEL and message types (recap for RAG)
- **What to cover:** Chaining with LCEL; system/user prompts; LangChain message types (`SystemMessage`, `HumanMessage`, `AIMessage`, `ToolMessage`); first LLM setup with Gemini.
- **Refer to:** [29 Jun 2026 notes](https://directai.blog/2026/06/29/gen-ai-developer-classroom-notes-29-jun-2026/)

### 29. Runnable methods & RAG phases
- **What to cover:** `Runnable` base class's 4 methods (`invoke`, `ainvoke`, `batch`, `stream`); LCEL chains; RAG phases (indexing vs retrieval); vector DB and RAG key terms.
- **Refer to:** [30 Jun 2026 notes](https://directai.blog/2026/06/30/gen-ai-developer-classroom-notes-30-jun-2026/)

### 30. GCP setup + prompt templates + output parsers
- **What to cover:** `gcloud` SDK, auth, Vertex AI project setup; `ChatPromptTemplate`; Output Parsers.
- **Refer to:** [2 Jul 2026 notes](https://directai.blog/2026/07/02/gen-ai-developer-classroom-notes-02-jul-2026/)

---

## PART I — RAG Pipeline: Indexing

### 31. Document loaders
- **What to cover:** `Document` object and Document Loaders (text, PDF, CSV, directory loaders).
- **Refer to:** [4 Jul 2026 notes](https://directai.blog/2026/07/04/gen-ai-developer-classroom-notes-04-jul-2026/)

### 32. Chunking/splitting strategies
- **What to cover:** Chunking/splitting strategies, chunk overlap, metadata during load-and-split.
- **Refer to:** [6 Jul 2026 notes](https://directai.blog/2026/07/06/gen-ai-developer-classroom-notes-06-jul-2026/)

### 33. Embeddings and vector databases
- **What to cover:** Embeddings (Word2Vec origins, embedding models); vector DB options (Chroma, FAISS, pgvector, MongoDB, Pinecone, Weaviate, cloud providers); indexing pipeline exercise.
- **Refer to:** [7 Jul 2026 notes](https://directai.blog/2026/07/07/gen-ai-developer-classroom-notes-07-jul-2026/)

### 34. First simple RAG pipeline
- **What to cover:** End-to-end build of a basic RAG pipeline.
- **Refer to:** [8 Jul 2026 notes](https://directai.blog/2026/07/08/gen-ai-developer-classroom-notes-08-jul-2026/)

### 35. RAG over relational/NoSQL databases
- **What to cover:** Schema understanding, NL-to-query; `SQLDatabaseToolkit`; NoSQL (MongoDB) retrieval.
- **Refer to:** [9 Jul 2026 notes](https://directai.blog/2026/07/09/gen-ai-developer-classroom-notes-09-jul-2026/)

---

## PART J — RAG in Production & Multi-modal

### 36. Productionizing RAG
- **What to cover:** Chunking strategy at scale, vector store choices, RAG scoring, guardrails, multi-modal data; CBSE Teacher idea case study (extracting text/images from PDFs, flowcharts, tables).
- **Refer to:** [11 Jul 2026 notes](https://directai.blog/2026/07/11/gen-ai-developer-classroom-notes-11-jul-2026/)

### 37. Multi-modal RAG (images)
- **What to cover:** Image captioning vs image embedding approaches.
- **Refer to:** [13 Jul 2026 notes](https://directai.blog/2026/07/13/gen-ai-developer-classroom-notes-13-jul-2026-2/)

### 38. Section-aware chunking (case study)
- **What to cover:** Splitting the NCERT book by section, then by length with overlap.
- **Refer to:** [15 Jul 2026 notes](https://directai.blog/2026/07/15/gen-ai-developer-classroom-notes-15-jul-2026/)

---

## PART K — Vector Database Internals & Retrieval

### 39. Vector DB internals
- **What to cover:** Recursive character splitting; vector storage challenges (handling source doc updates); embedding storage layer internals; indexing algorithms (Flat, IVF, HNSW); distance metrics (cosine, dot product, Euclidean); hybrid search; metadata filtering and CRUD updates.
- **Refer to:** [16 Jul 2026 notes](https://directai.blog/2026/07/16/gen-ai-developer-classroom-notes-16-jul-2026/)

### 40. Retrieval techniques
- **What to cover:** Dense retrieval, Sparse retrieval (keyword/BM25), Hybrid/Fuse search.
- **Refer to:** [21 Jul 2026 notes](https://directai.blog/2026/07/21/gen-ai-developer-classroom-notes-21-jul-2026/)

---

## PART L — RAG Interfaces, Evaluation & Maintenance

### 41. UI options for RAG apps
- **What to cover:** Chat API, Streamlit, Gradio; Streamlit basics — session state, widget keys vs plain variables, caching (`cache_data`, `cache_resource`).
- **Refer to:** [22 Jul 2026 notes](https://directai.blog/2026/07/22/gen-ai-developer-classroom-notes-22-jul-2026-2/)

### 42. Evaluating LLM/RAG outputs
- **What to cover:** Evaluation frameworks for LLM outputs; DeepEval setup and its three building blocks — TestCase, Metric, Evaluation Runner.
- **Refer to:** [23 Jul 2026 notes](https://directai.blog/2026/07/23/gen-ai-developer-classroom-notes-23-jul-2026/)
- **Extra material:** [DeepEval docs](https://deepeval.com/docs/getting-started)

### 43. Handling document updates in RAG (capstone)
- **What to cover:** Full re-index vs content hashing + incremental upsert vs LangChain's built-in indexing API vs document-level versioning + soft delete; applied in the NCERT Book RAG project.
- **Refer to:** [25 Jul 2026 notes](https://directai.blog/2026/07/25/gen-ai-developer-classroom-notes-25-jul-2026/)

---

## Suggested Study Order (at a glance)

1. **True basics** (Topics 0.1–0.4) — what LLMs are, ELIZA to transformers, neural network basics, attention deep dive
2. **Fine-Tuning group** (Topics 0.5–0.15) — how LLMs are trained, prompting vs RAG vs fine-tuning, PEFT/LoRA/QLoRA, GPU/TPU basics, Unsloth workflow, dataset design & synthetic data generation, hyperparameters, deployment to Ollama
3. **Foundations, reinforced** (Topics 1–4) — using AI tools, LLM evolution recap, key terms, hosting
4. **Foundations recap, Block 1** (Topics 0.16–0.19) — core terms, multimodal generation, training methods recap, tool calling & MCP intro
5. **Prompting** (Topics 5–7) — from basic to advanced prompt/context engineering
6. **Agent concepts** (Topics 8–9) — tool calling and agents conceptually
7. **Environment setup** (Topics 10–11) — get your dev stack ready
8. **LangChain basics** (Topics 12–15) — chains, prompt templates, tool calling
9. **LangGraph** (Topics 16–23) — state, control flow, memory, sub-graphs
10. **Capstone: Agent project** (Topics 24–26) — apply agentic concepts end-to-end
11. **Foundations recap, Block 2** (Topics 0.20–0.21) — token prediction, provider SDKs, prompt-processing pipeline (bridge into RAG)
12. **RAG foundations** (Topics 27–30) — vectors, RAG phases, setup
13. **RAG indexing pipeline** (Topics 31–35) — loaders, chunking, embeddings, first pipeline
14. **RAG production & multi-modal** (Topics 36–38)
15. **Vector DB internals & retrieval** (Topics 39–40)
16. **RAG UI, evaluation & maintenance** (Topics 41–43)

> Note: Item 2 in the original source (14 Mar 2026) has a mismatched blog
> link per the source repo's own annotation — it points to an older 14 Feb
> 2025 post. The content is still relevant to Topic 2 above, but flagged here
> for transparency.
# GenAI Revision Plan — Topic-wise, Basics → Advanced

Source: [GenAI_DayByDay_Plan.md](https://github.com/raffeemdai/REVIISON-PLAN/blob/main/GenAI_DayByDay_Plan.md) (raffeemdai/REVIISON-PLAN)

The original notes are ordered by class date. This version regroups the same
content **topic-wise**, starting from fundamentals and moving toward advanced
agentic/RAG systems, so you can study it as a structured course rather than a
chronological log. Each topic lists the original blog post(s) to refer to,
plus suggested external material to go deeper.

---

## PART A — GenAI & LLM Foundations

### 1. How to use AI tools to learn + prompt basics
- **What to cover:** Using ChatGPT/Claude/Gemini/NotebookLM as learning aids; anatomy of a basic prompt (role, understanding level, question, constraints); example study-plan prompt.
- **Refer to:** [13 Mar 2026 notes](https://directai.blog/2026/03/13/gen-ai-developer-classroom-notes-13-mar-2026/)
- **Extra material:** Anthropic's [prompt engineering overview](https://docs.claude.com/en/docs/build-with-claude/prompt-engineering/overview)

### 2. How LLMs actually work
- **What to cover:** Transformers, attention mechanism, temperature, word embeddings, foundational models (GPT/BERT).
- **Refer to:** [14 Mar 2026 notes (linked source is mismatched — points to a 14 Feb 2025 post, but content is correct for this topic)](https://directai.blog/2025/02/14/gen-ai-classroom-notes-14-02-2025/)
- **Extra material:** 3Blue1Brown's "Attention in transformers" video series; Jay Alammar's blog "The Illustrated Transformer"

### 3. Evolution of LLMs
- **What to cover:** AI → ML → Neural Networks → Deep Learning → NLP → LLMs; transformers and next-word prediction; embeddings and semantic search.
- **Refer to:** [16 Mar 2026 notes](https://directai.blog/2026/03/16/gen-ai-developer-classroom-notes-16-mar-2026/)

### 4. Core LLM terminology & hosting landscape
- **What to cover:** Temperature, context length, tokens; open-source vs proprietary cost models; cloud LLM hosting (GCP Vertex AI, AWS Bedrock, Azure AI Foundry) and available models.
- **Refer to:** [17 Mar 2026 notes](https://directai.blog/2026/03/17/gen-ai-developer-classroom-notes-17-mar-2026/)
- **Extra material:** Vertex AI, Bedrock, and Azure AI Foundry "getting started" docs (check current docs, as model/pricing pages change often)

---

## PART B — Prompt & Context Engineering

### 5. Writing effective prompts (intro)
- **What to cover:** Clear/explicit prompting, structured prompting.
- **Refer to:** [18 Mar 2026 notes](https://directai.blog/2026/03/18/gen-ai-developer-classroom-notes-18-mar-2026-2/)

### 6. Writing effective prompts (full)
- **What to cover:** Clear/explicit prompts, structured prompting, few-shot prompting, role-based prompting exercise.
- **Refer to:** [20 Mar 2026 notes](https://directai.blog/2026/03/20/gen-ai-developer-classroom-notes-20-mar-2026/)

### 7. Advanced prompting & context engineering
- **What to cover:** Chain-of-Thought prompting, multi-answer/plan-evaluation prompts, anatomy of a good prompt (role, task, audience, examples, constraints, output format); context engineering; short-term vs long-term memory.
- **Refer to:** [23 Mar 2026 notes](https://directai.blog/2026/03/23/gen-ai-developer-classroom-notes-23-mar-2026/)

---

## PART C — Tool Calling & Agent Concepts

### 8. Why LLMs need tool calling
- **What to cover:** Limits of pure LLM knowledge (no real-time data); simulated agent–LLM tool-call exercises (stock price, weather, news).
- **Refer to:** [24 Mar 2026 notes](https://directai.blog/2026/03/24/gen-ai-developer-classroom-notes-24-mar-2026/)

### 9. AI Agents — concepts
- **What to cover:** Agents vs rule-based chatbots; autonomous multi-step agentic workflows; multi-agent systems; "deep agents" (plans/skills/todos); customer support agent example; use cases.
- **Refer to:** [25 Mar 2026 notes](https://directai.blog/2026/03/25/gen-ai-developer-classroom-notes-25-mar-2026/)

---

## PART D — Dev Environment & Cloud Model Setup

### 10. Setting up the dev system
- **What to cover:** git, python, uv, VS Code, GCP/AWS/Azure CLIs; enabling and calling Vertex AI (gemini-2.5-flash-lite) via LangChain.
- **Refer to:** [28 Mar 2026 notes](https://directai.blog/2026/03/28/gen-ai-developer-classroom-notes-28-mar-2026/)

### 11. Connecting AWS Bedrock and Azure AI Foundry
- **What to cover:** AWS IAM/region setup, `ChatBedrockConverse`; Azure `az login`, serverless models, LangChain/LangGraph integration.
- **Refer to:** [29 Mar 2026 notes](https://directai.blog/2026/03/29/gen-ai-developer-classroom-notes-29-mar-2026/)

---

## PART E — LangChain Fundamentals

### 12. LangChain framework overview
- **What to cover:** Python/JS support, `Runnable` base class, `BaseChatModel`, message types, `BasePromptTemplate`; package structure (core, community, vendor packages).
- **Refer to:** [31 Mar 2026 notes](https://directai.blog/2026/03/31/gen-ai-developer-classroom-notes-31-mar-2026/)
- **Extra material:** [LangChain official docs](https://python.langchain.com/docs/introduction/)

### 13. Prompt templates & first chain
- **What to cover:** `BasePromptTemplate`; exercise — build a simple chain for "top n facts about a topic."
- **Refer to:** [1 Apr 2026 notes](https://directai.blog/2026/04/01/gen-ai-developer-classroom-notes-01-apr-2026/)

### 14. Tool calling in LangChain
- **What to cover:** `bind_tools`, `ToolMessage`; defining tools with `@tool`, type hints, docstrings; `create_agent` (built on LangGraph).
- **Refer to:** [2 Apr 2026 notes](https://directai.blog/2026/04/02/gen-ai-developer-classroom-notes-02-apr-2026/)

### 15. Third-party tools & toolkits
- **What to cover:** `@tool` decorator patterns continued; Tavily internet search tool; Yahoo Finance stock tool.
- **Refer to:** [4 Apr 2026 notes](https://directai.blog/2026/04/04/gen-ai-developer-classroom-notes-04-apr-2026-2/)

---

## PART F — LangGraph: Building Agentic Workflows

### 16. Intro to LangGraph
- **What to cover:** `StateGraph`, state, nodes, edges; first LangGraph app; running with `langgraph dev` / LangGraph Studio.
- **Refer to:** [6 Apr 2026 notes](https://directai.blog/2026/04/06/gen-ai-developer-classroom-notes-06-apr-2026/)
- **Extra material:** [LangGraph official docs](https://langchain-ai.github.io/langgraph/)

### 17. State management in LangGraph
- **What to cover:** Partial state updates at nodes; parallel graphs; handling state-update conflicts with reducers (`operator.add`, custom reducers).
- **Refer to:** [7 Apr 2026 notes](https://directai.blog/2026/04/07/gen-ai-developer-classroom-notes-07-apr-2026/)

### 18. Conditional edges and loops
- **What to cover:** Branching logic and cyclical graphs in LangGraph.
- **Refer to:** [9 Apr 2026 notes](https://directai.blog/2026/04/09/gen-ai-developer-classroom-notes-09-apr-2026/)

### 19. ReAct pattern
- **What to cover:** Reasoning + Acting technique; using LLMs in LangGraph with `ToolNode`; building and debugging a basic ReAct loop.
- **Refer to:** [11 Apr 2026 notes](https://directai.blog/2026/04/11/gen-ai-developer-classroom-notes-11-apr-2026/)

### 20. Making graphs stateful (short-term memory)
- **What to cover:** `MessagesState` shortcut, built-in `add_messages` reducer; converting stateless invocations to stateful ones (memory, checkpoints, threads).
- **Refer to:** [14 Apr 2026 notes](https://directai.blog/2026/04/14/gen-ai-developer-classroom-notes-14-apr-2026/)

### 21. LangGraph memory primitives
- **What to cover:** Threads, `config`/`thread_id`, checkpointer, `InMemorySaver`, multi-session scenarios.
- **Refer to:** [15 Apr 2026 notes](https://directai.blog/2026/04/15/gen-ai-developer-classroom-notes-15-apr-2026-2/)

### 22. Long-term memory
- **What to cover:** `Runtime`, `store`, `Namespace` for long-term memory vs short-term memory; capstone brainstorm (customer support, application failure analysis, proposal processing).
- **Refer to:** [18 Apr 2026 notes](https://directai.blog/2026/04/18/gen-ai-developer-classroom-notes-18-apr-2026/)

### 23. Sub-graphs
- **What to cover:** Composing graphs from sub-graphs.
- **Refer to:** [22 Apr 2026 notes](https://directai.blog/2026/04/22/gen-ai-developer-classroom-notes-22-apr-2026-3/)

---

## PART G — Capstone Project: Customer Support Agent

### 24. Project scoping
- **What to cover:** Case types (known/new/serious issues), candidate domains, MVP scope; things to learn next (pause/resume workflows, human-in-the-loop, sub-graphs, supervisor pattern, `langgraph-client`, REST API exposure).
- **Refer to:** [19 Apr 2026 notes](https://directai.blog/2026/04/19/gen-ai-developer-classroom-notes-19-apr-2026-2/)

### 25. Database setup for the agent
- **What to cover:** Setting up Postgres via Docker/Docker Compose; `SQLDatabaseToolkit` + LangGraph natural-language DB queries.
- **Refer to:** [27 Apr 2026 notes](https://directai.blog/2026/04/27/gen-ai-developer-classroom-notes-27-apr-2026/)

### 26. Assembling the full agent
- **What to cover:** Adding utils, state, SQL agent service, and a LangGraph graph; visualizing/debugging the graph in `langgraph dev`.
- **Refer to:** [7 Jun 2026 notes](https://directai.blog/2026/06/07/gen-ai-developer-classroom-notes-07-jun-2026/)

---

## PART H — RAG Foundations

### 27. Vectors and vector databases (intro)
- **What to cover:** Vector arithmetic (king − man + woman = queen analogy); intro to vector databases; RAG overview.
- **Refer to:** [27 Jun 2026 notes](https://directai.blog/2026/06/27/gen-ai-developer-classroom-notes-27-jun-2026/)

### 28. LCEL and message types (recap for RAG)
- **What to cover:** Chaining with LCEL; system/user prompts; LangChain message types (`SystemMessage`, `HumanMessage`, `AIMessage`, `ToolMessage`); first LLM setup with Gemini.
- **Refer to:** [29 Jun 2026 notes](https://directai.blog/2026/06/29/gen-ai-developer-classroom-notes-29-jun-2026/)

### 29. Runnable methods & RAG phases
- **What to cover:** `Runnable` base class's 4 methods (`invoke`, `ainvoke`, `batch`, `stream`); LCEL chains; RAG phases (indexing vs retrieval); vector DB and RAG key terms.
- **Refer to:** [30 Jun 2026 notes](https://directai.blog/2026/06/30/gen-ai-developer-classroom-notes-30-jun-2026/)

### 30. GCP setup + prompt templates + output parsers
- **What to cover:** `gcloud` SDK, auth, Vertex AI project setup; `ChatPromptTemplate`; Output Parsers.
- **Refer to:** [2 Jul 2026 notes](https://directai.blog/2026/07/02/gen-ai-developer-classroom-notes-02-jul-2026/)

---

## PART I — RAG Pipeline: Indexing

### 31. Document loaders
- **What to cover:** `Document` object and Document Loaders (text, PDF, CSV, directory loaders).
- **Refer to:** [4 Jul 2026 notes](https://directai.blog/2026/07/04/gen-ai-developer-classroom-notes-04-jul-2026/)

### 32. Chunking/splitting strategies
- **What to cover:** Chunking/splitting strategies, chunk overlap, metadata during load-and-split.
- **Refer to:** [6 Jul 2026 notes](https://directai.blog/2026/07/06/gen-ai-developer-classroom-notes-06-jul-2026/)

### 33. Embeddings and vector databases
- **What to cover:** Embeddings (Word2Vec origins, embedding models); vector DB options (Chroma, FAISS, pgvector, MongoDB, Pinecone, Weaviate, cloud providers); indexing pipeline exercise.
- **Refer to:** [7 Jul 2026 notes](https://directai.blog/2026/07/07/gen-ai-developer-classroom-notes-07-jul-2026/)

### 34. First simple RAG pipeline
- **What to cover:** End-to-end build of a basic RAG pipeline.
- **Refer to:** [8 Jul 2026 notes](https://directai.blog/2026/07/08/gen-ai-developer-classroom-notes-08-jul-2026/)

### 35. RAG over relational/NoSQL databases
- **What to cover:** Schema understanding, NL-to-query; `SQLDatabaseToolkit`; NoSQL (MongoDB) retrieval.
- **Refer to:** [9 Jul 2026 notes](https://directai.blog/2026/07/09/gen-ai-developer-classroom-notes-09-jul-2026/)

---

## PART J — RAG in Production & Multi-modal

### 36. Productionizing RAG
- **What to cover:** Chunking strategy at scale, vector store choices, RAG scoring, guardrails, multi-modal data; CBSE Teacher idea case study (extracting text/images from PDFs, flowcharts, tables).
- **Refer to:** [11 Jul 2026 notes](https://directai.blog/2026/07/11/gen-ai-developer-classroom-notes-11-jul-2026/)

### 37. Multi-modal RAG (images)
- **What to cover:** Image captioning vs image embedding approaches.
- **Refer to:** [13 Jul 2026 notes](https://directai.blog/2026/07/13/gen-ai-developer-classroom-notes-13-jul-2026-2/)

### 38. Section-aware chunking (case study)
- **What to cover:** Splitting the NCERT book by section, then by length with overlap.
- **Refer to:** [15 Jul 2026 notes](https://directai.blog/2026/07/15/gen-ai-developer-classroom-notes-15-jul-2026/)

---

## PART K — Vector Database Internals & Retrieval

### 39. Vector DB internals
- **What to cover:** Recursive character splitting; vector storage challenges (handling source doc updates); embedding storage layer internals; indexing algorithms (Flat, IVF, HNSW); distance metrics (cosine, dot product, Euclidean); hybrid search; metadata filtering and CRUD updates.
- **Refer to:** [16 Jul 2026 notes](https://directai.blog/2026/07/16/gen-ai-developer-classroom-notes-16-jul-2026/)

### 40. Retrieval techniques
- **What to cover:** Dense retrieval, Sparse retrieval (keyword/BM25), Hybrid/Fuse search.
- **Refer to:** [21 Jul 2026 notes](https://directai.blog/2026/07/21/gen-ai-developer-classroom-notes-21-jul-2026/)

---

## PART L — RAG Interfaces, Evaluation & Maintenance

### 41. UI options for RAG apps
- **What to cover:** Chat API, Streamlit, Gradio; Streamlit basics — session state, widget keys vs plain variables, caching (`cache_data`, `cache_resource`).
- **Refer to:** [22 Jul 2026 notes](https://directai.blog/2026/07/22/gen-ai-developer-classroom-notes-22-jul-2026-2/)

### 42. Evaluating LLM/RAG outputs
- **What to cover:** Evaluation frameworks for LLM outputs; DeepEval setup and its three building blocks — TestCase, Metric, Evaluation Runner.
- **Refer to:** [23 Jul 2026 notes](https://directai.blog/2026/07/23/gen-ai-developer-classroom-notes-23-jul-2026/)
- **Extra material:** [DeepEval docs](https://deepeval.com/docs/getting-started)

### 43. Handling document updates in RAG (capstone)
- **What to cover:** Full re-index vs content hashing + incremental upsert vs LangChain's built-in indexing API vs document-level versioning + soft delete; applied in the NCERT Book RAG project.
- **Refer to:** [25 Jul 2026 notes](https://directai.blog/2026/07/25/gen-ai-developer-classroom-notes-25-jul-2026/)

---

## Suggested Study Order (at a glance)

1. **Foundations** (Topics 1–4) — what LLMs are, how they work, key terms
2. **Prompting** (Topics 5–7) — from basic to advanced prompt/context engineering
3. **Agent concepts** (Topics 8–9) — tool calling and agents conceptually
4. **Environment setup** (Topics 10–11) — get your dev stack ready
5. **LangChain basics** (Topics 12–15) — chains, prompt templates, tool calling
6. **LangGraph** (Topics 16–23) — state, control flow, memory, sub-graphs
7. **Capstone: Agent project** (Topics 24–26) — apply agentic concepts end-to-end
8. **RAG foundations** (Topics 27–30) — vectors, RAG phases, setup
9. **RAG indexing pipeline** (Topics 31–35) — loaders, chunking, embeddings, first pipeline
10. **RAG production & multi-modal** (Topics 36–38)
11. **Vector DB internals & retrieval** (Topics 39–40)
12. **RAG UI, evaluation & maintenance** (Topics 41–43)

> Note: Item 2 in the original source (14 Mar 2026) has a mismatched blog
> link per the source repo's own annotation — it points to an older 14 Feb
> 2025 post. The content is still relevant to Topic 2 above, but flagged here
> for transparency.

## Fine Tuning Topic Group (ascending date order)

| Class Date  | Topics Covered                                                                                                                                                                                                                                                 | Blog URL                                                                                 |
| ----------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------ |
| 17 Feb 2026 | Introduction to Large Language Models — what LLMs are, content types they generate (text, code, images, audio, video); hosting options (self-hosted, cloud-hosted via AWS/Azure/GCP, direct provider hosting); open-source vs proprietary LLMs                | <https://directai.blog/2026/02/17/gen-ai-developer-classroom-notes-17-feb-2026/>            |
| 18 Feb 2026 | How LLMs work — ELIZA (1966) as an early example; ML model as a mathematical function; text-to-number conversion; word2vec (2013) and vector embeddings; semantic search; tokenization; LLMs as next-token-prediction neural networks; role of transformers    | <https://directai.blog/2026/02/18/gen-ai-developer-classroom-notes-18-feb-2026/>            |
| 19 Feb 2026 | How LLMs work continued — LLM as a deep learning model; supervised vs unsupervised ML; neural networks modeled on the human brain; neurons arranged in layers; activation functions                                                                            | <https://directai.blog/2026/02/19/gen-ai-developer-classroom-notes-19-feb-2026/>            |
| 21 Feb 2026 | How Transformers and Attention work — transformer architecture and components; prompting exercises to explain transformer components in order and trace what each component does to a sample sentence; transformer visualizer tool                            | <https://directai.blog/2026/02/21/gen-ai-developer-classroom-notes-21-feb-2026/>            |
| 22 Feb 2026 | How LLMs are trained — three major phases (Pretraining, Supervised Fine-Tuning, Alignment/RLHF); simple and technical walkthroughs of each phase (data collection, tokenization, forward pass, loss, backpropagation, instruction datasets, reward models, PPO); LLaMA 1 training dataset mix | <https://directai.blog/2026/02/22/gen-ai-developer-classroom-notes-22-feb-2026/>            |
| 24 Feb 2026 | Fine-tuning motivation via ACME Airline case study — comparing Prompting, RAG, and Fine-Tuning approaches for FAQs/complaints; why fine-tuning suits brand tone (empathy, soft tone); Full Fine-Tuning vs PEFT overview                                         | <https://directai.blog/2026/02/24/gen-ai-developer-classroom-notes-24-feb-2026/>            |
| 25 Feb 2026 | Distillation; pre-trained vs instruct base models and when to choose each (dataset size, who tunes them); Full Fine-Tuning vs PEFT; LoRA (adapters reduce trainable parameters); QLoRA                                                                          | <https://directai.blog/2026/02/25/gen-ai-developer-classroom-notes-25-feb-2026-2/>          |
| 26 Feb 2026 | Fine-tuning tooling with Unsloth; GPU fundamentals (parallel processing, FLOPS/TFLOPS/PFLOPS); CUDA and CUDA cores; Tensor cores; VRAM contents during fine-tuning (weights, activations, gradients, optimizer states); TPUs                                    | <https://directai.blog/2026/02/26/gen-ai-developer-classroom-notes-26-feb-2026/>            |
| 28 Feb 2026 | Fine-tuning with Unsloth continued — dataset formats (Alpaca, ChatML, ShareGPT, Plain Text, JSONL, Custom Prompt, HuggingFace Dataset); LoRA hyperparameters overview; end-to-end fine-tuning workflow from defining objective through deployment and monitoring | <https://directai.blog/2026/02/28/gen-ai-developer-classroom-notes-28-feb-2026/>            |
| 2 Mar 2026  | ACME Bharat Airlines scenario — choosing base model (Qwen 8B instruct vs LLaMA/Mistral); dataset planning (SOPs, scenario buckets like delays, cancellations, baggage, refunds); ShareGPT format deep dive (single-turn, multi-turn, metadata keys); 90/10 train-test split | <https://directai.blog/2026/03/02/gen-ai-developer-classroom-notes-02-mar-2026/>            |
| 5 Mar 2026  | Synthetic data generation for fine-tuning — complaint categories (flight delay, cancellation); data generation pipeline (scenario generator → dataset generator → quality filter → final dataset → train/test split); few-shot prompting to generate realistic complaint datasets | <https://directai.blog/2026/03/05/gen-ai-developer-classroom-notes-05-mar-2026/>            |
| 7 Mar 2026  | Building the fine-tuning dataset with Unsloth — converting an Excel complaints sheet into a ShareGPT-style JSONL dataset via prompting; setting up the Google Colab notebook and uploading the dataset for fine-tuning                                          | <https://directai.blog/2026/03/07/gen-ai-developer-classroom-notes-07-mar-2026-2/>          |
| 9 Mar 2026  | Understanding fine-tuning parameters — CPU vs GPU vs TPU; Unsloth model naming conventions; max_seq_length; dtype selection (auto, bfloat16, float16, float32); load_in_4bit quantization; LoRA parameters (rank, lora_alpha, target_modules, lora_dropout)     | <https://directai.blog/2026/03/09/gen-ai-developer-classroom-notes-09-mar-2026/>            |
| 10 Mar 2026 | Tuning hyperparameters and reading training loss — batch size, gradient accumulation, warmup steps, learning rate, optimizer, weight decay, LR scheduler; training loss quality bands; Unsloth QLoRA recommended starter config; response-only training and label-masking pitfalls; loss-troubleshooting flow (not decreasing, exploding, near zero) | <https://directai.blog/2026/03/10/gen-ai-developer-classroom-notes-10-mar-2026/>            |
| 11 Mar 2026 | Loading the fine-tuned model into Ollama and other deployment targets; wrapping up the Google Colab fine-tuning notebook                                                                                                                                         | <https://directai.blog/2026/03/11/gen-ai-developer-classroom-notes-11-mar-2026/>            |

*Topics above are taken directly from each post's actual heading/content, not just the source list's group label.*

## Agentic AI Topic Group (ascending date order)

| Class Date | Topics Covered | Blog URL |
|---|---|---|
| 13 Mar 2026 | Using AI to learn (ChatGPT, Claude, Gemini, NotebookLM); anatomy of a basic prompt (role, understanding level, question, constraints); example study-plan prompt | https://directai.blog/2026/03/13/gen-ai-developer-classroom-notes-13-mar-2026/ |
| 14 Mar 2026 | ⚠️ Source link is mismatched — it points to an older "How LLMs Work" post (Transformers, Attention, Temperature, Word Embeddings, Foundational Models like GPT/BERT) dated 14 Feb 2025, not a Mar 2026 session | https://directai.blog/2025/02/14/gen-ai-classroom-notes-14-02-2025/ |
| 16 Mar 2026 | LLM evolution (AI → ML → Neural Networks → Deep Learning → NLP → LLMs); transformers and next-word prediction; embeddings and semantic search | https://directai.blog/2026/03/16/gen-ai-developer-classroom-notes-16-mar-2026/ |
| 17 Mar 2026 | LLM terms (temperature, context length, tokens); LLM cost models (open-source vs proprietary); cloud LLM hosting (GCP Vertex, AWS Bedrock, Azure AI Foundry) and available models; lab credits plan; prompt engineering resources | https://directai.blog/2026/03/17/gen-ai-developer-classroom-notes-17-mar-2026/ |
| 18 Mar 2026 | Writing effective prompts — clear/explicit prompts and structured prompting (partial repeat of the 20-Mar session) | https://directai.blog/2026/03/18/gen-ai-developer-classroom-notes-18-mar-2026-2/ |
| 20 Mar 2026 | Writing effective prompts — clear/explicit prompts, structured prompting, few-shot prompting, role-based prompting exercise | https://directai.blog/2026/03/20/gen-ai-developer-classroom-notes-20-mar-2026/ |
| 23 Mar 2026 | Prompt engineering continued — Chain of Thought prompting, multiple-answer/plan-evaluation prompts, anatomy of a good prompt (role, task, audience, examples, constraints, output format); context engineering and memory (short/long term) | https://directai.blog/2026/03/23/gen-ai-developer-classroom-notes-23-mar-2026/ |
| 24 Mar 2026 | Tool calling by LLMs — why it's needed (real-time data like stock price, weather, news); simulated agent–LLM tool-call exercises | https://directai.blog/2026/03/24/gen-ai-developer-classroom-notes-24-mar-2026/ |
| 25 Mar 2026 | AI Agents vs simple rule-based chatbots; autonomous multi-step agentic workflows and multi-agent systems; deep agents (plans/skills, todos); customer support agent example; agent use cases in a university setting | https://directai.blog/2026/03/25/gen-ai-developer-classroom-notes-25-mar-2026/ |
| 28 Mar 2026 | Configuring the dev system for LLMs (git, python, uv, VS Code, GCP/AWS/Azure CLIs); enabling and calling Vertex AI (gemini-2.5-flash-lite) via LangChain | https://directai.blog/2026/03/28/gen-ai-developer-classroom-notes-28-mar-2026/ |
| 29 Mar 2026 | Configuring models from AWS Bedrock (IAM, region, ChatBedrockConverse) and Azure AI Foundry (az login, serverless models, LangChain/LangGraph integration) | https://directai.blog/2026/03/29/gen-ai-developer-classroom-notes-29-mar-2026/ |
| 31 Mar 2026 | LangChain framework overview (Python/JS support, Runnable base class, BaseChatModel, message types, BasePromptTemplate); LangChain package structure (core, community, vendor packages); first project using GCP Vertex models | https://directai.blog/2026/03/31/gen-ai-developer-classroom-notes-31-mar-2026/ |
| 1 Apr 2026 | LangChain Prompt Templates (BasePromptTemplate); exercise building a simple chain for "top n facts about a topic" | https://directai.blog/2026/04/01/gen-ai-developer-classroom-notes-01-apr-2026/ |
| 2 Apr 2026 | Tool calling fundamentals (bind_tools, ToolMessage); defining tools with @tool, type hints, docstrings; LangChain's create_agent (built on LangGraph) | https://directai.blog/2026/04/02/gen-ai-developer-classroom-notes-02-apr-2026/ |
| 4 Apr 2026 | Tools in LangChain continued (@tool decorator, docstrings); third-party tools/toolkits; Tavily internet search; Yahoo Finance stock tool | https://directai.blog/2026/04/04/gen-ai-developer-classroom-notes-04-apr-2026-2/ |
| 6 Apr 2026 | Intro to LangGraph (StateGraph, state, nodes, edges); building the first LangGraph app; running with `langgraph dev` / LangGraph Studio | https://directai.blog/2026/04/06/gen-ai-developer-classroom-notes-06-apr-2026/ |
| 7 Apr 2026 | LangGraph partial state updates at nodes; parallel graphs; handling state-update conflicts with reducers (operator.add, custom reducer) | https://directai.blog/2026/04/07/gen-ai-developer-classroom-notes-07-apr-2026/ |
| 9 Apr 2026 | Conditional edges and loops in LangGraph | https://directai.blog/2026/04/09/gen-ai-developer-classroom-notes-09-apr-2026/ |
| 11 Apr 2026 | ReAct (Reasoning + Acting) technique; using LLMs in LangGraph with ToolNode; building and fixing a basic ReAct loop | https://directai.blog/2026/04/11/gen-ai-developer-classroom-notes-11-apr-2026/ |
| 14 Apr 2026 | MessagesState shortcut and built-in add_messages reducer; making stateless graph invocations stateful (memory, checkpoints, threads) | https://directai.blog/2026/04/14/gen-ai-developer-classroom-notes-14-apr-2026/ |
| 15 Apr 2026 | LangGraph primitives — threads, config/thread_id, checkpointer, InMemorySaver, multi-session scenarios | https://directai.blog/2026/04/15/gen-ai-developer-classroom-notes-15-apr-2026-2/ |
| 18 Apr 2026 | Long-term memory in LangGraph (Runtime, store, Namespace) vs short-term memory; small capstone ideas (customer support, application failure analysis, proposal processing) | https://directai.blog/2026/04/18/gen-ai-developer-classroom-notes-18-apr-2026/ |
| 19 Apr 2026 | Customer Support Tickets and grievance — case types (known/new/serious issues), candidate domains, MVP scope; things to learn (pause/resume workflows, human-in-the-loop, sub graphs, supervisor, langgraph-client, REST API exposure) | https://directai.blog/2026/04/19/gen-ai-developer-classroom-notes-19-apr-2026-2/ |
| 22 Apr 2026 | Sub graphs in LangGraph | https://directai.blog/2026/04/22/gen-ai-developer-classroom-notes-22-apr-2026-3/ |
| 27 Apr 2026 | Customer Support Tickets and grievance — setting up the customer care Postgres DB with Docker/Docker Compose; SQLDatabaseToolkit + LangGraph natural-language DB queries | https://directai.blog/2026/04/27/gen-ai-developer-classroom-notes-27-apr-2026/ |
| 7 Jun 2026 | Customer Support Tickets and grievance project — adding utils, state, SQL agent service, and a LangGraph graph; visualizing/fixing the graph in `langgraph dev` | https://directai.blog/2026/06/07/gen-ai-developer-classroom-notes-07-jun-2026/ |

---

## RAG Topic Group (ascending date order)

| Class Date | Topics Covered | Blog URL |
|---|---|---|
| 27 Jun 2026 | Vectors and vector arithmetic (king − man + woman = queen analogy); intro to vector databases; RAG trailer/overview | https://directai.blog/2026/06/27/gen-ai-developer-classroom-notes-27-jun-2026/ |
| 29 Jun 2026 | Chaining with LCEL; prompts (system/user); LangChain message types (SystemMessage, HumanMessage, AIMessage, ToolMessage); first LLM interaction setup with Gemini | https://directai.blog/2026/06/29/gen-ai-developer-classroom-notes-29-jun-2026/ |
| 30 Jun 2026 | Runnable base class and its 4 methods (invoke, ainvoke, batch, stream); LCEL chains; RAG phases (indexing vs retrieval); vector databases and key RAG terms | https://directai.blog/2026/06/30/gen-ai-developer-classroom-notes-30-jun-2026/ |
| 2 Jul 2026 | Setting up GCP with LangChain (gcloud SDK, auth, Vertex AI project setup); ChatPromptTemplate; Output Parsers | https://directai.blog/2026/07/02/gen-ai-developer-classroom-notes-02-jul-2026/ |
| 4 Jul 2026 | Document and Document Loaders (text, PDF, CSV, directory loaders) | https://directai.blog/2026/07/04/gen-ai-developer-classroom-notes-04-jul-2026/ |
| 6 Jul 2026 | Chunking/splitting strategies in RAG; chunk overlap; metadata during load-and-split | https://directai.blog/2026/07/06/gen-ai-developer-classroom-notes-06-jul-2026/ |
| 7 Jul 2026 | Embeddings (Word2Vec origins, embedding models); Vector databases (Chroma, FAISS, pgvector, MongoDB, Pinecone, Weaviate, cloud providers); indexing pipeline exercise | https://directai.blog/2026/07/07/gen-ai-developer-classroom-notes-07-jul-2026/ |
| 8 Jul 2026 | Building the first simple RAG pipeline | https://directai.blog/2026/07/08/gen-ai-developer-classroom-notes-08-jul-2026/ |
| 9 Jul 2026 | Building RAG with relational databases (schema understanding, NL-to-query); SQLDatabaseToolkit; NoSQL (MongoDB) | https://directai.blog/2026/07/09/gen-ai-developer-classroom-notes-09-jul-2026/ |
| 11 Jul 2026 | Productionizing RAG (chunking, vector stores, RAG scoring, guardrails, multi-modal data); CBSE Teacher idea — extracting text/images from PDFs, handling flowcharts/tables | https://directai.blog/2026/07/11/gen-ai-developer-classroom-notes-11-jul-2026/ |
| 13 Jul 2026 | Dealing with images in RAG (image captioning vs image embedding) — Multi-modal RAG | https://directai.blog/2026/07/13/gen-ai-developer-classroom-notes-13-jul-2026-2/ |
| 15 Jul 2026 | Section-aware chunking for the NCERT book (splitting by section then by length with overlap) | https://directai.blog/2026/07/15/gen-ai-developer-classroom-notes-15-jul-2026/ |
| 16 Jul 2026 | Recursive character splitting; vector storage challenges (source doc updates); embedding storage layer internals; indexing algorithms (Flat, IVF, HNSW); distance metrics (cosine, dot product, Euclidean); hybrid search; metadata filtering and CRUD updates | https://directai.blog/2026/07/16/gen-ai-developer-classroom-notes-16-jul-2026/ |
| 21 Jul 2026 | Vector database retrieval techniques — Dense, Sparse (keyword/BM25), and Hybrid/Fuse search | https://directai.blog/2026/07/21/gen-ai-developer-classroom-notes-21-jul-2026/ |
| 22 Jul 2026 | User interface options for RAG (chat API, Streamlit, Gradio); Streamlit basics — session state, widget keys vs plain variables, and caching (cache_data, cache_resource) | https://directai.blog/2026/07/22/gen-ai-developer-classroom-notes-22-jul-2026-2/ |
| 23 Jul 2026 | Evaluation frameworks for LLM outputs; DeepEval setup and its three building blocks (TestCase, Metric, Evaluation Runner) | https://directai.blog/2026/07/23/gen-ai-developer-classroom-notes-23-jul-2026/ |
| 25 Jul 2026 | Handling document updates in RAG (full re-index, content hashing + incremental upsert, LangChain's built-in indexing API, document-level versioning + soft delete); NCERT Book RAG project | https://directai.blog/2026/07/25/gen-ai-developer-classroom-notes-25-jul-2026/ |

*Topics above are taken directly from each post's actual heading/content, not just the source list's group label.*
