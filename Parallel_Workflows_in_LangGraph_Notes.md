# Parallel Workflows in LangGraph — Complete Notes

**Theory · Full Code Walkthrough · Interview Q&A**
Based on: Cricket Batsman Workflow (non-LLM) & UPSC Essay Evaluation Workflow (LLM-based)



https://github.com/campusx-official/langgraph-tutorials/blob/main/4_batsman_workflow.ipynb

https://github.com/campusx-official/langgraph-tutorials/blob/main/5_UPSC_essay_workflow.ipynb

---

## Table of Contents

1. [What is a Parallel Workflow?](#1-what-is-a-parallel-workflow)
2. [Sequential vs Parallel Workflow](#2-sequential-vs-parallel-workflow)
3. [Example 1: Cricket Batsman Workflow (Non-LLM)](#3-example-1-cricket-batsman-workflow-non-llm)
4. [The InvalidUpdateError & Partial State Updates](#4-the-invalidupdateerror--partial-state-updates)
5. [Example 2: UPSC Essay Evaluation Workflow (LLM-based)](#5-example-2-upsc-essay-evaluation-workflow-llm-based)
6. [Structured Output — Why and How](#6-structured-output--why-and-how)
7. [Reducers — Merging Parallel Updates](#7-reducers--merging-parallel-updates)
8. [Key Concepts Summary Table](#8-key-concepts-summary-table)
9. [Interview Questions & Answers](#9-interview-questions--answers)
10. [Conclusion](#10-conclusion)

---

## 1. What is a Parallel Workflow?

In LangGraph, a workflow is built using a graph made of **nodes** (functions) and **edges** (connections that decide execution order). In a **sequential (linear) workflow**, one node finishes completely before the next one starts — like a single queue. A **parallel workflow**, on the other hand, allows multiple nodes to branch out from the same point and execute independently, at the same time, before their results are combined again.

The key requirement for two tasks to run in parallel is that **they must not depend on each other's output**. If Task B needs the result of Task A, they cannot run in parallel — Task A must finish first. But if Task A, Task B, and Task C all only need the same original input, there's no reason to make them wait for one another.

> 🍳 **Analogy — The Restaurant Kitchen**
>
> Imagine you order a Thali (a full meal set) at a restaurant. The kitchen doesn't cook the rice, then wait for it to finish before starting the dal, then wait again before starting the sabzi. Instead, three different cooks start working on rice, dal, and sabzi at the same time, because none of them needs the others to be ready first. Once all three are done, a fourth person plates them together into one Thali and sends it to your table. That final plating step is exactly like the **"Summary"** or **"Final Evaluation"** node in a LangGraph parallel workflow — it waits for all parallel branches to finish, then combines their outputs into one result.

---

## 2. Sequential vs Parallel Workflow

| Sequential Workflow | Parallel Workflow |
|---|---|
| One node runs after another, in a single line | Multiple nodes run independently from the same starting point |
| Each node typically depends on the previous node's output | Each parallel node only needs the original/shared input |
| Slower — total time = sum of all node times | Faster — total time ≈ time of the slowest branch, not the sum |
| Example: Prompt chaining — Topic → Outline → Draft → Final polish | Example: Batsman stats — Strike Rate, Boundary %, Balls/Boundary calculated together |
| Diagram: `START → A → B → C → END` | Diagram: `START → {A, B, C} (parallel) → Summary → END` |

---

## 3. Example 1: Cricket Batsman Workflow (Non-LLM)

This first example does not use any LLM at all — it's pure Python logic — but it is the perfect way to learn how parallel branching works in LangGraph before adding the complexity of LLM calls.

### 3.1 Problem Statement

Given a batsman's raw stats — runs scored, balls played, number of fours, and number of sixes — we want to calculate three independent metrics and then generate a text summary:

- **Strike Rate** = (Runs ÷ Balls) × 100
- **Balls per Boundary (BPB)** = Balls ÷ (Fours + Sixes)
- **Boundary Percentage** = ((Fours × 4) + (Sixes × 6)) ÷ Runs × 100

Notice that all three calculations only need the original four inputs (runs, balls, fours, sixes). None of them depends on another calculated value — so all three can run in parallel.

### 3.2 Workflow Diagram

```
                 ┌──────────────────────┐
                 │        START         │
                 └──────────┬───────────┘
        ┌───────────────────┼───────────────────┐
        ▼                   ▼                   ▼
 calculate_sr      calculate_bpb      calculate_boundary_percent
        │                   │                   │
        └───────────────────┼───────────────────┘
                             ▼
                         summary
                             │
                             ▼
                           END
```

### 3.3 Full Notebook Code — Explained Step by Step

Below is the complete, unmodified code from `4_batsman_workflow.ipynb`.

#### Step 1 — Imports

```python
from langgraph.graph import StateGraph, START, END
from typing import TypedDict
```

We import `StateGraph` (the class used to build the graph), `START` and `END` (special markers for the entry and exit points of the graph), and `TypedDict` (used to define the shape of our state).

#### Step 2 — Defining the State

```python
class BatsmanState(TypedDict):

    runs: int
    balls: int
    fours: int
    sixes: int

    sr: float
    bpb: float
    boundary_percent: float
    summary: str
```

`BatsmanState` is a `TypedDict` that describes every piece of data our graph will carry around. The first four fields (`runs`, `balls`, `fours`, `sixes`) are inputs supplied by the user. The remaining four fields (`sr`, `bpb`, `boundary_percent`, `summary`) are outputs that will be filled in as the graph executes. Think of this as the shared whiteboard that every node in the graph can read from and write to.

#### Step 3 — Node: Calculate Strike Rate

```python
def calculate_sr(state: BatsmanState):

    sr = (state['runs']/state['balls'])*100
    
    return {'sr': sr}
```

This function receives the whole state as input but only reads `runs` and `balls`. It calculates the strike rate using the standard cricket formula and returns a dictionary containing **only the key it updated** — `{'sr': sr}` — instead of the entire state. This is called a **partial state update**, and it is critical for parallel workflows (explained in detail in Section 4).

#### Step 4 — Node: Calculate Balls Per Boundary

```python
def calculate_bpb(state: BatsmanState):

    bpb = state['balls']/(state['fours'] + state['sixes'])

    return {'bpb': bpb}
```

This tells us, on average, after how many balls the batsman hits a boundary (a four or a six). It only reads `balls`, `fours`, and `sixes` — completely independent of the strike rate calculation — which is exactly why it can run in parallel with `calculate_sr`.

#### Step 5 — Node: Calculate Boundary Percentage

```python
def calculate_boundary_percent(state: BatsmanState):

    boundary_percent = (((state['fours'] * 4) + (state['sixes'] * 6))/state['runs'])*100

    return {'boundary_percent': boundary_percent}
```

This calculates what percentage of the batsman's total runs came specifically from boundaries. Runs from fours (`fours × 4`) plus runs from sixes (`sixes × 6`), divided by total runs, times 100. Again, it only needs the raw inputs, so it too runs independently and in parallel.

#### Step 6 — Node: Summary

```python
def summary(state: BatsmanState):

    summary = f"""
Strike Rate - {state['sr']} \n
Balls per boundary - {state['bpb']} \n
Boundary percent - {state['boundary_percent']}
"""
    
    return {'summary': summary}
```

The summary node is the **"join point"** of the parallel workflow. It runs only after all three parallel branches have completed, because it depends on their outputs (`sr`, `bpb`, `boundary_percent`). It formats them into a readable string and returns it as a partial update to the `summary` key.

#### Step 7 — Building the Graph

```python
graph = StateGraph(BatsmanState)

graph.add_node('calculate_sr', calculate_sr)
graph.add_node('calculate_bpb', calculate_bpb)
graph.add_node('calculate_boundary_percent', calculate_boundary_percent)
graph.add_node('summary', summary)

# edges

graph.add_edge(START, 'calculate_sr')
graph.add_edge(START, 'calculate_bpb')
graph.add_edge(START, 'calculate_boundary_percent')

graph.add_edge('calculate_sr', 'summary')
graph.add_edge('calculate_bpb', 'summary')
graph.add_edge('calculate_boundary_percent', 'summary')

graph.add_edge('summary', END)

workflow = graph.compile()
```

This is the heart of the parallel design. Notice that `START` is connected to all three calculation nodes **separately** — this is what makes them run in parallel rather than one after another. Then, all three nodes are separately connected to the `summary` node — this tells LangGraph that `summary` must wait until **all three** of its incoming branches have finished. Finally `summary` connects to `END`, and `graph.compile()` turns this definition into a runnable workflow object.

#### Step 8 — Visualizing the Workflow

```python
workflow
```

Running this in a Jupyter cell renders a visual diagram of the graph, confirming the fan-out/fan-in shape shown in Section 3.2.

#### Step 9 — Running the Workflow

```python
intial_state = {
    'runs': 100,
    'balls': 50,
    'fours': 6,
    'sixes': 4
}

workflow.invoke(intial_state)
```

We provide only the four input fields — the rest of the state fields will be filled in automatically as the graph runs. The final output of this call is:

```python
{'runs': 100,
 'balls': 50,
 'fours': 6,
 'sixes': 4,
 'sr': 200.0,
 'bpb': 5.0,
 'boundary_percent': 48.0,
 'summary': '\nStrike Rate - 200.0 \n\nBalls per boundary - 5.0 \n\nBoundary percent - 48.0\n'}
```

- **Strike Rate** = (100/50) × 100 = **200**
- **Balls per Boundary** = 50/(6+4) = **5**
- **Boundary Percentage** = ((6×4)+(4×6))/100 × 100 = **48%**

All three were computed independently and merged automatically into one final state object.

---

## 4. The InvalidUpdateError & Partial State Updates

This is the single most important lesson from the cricket example. If each node had instead returned the **entire state object** (as is commonly done in simple sequential workflows), running the graph would throw an error:

```
InvalidUpdateError
At key 'runs': Can receive only one value per step.
```

### 4.1 Why Does This Happen?

Even though `calculate_sr`, `calculate_bpb`, and `calculate_boundary_percent` only *read* from `runs`, `balls`, `fours`, and `sixes` and never modify them, returning the full state dictionary makes LangGraph think that all three nodes are potentially trying to update **every** key — including `runs`. Since all three run in the same execution step (in parallel), LangGraph suddenly receives three different "updates" for the same key `runs` at the same time, and it has no rule for deciding which one should win. That ambiguity is exactly what the error is complaining about.

> 📮 **Analogy — Three People Editing the Same Line**
>
> Imagine three colleagues editing the same single line of a shared document at the exact same second, each pasting their own version of that line, with no "track changes" or merge rule in place. The system has no way to know whose version to keep. LangGraph reacts the same way when multiple parallel nodes "touch" the same key without a merge rule.

### 4.2 The Fix — Return Only What You Changed

The solution used in the notebook is simple and is considered a best practice: instead of returning the entire state from a node, return a dictionary containing **only the keys that node is actually updating**.

- `calculate_sr` returns only `{'sr': sr}`
- `calculate_bpb` returns only `{'bpb': bpb}`
- `calculate_boundary_percent` returns only `{'boundary_percent': boundary_percent}`

Because each node now only claims ownership of the one key it computed, there is no overlap between the three parallel updates, and LangGraph can safely merge all three partial dictionaries into the single shared state without any conflict.

> 💡 **Rule of Thumb**
>
> In sequential workflows, returning the full state technically works because there's never more than one "writer" active at a time. But in parallel workflows, you **must** return partial updates (only the changed keys). Since partial updates work correctly in both sequential and parallel graphs, it is good practice to always use them.

---

## 5. Example 2: UPSC Essay Evaluation Workflow (LLM-based)

The second example is a realistic, LLM-powered parallel workflow. It mimics a website where a UPSC (Indian civil services exam) aspirant submits an essay and receives feedback on three separate dimensions, each evaluated by an LLM call, followed by a combined final evaluation.

### 5.1 Problem Statement

Given an essay, evaluate it on three independent criteria — **in parallel** — using an LLM for each:

- Clarity of Thought
- Depth of Analysis
- Language Quality

Each of these three LLM calls should return two things: (1) written feedback and (2) a numeric score from 0–10. A fourth node then merges the three feedbacks into one summarized feedback and averages the three scores into a final score.

> 🎓 **Analogy — Three Examiners Grading the Same Answer Sheet**
>
> Picture a UPSC answer sheet being sent simultaneously to three different examiners: one checks only the language, one checks only the depth of the arguments, and one checks only the clarity of thought — all working at the same time, without waiting on each other. Once all three finish, a head examiner reads all three reports, writes one combined remark, and calculates the average marks. That head examiner is the `final_evaluation` node.

### 5.2 Three New Concepts in This Example

1. **Parallel workflow** — same fan-out/fan-in pattern as Example 1.
2. **Structured Output** — forcing the LLM to always reply in a fixed, predictable schema (feedback + score).
3. **Reducers** — a special way to merge values from multiple parallel nodes into a single list, instead of one overwriting another.

---

## 6. Structured Output — Why and How

If we simply asked an LLM in plain language to "give feedback and a score out of 10," it would probably work most of the time — but not always. For example, instead of the number `7`, it might occasionally return the word `"seven"`, or add extra commentary around the number. That unpredictability breaks any code that tries to do maths on the score (like averaging). **Structured Output** solves this by defining a strict schema the model must follow every single time.

### 6.1 Code Walkthrough

#### Imports and Environment Setup

```python
from langgraph.graph import StateGraph, START, END
from langchain_openai import ChatOpenAI
from dotenv import load_dotenv
from typing import TypedDict, Annotated
from pydantic import BaseModel, Field
import operator
```

Along with the usual LangGraph imports, we bring in `ChatOpenAI` (to call an OpenAI chat model), `load_dotenv` (to load the API key from a `.env` file), `Annotated` (needed later for the reducer), `BaseModel` and `Field` from Pydantic (to define our structured output schema), and Python's built-in `operator` module (needed for the reducer function).

```python
load_dotenv()
```

This loads environment variables (like `OPENAI_API_KEY`) from a local `.env` file so the model can authenticate with OpenAI.

```python
model = ChatOpenAI(model='gpt-4o-mini')
```

We create our base LLM instance using the lightweight and cost-efficient GPT-4o-mini model.

#### Defining the Output Schema

```python
class EvaluationSchema(BaseModel):

    feedback: str = Field(description='Detailed feedbackfor the essay')
    score: int = Field(description='Score out of 10', ge=0, le=10)
```

`EvaluationSchema` is a Pydantic model that defines exactly what shape we want the LLM's answer to take: a `feedback` string, and a `score` integer that must be between 0 and 10 (`ge` = greater-than-or-equal, `le` = less-than-or-equal). The `Field` description strings act like extra instructions that help the LLM understand what each field should contain — the more descriptive, the better the results.

#### Binding Structured Output to the Model

```python
structured_model = model.with_structured_output(EvaluationSchema)
```

`with_structured_output(EvaluationSchema)` wraps our base model so that every response it returns is automatically parsed and validated into an `EvaluationSchema` object, instead of a raw block of text. From this point on, calling `structured_model.invoke(...)` returns an object with `.feedback` and `.score` attributes we can rely on.

#### Sample Essay & Quick Test

The notebook defines a sample essay on the topic *"India in the Age of AI"* (a multi-paragraph essay about India's opportunities and challenges in artificial intelligence adoption), then tests the structured model with it:

```python
essay = """India in the Age of AI
As the world enters a transformative era defined by artificial intelligence (AI), India stands at a critical juncture...
"""
```

```python
prompt = f'Evaluate the language quality of the following essay and provide a feedback and assign a score out of 10 \n {essay}'
structured_model.invoke(prompt).feedback
```

This confirms that `structured_model` reliably returns an object we can pull `.feedback` (and similarly `.score`) out of, before we plug it into the graph.

---

## 7. Reducers — Merging Parallel Updates

In Example 1, each parallel node updated a **different** key (`sr`, `bpb`, `boundary_percent`), so there was no conflict once we used partial updates. But in this example, all three evaluation nodes want to contribute to the **same** field: the list of scores. If clarity, analysis, and language all tried to simply "set" `individual_scores`, one would silently overwrite the others. We need a rule that tells LangGraph to **combine (merge)** the values instead of overwriting them — that rule is called a **reducer**.

### 7.1 Defining the State with a Reducer

```python
class UPSCState(TypedDict):

    essay: str
    language_feedback: str
    analysis_feedback: str
    clarity_feedback: str
    overall_feedback: str
    individual_scores: Annotated[list[int], operator.add]
    avg_score: float
```

Every field here should look familiar except one: `individual_scores: Annotated[list[int], operator.add]`. `Annotated` lets us attach extra metadata to a type — here, `list[int]` is the actual data type, and `operator.add` is the **reducer function** that tells LangGraph how to combine updates to this key whenever more than one node writes to it in the same step.

> ➕ **Analogy — A Shared Suggestion Box**
>
> Think of `individual_scores` like a suggestion box that three examiners each drop one slip of paper into, at the same time. Without a reducer, LangGraph would think there can only be one slip in the box and would complain about the conflict (the same `InvalidUpdateError` from Section 4). With `operator.add` as the reducer, LangGraph knows: "whenever a new slip arrives, just add it to the box alongside the existing ones" — so if examiner A drops `[8]`, examiner B drops `[7]`, and examiner C drops `[6]`, the box ends up containing `[8, 7, 6]`. This is Python's `list + list = list` behaviour, exposed as a reusable rule.

### 7.2 The Three Evaluation Nodes

```python
def evaluate_language(state: UPSCState):

    prompt = f'Evaluate the language quality of the following essay and provide a feedback and assign a score out of 10 \n {state["essay"]}'
    output = structured_model.invoke(prompt)

    return {'language_feedback': output.feedback, 'individual_scores': [output.score]}
```

This node builds a prompt asking specifically for a language-quality evaluation of the essay, sends it to `structured_model`, and returns a partial update containing the language feedback text plus the score wrapped in a single-element list (`[output.score]`) — this is essential, because `operator.add` merges **lists**, not raw integers.

```python
def evaluate_analysis(state: UPSCState):

    prompt = f'Evaluate the depth of analysis of the following essay and provide a feedback and assign a score out of 10 \n {state["essay"]}'
    output = structured_model.invoke(prompt)

    return {'analysis_feedback': output.feedback, 'individual_scores': [output.score]}
```

Identical pattern to the previous node, but the prompt now asks the LLM to judge the **depth of analysis** instead of language.

```python
def evaluate_thought(state: UPSCState):

    prompt = f'Evaluate the clarity of thought of the following essay and provide a feedback and assign a score out of 10 \n {state["essay"]}'
    output = structured_model.invoke(prompt)

    return {'clarity_feedback': output.feedback, 'individual_scores': [output.score]}
```

Again the same pattern, this time judging **clarity of thought**. All three nodes are independent LLM calls — none needs the output of the other two — so they qualify to run in parallel.

### 7.3 The Final Evaluation Node

```python
def final_evaluation(state: UPSCState):

    # summary feedback
    prompt = f'Based on the following feedbacks create a summarized feedback \n language feedback - {state["language_feedback"]} \n depth of analysis feedback - {state["analysis_feedback"]} \n clarity of thought feedback - {state["clarity_feedback"]}'
    overall_feedback = model.invoke(prompt).content

    # avg calculate
    avg_score = sum(state['individual_scores'])/len(state['individual_scores'])

    return {'overall_feedback': overall_feedback, 'avg_score': avg_score}
```

This node does two things:

1. It builds a prompt that feeds all three textual feedbacks (language, analysis, clarity) back into the **plain** `model` (not `structured_model` — because we don't want it to invent another score here, just a natural-language summary), and stores the resulting text as `overall_feedback`.
2. It computes the numeric average of `individual_scores` — which, thanks to the reducer, is guaranteed to already contain all three scores by the time this node runs.

Note that `final_evaluation` only executes after all three parallel branches finish, since it depends on their outputs.

### 7.4 Building and Running the Graph

```python
graph = StateGraph(UPSCState)

graph.add_node('evaluate_language', evaluate_language)
graph.add_node('evaluate_analysis', evaluate_analysis)
graph.add_node('evaluate_thought', evaluate_thought)
graph.add_node('final_evaluation', final_evaluation)

# edges
graph.add_edge(START, 'evaluate_language')
graph.add_edge(START, 'evaluate_analysis')
graph.add_edge(START, 'evaluate_thought')

graph.add_edge('evaluate_language', 'final_evaluation')
graph.add_edge('evaluate_analysis', 'final_evaluation')
graph.add_edge('evaluate_thought', 'final_evaluation')

graph.add_edge('final_evaluation', END)

workflow = graph.compile()
```

Exactly the same fan-out/fan-in pattern as the batsman example: `START` branches into three parallel nodes, all three converge into `final_evaluation`, which then leads to `END`.

```python
workflow
```

Displaying the compiled workflow renders/confirms the graph structure — three parallel evaluation nodes feeding into one final evaluation node.

#### Running with a Good Essay

```python
intial_state = {
    'essay': essay
}

workflow.invoke(intial_state)
```

Only the essay needs to be provided — every other field is produced by the graph. Because the sample essay is well-written, the notebook shows all three scores coming back high (for example, something like `[7, 8, 8]` for language, analysis, and clarity respectively), and a good average score.

#### Running with a Deliberately Poor Essay

```python
essay2 = """India and AI Time

Now world change very fast because new tech call Artificial Intel… something (AI). India also want become big in this AI thing...
"""

intial_state = {
    'essay': essay2
}

workflow.invoke(intial_state)
```

To sanity-check the workflow, a second essay is used — one intentionally written with poor grammar and shallow arguments. Running the same workflow on this essay produces noticeably lower scores across all three criteria and a lower overall average, confirming the evaluation logic is actually responding to essay quality rather than returning fixed values.

---

## 8. Key Concepts Summary Table

| Concept | What It Means |
|---|---|
| **Parallel Workflow** | Multiple nodes branch out from the same point and run independently, then converge into a join node |
| **Fan-out** | One node (or `START`) connects to multiple next nodes — this is what creates parallel branches |
| **Fan-in** | Multiple nodes all connect into one common next node, which waits for all of them |
| **Partial State Update** | A node returns only the keys it changed (e.g. `{'sr': sr}`) instead of the whole state — required to avoid write conflicts in parallel branches |
| **InvalidUpdateError** | Raised when two or more parallel nodes try to update the same state key in the same step without a reducer |
| **Reducer** | A merge function (e.g. `operator.add`) attached to a state field via `Annotated[type, reducer]`, telling LangGraph how to combine multiple simultaneous updates to that field |
| **Structured Output** | Using `with_structured_output(Schema)` with a Pydantic model to force an LLM to always reply in a fixed, validated format |
| **`with_structured_output()`** | A LangChain method that wraps a chat model so its responses are parsed directly into the given schema object |

---

## 9. Interview Questions & Answers

**Q1. What is a parallel workflow in LangGraph, and when should you use one?**
Ans: A parallel workflow is a graph structure where multiple nodes execute independently and simultaneously from a common starting point, and their results are later combined by a join node. You should use it whenever two or more tasks only depend on the same original input and not on each other's output — this reduces total execution time compared to running them one after another.

**Q2. How do you create parallel branches in a LangGraph graph?**
Ans: You connect the same source node (often `START`) to multiple different nodes using separate `add_edge` calls, e.g. `graph.add_edge(START, 'node_a')` and `graph.add_edge(START, 'node_b')`. Because both edges originate from the same node, LangGraph treats `node_a` and `node_b` as running in the same execution step, i.e. in parallel.

**Q3. What is the InvalidUpdateError and why does it happen in parallel graphs?**
Ans: It's an error LangGraph raises when two or more nodes running in the same step attempt to write a value to the same state key, and there is no reducer defined to say how those values should be combined. LangGraph cannot decide which value should "win," so it refuses the update and raises the error.

**Q4. How do you fix the InvalidUpdateError?**
Ans: Two ways: (1) make sure each parallel node returns a partial update containing only the keys it actually computed, so there's no overlap between nodes; or (2) if multiple nodes genuinely need to update the same key, attach a reducer function to that key using `Annotated[type, reducer]` (e.g. `operator.add` for lists) so LangGraph knows how to merge the values instead of overwriting them.

**Q5. What is a partial state update, and why is it considered a best practice?**
Ans: It means a node returns a dictionary containing only the state keys it modified (e.g. `{'sr': sr}`) instead of returning the entire state object. It's best practice because it avoids ambiguous, conflicting writes in parallel workflows, and it works equally well in sequential workflows — so using it everywhere keeps your code consistent and safe by default.

**Q6. What is a reducer in LangGraph, and how do you define one?**
Ans: A reducer is a function that tells LangGraph how to merge multiple simultaneous updates to the same state key, instead of the default overwrite behaviour. You define one by wrapping the field's type with `Annotated`, e.g. `individual_scores: Annotated[list[int], operator.add]` — here `operator.add` merges lists using Python's `+` operator, so `[8]`, `[7]`, and `[6]` combine into `[8, 7, 6]`.

**Q7. Why does each score need to be returned as a list, e.g. `[output.score]`, instead of just `output.score`?**
Ans: Because the reducer `operator.add` is designed to combine lists (using the `+` operator). If a node returned a plain integer instead of a single-item list, `operator.add` would try to add integers together (or fail, depending on types) rather than appending each new score into a growing list. Wrapping it as `[output.score]` ensures the three scores end up merged into one list like `[8, 7, 6]` rather than summed into one number.

**Q8. What is Structured Output in LangChain/LangGraph, and why is it needed here?**
Ans: Structured Output forces an LLM's response to conform to a predefined schema (typically a Pydantic model) rather than free-form text. It's needed here because we must reliably extract a numeric score (0-10) and a feedback string from every LLM call — without it, the model might occasionally return the score as text (like "seven") or in an inconsistent format, breaking any downstream maths like averaging.

**Q9. How is Structured Output implemented in the UPSC essay workflow?**
Ans: A Pydantic `BaseModel` called `EvaluationSchema` is defined with two fields — `feedback` (str) and `score` (int, constrained between 0 and 10 using `ge`/`le`). The base chat model is then wrapped using `model.with_structured_output(EvaluationSchema)`, producing `structured_model`, whose `.invoke()` calls return an `EvaluationSchema` object with typed, validated `.feedback` and `.score` attributes.

**Q10. In the essay evaluation workflow, why does the `final_evaluation` node use the plain model instead of `structured_model` for generating the overall feedback?**
Ans: Because `structured_model` is bound to `EvaluationSchema`, which expects (and would try to produce) another feedback + score pair. But `final_evaluation` only needs a free-form summarized feedback paragraph, not another score — using the plain model avoids forcing an unnecessary/irrelevant score field into that step.

**Q11. What determines whether a node in LangGraph should run after another node, or in parallel with it?**
Ans: It comes down to data dependency, expressed via edges. If node B's logic requires a value produced by node A, you connect A to B directly, making B wait for A (sequential). If node B does not need anything from node A and both only depend on the same shared input, you connect both A and B from the same source node (like `START`), allowing LangGraph to execute them in the same step (parallel).

**Q12. What would happen if the summary/final_evaluation node were connected directly from START instead of from the three parallel nodes?**
Ans: It would run too early, before the parallel calculation/evaluation nodes have produced their outputs, so it would try to read state keys (like `sr`, `bpb`, `boundary_percent`, or the three feedbacks/scores) that don't exist yet, causing missing-key errors or incorrect/empty results. The join node must be wired to depend on (receive edges from) every parallel branch it needs data from.

---

## 10. Conclusion

Parallel workflows in LangGraph follow one simple mental model: **fan-out** from a shared starting point into independent branches, then **fan-in** into a join node that waits for all of them. The cricket batsman example teaches the mechanics — how to wire up parallel edges and, critically, why partial state updates (returning only changed keys) are necessary to avoid the `InvalidUpdateError`. The UPSC essay example builds on that foundation with real LLM calls, adding two more essential tools: **Structured Output** (to make LLM responses reliable and machine-readable) and **Reducers** (to safely merge values — like a growing list of scores — that multiple parallel nodes contribute to at once). Together, these two examples give you everything you need to design your own parallel, LLM-powered workflows in LangGraph.

> 🔑 **One-Line Takeaway**
>
> Parallel workflow = fan-out (`START` → many nodes) + fan-in (many nodes → one node), always returning partial updates, and using a reducer whenever more than one parallel branch needs to write to the same field.
