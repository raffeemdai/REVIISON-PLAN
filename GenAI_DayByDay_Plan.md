# Gen AI — Day-by-Day Study Plan (Basics → Advanced)

**Order:** Foundation for Gen AI → RAG → Fine-tuning → Agentic AI → MCP → Deep Agents → Interview Prep

One topic/link per day. Each day: read the notes (45–60 min), do a small hands-on exercise from the linked repo (45–60 min), write a 3-line summary in your own words.

Total: **~90 study days** (about 13 weeks at one day per weekday, or compress by doing 2 days' worth on weekends).

---

## Stage 1 — Foundation for Gen AI (Days 1–6)
*LLM basics, tokens, embeddings, transformer concepts, prompt engineering*

| Day | Topic |
|---|---|
| 1 | [21st April – Foundation](https://directai.blog/2026/04/21/gen-ai-developer-classroom-notes-21-apr-2026/) |
| 2 | [22nd April – Foundation](https://directai.blog/2026/04/22/gen-ai-developer-classroom-notes-22-apr-2026/) |
| 3 | [23rd April – Foundation](https://directai.blog/2026/04/23/gen-ai-developer-classroom-notes-23-apr-2026/) |
| 4 | [26th April – Foundation](https://directai.blog/2026/04/26/gen-ai-developer-classroom-notes-26-apr-2026/) |
| 5 | [23rd June – Foundation](https://directai.blog/2026/06/23/gen-ai-developer-classroom-notes-23-jun-2026/) |
| 6 | [26th June – Foundation](https://directai.blog/2026/06/26/gen-ai-developer-classroom-notes-26-jun-2026/) |

**Reference throughout:** [prompt_engineering_notes_euri.md](https://github.com/raffeemdai/LangChain_LangGraph/blob/main/prompt_engineering_notes_euri.md), [AI Engineering Guidebook (PDF)](https://github.com/raffeemdai/RAG_BY_ME/blob/main/AI%20Engineering%20Guidebook%20Full-compressed.pdf)

**Checkpoint (end of Day 6):** Explain tokens vs. embeddings, how attention works at a high level, and 2 prompt engineering techniques — out loud, no notes.

---

## Stage 2 — RAG (Days 7–23)
*Chunking, embeddings, vector DBs, retrieval strategies, evaluation*

| Day | Topic |
|---|---|
| 7 | [27th June](https://directai.blog/2026/06/27/gen-ai-developer-classroom-notes-27-jun-2026/) |
| 8 | [29th June](https://directai.blog/2026/06/29/gen-ai-developer-classroom-notes-29-jun-2026/) |
| 9 | [30th June](https://directai.blog/2026/06/30/gen-ai-developer-classroom-notes-30-jun-2026/) |
| 10 | [2nd July](https://directai.blog/2026/07/02/gen-ai-developer-classroom-notes-02-jul-2026/) |
| 11 | [4th July](https://directai.blog/2026/07/04/gen-ai-developer-classroom-notes-04-jul-2026/) |
| 12 | [6th July](https://directai.blog/2026/07/06/gen-ai-developer-classroom-notes-06-jul-2026/) |
| 13 | [7th July](https://directai.blog/2026/07/07/gen-ai-developer-classroom-notes-07-jul-2026/) |
| 14 | [8th July](https://directai.blog/2026/07/08/gen-ai-developer-classroom-notes-08-jul-2026/) |
| 15 | [9th July](https://directai.blog/2026/07/09/gen-ai-developer-classroom-notes-09-jul-2026/) |
| 16 | [11th July](https://directai.blog/2026/07/11/gen-ai-developer-classroom-notes-11-jul-2026/) |
| 17 | [13th July](https://directai.blog/2026/07/13/gen-ai-developer-classroom-notes-13-jul-2026-2/) |
| 18 | [15th July](https://directai.blog/2026/07/15/gen-ai-developer-classroom-notes-15-jul-2026/) |
| 19 | [16th July](https://directai.blog/2026/07/16/gen-ai-developer-classroom-notes-16-jul-2026/) |
| 20 | [21st July](https://directai.blog/2026/07/21/gen-ai-developer-classroom-notes-21-jul-2026/) |
| 21 | [22nd July](https://directai.blog/2026/07/22/gen-ai-developer-classroom-notes-22-jul-2026-2/) |
| 22 | [23rd July](https://directai.blog/2026/07/23/gen-ai-developer-classroom-notes-23-jul-2026/) |
| 23 | [25th July](https://directai.blog/2026/07/25/gen-ai-developer-classroom-notes-25-jul-2026/) |

**Hands-on repo:** [RAG_BY_ME](https://github.com/raffeemdai/RAG_BY_ME) — build chunking → embedding → retrieval → generation piece by piece as you go.

**Checkpoint (end of Day 23):** Have a working RAG pipeline on your own dataset. Explain how you'd evaluate it (faithfulness, relevance, hallucination rate).

---

## Stage 3 — Fine-tuning (Days 24–38)
*LoRA/QLoRA, dataset prep, when to fine-tune vs. prompt/RAG*

| Day | Topic |
|---|---|
| 24 | [17th Feb](https://directai.blog/2026/02/17/gen-ai-developer-classroom-notes-17-feb-2026/) |
| 25 | [18th Feb](https://directai.blog/2026/02/18/gen-ai-developer-classroom-notes-18-feb-2026/) |
| 26 | [19th Feb](https://directai.blog/2026/02/19/gen-ai-developer-classroom-notes-19-feb-2026/) |
| 27 | [21st Feb](https://directai.blog/2026/02/21/gen-ai-developer-classroom-notes-21-feb-2026/) |
| 28 | [22nd Feb](https://directai.blog/2026/02/22/gen-ai-developer-classroom-notes-22-feb-2026/) |
| 29 | [24th Feb](https://directai.blog/2026/02/24/gen-ai-developer-classroom-notes-24-feb-2026/) |
| 30 | [25th Feb](https://directai.blog/2026/02/25/gen-ai-developer-classroom-notes-25-feb-2026-2/) |
| 31 | [26th Feb](https://directai.blog/2026/02/26/gen-ai-developer-classroom-notes-26-feb-2026/) |
| 32 | [28th Feb](https://directai.blog/2026/02/28/gen-ai-developer-classroom-notes-28-feb-2026/) |
| 33 | [2nd March](https://directai.blog/2026/03/02/gen-ai-developer-classroom-notes-02-mar-2026/) |
| 34 | [5th March](https://directai.blog/2026/03/05/gen-ai-developer-classroom-notes-05-mar-2026/) |
| 35 | [7th March](https://directai.blog/2026/03/07/gen-ai-developer-classroom-notes-07-mar-2026-2/) |
| 36 | [9th March](https://directai.blog/2026/03/09/gen-ai-developer-classroom-notes-09-mar-2026/) |
| 37 | [10th March](https://directai.blog/2026/03/10/gen-ai-developer-classroom-notes-10-mar-2026/) |
| 38 | [11th March](https://directai.blog/2026/03/11/gen-ai-developer-classroom-notes-11-mar-2026/) |

**Checkpoint (end of Day 38):** Explain LoRA/QLoRA in plain terms, and when you'd choose fine-tuning over RAG or better prompting (cost, data volume, latency).

---

## Stage 4 — Agentic AI (Days 39–62)
*Agent architectures, tool calling, reasoning loops, decision-making*

| Day | Topic |
|---|---|
| 39 | [13th March](https://directai.blog/2026/03/13/gen-ai-developer-classroom-notes-13-mar-2026/) |
| 40 | [16th March](https://directai.blog/2026/03/16/gen-ai-developer-classroom-notes-16-mar-2026/) |
| 41 | [17th March](https://directai.blog/2026/03/17/gen-ai-developer-classroom-notes-17-mar-2026/) |
| 42 | [18th March](https://directai.blog/2026/03/18/gen-ai-developer-classroom-notes-18-mar-2026-2/) |
| 43 | [20th March](https://directai.blog/2026/03/20/gen-ai-developer-classroom-notes-20-mar-2026/) |
| 44 | [23rd March](https://directai.blog/2026/03/23/gen-ai-developer-classroom-notes-23-mar-2026/) |
| 45 | [24th March](https://directai.blog/2026/03/24/gen-ai-developer-classroom-notes-24-mar-2026/) |
| 46 | [25th March](https://directai.blog/2026/03/25/gen-ai-developer-classroom-notes-25-mar-2026/) |
| 47 | [28th March](https://directai.blog/2026/03/28/gen-ai-developer-classroom-notes-28-mar-2026/) |
| 48 | [29th March](https://directai.blog/2026/03/29/gen-ai-developer-classroom-notes-29-mar-2026/) |
| 49 | [31st March](https://directai.blog/2026/03/31/gen-ai-developer-classroom-notes-31-mar-2026/) |
| 50 | [1st April](https://directai.blog/2026/04/01/gen-ai-developer-classroom-notes-01-apr-2026/) |
| 51 | [2nd April](https://directai.blog/2026/04/02/gen-ai-developer-classroom-notes-02-apr-2026/) |
| 52 | [4th April](https://directai.blog/2026/04/04/gen-ai-developer-classroom-notes-04-apr-2026-2/) |
| 53 | [6th April](https://directai.blog/2026/04/06/gen-ai-developer-classroom-notes-06-apr-2026/) |
| 54 | [7th April](https://directai.blog/2026/04/07/gen-ai-developer-classroom-notes-07-apr-2026/) |
| 55 | [9th April](https://directai.blog/2026/04/09/gen-ai-developer-classroom-notes-09-apr-2026/) |
| 56 | [11th April](https://directai.blog/2026/04/11/gen-ai-developer-classroom-notes-11-apr-2026/) |
| 57 | [14th April](https://directai.blog/2026/04/14/gen-ai-developer-classroom-notes-14-apr-2026/) |
| 58 | [18th April](https://directai.blog/2026/04/18/gen-ai-developer-classroom-notes-18-apr-2026/) |
| 59 | [19th April](https://directai.blog/2026/04/19/gen-ai-developer-classroom-notes-19-apr-2026-2/) |
| 60 | [22nd April](https://directai.blog/2026/04/22/gen-ai-developer-classroom-notes-22-apr-2026-3/) |
| 61 | [27th April](https://directai.blog/2026/04/27/gen-ai-developer-classroom-notes-27-apr-2026/) |
| 62 | [7th June](https://directai.blog/2026/06/07/gen-ai-developer-classroom-notes-07-jun-2026/) |

**Hands-on repo:** [LangChain_LangGraph](https://github.com/raffeemdai/LangChain_LangGraph) — build a basic tool-calling agent.

**Checkpoint (end of Day 62):** Explain the difference between a chain and an agent, and how an agent decides which tool to call.

---

## Stage 5 — MCP (Model Context Protocol) (Days 63–76)
*Standardizing how agents connect to tools/data*

| Day | Topic |
|---|---|
| 63 | [30th April](https://directai.blog/2026/04/30/gen-ai-developer-classroom-notes-30-apr-2026/) |
| 64 | [4th May](https://directai.blog/2026/05/04/gen-ai-developer-classroom-notes-04-may-2026/) |
| 65 | [5th May](https://directai.blog/2026/05/05/gen-ai-developer-classroom-notes-05-may-2026-2/) |
| 66 | [6th May](https://directai.blog/2026/05/06/gen-ai-developer-classroom-notes-06-may-2026/) |
| 67 | [9th May](https://directai.blog/2026/05/09/gen-ai-developer-classroom-notes-09-may-2026/) |
| 68 | [10th May](https://directai.blog/2026/05/10/gen-ai-developer-classroom-notes-10-may-2026/) |
| 69 | [12th May](https://directai.blog/2026/05/12/gen-ai-developer-classroom-notes-12-may-2026/) |
| 70 | [13th May](https://directai.blog/2026/05/13/gen-ai-developer-classroom-notes-13-may-2026/) |
| 71 | [14th May](https://directai.blog/2026/05/14/gen-ai-developer-classroom-notes-14-may-2026-2/) |
| 72 | [16th May](https://directai.blog/2026/05/16/gen-ai-developer-classroom-notes-16-may-2026/) |
| 73 | [17th May](https://directai.blog/2026/05/17/gen-ai-developer-classroom-notes-17-may-2026/) |
| 74 | [19th May](https://directai.blog/2026/05/19/gen-ai-developer-classroom-notes-19-may-2026/) |
| 75 | [20th May](https://directai.blog/2026/05/20/gen-ai-developer-classroom-notes-20-may-2026/) |
| 76 | [23rd May](https://directai.blog/2026/05/23/gen-ai-developer-classroom-notes-23-may-2026/) |

**Checkpoint (end of Day 76):** Explain what problem MCP solves that raw function-calling doesn't, and sketch a simple MCP server/client setup.

---

## Stage 6 — Deep Agents (Days 77–85)
*Planning, memory, sub-agent delegation — the most advanced patterns*

| Day | Topic |
|---|---|
| 77 | [8th June](https://directai.blog/2026/06/08/gen-ai-developer-classroom-notes-08-jun-2026/) |
| 78 | [9th June](https://directai.blog/2026/06/09/gen-ai-developer-classroom-notes-09-jun-2026/) |
| 79 | [11th June](https://directai.blog/2026/06/11/gen-ai-developer-classroom-notes-11-jun-2026/) |
| 80 | [12th June](https://directai.blog/2026/06/12/gen-ai-developer-classroom-notes-12-jun-2026/) |
| 81 | [15th June](https://directai.blog/2026/06/15/gen-ai-developer-classroom-notes-15-jun-2026/) |
| 82 | [17th June](https://directai.blog/2026/06/17/gen-ai-developer-classroom-notes-17-jun-2026/) |
| 83 | [18th June](https://directai.blog/2026/06/18/gen-ai-developer-classroom-notes-18-jun-2026/) |
| 84 | [21st June](https://directai.blog/2026/06/21/gen-ai-developer-classroom-notes-21-jun-2026/) |
| 85 | [18th/19th July](https://directai.blog/2026/07/19/gen-ai-developer-classroom-notes-19-jul-2026/) |

**Checkpoint (end of Day 85):** Design a "deep agent" for a multi-step task (e.g., research assistant) covering planning, memory, and sub-agent delegation.

---

## Stage 7 — Interview Prep (Days 86–90)

| Day | Focus |
|---|---|
| 86 | Go through [ai-engineering-interview-questions](https://github.com/raffeemdai/ai-engineering-interview-questions) — Foundation & RAG questions |
| 87 | Interview questions — Fine-tuning & Agentic AI |
| 88 | Interview questions — MCP & Deep Agents |
| 89 | Polish your portfolio project (RAG or agent build) — clean README, clear write-up of design decisions |
| 90 | Mock interview: walk through your project end-to-end out loud, including one failure and how you fixed it |

---

## How to use this
- One day = one link. Don't skip ahead even if a day looks short — later topics assume you've internalized earlier ones.
- If you miss a day, don't try to "double up" on notes — just shift the schedule. Skipping the hands-on part is worse than skipping a reading.
- Re-do the checkpoint question at the start of the *next* stage as a warm-up — this is your spaced repetition.
