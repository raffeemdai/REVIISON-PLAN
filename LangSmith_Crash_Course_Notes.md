# LangSmith Crash Course — Complete Notes

> Beginner-friendly notes based on the CampusX "LangSmith Masterclass" video + code repo
> (`campusx-official/langsmith-masterclass`). Every code file is explained line-by-line with
> simple analogies, the underlying theory, and interview Q&A.

> ⚠️ **Security note before we start:** The video description pasted a *real* `OPENAI_API_KEY`
> and a real `LANGCHAIN_API_KEY` in plain text. Treat any key that has ever appeared in a
> public place (video description, GitHub, Slack, etc.) as **compromised** — go to the OpenAI
> and LangSmith dashboards and **rotate/delete** those keys immediately. In this document, all
> keys are replaced with placeholders like `sk-...` — never commit real keys to a file or a repo.

---

## Table of Contents

1. [Why do we even need LangSmith?](#1-why-do-we-even-need-langsmith)
2. [What is Observability?](#2-what-is-observability)
3. [What is LangSmith?](#3-what-is-langsmith)
4. [What does LangSmith actually trace?](#4-what-does-langsmith-actually-trace)
5. [Setup — .env file and installation](#5-setup--env-file-and-installation)
6. [Core Concepts: Project → Trace → Run](#6-core-concepts-project--trace--run)
7. [Code Walkthrough](#7-code-walkthrough)
   - [7.1 `1_simple_llm_call.py`](#71-1_simple_llm_callpy)
   - [7.2 `2_sequential_chain.py`](#72-2_sequential_chainpy)
   - [7.3 `3_rag_v1.py`](#73-3_rag_v1py-rag-without-tracing-of-plain-python-steps)
   - [7.4 `3_rag_v2.py`](#74-3_rag_v2py-adding-traceable)
   - [7.5 `3_rag_v3.py`](#75-3_rag_v3py-one-clean-root-trace)
   - [7.6 `3_rag_v4.py`](#76-3_rag_v4py-persistent-cached-index)
   - [7.7 `4_agent.py`](#77-4_agentpy-tracing-a-react-agent)
   - [7.8 `5_langgraph.py`](#78-5_langgraphpy-tracing-a-langgraph-workflow)
8. [LLMOps — Beyond Tracing](#8-llmops--beyond-tracing)
9. [Full Interview Q&A Bank](#9-full-interview-qa-bank)
10. [One-Page Cheat Sheet](#10-one-page-cheat-sheet)

---

## 1. Why do we even need LangSmith?

Think of a normal Python program like a **calculator**. Give it `2 + 2`, and it always returns
`4`. If something breaks, you get a clean error with a line number — easy to debug.

An **LLM application** is not like that. It's more like a **kitchen with many chefs**, where
each chef (component) hands a half-cooked dish to the next chef, and the final plate depends on
mood, ingredients, and a bit of randomness every time. Something can go wrong "quietly" —
the dish looks fine, tastes wrong, costs more, or takes longer — with no red error message
anywhere. LangSmith is the **kitchen CCTV + receipts system**: it lets you rewind and see
exactly what every chef did, how long they took, and what ingredients they used.

The video walks through **three real-world horror stories** to motivate this:

### Scenario 1 — The Slow Cover-Letter App (Latency problem)
A startup builds an app that reads a job description, pulls your resume/portfolio from Google
Drive, matches skills, and writes a custom cover letter. It normally takes **2 minutes**.
One day it starts taking **7–10 minutes** and users get angry.

The problem: the app has many steps (read JD → fetch Drive docs → match skills → generate →
proofread), but the team can only see **total time**, not a **per-step breakdown**. Someone
had quietly changed the Google Drive logic to scan the *entire* drive instead of one folder —
but without step-by-step visibility, nobody can find this quickly.

**Analogy:** You know a courier took 3 hours instead of 30 minutes, but you have no tracking
number — you can't tell if the delay was at pickup, sorting, or delivery.

### Scenario 2 — The Expensive Research Agent (Cost problem)
A "research assistant" agent searches papers, reads them, summarizes them, and writes a report.
It normally costs about ₹0.50 per report (LLM token cost). Suddenly some reports cost ₹2.00.

The cause: someone tweaked a prompt to say *"keep generating until you are satisfied with the
quality"* — for some topics the agent loops multiple times (search → read → summarize → "not
good enough, retry"), burning 4x the tokens. No crash, no stack trace — just a silent
**behavior change** in an autonomous loop.

**Analogy:** Imagine paying someone by the hour to mow your lawn, and one day they "feel like"
mowing it 4 times because they're a perfectionist. Your bill goes up — but nothing "broke."

### Scenario 3 — The Hallucinating HR Chatbot (RAG problem)
A RAG (Retrieval-Augmented Generation) chatbot answers employee questions about HR policy using
company documents. It starts **hallucinating** — e.g., wrongly telling an employee they can
take leave whenever they want.

Two possible root causes, and from the final answer alone you cannot tell which one it is:
- **Retriever problem:** wrong documents were fetched (e.g. `k=1` retrieved doc count is too
  low, or the retriever grabbed unrelated "company history" docs instead of "leave policy" docs).
- **Generator problem:** the right documents were fetched, but the LLM ignored them and made
  something up because the prompt didn't strongly enforce "answer only from context."

**Analogy:** A student fails an exam. Was it because they were given the wrong textbook chapter
to study (retriever), or because they had the right chapter but didn't actually use it while
answering (generator)? You need to see both the chapter they got *and* their working to know.

**The common thread:** in all three cases, the app isn't "crashing" in the traditional sense —
it's misbehaving in ways that are invisible unless you can see *inside* the pipeline,
step by step. That "seeing inside" is **observability**, and LangSmith is the tool for it.

---

## 2. What is Observability?

> **Definition:** Observability is the ability to understand what is happening *inside* a
> system by looking at what comes *out* of it — logs, metrics, and traces — even for problems
> you didn't anticipate in advance.

**Simple analogy:** A doctor doesn't need to cut you open to know something is wrong — they
look at *external signals*: your temperature, blood pressure, an X-ray. Those signals are like
logs/metrics/traces for a software system.

Why is this especially important for LLM apps?

| Traditional software | LLM-based software |
|---|---|
| Deterministic — same input → same output | Non-deterministic — same input can give different outputs |
| Errors usually throw a clear exception/stack trace | "Errors" can be silent — a wrong-but-fluent answer, a slow step, a costly loop |
| Easy to unit test with fixed expected outputs | Hard to test — you often need to *evaluate* quality, not just check equality |

Observability lets you answer the question: **"Why did *this specific run* behave the way it
did?"** — even after the fact, even if you never predicted this particular failure mode.

---

## 3. What is LangSmith?

> **Definition:** LangSmith is a **unified observability and evaluation platform** for LLM
> applications. It lets teams **trace**, **test**, **monitor**, and **evaluate** how their
> GenAI apps behave, whether built with LangChain, LangGraph, or plain Python.

Once you wire your app up to LangSmith (mostly through environment variables), it automatically
records what happens at **every step** — without you writing custom logging code everywhere.

**Analogy:** It's like installing a **flight data recorder (black box)** in an airplane. You
don't change how the plane flies; you just get a device that quietly records everything, so if
something goes wrong, you can replay exactly what happened.

---

## 4. What does LangSmith actually trace?

For every execution, LangSmith can capture:

- **Inputs & Outputs** — e.g. input: `"What is the capital of India?"`, output: `"New Delhi"`.
- **Intermediate steps** — e.g. in RAG: the question sent to the retriever, the context it
  returned, the final prompt built from question + context, the raw LLM response, the parsed
  output.
- **Latency** — total time, and time per individual component.
- **Token usage & cost** — input tokens, output tokens, estimated $ cost per model.
- **Errors** — if any component throws an exception.
- **Tags** — labels you (or LangSmith) attach to classify traces, e.g. `report-generation`.
- **Metadata** — extra structured info you attach, e.g. `{"embedding_model": "text-embedding-3-small"}`.
- **User feedback** — thumbs up/down or ratings linked back to the exact trace that produced
  that response.

---

## 5. Setup — .env file and installation

High-level setup steps from the video:

1. Clone the repo: `git clone https://github.com/campusx-official/langsmith-masterclass`
2. Create and activate a Python virtual environment.
3. `pip install -r requirements.txt`
4. Create a LangSmith account → generate an API key from the LangSmith dashboard.
5. Create a `.env` file in the project root with:

```bash
# .env  (NEVER commit this file or share it in videos/chats — rotate immediately if leaked)
OPENAI_API_KEY="sk-...your-own-key..."

LANGCHAIN_TRACING_V2=true
LANGCHAIN_ENDPOINT="https://api.smith.langchain.com"
LANGCHAIN_API_KEY="lsv2_pt_...your-own-key..."
LANGCHAIN_PROJECT="langsmith-demo"
```

What each variable means:

| Variable | Meaning |
|---|---|
| `OPENAI_API_KEY` | Lets `langchain_openai` call OpenAI's models (chat + embeddings). |
| `LANGCHAIN_TRACING_V2=true` | The master switch — turns tracing **ON** for every LangChain run in this process. |
| `LANGCHAIN_ENDPOINT` | Where trace data is sent (LangSmith's cloud API). |
| `LANGCHAIN_API_KEY` | Authenticates your app to your LangSmith account/workspace. |
| `LANGCHAIN_PROJECT` | Which "folder" (Project) in LangSmith the traces should land in. |

**The magic:** once these env vars are set and `load_dotenv()` is called, you generally **do
not need to add any special tracing code** to plain LangChain chains — LangSmith auto-instruments
them. You only add the `@traceable` decorator for *plain Python functions* that aren't
LangChain objects (loading a PDF, hashing a file, etc.) — covered below.

> **Analogy:** `LANGCHAIN_TRACING_V2=true` is like flipping on a security camera system for a
> building — every LangChain "room" (component) it can see automatically gets recorded. But a
> hallway the camera *can't* see (a plain Python function) needs its own dedicated camera —
> that's what `@traceable` is for.

---

## 6. Core Concepts: Project → Trace → Run

LangSmith organizes everything in a 3-level hierarchy:

```
Project
 └── Trace (one full end-to-end execution)
      └── Run (execution of one component inside that trace)
           └── Run (can be nested further — a run can have child runs)
```

| Concept | What it is | Analogy |
|---|---|---|
| **Project** | A logical grouping for one application — e.g. "langsmith-demo". | A **case file folder** for one investigation unit / product. |
| **Trace** | One complete execution of your app, start to finish, for one request. | One **complete customer order**, from order placed to delivery. |
| **Run** | The execution of a single component/function within a trace (Prompt, LLM call, Retriever, a Python function, etc.). Can be nested (parent run → child runs). | Each **individual step** in fulfilling that order — packing, labeling, shipping — each logged separately, but all tied to the same order number. |

**Example:** A simple chain `Prompt → ChatOpenAI → StringOutputParser`, run once with the
question "What is the capital of Peru?", produces **one Trace** containing **three Runs**
(one per component). Run the app 10 times → 10 Traces, each with their own Runs, all inside
the same Project.

---

## 7. Code Walkthrough

> All code below is reproduced from `campusx-official/langsmith-masterclass` on GitHub, file by
> file, in the order the video builds them up (simple call → chain → RAG v1 → v2 → v3 → v4 →
> agent → LangGraph).

### 7.1 `1_simple_llm_call.py`

**Goal:** The simplest possible traced LLM call — no manual tracing code needed at all.

```python
from dotenv import load_dotenv
from langchain_openai import ChatOpenAI
from langchain_core.prompts import PromptTemplate
from langchain_core.output_parsers import StrOutputParser

load_dotenv()

# Simple one-line prompt
prompt = PromptTemplate.from_template("{question}")

model = ChatOpenAI()

parser = StrOutputParser()

# Chain: prompt → model → parser
chain = prompt | model | parser

# Run it
result = chain.invoke({"question": "What is the capital of Peru?"})

print(result)
```

**Line-by-line, in plain English:**

- `load_dotenv()` — reads your `.env` file and loads `OPENAI_API_KEY`, `LANGCHAIN_TRACING_V2`,
  etc. into the environment so the libraries can see them.
- `PromptTemplate.from_template("{question}")` — creates a template that just inserts whatever
  you pass as `question` directly (a "pass-through" template, no extra wording).
- `ChatOpenAI()` — the LLM client; by default uses OpenAI's chat model.
- `StrOutputParser()` — takes the LLM's raw message object and extracts just the plain text
  string out of it.
- `chain = prompt | model | parser` — the **pipe operator (`|`)** chains components together
  using LangChain's **LCEL (LangChain Expression Language)**: output of `prompt` feeds into
  `model`, whose output feeds into `parser`.
- `chain.invoke({"question": "..."})` — runs the whole pipeline once and returns the final
  string, e.g. `"Lima"`.

**Analogy:** Think of `prompt | model | parser` like a **factory assembly line** with 3
stations: Station 1 wraps your raw question into a nicely formatted package (prompt), Station 2
sends it to the "brain" (the LLM) and gets a reply, Station 3 unwraps the reply into plain
text. The `|` symbol is the conveyor belt between stations.

**What happens in LangSmith:** Because `LANGCHAIN_TRACING_V2=true` is set and everything used
here is a native LangChain object (`PromptTemplate`, `ChatOpenAI`, `StrOutputParser`), LangSmith
**automatically** creates one Trace with three nested Runs — Prompt, LLM, Parser — showing
input/output/latency for each, with **zero extra tracing code**.

**Theory box — why LCEL (`|`) matters:**
LCEL is LangChain's declarative way to compose "Runnables." Every LCEL component implements a
common interface (`invoke`, `batch`, `stream`, etc.), so they can be chained with `|` just like
Unix pipes (`cat file | grep x | sort`). This uniformity is *exactly* why LangSmith can trace
them automatically — it just walks the Runnable graph.

**Interview Q&A:**

- **Q: What is LCEL and why is the `|` operator used?**
  A: LCEL (LangChain Expression Language) lets you compose "Runnable" components declaratively.
  The `|` operator pipes the output of one Runnable into the input of the next, similar to Unix
  pipes, producing a `RunnableSequence`.
- **Q: Why don't we need to write any tracing code in this file?**
  A: Because tracing is controlled by environment variables (`LANGCHAIN_TRACING_V2=true`,
  `LANGCHAIN_API_KEY`, etc.), and every component here (`PromptTemplate`, `ChatOpenAI`,
  `StrOutputParser`) is a native LangChain Runnable that LangSmith auto-instruments.
- **Q: What does `StrOutputParser` do and why is it needed?**
  A: `ChatOpenAI` returns a rich `AIMessage` object (with metadata like token usage). If you
  just want the plain text answer, `StrOutputParser` extracts the `.content` string from it.

---

### 7.2 `2_sequential_chain.py`

**Goal:** Show a **multi-step (sequential) chain** — a report generator followed by a
summarizer — and see how LangSmith represents multiple LLM calls inside one trace.

```python
from langchain_openai import ChatOpenAI
from dotenv import load_dotenv
from langchain_core.prompts import PromptTemplate
from langchain_core.output_parsers import StrOutputParser

load_dotenv()

prompt1 = PromptTemplate(
    template='Generate a detailed report on {topic}',
    input_variables=['topic']
)

prompt2 = PromptTemplate(
    template='Generate a 5 pointer summary from the following text \n {text}',
    input_variables=['text']
)

model = ChatOpenAI()

parser = StrOutputParser()

chain = prompt1 | model | parser | prompt2 | model | parser

result = chain.invoke({'topic': 'Unemployment in India'})

print(result)
```

**Line-by-line:**

- `prompt1` — takes a `topic` and asks the LLM to write a **detailed report**.
- `prompt2` — takes `text` (the output of the previous step) and asks for a **5-point summary**.
- `chain = prompt1 | model | parser | prompt2 | model | parser` — this is a **6-station**
  assembly line: generate report → parse to string → feed that string as `text` into prompt2 →
  summarize → parse to string again. Note: `model` is reused for both LLM calls here (same
  object, called twice).
- `chain.invoke({'topic': 'Unemployment in India'})` — kicks off the whole two-stage pipeline
  in one call.

**Analogy:** This is like a **news pipeline**: a journalist writes a full detailed article
(prompt1 + model), then an editor condenses it into 5 bullet-point headlines for the homepage
(prompt2 + model). Both jobs happen back-to-back, but they are two *separate* trips to the
"newsroom" (two separate LLM calls).

**What happens in LangSmith:** One Trace is created for the whole `chain.invoke(...)` call, and
inside it you'll see **two separate LLM Runs** (one for the report, one for the summary), each
with its own input/output/latency/tokens — even though it's all "one line of code" to you.
This is exactly the kind of visibility that would have caught the cost-explosion problem in
Scenario 2 (you'd see two LLM calls where you expected one, or a suspiciously large output on
the first call).

**Interview Q&A:**

- **Q: In a sequential chain like this, how many LLM calls actually happen, and why does that
  matter for cost/latency?**
  A: Two — `model` is invoked once for the report and once for the summary. Each call costs
  tokens and adds latency; LangSmith shows both calls separately so you can see which one is
  the bottleneck or the expensive one.
- **Q: Why is `parser` placed between `prompt1|model` and `prompt2`?**
  A: Because `model` returns an `AIMessage` object, but `prompt2`'s template expects a plain
  string for `{text}`. `StrOutputParser` converts the message to text so the next prompt can
  use it.
- **Q: How does this connect back to Scenario 2 (the expensive research agent) from the video?**
  A: A "silent" extra LLM call or a runaway loop is invisible from the outside (same code, same
  final output shape) but shows up immediately as extra Runs and extra token cost inside a
  LangSmith trace.

---

### 7.3 `3_rag_v1.py` — RAG without tracing of plain-Python steps

**Goal:** A working PDF Question-Answering (RAG) app. This version reveals the **"partial
tracing" problem**: LangChain-native steps (retriever, prompt, LLM, parser) get traced
automatically, but plain Python steps (PDF loading, chunking, embedding) do **not**.

```python
# pip install -U langchain langchain-openai langchain-community faiss-cpu pypdf python-dotenv
import os
from dotenv import load_dotenv
from langchain_community.document_loaders import PyPDFLoader
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain_openai import OpenAIEmbeddings, ChatOpenAI
from langchain_community.vectorstores import FAISS
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.runnables import RunnableParallel, RunnablePassthrough, RunnableLambda
from langchain_core.output_parsers import StrOutputParser

load_dotenv()  # expects OPENAI_API_KEY in .env

PDF_PATH = "islr.pdf"  # <-- change to your PDF filename

# 1) Load PDF
loader = PyPDFLoader(PDF_PATH)
docs = loader.load()  # one Document per page

# 2) Chunk
splitter = RecursiveCharacterTextSplitter(chunk_size=1000, chunk_overlap=150)
splits = splitter.split_documents(docs)

# 3) Embed + index
emb = OpenAIEmbeddings(model="text-embedding-3-small")
vs = FAISS.from_documents(splits, emb)
retriever = vs.as_retriever(search_type="similarity", search_kwargs={"k": 4})

# 4) Prompt
prompt = ChatPromptTemplate.from_messages([
    ("system", "Answer ONLY from the provided context. If not found, say you don't know."),
    ("human", "Question: {question}\n\nContext:\n{context}")
])

# 5) Chain
llm = ChatOpenAI(model="gpt-4o-mini", temperature=0)

def format_docs(docs): return "\n\n".join(d.page_content for d in docs)

parallel = RunnableParallel({
    "context": retriever | RunnableLambda(format_docs),
    "question": RunnablePassthrough()
})

chain = parallel | prompt | llm | StrOutputParser()

# 6) Ask questions
print("PDF RAG ready. Ask a question (or Ctrl+C to exit).")
q = input("\nQ: ")
ans = chain.invoke(q.strip())
print("\nA:", ans)
```

**Step-by-step in plain English:**

1. **Load PDF** — `PyPDFLoader` reads the PDF and returns a list of `Document` objects, one
   per page (e.g. 441 pages → 441 documents).
2. **Chunk** — `RecursiveCharacterTextSplitter` breaks each page into smaller overlapping pieces
   (`chunk_size=1000` characters, `chunk_overlap=150`). Overlap helps avoid cutting a sentence
   or idea exactly in half between two chunks.
3. **Embed + index** — `OpenAIEmbeddings` turns each text chunk into a vector (a list of
   numbers representing meaning). `FAISS.from_documents(...)` stores all these vectors in a
   fast searchable index.
4. **Retriever** — `vs.as_retriever(search_kwargs={"k": 4})` — given a question, find the
   **top 4** most similar chunks by vector similarity.
5. **Prompt** — a strict system instruction: *"Answer ONLY from the provided context. If not
   found, say you don't know."* This is the key guardrail against hallucination discussed in
   Scenario 3.
6. **`RunnableParallel`** — runs two branches at the same time: one branch sends the question
   through the retriever and formats the docs into one big string (`context`); the other branch
   just passes the raw `question` straight through (`RunnablePassthrough`). Both results are
   combined into a dict `{"context": ..., "question": ...}` that matches the prompt's template
   variables.
7. **`chain = parallel | prompt | llm | StrOutputParser()`** — combine context+question → build
   the final prompt → call the LLM → parse to text.

**Analogy for RAG as a whole:** RAG is like an **open-book exam**. Instead of memorizing every
policy from training (closed-book, and possibly wrong/outdated), the model is handed the
*exact* relevant pages from the textbook (retrieved context) right before answering. The
retriever is the "librarian" who finds the right pages; the LLM is the "student" who must
answer *using only those pages*.

**The problem highlighted in the video:** Steps 1–3 (load, chunk, embed) are **plain Python
function calls / library calls**, not LangChain Runnables chained with `|`. So when this script
runs, LangSmith will trace the retriever, prompt, LLM, and parser nicely (they're inside the
`chain`), but it has **no visibility at all** into how long loading the PDF took, how many
chunks were created, or which embedding model/dimensions were used. This is called
**partial tracing** — and it's exactly the gap `@traceable` (next file) is built to close.

**Interview Q&A:**

- **Q: Why do we chunk documents before embedding instead of embedding the whole PDF at once?**
  A: Embedding models have input size limits, and smaller chunks give more precise, focused
  retrieval — you want to retrieve the specific paragraph that answers the question, not an
  entire 40-page chapter which dilutes relevance.
- **Q: What does `chunk_overlap=150` do and why is it useful?**
  A: It repeats the last 150 characters of one chunk at the start of the next chunk, so an idea
  or sentence that spans a chunk boundary isn't completely lost in either chunk.
- **Q: What is `k=4` in `search_kwargs`, and how could it cause hallucination if misconfigured?**
  A: `k` is the number of top-matching chunks retrieved. If `k` is too small (e.g. 1) for a
  question that needs information spread across multiple chunks, the LLM won't have enough
  context and may guess/hallucinate an answer instead of saying "I don't know."
- **Q: Why does the prompt explicitly say "Answer ONLY from the provided context... say you
  don't know"?**
  A: LLMs are trained to be helpful and will often "fill gaps" with plausible-sounding but
  incorrect information. A strict instruction forces the model to stay grounded in the
  retrieved context rather than inventing facts — directly addressing the "Generator problem"
  from Scenario 3.
- **Q: Why is this version of the RAG app only "partially traceable" in LangSmith?**
  A: Because PDF loading, chunking, and embedding-index construction are plain Python/library
  calls outside any LangChain Runnable chain — LangSmith's auto-instrumentation only sees
  Runnables composed with `|`, not arbitrary function calls.

---

### 7.4 `3_rag_v2.py` — Adding `@traceable`

**Goal:** Fix the partial-tracing gap using LangSmith's `@traceable` decorator, so PDF loading,
chunking, and embedding also show up as their own Runs.

```python
# pip install -U langchain langchain-openai langchain-community faiss-cpu pypdf python-dotenv langsmith
import os
from dotenv import load_dotenv
from langsmith import traceable  # <-- key import
from langchain_community.document_loaders import PyPDFLoader
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain_openai import OpenAIEmbeddings, ChatOpenAI
from langchain_community.vectorstores import FAISS
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.runnables import RunnableParallel, RunnablePassthrough, RunnableLambda
from langchain_core.output_parsers import StrOutputParser

# --- LangSmith env (make sure these are set) ---
# LANGCHAIN_TRACING_V2=true
# LANGCHAIN_API_KEY=...
# LANGCHAIN_PROJECT=pdf_rag_demo

load_dotenv()

PDF_PATH = "islr.pdf"  # change to your file

# ---------- traced setup steps ----------
@traceable(name="load_pdf")
def load_pdf(path: str):
    loader = PyPDFLoader(path)
    return loader.load()  # list[Document]

@traceable(name="split_documents")
def split_documents(docs, chunk_size=1000, chunk_overlap=150):
    splitter = RecursiveCharacterTextSplitter(
        chunk_size=chunk_size, chunk_overlap=chunk_overlap
    )
    return splitter.split_documents(docs)

@traceable(name="build_vectorstore")
def build_vectorstore(splits):
    emb = OpenAIEmbeddings(model="text-embedding-3-small")
    # FAISS.from_documents internally calls the embedding model:
    vs = FAISS.from_documents(splits, emb)
    return vs

# You can also trace a "setup" umbrella span if you want:
@traceable(name="setup_pipeline")
def setup_pipeline(pdf_path: str):
    docs = load_pdf(pdf_path)
    splits = split_documents(docs)
    vs = build_vectorstore(splits)
    return vs

# ---------- pipeline ----------
llm = ChatOpenAI(model="gpt-4o-mini", temperature=0)

prompt = ChatPromptTemplate.from_messages([
    ("system", "Answer ONLY from the provided context. If not found, say you don't know."),
    ("human", "Question: {question}\n\nContext:\n{context}")
])

def format_docs(docs):
    return "\n\n".join(d.page_content for d in docs)

# Build the index under traced setup
vectorstore = setup_pipeline(PDF_PATH)

retriever = vectorstore.as_retriever(search_type="similarity", search_kwargs={"k": 4})

parallel = RunnableParallel({
    "context": retriever | RunnableLambda(format_docs),
    "question": RunnablePassthrough(),
})

chain = parallel | prompt | llm | StrOutputParser()

# ---------- run a query (also traced) ----------
print("PDF RAG ready. Ask a question (or Ctrl+C to exit).")
q = input("\nQ: ").strip()

# Give the visible run name + tags/metadata so it's easy to find:
config = {
    "run_name": "pdf_rag_query"
}

ans = chain.invoke(q, config=config)
print("\nA:", ans)
```

**What changed vs v1, and why:**

- `from langsmith import traceable` — this is the key new import. `@traceable` is a **decorator**
  that wraps any ordinary Python function so LangSmith records it as a Run — capturing its
  inputs, outputs, latency, and any exceptions — **without** needing that function to be a
  LangChain "Runnable."
- `load_pdf`, `split_documents`, `build_vectorstore` are now separate, named, `@traceable`
  functions instead of loose top-level code. Each becomes its **own labeled Run** in LangSmith
  (e.g. "load_pdf" took 2.1s, "build_vectorstore" took 14.3s).
- `setup_pipeline(...)` is itself `@traceable`, and it **calls** the three functions above — so
  LangSmith records it as a **parent Run**, with `load_pdf`, `split_documents`, and
  `build_vectorstore` appearing as **child Runs nested inside it**. This nesting happens
  automatically because `traceable` uses Python's context system to detect "I was called while
  another traced function was running."
- `config = {"run_name": "pdf_rag_query"}` passed into `chain.invoke(q, config=config)` — this
  renames the LangChain chain's root Run from the generic auto-generated name (like
  `RunnableSequence`) to something human-readable in the LangSmith UI.

**Analogy:** In v1, only the "public-facing counter staff" (retriever, prompt, LLM, parser)
wore body cameras. In v2, we hand body cameras to the "kitchen staff" too (`load_pdf`,
`split_documents`, `build_vectorstore`) by literally putting a `@traceable` sticker on each of
their job descriptions. Now every single worker involved in making your meal is recorded.

**Remaining problem this file doesn't fix:** `setup_pipeline(...)` (loading, chunking,
embedding) runs as **one top-level Trace**, and `chain.invoke(...)` (the actual query) runs as
a **separate, unrelated top-level Trace**. Logically they're one application, but in the
LangSmith UI they show up as two disconnected traces — fixed in v3.

**Interview Q&A:**

- **Q: What problem does `@traceable` solve that plain LangChain tracing doesn't?**
  A: LangChain's auto-tracing only sees LCEL Runnables (things chained with `|`). Plain Python
  functions — like loading a file, hashing it, or calling a non-LangChain library — are
  invisible to it. `@traceable` manually instruments any function so it also produces a
  LangSmith Run.
- **Q: How does LangSmith know that `load_pdf`, `split_documents`, and `build_vectorstore` are
  "children" of `setup_pipeline` rather than three separate unrelated traces?**
  A: `traceable` uses Python's `contextvars` to track "the currently active run." When
  `setup_pipeline` (itself traced) calls another traced function, LangSmith sees it's running
  inside an active trace context and automatically nests it as a child run of that trace.
- **Q: What does the `config={"run_name": ...}` dict do when passed to `.invoke()`?**
  A: It's LangChain's `RunnableConfig` — used here just to give the root Run of that specific
  invocation a custom, readable name instead of the default auto-generated class name.

---

### 7.5 `3_rag_v3.py` — one clean root trace

**Goal:** Fix the "two separate traces" problem from v2 by wrapping *both* setup and query
under a single `@traceable` root function, so the whole app shows up as **one trace** with a
clean parent → child hierarchy.

```python
# pip install -U langchain langchain-openai langchain-community faiss-cpu pypdf python-dotenv langsmith
import os
from dotenv import load_dotenv
from langsmith import traceable
from langchain_community.document_loaders import PyPDFLoader
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain_openai import OpenAIEmbeddings, ChatOpenAI
from langchain_community.vectorstores import FAISS
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.runnables import RunnableParallel, RunnablePassthrough, RunnableLambda
from langchain_core.output_parsers import StrOutputParser

load_dotenv()

PDF_PATH = "islr.pdf"  # <- change to your file

# ----------------- helpers (not traced individually) -----------------
@traceable(name="load_pdf")
def load_pdf(path: str):
    loader = PyPDFLoader(path)
    return loader.load()  # list[Document]

@traceable(name="split_documents")
def split_documents(docs, chunk_size=1000, chunk_overlap=150):
    splitter = RecursiveCharacterTextSplitter(
        chunk_size=chunk_size, chunk_overlap=chunk_overlap
    )
    return splitter.split_documents(docs)

@traceable(name="build_vectorstore")
def build_vectorstore(splits):
    emb = OpenAIEmbeddings(model="text-embedding-3-small")
    return FAISS.from_documents(splits, emb)

# ----------------- parent setup function (traced) -----------------
@traceable(name="setup_pipeline", tags=["setup"])
def setup_pipeline(pdf_path: str, chunk_size=1000, chunk_overlap=150):
    # These three steps are "clubbed" under this parent function
    docs = load_pdf(pdf_path)
    splits = split_documents(docs, chunk_size=chunk_size, chunk_overlap=chunk_overlap)
    vs = build_vectorstore(splits)
    return vs

# ----------------- model, prompt, and run -----------------
llm = ChatOpenAI(model="gpt-4o-mini", temperature=0)

prompt = ChatPromptTemplate.from_messages([
    ("system", "Answer ONLY from the provided context. If not found, say you don't know."),
    ("human", "Question: {question}\n\nContext:\n{context}")
])

def format_docs(docs):
    return "\n\n".join(d.page_content for d in docs)

# ----------------- one top-level (root) run -----------------
@traceable(name="pdf_rag_full_run")
def setup_pipeline_and_query(pdf_path: str, question: str):
    # Parent setup run (child of root)
    vectorstore = setup_pipeline(pdf_path, chunk_size=1000, chunk_overlap=150)
    retriever = vectorstore.as_retriever(search_type="similarity", search_kwargs={"k": 4})

    parallel = RunnableParallel({
        "context": retriever | RunnableLambda(format_docs),
        "question": RunnablePassthrough(),
    })
    chain = parallel | prompt | llm | StrOutputParser()

    # This LangChain run stays under the same root (since we're inside this traced function)
    lc_config = {"run_name": "pdf_rag_query"}
    return chain.invoke(question, config=lc_config)

# ----------------- CLI -----------------
if __name__ == "__main__":
    print("PDF RAG ready. Ask a question (or Ctrl+C to exit).")
    q = input("\nQ: ").strip()
    ans = setup_pipeline_and_query(PDF_PATH, q)
    print("\nA:", ans)
```

**What's new, and why it matters:**

- `setup_pipeline` now has `tags=["setup"]` — tags let you filter/search traces in the LangSmith
  UI later (e.g. "show me only the setup-phase runs across the last 1000 traces").
- **`setup_pipeline_and_query(...)`** is the big addition — a brand-new `@traceable` function
  that becomes the **single root of the entire trace**. Inside it, we call `setup_pipeline(...)`
  (which itself calls `load_pdf` → `split_documents` → `build_vectorstore`) *and* we build and
  invoke the LangChain `chain`. Because both happen inside the same active trace context, the
  final hierarchy in LangSmith looks like:

```
pdf_rag_full_run                 (root trace)
 └── setup_pipeline               (tags: setup)
      ├── load_pdf
      ├── split_documents
      └── build_vectorstore
 └── pdf_rag_query                (the LangChain chain: retriever → prompt → llm → parser)
```

  One trace, everything nested logically — exactly matching the ideal hierarchy the video
  describes: *"RAG Application → Setup Pipeline → {Load PDF, Split, Build Vector Store} →
  RAG Query → {Retriever, Prompt, LLM, Parser}."*

**Analogy:** v2 was like having two separate CCTV tapes — one for "kitchen prep" and one for
"serving the customer" — with no way to tell they were for the same meal. v3 puts both tapes
under **one folder labeled with the order number**, so you can watch the whole story, from
chopping vegetables to handing over the plate, in one continuous timeline.

**Interview Q&A:**

- **Q: Why wrap both `setup_pipeline(...)` and `chain.invoke(...)` inside one more
  `@traceable` function (`setup_pipeline_and_query`) instead of calling them separately at the
  top level like in v2?**
  A: Because each independently-called `@traceable`/traced call starts its **own** root trace.
  By calling both from inside one shared `@traceable` parent function, they both become child
  runs of that single parent — merging what used to be two disconnected traces into one
  coherent trace tree.
- **Q: What's the practical benefit of tags like `tags=["setup"]`?**
  A: They let you filter and search traces in the LangSmith dashboard — e.g., quickly pull up
  every "setup" run across thousands of traces to check index-build times, without manually
  opening each trace.

---

### 7.6 `3_rag_v4.py` — persistent (cached) index

**Goal:** Solve the **performance problem** the video calls out: v1–v3 rebuild the PDF index
(load + chunk + embed) on **every single run**, which is slow and wastes API calls. v4 adds a
**persistent FAISS index on disk**, built once and reused afterwards — while still tracing the
load/build-vs-reuse decision.

```python
# pip install -U langchain langchain-openai langchain-community faiss-cpu pypdf python-dotenv langsmith
import os
import json
import hashlib
from pathlib import Path
from dotenv import load_dotenv
from langsmith import traceable
from langchain_community.document_loaders import PyPDFLoader
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain_openai import OpenAIEmbeddings, ChatOpenAI
from langchain_community.vectorstores import FAISS
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.runnables import RunnableParallel, RunnablePassthrough, RunnableLambda
from langchain_core.output_parsers import StrOutputParser

load_dotenv()

PDF_PATH = "islr.pdf"          # change to your file
INDEX_ROOT = Path(".indices")
INDEX_ROOT.mkdir(exist_ok=True)

# ----------------- helpers (traced) -----------------
@traceable(name="load_pdf")
def load_pdf(path: str):
    return PyPDFLoader(path).load()  # list[Document]

@traceable(name="split_documents")
def split_documents(docs, chunk_size=1000, chunk_overlap=150):
    splitter = RecursiveCharacterTextSplitter(
        chunk_size=chunk_size, chunk_overlap=chunk_overlap
    )
    return splitter.split_documents(docs)

@traceable(name="build_vectorstore")
def build_vectorstore(splits, embed_model_name: str):
    emb = OpenAIEmbeddings(model=embed_model_name)
    return FAISS.from_documents(splits, emb)

# ----------------- cache key / fingerprint -----------------
def _file_fingerprint(path: str) -> dict:
    p = Path(path)
    h = hashlib.sha256()
    with p.open("rb") as f:
        for chunk in iter(lambda: f.read(1024 * 1024), b""):
            h.update(chunk)
    return {"sha256": h.hexdigest(), "size": p.stat().st_size, "mtime": int(p.stat().st_mtime)}

def _index_key(pdf_path: str, chunk_size: int, chunk_overlap: int, embed_model_name: str) -> str:
    meta = {
        "pdf_fingerprint": _file_fingerprint(pdf_path),
        "chunk_size": chunk_size,
        "chunk_overlap": chunk_overlap,
        "embedding_model": embed_model_name,
        "format": "v1",
    }
    return hashlib.sha256(json.dumps(meta, sort_keys=True).encode("utf-8")).hexdigest()

# ----------------- explicitly traced load/build runs -----------------
@traceable(name="load_index", tags=["index"])
def load_index_run(index_dir: Path, embed_model_name: str):
    emb = OpenAIEmbeddings(model=embed_model_name)
    return FAISS.load_local(
        str(index_dir), emb, allow_dangerous_deserialization=True
    )

@traceable(name="build_index", tags=["index"])
def build_index_run(pdf_path: str, index_dir: Path, chunk_size: int, chunk_overlap: int, embed_model_name: str):
    docs = load_pdf(pdf_path)                                            # child
    splits = split_documents(docs, chunk_size=chunk_size, chunk_overlap=chunk_overlap)  # child
    vs = build_vectorstore(splits, embed_model_name)                     # child
    index_dir.mkdir(parents=True, exist_ok=True)
    vs.save_local(str(index_dir))
    (index_dir / "meta.json").write_text(json.dumps({
        "pdf_path": os.path.abspath(pdf_path),
        "chunk_size": chunk_size,
        "chunk_overlap": chunk_overlap,
        "embedding_model": embed_model_name,
    }, indent=2))
    return vs

# ----------------- dispatcher (not traced) -----------------
def load_or_build_index(
    pdf_path: str,
    chunk_size: int = 1000,
    chunk_overlap: int = 150,
    embed_model_name: str = "text-embedding-3-small",
    force_rebuild: bool = False,
):
    key = _index_key(pdf_path, chunk_size, chunk_overlap, embed_model_name)
    index_dir = INDEX_ROOT / key
    cache_hit = index_dir.exists() and not force_rebuild

    if cache_hit:
        return load_index_run(index_dir, embed_model_name)
    else:
        return build_index_run(pdf_path, index_dir, chunk_size, chunk_overlap, embed_model_name)

# ----------------- model, prompt, and pipeline -----------------
llm = ChatOpenAI(model="gpt-4o-mini", temperature=0)

prompt = ChatPromptTemplate.from_messages([
    ("system", "Answer ONLY from the provided context. If not found, say you don't know."),
    ("human", "Question: {question}\n\nContext:\n{context}")
])

def format_docs(docs):
    return "\n\n".join(d.page_content for d in docs)

@traceable(name="setup_pipeline", tags=["setup"])
def setup_pipeline(pdf_path: str, chunk_size=1000, chunk_overlap=150, embed_model_name="text-embedding-3-small", force_rebuild=False):
    return load_or_build_index(
        pdf_path=pdf_path,
        chunk_size=chunk_size,
        chunk_overlap=chunk_overlap,
        embed_model_name=embed_model_name,
        force_rebuild=force_rebuild,
    )

@traceable(name="pdf_rag_full_run")
def setup_pipeline_and_query(
    pdf_path: str,
    question: str,
    chunk_size: int = 1000,
    chunk_overlap: int = 150,
    embed_model_name: str = "text-embedding-3-small",
    force_rebuild: bool = False,
):
    vectorstore = setup_pipeline(pdf_path, chunk_size, chunk_overlap, embed_model_name, force_rebuild)
    retriever = vectorstore.as_retriever(search_type="similarity", search_kwargs={"k": 4})

    parallel = RunnableParallel({
        "context": retriever | RunnableLambda(format_docs),
        "question": RunnablePassthrough(),
    })
    chain = parallel | prompt | llm | StrOutputParser()

    return chain.invoke(
        question,
        config={"run_name": "pdf_rag_query", "tags": ["qa"], "metadata": {"k": 4}}
    )

# ----------------- CLI -----------------
if __name__ == "__main__":
    print("PDF RAG ready. Ask a question (or Ctrl+C to exit).")
    q = input("\nQ: ").strip()
    ans = setup_pipeline_and_query(PDF_PATH, q)
    print("\nA:", ans)
```

**Concept-by-concept:**

- **Fingerprinting (`_file_fingerprint`)** — reads the PDF file in 1MB chunks and computes a
  SHA-256 hash, plus file size and modified-time. This uniquely "fingerprints" the exact
  content of the file — if even one character in the PDF changes, the hash changes completely.
- **Cache key (`_index_key`)** — combines the PDF's fingerprint with `chunk_size`,
  `chunk_overlap`, and `embedding_model` into one dictionary, then hashes *that whole
  dictionary* into a single string. This key uniquely represents "this exact PDF, chunked and
  embedded this exact way." If you change the embedding model or chunk size, you automatically
  get a **different** key, and therefore a fresh (correct) index — no stale/mismatched index.
- **`load_or_build_index(...)`** — the decision-maker: computes the key, checks if a folder for
  that key already exists on disk (`.indices/<key>/`). If yes → **load** the cached FAISS index
  (fast). If no → **build** it from scratch, then save it to disk for next time.
- **`FAISS.save_local(...)` / `FAISS.load_local(...)`** — FAISS's built-in way to persist a
  vector index to disk and reload it later, instead of recomputing embeddings every run.
- Note `load_or_build_index` itself is **not** `@traceable` — it's a lightweight dispatcher;
  the actual work happens inside `load_index_run` or `build_index_run`, which *are* traced.

**Analogy:** This is exactly like **browser caching**. The first time you visit a website,
your browser downloads all the images/CSS/JS (slow). On your next visit, if nothing changed, it
reuses the cached files instantly. If the website updates a file, the browser detects the
change (via a hash/ETag, conceptually similar to our fingerprint) and re-downloads just that
piece. Here, the "website" is your PDF+config, and the "cached files" are the FAISS index.

**Result:** First run might take, say, 30–60 seconds (load + chunk + embed + save). Every
subsequent run with the *same* PDF/config takes only a few seconds (just `load_index_run`) —
and you can literally **see this difference measured** in LangSmith, because `load_index_run`
and `build_index_run` are separate, clearly labeled, traced functions with very different
latencies.

**Interview Q&A:**

- **Q: Why hash the PDF's contents instead of just using the filename to decide whether to
  rebuild the index?**
  A: The filename can stay the same while the content changes (e.g., someone updates the
  policy PDF but keeps calling it `islr.pdf`). Hashing the actual bytes guarantees the cache is
  invalidated whenever the *content* changes, not just when the name changes.
  
- **Q: Besides the PDF content, what else is included in the cache key, and why?**
  A: `chunk_size`, `chunk_overlap`, and `embed_model_name`. If any of these change, the
  resulting chunks/embeddings would be different from what's cached, so they must be part of
  the key — otherwise you'd silently serve a **wrong/stale** index for the new settings.
  
- **Q: What's the difference between `load_index_run` and `build_index_run`, and why are both
  separately `@traceable`?**
  A: `load_index_run` reads an already-built FAISS index off disk (fast, cheap — no embedding
  API calls). `build_index_run` does the full pipeline (load PDF → split → embed → save) and is
  slow/costly. Tracing them separately lets you see in LangSmith exactly which path was taken
  and how long each took — this is the direct fix for the "latency mystery" from Scenario 1.
  
- **Q: What real-world production concept does this file demonstrate?**
  A: Idempotent caching / memoization with cache invalidation based on a content+config hash —
  a very common pattern to avoid redundant, expensive computation (here: PDF parsing + LLM
  embedding calls).

---

### 7.7 `4_agent.py` — tracing a ReAct agent

**Goal:** Move from a fixed pipeline (chain) to an **agent** — a system that decides *which*
tool to call and *when*, in a loop, until it thinks it has the answer. This directly relates to
Scenario 2 (the "keeps looping and burning cost" agent).

```python
from langchain_openai import ChatOpenAI
from langchain_core.tools import tool
import requests
from langchain_community.tools import DuckDuckGoSearchRun
from langchain.agents import create_react_agent, AgentExecutor
from langchain import hub
from dotenv import load_dotenv

load_dotenv()

search_tool = DuckDuckGoSearchRun()

@tool
def get_weather_data(city: str) -> str:
    """
    This function fetches the current weather data for a given city
    """
    url = f'https://api.weatherstack.com/current?access_key=YOUR_WEATHERSTACK_KEY&query={city}'
    response = requests.get(url)
    return response.json()

llm = ChatOpenAI()

# Step 2: Pull the ReAct prompt from LangChain Hub
prompt = hub.pull("hwchase17/react")  # pulls the standard ReAct agent prompt

# Step 3: Create the ReAct agent manually with the pulled prompt
agent = create_react_agent(
    llm=llm,
    tools=[search_tool, get_weather_data],
    prompt=prompt
)

# Step 4: Wrap it with AgentExecutor
agent_executor = AgentExecutor(
    agent=agent,
    tools=[search_tool, get_weather_data],
    verbose=True,
    max_iterations=5
)

# Example questions to try:
# What is the release date of Dhadak 2?
# What is the current temp of gurgaon
# Identify the birthplace city of Kalpana Chawla (search) and give its current temperature.

# Step 5: Invoke
response = agent_executor.invoke({"input": "What is the current temp of gurgaon"})
print(response)
print(response['output'])
```

> Note: the original file hardcodes a Weatherstack access key. Never hardcode real API keys in
> source code — use `os.environ["WEATHERSTACK_KEY"]` loaded from your `.env` instead.

**Concept-by-concept:**

- **`@tool` decorator** — turns a normal Python function (`get_weather_data`) into a **Tool**
  the agent is allowed to call. The function's **docstring** ("This function fetches...")
  becomes the description the LLM reads to decide *when* this tool is useful — so a clear
  docstring is not optional, it's functionally part of the "API" the LLM sees.
- **`DuckDuckGoSearchRun()`** — a ready-made LangChain tool that performs a web search and
  returns text results — used for general knowledge questions the model doesn't already know.
- **`hub.pull("hwchase17/react")`** — downloads a pre-written, battle-tested prompt template
  from LangChain Hub that instructs the LLM how to "think out loud" in the **ReAct** format
  (explained below), rather than writing this prompt from scratch.
- **`create_react_agent(llm, tools, prompt)`** — builds the "brain" of the agent: given the
  ReAct prompt, the LLM will output a *Thought*, then an *Action* (which tool + what input),
  then wait for an *Observation* (the tool's result), and repeat until it decides it's ready to
  give a *Final Answer*.
- **`AgentExecutor(agent, tools, verbose=True, max_iterations=5)`** — the "runtime" that
  actually executes this loop: calls the agent for a decision, executes the chosen tool, feeds
  the result back, and repeats — up to `max_iterations=5` times, as a safety cap against
  infinite loops (directly related to Scenario 2's runaway-cost problem!).

**What is ReAct, simply?** ReAct = **Rea**soning + **Act**ing. Instead of the LLM answering in
one shot, it's prompted to alternate between "thinking" and "doing":

```
Thought: I need to find the current temperature in Gurgaon.
Action: get_weather_data
Action Input: Gurgaon
Observation: {"current": {"temperature": 34, ...}}
Thought: I now have the temperature.
Final Answer: The current temperature in Gurgaon is 34°C.
```

**Analogy:** A fixed chain (from earlier files) is like a **vending machine** — press button A,
get snack A, every time, no decisions made. An **agent** is like a **personal assistant** —
you say "find out how hot it is in Gurgaon," and *they* decide: "I should use the weather tool,
not the search engine, and here's the city name to give it." The assistant reasons about
*which* tool to use and *what* to hand it, in a loop, until satisfied.

**Why tracing matters here (tying back to Scenario 2):** With `AgentExecutor`, each
Thought → Action → Observation cycle is one iteration. LangSmith traces **every single
iteration** as its own Run under the agent's Trace — you can see exactly: which tool was
called, with what input, what it returned, how many iterations happened, and how many tokens
each LLM "thinking" step cost. If an agent starts looping 5 times instead of 1 for a "simple"
question, you will see it immediately as 5 tool-call Runs stacked inside one trace, instead of
just a mysteriously bigger bill at the end of the month.

**Interview Q&A:**

- **Q: What is the difference between a "chain" and an "agent" in LangChain?**
  A: A chain follows a **fixed**, predetermined sequence of steps every time. An agent uses an
  LLM to **dynamically decide**, at runtime, which tool(s) to call, in what order, and when to
  stop — the execution path is not fixed in advance.
- **Q: What does the ReAct pattern stand for, and what problem does it solve?**
  A: "Reasoning + Acting." It interleaves the model's internal reasoning ("Thought") with
  concrete tool calls ("Action") and their results ("Observation"), which tends to produce more
  reliable, explainable, and grounded multi-step behavior than asking for a single final answer
  directly.
- **Q: What is `max_iterations` for, and why is it important in production?**
  A: It caps how many Thought→Action→Observation loops the agent can run before being forced to
  stop. Without a cap, a confused or overly perfectionist agent could loop indefinitely (or for
  a very long time), burning tokens and money — exactly the failure mode from Scenario 2.
- **Q: Why does the docstring inside `@tool def get_weather_data(...)` matter so much?**
  A: The LLM decides *which* tool to call by reading each tool's name and docstring/description
  — it never sees the function's actual code. A vague or missing docstring makes the agent more
  likely to pick the wrong tool or misuse it.
- **Q: How would you debug an agent that suddenly became slow/expensive, using LangSmith?**
  A: Open the trace for a slow/expensive run and look at the number of iterations (Action/
  Observation Runs), which tool(s) were called repeatedly, and the token usage per LLM
  "thinking" call — this pinpoints whether it's looping too much, calling the wrong tool
  repeatedly, or just making unusually large/expensive individual calls.

---

### 7.8 `5_langgraph.py` — tracing a LangGraph workflow

**Goal:** Show that LangSmith isn't limited to chains and simple agents — it also traces
**LangGraph** applications, including parallel ("fan-out/fan-in") branches. The example
evaluates an essay along three dimensions in parallel, then combines the results.

```python
# pip install -U langgraph langchain-openai pydantic python-dotenv langsmith
import operator
from typing import TypedDict, Annotated, List
from dotenv import load_dotenv
from pydantic import BaseModel, Field
from langsmith import traceable
from langchain_openai import ChatOpenAI
from langgraph.graph import StateGraph, START, END

# ---------- Setup ----------
load_dotenv()
model = ChatOpenAI(model="gpt-4o-mini", temperature=0)

# ---------- Structured schema & model ----------
class EvaluationSchema(BaseModel):
    feedback: str = Field(description="Detailed feedback for the essay")
    score: int = Field(description="Score out of 10", ge=0, le=10)

structured_model = model.with_structured_output(EvaluationSchema)

# ---------- Sample essay ----------
essay2 = """India and AI Time
... (sample student essay text) ...
"""

# ---------- LangGraph state ----------
class UPSCState(TypedDict, total=False):
    essay: str
    language_feedback: str
    analysis_feedback: str
    clarity_feedback: str
    overall_feedback: str
    individual_scores: Annotated[List[int], operator.add]  # merges parallel lists
    avg_score: float

# ---------- Traced node functions ----------
@traceable(name="evaluate_language_fn", tags=["dimension:language"], metadata={"dimension": "language"})
def evaluate_language(state: UPSCState):
    prompt = (
        "Evaluate the language quality of the following essay and provide feedback "
        "and assign a score out of 10.\n\n" + state["essay"]
    )
    out = structured_model.invoke(prompt)
    return {"language_feedback": out.feedback, "individual_scores": [out.score]}

@traceable(name="evaluate_analysis_fn", tags=["dimension:analysis"], metadata={"dimension": "analysis"})
def evaluate_analysis(state: UPSCState):
    prompt = (
        "Evaluate the depth of analysis of the following essay and provide feedback "
        "and assign a score out of 10.\n\n" + state["essay"]
    )
    out = structured_model.invoke(prompt)
    return {"analysis_feedback": out.feedback, "individual_scores": [out.score]}

@traceable(name="evaluate_thought_fn", tags=["dimension:clarity"], metadata={"dimension": "clarity_of_thought"})
def evaluate_thought(state: UPSCState):
    prompt = (
        "Evaluate the clarity of thought of the following essay and provide feedback "
        "and assign a score out of 10.\n\n" + state["essay"]
    )
    out = structured_model.invoke(prompt)
    return {"clarity_feedback": out.feedback, "individual_scores": [out.score]}

@traceable(name="final_evaluation_fn", tags=["aggregate"])
def final_evaluation(state: UPSCState):
    prompt = (
        "Based on the following feedback, create a summarized overall feedback.\n\n"
        f"Language feedback: {state.get('language_feedback','')}\n"
        f"Depth of analysis feedback: {state.get('analysis_feedback','')}\n"
        f"Clarity of thought feedback: {state.get('clarity_feedback','')}\n"
    )
    overall = model.invoke(prompt).content
    scores = state.get("individual_scores", []) or []
    avg = (sum(scores) / len(scores)) if scores else 0.0
    return {"overall_feedback": overall, "avg_score": avg}

# ---------- Build graph ----------
graph = StateGraph(UPSCState)
graph.add_node("evaluate_language", evaluate_language)
graph.add_node("evaluate_analysis", evaluate_analysis)
graph.add_node("evaluate_thought", evaluate_thought)
graph.add_node("final_evaluation", final_evaluation)

# Fan-out → join
graph.add_edge(START, "evaluate_language")
graph.add_edge(START, "evaluate_analysis")
graph.add_edge(START, "evaluate_thought")
graph.add_edge("evaluate_language", "final_evaluation")
graph.add_edge("evaluate_analysis", "final_evaluation")
graph.add_edge("evaluate_thought", "final_evaluation")
graph.add_edge("final_evaluation", END)

workflow = graph.compile()

# ---------- Direct invoke without wrapper ----------
if __name__ == "__main__":
    result = workflow.invoke(
        {"essay": essay2},
        config={
            "run_name": "evaluate_upsc_essay",  # becomes root run name
            "tags": ["essay", "langgraph", "evaluation"],
            "metadata": {
                "essay_length": len(essay2),
                "model": "gpt-4o-mini",
                "dimensions": ["language", "analysis", "clarity"],
            },
        },
    )
    print("\n=== Evaluation Results ===")
    print("Language feedback:\n", result.get("language_feedback", ""), "\n")
    print("Analysis feedback:\n", result.get("analysis_feedback", ""), "\n")
    print("Clarity feedback:\n", result.get("clarity_feedback", ""), "\n")
    print("Overall feedback:\n", result.get("overall_feedback", ""), "\n")
    print("Individual scores:", result.get("individual_scores", []))
    print("Average score:", result.get("avg_score", 0.0))
```

**Concept-by-concept:**

- **`StateGraph(UPSCState)`** — LangGraph models your workflow as a **graph**: nodes are
  functions, edges define which node runs next. `UPSCState` is a `TypedDict` describing the
  shared "state" object that flows through the graph, getting updated by each node.
- **`with_structured_output(EvaluationSchema)`** — instead of parsing free text, this forces
  the LLM's output to conform to a `pydantic` schema (`feedback: str`, `score: int` between 0
  and 10) — no manual string parsing/regex needed, and invalid outputs are caught early.
- **`Annotated[List[int], operator.add]`** — this is the key trick for **parallel branches**.
  Three nodes (`evaluate_language`, `evaluate_analysis`, `evaluate_thought`) each return
  `{"individual_scores": [out.score]}` — a **list with one number**. Because the state field is
  annotated with `operator.add` as its "reducer," LangGraph knows to **concatenate** (not
  overwrite) these lists when merging results from parallel branches — so after all three run,
  `individual_scores` correctly contains all three scores, e.g. `[7, 6, 8]`.
- **Fan-out / fan-in edges** — `START` connects to all three evaluator nodes at once (fan-out:
  they run **in parallel**), and all three connect into `final_evaluation` (fan-in: it waits
  for all three to finish before running).
- **`final_evaluation`** — reads all three feedback strings + the merged scores list, asks the
  LLM for one combined summary, and computes the numeric average.
- **`workflow.invoke({...}, config={...})`** — same `RunnableConfig` pattern as before:
  `run_name` labels the root trace, `tags` and `metadata` make it searchable/filterable later.

**Analogy:** Think of this like a **school report card system with three subject teachers**
grading the same essay for Language, Analysis, and Clarity *simultaneously* (fan-out — they
don't wait for each other). Once **all three** teachers submit their grades, the **head teacher**
(`final_evaluation`) combines all three reports into one overall report card (fan-in). LangGraph
is the timetable/scheduler that makes sure the head teacher only starts once all three subject
grades have arrived.

**What LangSmith shows for a graph:** Each node (`evaluate_language`, `evaluate_analysis`,
`evaluate_thought`, `final_evaluation`) appears as its own Run, nested under the graph's root
Trace (named `evaluate_upsc_essay` here thanks to `run_name`). You can see that the three
evaluator nodes ran **in parallel** (overlapping timestamps), each with its own latency and
token cost, and that `final_evaluation` only started after all three finished — making it easy
to spot, say, which one of the three dimensions is slowest, or whether the branches actually
ran in parallel as intended.

**Interview Q&A:**

- **Q: What problem does `Annotated[List[int], operator.add]` solve in LangGraph state?**
  A: By default, when multiple parallel branches return updates to the same state key,
  LangGraph needs to know how to **combine** them (overwrite? merge? error?). Annotating the
  field with a reducer function (`operator.add`, i.e. list concatenation) tells LangGraph to
  append each branch's contribution instead of one overwriting another.
- **Q: How does LangGraph know that `evaluate_language`, `evaluate_analysis`, and
  `evaluate_thought` should run in parallel rather than sequentially?**
  A: All three have an edge directly from `START`, and none of them depend on each other's
  output. LangGraph's execution engine runs nodes with no unmet dependencies concurrently.
- **Q: Why use `with_structured_output(EvaluationSchema)` instead of asking the LLM to just
  return free text and parsing it with string logic?**
  A: It guarantees a well-typed, validated response directly from the model call (score is
  forced to be an int between 0–10, feedback is forced to be a string) — this avoids fragile
  regex/string-parsing bugs and produces cleaner, more reliable traces to debug.
- **Q: Why is LangGraph tracing especially valuable compared to tracing a plain chain?**
  A: Graphs can have conditional branches, loops, and parallel paths — the actual execution
  path isn't fixed in code the way a linear chain is. Tracing lets you see, for any specific
  run, exactly *which* path was taken, which branch triggered a loop, and where time/cost was
  spent — critical for debugging agentic or branching workflows.
- **Q: What's the purpose of passing `config={"run_name": ..., "tags": [...], "metadata": {...}}`
  to `workflow.invoke(...)`?**
  A: Same `RunnableConfig` pattern used throughout the course — it renames the root trace for
  readability and attaches tags/metadata so this specific run can be found and filtered later in
  the LangSmith dashboard (e.g., "show me all `langgraph` + `evaluation` traces from this week").

---

## 8. LLMOps — Beyond Tracing

The video's final section makes clear: **LangSmith is not just a tracing tool** — it covers the
broader lifecycle of running LLM apps in production, often called **LLMOps** (the LLM
equivalent of DevOps/MLOps).

| Feature | What it does | Analogy |
|---|---|---|
| **Monitoring** | Dashboards of metrics **across many traces over time**: traces/day, latency, error rate, LLM-call count/latency, cost, cost-per-trace, input/output tokens, tool usage. | Not just one patient's chart, but the whole **hospital's daily vitals dashboard** — spotting trends before one patient becomes a crisis. |
| **Alerting** | Set thresholds on monitored metrics (latency, cost, error rate, token spikes); get notified when crossed. | A **smoke alarm** — you don't wait to smell smoke yourself, the system pages you the moment a threshold is crossed. |
| **Evaluation** | Systematically score outputs against gold-standard datasets using metrics like faithfulness, relevance, completeness, semantic similarity, "LLM-as-a-judge," or custom Python evaluators — online or offline. | A **standardized exam** with an answer key, instead of eyeballing one or two examples and guessing if the new model version is "better." |
| **Evaluators** | Configurable scoring functions per project — built-in (hallucination, conciseness, code-correctness) or custom. | Different **grading rubrics** you can plug in depending on what you're testing for. |
| **Prompt Experimentation** | Run Prompt A vs Prompt B (or Model A vs Model B) against the same dataset with the same evaluation criteria — real A/B testing, not a one-off "looks better to me." | A **clinical trial** comparing two treatments on the same patient population, not just trying each on one random patient. |
| **Playground** | Interactively test different prompts, schemas, and models side by side. | A **sandbox/workbench** to tinker before shipping. |
| **Prompt Versioning** | Store, version, and collaborate on prompts centrally, like source control for prompts. | **Git, but for prompts.** |
| **Dataset Creation & Annotation** | Build reusable datasets (from scratch, imported rows, or captured traces); manually label/annotate examples; version and reuse across projects. | A **question bank** you keep reusing every time you release a new "student" (model version) to make sure it still knows the material. |
| **User Feedback Integration** | Link thumbs up/down or ratings directly to the exact trace that produced that response. | **Customer complaint forms that are stapled directly to the receipt** for that exact transaction — you know precisely what led to the complaint. |
| **Collaboration** | Share a trace link with a teammate instead of screenshots/manual descriptions; collaborate on prompts, experiments, dashboards, datasets, evaluations. | Sending someone a **direct link to the exact security-camera clip**, instead of describing what you think you saw. |

**Why this matters (theory):** Building an LLM app that works on your test question is one
problem. **Keeping it working reliably as usage scales, models change, and prompts evolve** is
a different, ongoing problem — that's the LLMOps lifecycle LangSmith is designed to support:
Observability → Monitoring → Alerting → Evaluation → Experimentation → Dataset/Feedback
management → Collaboration.

---

## 9. Full Interview Q&A Bank

**Conceptual / "explain it to me" questions**

1. **Q: In one sentence, what problem does LangSmith solve?**
   A: It gives you step-by-step visibility (observability) into non-deterministic, multi-step
   LLM applications, so you can debug latency, cost, and quality problems that don't throw
   normal errors.

2. **Q: Why is debugging an LLM app harder than debugging traditional software?**
   A: Traditional software is deterministic (same input → same output, clear stack traces on
   failure). LLM apps are non-deterministic, multi-step, and often fail "silently" — a fluent
   but wrong answer, a slow step, or a costly loop, with no exception thrown.

3. **Q: Explain the Project → Trace → Run hierarchy.**
   A: A Project groups all traces for one application. A Trace is one complete end-to-end
   execution (e.g., one user request). A Run is the execution of a single component/function
   inside that trace, and Runs can be nested (parent-child) to reflect the call structure.

4. **Q: What's the difference between observability and monitoring?**
   A: Observability is about deeply inspecting a **single** execution (a trace) to understand
   what happened and why. Monitoring is about tracking **aggregate trends across many
   executions over time** (e.g., average latency this week vs last week) to catch systemic
   issues early.

5. **Q: What's the difference between evaluation and monitoring?**
   A: Monitoring watches operational metrics (latency, cost, errors) in production. Evaluation
   measures **output quality** (correctness, faithfulness, relevance) against a reference
   dataset — usually before shipping a change, to prove a new version is actually better, not
   just "looks fine on one example."

6. **Q: Give an example of a bug that would be invisible without tracing but obvious with it.**
   A: An agent looping 4 extra times because of a vague prompt ("keep going until satisfied") —
   no crash occurs, the final report still "looks fine," but the cost quadruples. Tracing shows
   the extra loop iterations directly as repeated tool/LLM Runs.

7. **Q: In a RAG system, how do you tell whether hallucination is a retriever problem or a
   generator problem, using tracing?**
   A: Inspect the trace: look at what documents the retriever actually returned for that
   question. If the returned context is irrelevant/insufficient → retriever problem. If the
   context is correct and relevant but the final answer still contradicts it → generator/prompt
   problem.

8. **Q: What are Tags and Metadata used for, and how do they differ?**
   A: Both help organize/search traces. Tags are simple labels (short strings) for
   categorization/filtering, e.g. `["setup"]`, `["qa"]`. Metadata is structured key-value data
   for richer detail, e.g. `{"embedding_model": "text-embedding-3-small", "k": 4}`.

9. **Q: What is LLMOps, and how does LangSmith fit into it?**
   A: LLMOps is the set of practices/tools for reliably building, deploying, and maintaining
   LLM applications in production — observability, monitoring, alerting, evaluation, prompt
   management, dataset/feedback management, and collaboration. LangSmith provides tooling
   across this whole lifecycle, not just tracing.

10. **Q: Why is `@traceable` needed even though LangChain has "automatic" tracing?**
    A: LangChain's automatic tracing only covers LCEL Runnables (things composed with `|`).
    Plain Python functions, calls to other libraries, and custom logic are invisible to it
    unless explicitly wrapped with `@traceable`.

11. **Q: How would you use LangSmith to prove that a new prompt version is actually better
    before deploying it?**
    A: Run both the old and new prompt against the same evaluation dataset using LangSmith's
    prompt experimentation/evaluation tools, apply the same evaluators (e.g., faithfulness,
    LLM-as-a-judge) to both, and compare aggregate scores rather than eyeballing a couple of
    examples.

12. **Q: How does LangSmith help a team collaborate on debugging a production issue?**
    A: Instead of screenshots and verbal descriptions, an engineer can share a direct link to
    the exact problematic trace; teammates open the same trace and see the same inputs, outputs,
    latencies, and errors — a single shared source of truth.

---

## 10. One-Page Cheat Sheet

**Setup (once per project):**
```bash
pip install -U langchain langchain-openai langchain-community langsmith python-dotenv
```
```bash
# .env
OPENAI_API_KEY="sk-..."
LANGCHAIN_TRACING_V2=true
LANGCHAIN_ENDPOINT="https://api.smith.langchain.com"
LANGCHAIN_API_KEY="lsv2_pt_..."
LANGCHAIN_PROJECT="my-project-name"
```

**Auto-traced (no extra code needed):** any LCEL chain built with `|` — `PromptTemplate`,
`ChatOpenAI`, `StrOutputParser`, retrievers, `RunnableParallel`, LangGraph `StateGraph`, etc.

**Needs manual instrumentation:** plain Python functions (file I/O, hashing, calling non-
LangChain libraries) → wrap with:
```python
from langsmith import traceable

@traceable(name="my_step", tags=["my-tag"], metadata={"key": "value"})
def my_step(...):
    ...
```

**Naming/organizing a specific call:**
```python
chain.invoke(input, config={
    "run_name": "readable_name",
    "tags": ["tag1", "tag2"],
    "metadata": {"key": "value"},
})
```

**Hierarchy recap:**
```
Project
 └── Trace (one end-to-end run)
      └── Run (one component; can nest child Runs)
```

**Beyond tracing:** Monitoring (trends over time) · Alerting (threshold-based notifications) ·
Evaluation (scored comparison against datasets) · Prompt Experimentation (A/B testing prompts
and models) · Prompt Versioning · Dataset creation & Annotation · User Feedback linking ·
Collaboration (shareable trace links).

---

*Source code: [campusx-official/langsmith-masterclass](https://github.com/campusx-official/langsmith-masterclass)
· Notes compiled from the accompanying CampusX video transcript.*
