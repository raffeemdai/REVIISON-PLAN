 langgraph campusx 

 Langgraph campusx code github url :  https://github.com/campusx-official/langgraph-tutorials/blob/main/1_bmi_workflow.ipynb

 
important chapters

Ch 3 LangChain vs LangGraph

Ch 4 Core Concepts

Ch 5-8 Workflow Types

Ch 10 Persistence

Ch 17 Tools

Ch 19 RAG

Ch 20 HITL

Ch 15-16 LangSmith

Ch 21 Subgraphs

Ch 22-24 Memory

Ch 25 Planning Agent



# LangGraph Interview Cheat Sheet

## Chapter-wise Important Topics

| Chapter | Chapter Name | Important Topics |
|----------|-------------|------------------|
| 3 | LangChain vs LangGraph | Stateful vs Stateless, Event-Driven Execution, Nested Workflows, Observability, Framework Selection, LangChain + LangGraph Together |
| 4 | LangGraph Core Concepts | State, Nodes, Edges, Reducers, Execution Model, Supersteps, Message Passing, Workflow Patterns |
| 5 | Sequential Workflows | StateGraph, START/END Nodes, Prompt Chaining, Graph Construction, invoke(), compile() |
| 6 | Parallel Workflows | Parallel Execution, Reducers, Partial State Updates, Structured Output, State Merging |
| 7 | Conditional Workflows | Routing Functions, add_conditional_edges(), Conditional Branching, Decision Logic |
| 8 | Iterative (Looping) Workflows | Evaluation Loops, Feedback Loops, Evaluator-Optimizer Pattern, Max Iterations, Self-Improvement |
| 10 | Persistence | Checkpointer, Checkpoints, Threads, Thread ID, MemorySaver, SqliteSaver, PostgresSaver, Time Travel, Fault Tolerance |
| 15 | LangSmith Fundamentals | Observability, Tracing, Debugging, Monitoring, Project-Trace-Run Hierarchy, Evaluation |
| 17 | Tools in LangGraph | ToolNode, @tool, bind_tools(), tools_condition(), Feedback Loop, Tool Calling Lifecycle |
| 19 | RAG Integration | Embeddings, Chunking, Vector Store, Retriever, Similarity Search, RAG Tool, Agentic RAG |
| 20 | Human-in-the-Loop (HITL) | interrupt(), Command(resume), Approval Workflows, Review/Edit Pattern, Escalation Pattern |
| 21 | Subgraphs | Graph-in-Graph, Multi-Agent Systems, Failure Isolation, State Separation, Reusability |
| 22 | Memory Fundamentals | LLM Statelessness, Context Window, In-Context Learning, STM vs LTM |
| 23 | Short-Term Memory | Checkpointer, Threads, Conversation Buffer, Trimming, Deletion, Summarization |
| 24 | Long-Term Memory | Memory Store, Namespace, Semantic Search, Episodic Memory, Semantic Memory, Procedural Memory |
| 25 | Planning Agents | Planning vs Execution, Orchestrator-Worker Pattern, Dynamic Fanout, Send API, Reducer Pattern |

---

# Most Important Topics (High Interview Probability)

| Rank | Topic | Related Chapters |
|--------|--------|--------|
| 1 | State Management | 3, 4 |
| 2 | Reducers | 4, 6 |
| 3 | Sequential Workflow | 5 |
| 4 | Parallel Workflow | 6 |
| 5 | Conditional Workflow | 7 |
| 6 | Iterative Workflow | 8 |
| 7 | Persistence & Thread ID | 10, 23 |
| 8 | Tool Calling & ToolNode | 17 |
| 9 | RAG Architecture | 19 |
| 10 | Human-in-the-Loop | 20 |
| 11 | LangSmith Observability | 15 |
| 12 | Subgraphs | 21 |
| 13 | STM vs LTM | 22, 23, 24 |
| 14 | Planning Agents | 25 |

---

# Workflow Types (Must Know)

| Workflow Type | Concept | Example |
|---------------|----------|----------|
| Sequential | One node after another | A → B → C |
| Parallel | Multiple nodes simultaneously | A → (B, C, D) |
| Conditional | Route based on decision | A → B or C |
| Iterative | Loop until success | Generate → Evaluate → Improve |

---

# State Management

| Topic | What to Know |
|---------|-------------|
| State | Shared memory across nodes |
| Stateful Execution | Data flows from node to node |
| TypedDict | Standard state definition |
| State Propagation | Automatic state passing |
| State Evolution | State changes over time |
| State Access | Every node receives current state |

Example:

```python
class State(TypedDict):
    query: str
    response: str
```

---

# Reducers

| Topic | What to Know |
|---------|-------------|
| Purpose | Handle parallel state updates |
| Default Behavior | Replace old value |
| Reducer Behavior | Merge or append values |
| Annotated | Used for reducers |
| operator.add | Common reducer |

Example:

```python
messages: Annotated[list, add]
```

---

# Persistence Topics

| Topic | Description |
|---------|-------------|
| Checkpointer | Saves workflow state |
| Checkpoint | Snapshot of state |
| Thread ID | Unique conversation identifier |
| MemorySaver | In-memory checkpointing |
| SqliteSaver | SQLite persistence |
| PostgresSaver | Production persistence |
| RedisSaver | Scalable persistence |
| Time Travel | Replay from checkpoint |
| Fault Tolerance | Resume after failure |

Example:

```python
config = {
    "configurable": {
        "thread_id": "123"
    }
}
```

---

# Tool Calling Topics

| Topic | What to Know |
|---------|-------------|
| ToolNode | Executes tool calls |
| tools_condition | Determines Tool vs END |
| bind_tools() | Binds tools to LLM |
| Custom Tools | @tool decorator |
| Prebuilt Tools | DuckDuckGo, Wikipedia, Tavily |
| Feedback Loop | Tool → LLM → Tool → Final Answer |

Tool Flow:

```text
User
 ↓
LLM
 ↓
ToolNode
 ↓
Tool
 ↓
LLM
 ↓
Answer
```

---

# RAG Topics

| Area | Topics |
|--------|--------|
| Document Processing | Document Loader, Chunking |
| Embeddings | Text → Vector |
| Storage | FAISS, Chroma, Pinecone |
| Retrieval | Similarity Search, Top-K Retrieval |
| Generation | Query + Context + LLM |
| Agentic RAG | RAG Tool inside LangGraph |

RAG Flow:

```text
Question
 ↓
Embedding
 ↓
Retriever
 ↓
Relevant Chunks
 ↓
LLM
 ↓
Answer
```

---

# Human-in-the-Loop (HITL)

| Topic | Description |
|---------|-------------|
| interrupt() | Pause execution |
| Command(resume) | Resume execution |
| Approval Workflow | Human approves actions |
| Clarification Pattern | Ask user for clarification |
| Escalation Pattern | Hand over to human |
| Output Review | Human edits AI output |

Example:

```python
decision = interrupt(...)
```

```python
Command(resume="yes")
```

---

# LangSmith Topics

| Concept | Description |
|----------|-------------|
| Project | Complete application |
| Trace | Single execution |
| Run | Individual component execution |
| Monitoring | Production metrics |
| Evaluation | LLM quality checks |
| Cost Tracking | Token usage & spend |
| Latency Tracking | Response timings |
| Debugging | Root cause analysis |

Hierarchy:

```text
Project
   ↓
Trace
   ↓
Run
```

---

# Subgraphs

| Topic | Description |
|----------|-------------|
| Graph-in-Graph | Graph inside another graph |
| Multi-Agent Systems | Each agent can be a subgraph |
| Failure Isolation | One subgraph failure isolated |
| State Separation | Independent state management |
| Reusability | Reuse across workflows |

### Method 1

```python
subgraph.invoke(...)
```

### Method 2

```python
parent.add_node(
    "agent",
    compiled_subgraph
)
```

---

# Memory Topics

## Short-Term Memory (STM)

| Topic | Description |
|---------|-------------|
| Conversation Buffer | Holds chat history |
| Thread Scoped | Memory per conversation |
| Checkpointer | Saves chat state |
| Trimming | Keep recent messages |
| Deletion | Remove old messages |
| Summarization | Compress old history |

---

## Long-Term Memory (LTM)

| Memory Type | Description | Example |
|-------------|-------------|----------|
| Episodic Memory | Past events | Last session user rejected plan A |
| Semantic Memory | Facts & preferences | User prefers Python |
| Procedural Memory | Strategies | Use window functions instead of subqueries |

---

## Long-Term Memory Components

| Topic | Description |
|---------|-------------|
| Memory Store | Stores memories |
| Namespace | Organizes memories |
| Semantic Search | Retrieves relevant memories |
| Embeddings | Memory vectorization |
| PostgresStore | Production memory storage |

---

# Planning Agents

| Component | Purpose |
|------------|----------|
| Planner | Creates execution plan |
| Orchestrator | Manages workers |
| Workers | Execute tasks |
| Fanout | Create workers dynamically |
| Send API | Spawn worker nodes |
| Reducer | Merge worker outputs |
| Planning Phase | Think first |
| Execution Phase | Act later |

Flow:

```text
Goal
 ↓
Plan
 ↓
Execute
```

Orchestrator-Worker Pattern:

```text
Topic
 ↓
Planner
 ↓
Tasks
 ↓
Workers
 ↓
Reducer
 ↓
Output
```

---

# Top 10 Topics To Master Before Interview

| Rank | Topic |
|--------|--------|
| 1 | State Management |
| 2 | Reducers |
| 3 | Workflow Types |
| 4 | Persistence + Thread ID |
| 5 | Tool Calling + ToolNode |
| 6 | RAG |
| 7 | Human-in-the-Loop (HITL) |
| 8 | LangSmith |
| 9 | Subgraphs |
| 10 | STM vs LTM + Planning Agents |

---

# Interview Questions Mapping

| Topic | Common Questions |
|---------|-----------------|
| State | What is State in LangGraph? |
| Reducers | Why are Reducers required? |
| Sequential | Explain Sequential Workflow |
| Parallel | Explain Parallel Workflow |
| Conditional | Explain add_conditional_edges() |
| Iterative | How do you create loops in LangGraph? |
| Persistence | What is a Checkpointer? |
| Tools | Explain ToolNode and Tool Calling |
| RAG | How would you implement RAG in LangGraph? |
| HITL | How does interrupt() work? |
| LangSmith | Difference between Project, Trace, Run |
| Subgraphs | What are Subgraphs and why use them? |
| STM/LTM | Difference between STM and LTM |
| Planning Agents | Explain Orchestrator-Worker Pattern |

#################################################33333


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


