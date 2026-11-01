# RAG Chapter 3 — Text Splitters (Complete, Organized Guide)

> All content and all Python code from your notes, reorganized into a clear learning order.
> Every strategy follows the **same template**: Idea → Parameters → Code → Observations → Pros/Cons → Memory trick.

**Legend:** 🔵 Core concept · 🟠 Important · 🟣 Memory trick · 🔴 Caution · 🟢 Practice

refer : https://medium.com/@dewasheesh.rana/the-definitive-guide-to-agentic-rag-text-splitting-chunking-a75766ee307c


---

## 📑 Table of Contents

**PART A — Foundations**
1. Where text splitters fit in RAG
2. What text splitters are
3. Why we need to split text
4. The 3 LangChain methods (+ mixing them)

**PART B — The 5 Strategies**
5. Strategy 1 — Length-based splitting
6. Chunk overlap (works with every strategy)
7. Token-based splitting with `tiktoken` *(an add-on: how to count size, not a strategy)*
8. Splitting `Document` objects *(an add-on: input/output method, not a strategy)*
9. Length-based: pros and cons
10. Strategy 2 — Recursive splitting
11. Strategy 3 — Document-structure-based (Python, JSON, Markdown)
12. Strategy 4 — Semantic chunking
13. Strategy 5 — LLM-based chunking

**PART C — Choosing**
14. Strategy comparison, decision table and decision flow

**PART D — Rules and Revision**
15. Golden rules
16. Common mistakes and fixes
17. Interview Q&A
18. Quick revision sheet
19. Source section map (PDF 3.1–3.15)

---

# PART A — FOUNDATIONS

---

## 1. Where text splitters fit in the RAG pipeline

### 1.1 The story: preparing a library for a smart librarian

You are building an assistant that answers questions from PDFs, text files, web pages and reports.

- **Document loaders** load source files and extract text.
- **Text splitters** divide loaded text into small passages called **chunks**.
- **Embedding models** convert chunks into vectors (lists of numbers representing meaning).
- A **vector database** stores vectors and chunk information.
- At query time, the **retriever** finds relevant chunks and gives them to the LLM as context.

```text
Raw files / URLs / databases
          |
          v
   Document loaders
          |
          v
   Loaded Document objects
          |
          v
     Text splitters      <-- THIS CHAPTER
          |
          v
      Small chunks
          |
          v
     Embedding model
          |
          v
 Vector database / index
          |
          v
Retrieve relevant chunks -> LLM answer
```

### Knowledge source vs. knowledge base

| Term | Meaning | Example |
|---|---|---|
| **Knowledge source** | Where original information lives | A folder of PDFs and Markdown files |
| **Knowledge base** | Indexed representation used for retrieval | A vector database with chunk embeddings and metadata |

### 1.2 Quick recap: loaders and `Document` objects

A loader generally:
1. Loads a source into memory (disk, database, cloud drive, website, etc.).
2. Parses its format and extracts text.
3. Returns a consistent LangChain `Document`.

A `Document` has two attributes:
- `page_content`: extracted text.
- `metadata`: key-value info such as source path, page number, file type, file size.

```python
from langchain_core.documents import Document

doc = Document(
    page_content="RAG combines retrieval with language generation.",
    metadata={"source": "rag_notes.pdf", "page": 12}
)

print(doc.page_content)
print(doc.metadata)
```

🟠 **Important:** Keep metadata when splitting. It lets a retrieved chunk be traced back to its source and page.

LangChain loaders and splitters follow common interface patterns, so you can often swap one without rewriting the whole pipeline.

---

## 2. What are text splitters?

A **text splitter** divides a large string or loaded `Document` into smaller chunks. In LangChain, "text splitting" and "chunking" mean the same general process.

A good chunk is:
- **Coherent:** one related idea.
- **Small enough:** retrieval finds a precise passage.
- **Large enough:** enough context to make sense.
- **Traceable:** keeps source metadata.
- **Usable downstream:** can be embedded and fits the final prompt budget.

```text
One large document
        |
        v
+----------------+  +----------------+  +----------------+
| Chunk 1        |  | Chunk 2        |  | Chunk 3        |
| useful passage |  | useful passage |  | useful passage |
+----------------+  +----------------+  +----------------+
```

🟣 **Memory trick — S-M-T:** **S**mall enough to retrieve precisely, **M**eaningful enough to understand, **T**raceable to its source.

---

## 3. Why do we need to split text?

### 3.1 Context-window limits
An LLM can process only a limited amount of input at once (its **context window**). A document might contain a million tokens while the model accepts far fewer. Too much text can exceed the limit or leave little room for the question and answer. Chunking lets the retriever select a smaller relevant passage. The actual limit depends on the model, and the final prompt still needs a budget.

### 3.2 The "lost in the middle" problem
Even when a long prompt fits, a model may use information near the beginning and end more reliably than details buried in the middle. Smaller, focused passages can help, although chunking does not eliminate this problem entirely.

### 3.3 The most important reason: better retrieval selectivity

Imagine **5 text files** and a retriever returning the **top 5** results.

**Without chunking**
- The loader returns 5 `Document` objects, one per file.
- With a 512-dimensional embedding each, the embedding array has shape **5 × 512**.
- Top 5 out of 5 returns every file. Search cannot select a small passage within a file.
- Entire files may go into context, including irrelevant material.

**With chunking**
- Split each file into about 10 chunks: **50 chunks**.
- Embeddings have shape **50 × 512**.
- Top 5 out of 50 can select the most relevant passages.
- If each file has ~1,000 lines, unchunked context could be 5,000 lines. Five retrieved chunks of ~100 lines each is ~500 lines.

| Setup | Searchable vectors | What top-5 retrieval can do |
|---|---:|---|
| Five whole files | 5 | Returns all five files |
| Five files, ten chunks each | 50 | Selects five passages from many candidates |

🔵 **Key takeaway:** Chunking makes retrieval selective by giving similarity search smaller candidates instead of whole documents.

### 3.4 Does page-wise PDF loading make splitting unnecessary?

No. A loader such as `PyPDFLoader` may return one `Document` per page, but a page is not always a good chunk. Further splitting helps because:

1. **A page may contain multiple topics.** Splitting makes passages more focused.
2. **Embeddings compress text.** Encoding 10,000 words into 512 numbers is heavier compression than 1,000 words into 512 numbers, so smaller coherent passages may preserve topic signals better.
3. **Embedding can be batched or parallelized.** Many chunks can be processed in batches; one huge document produces only one vector.

🔴 **Caution:** Smaller is not always better. Tiny chunks lose context. Choose sizes by testing retrieval quality.

---

## 4. The three LangChain splitter methods

| Method | Input | Output | When to use |
|---|---|---|---|
| `split_text(text)` | One string | List of strings | You have plain text |
| `split_documents(documents)` | List of `Document` | List of smaller `Document` | You used a loader and need metadata preserved |
| `create_documents(texts)` | List of strings | List of `Document` | You want to wrap strings as documents |

```python
from langchain_core.documents import Document
from langchain_text_splitters import RecursiveCharacterTextSplitter

text = """
RAG retrieves information from external sources.
Text is split into chunks before embedding.
The retriever selects relevant chunks for a question.
"""

splitter = RecursiveCharacterTextSplitter(
    chunk_size=80,
    chunk_overlap=10
)

string_chunks = splitter.split_text(text)  # str -> list[str]

docs_from_strings = splitter.create_documents(
    [text],
    metadatas=[{"source": "notes.txt"}]
)

original_docs = [
    Document(page_content=text, metadata={"source": "notes.txt", "page": 1})
]
chunk_docs = splitter.split_documents(original_docs)

print(type(string_chunks[0]))  # str
print(type(chunk_docs[0]))     # Document
print(chunk_docs[0].metadata)  # original metadata is carried forward
```

**Remember the types:**
- `split_text`: `str` → `list[str]`
- `create_documents`: `list[str]` → `list[Document]`
- `split_documents`: `list[Document]` → `list[Document]`

### 4.1 Mixing splitter methods in sequence

You can run two splitters one after another. This is useful when the first step creates text pieces and the next needs `Document` objects.

```text
One long string
     |
     v
First splitter: split_text()
     |
     v
List[str]
     |
     v
Second step: create_documents()
     |
     v
List[Document] ready for metadata and downstream processing
```

```python
from langchain_text_splitters import CharacterTextSplitter
from langchain_core.documents import Document

text = """
RAG retrieves information from a knowledge base.
The retrieved passages are added to the prompt.

The language model uses the passages to answer the question.
"""

# Step 1: Split the string into smaller strings.
first_splitter = CharacterTextSplitter(
    chunk_size=100,
    chunk_overlap=0,
    separator="\n\n"
)
text_pieces = first_splitter.split_text(text)

# Step 2: Turn those strings into Document objects.
second_splitter = CharacterTextSplitter(
    chunk_size=70,
    chunk_overlap=10,
    separator=" "
)
documents = second_splitter.create_documents(
    text_pieces,
    metadatas=[{"source": f"piece_{i}"} for i in range(len(text_pieces))]
)

for doc in documents:
    print(doc.page_content)
    print(doc.metadata)
```

Use a sequence like this only when it solves a real pipeline need. You do not have to use all three methods for every document.

---

# PART B — THE 5 STRATEGIES

> 🔵 **Quick map before the details**
>
> | # | Strategy | Cuts based on | Cost |
> |---|---|---|---|
> | 1 | Length-based | Size | Lowest |
> | 2 | Recursive | Separator hierarchy | Low |
> | 3 | Document-structure-based | File format (headings, code, JSON) | Low |
> | 4 | Semantic | Meaning change (embeddings) | Medium |
> | 5 | LLM-based | Model's judgment | Highest |

---

## 5. Strategy 1 — Length-based splitting

### 5.1 The idea
Cuts text using a configured length and a chosen separator. Simple and fast, but it does not understand meaning.

### 5.2 Core parameters

| Parameter | Meaning |
|---|---|
| `chunk_size` | Target chunk length, measured by the chosen length function |
| `length_function` | Measures length; `len` counts characters |
| `separator` | Preferred place to split |
| `chunk_overlap` | Text repeated between neighbouring chunks |

### 5.3 Common separators

| Separator | Preferred boundary |
|---|---|
| `""` | Character |
| `" "` | Space / word |
| `"\n"` | Line |
| `"\n\n"` | Paragraph |

### 5.4 How size and separator interact
- If a suitable separator occurs before the target length, the splitter can split there and create a shorter chunk.
- If the separator is too coarse (paragraph breaks far apart), a chunk may exceed `chunk_size`.
- Some configurations issue warnings when chunks exceed the target. Exact behavior depends on the splitter and version.

🔴 **Important:** `chunk_size` is not always a strict guarantee. The separator and implementation matter.

### 5.5 Code: `CharacterTextSplitter`

Install the package:

```bash
uv add langchain-text-splitters
```

Or:

```bash
pip install langchain-text-splitters
```

```python
from langchain_text_splitters import CharacterTextSplitter

text = """Artificial intelligence helps computers perform tasks.
Machine learning lets systems learn patterns from data.

Deep learning uses neural networks.
Natural language processing works with human language.

AI is used in healthcare, finance, and education."""

splitter = CharacterTextSplitter(
    chunk_size=100,
    chunk_overlap=0,
    length_function=len,
    separator=" "
)

chunks = splitter.split_text(text)

for i, chunk in enumerate(chunks, start=1):
    print(f"\n--- Chunk {i} | {len(chunk)} characters ---")
    print(chunk)
```

**Helper function for experiments:**

```python
def make_chunks(text, chunk_size, separator, chunk_overlap=0):
    splitter = CharacterTextSplitter(
        chunk_size=chunk_size,
        chunk_overlap=chunk_overlap,
        length_function=len,
        separator=separator
    )
    return splitter.split_text(text)

print(make_chunks(text, 100, ""))
print(make_chunks(text, 100, " "))
print(make_chunks(text, 250, "\n\n"))
```

**Optional colored display** (install first with `uv add termcolor`). The lecture used `termcolor` to print chunks in different colours with indices and lengths. It is a display aid only and does not change splitting.

```python
from termcolor import colored

def print_chunks(chunks):
    colors = ["red", "green", "yellow", "blue", "magenta", "cyan"]
    for i, chunk in enumerate(chunks, start=1):
        color = colors[(i - 1) % len(colors)]
        print(colored(f"\n--- Chunk {i} | {len(chunk)} characters ---", color))
        print(colored(chunk, color))

print_chunks(chunks)
```

### 5.6 Experiments and observations (from the lecture sample)

| Configuration | Observed result | Lesson |
|---|---|---|
| `chunk_size=100`, `separator=""` | ~8 chunks; words such as "algorithm" and "computer" were cut | Character cuts damage context |
| `chunk_size=300`, `separator=""` | ~3 chunks | Larger size → fewer chunks |
| `chunk_size=50`, `separator=""` | More chunks than size 100 | Smaller size → more chunks |
| `chunk_size=100`, `separator=" "` | ~8 chunks, examples around 92 and 95 characters | Word boundaries are more readable |
| `chunk_size=100`, `separator="\n\n"` | Warnings; chunks of 217 and 227 characters | Paragraph breaks too far apart |
| `chunk_size=250`, `separator="\n\n"` | Same 3 paragraph chunks (~217, 227, 263 chars); the last slightly exceeded 250 | A coarse separator can still produce an oversized chunk |

- A trailing space at a cut point may be dropped, so a displayed "100-character" chunk can show 99 characters.
- Do not memorize the numbers as universal. Memorize the relationship:

🟠 **The separator decides where a split is allowed; `chunk_size` guides the target; coarse separators can produce oversized chunks. Tune `chunk_size` and `separator` together.**

---

## 6. Chunk overlap (works with every strategy)

### 6.1 Why overlap is needed
A character cut might split `algorithm` into `algo` and `rithm`, or separate a definition from its explanation. **Chunk overlap** repeats some text from the end of one chunk at the beginning of the next.

```text
Chunk 1: ... machine learning algorithm | retrieval ...
Chunk 2:                 algorithm | retrieval works ...
                         <--- repeated text --->
```

With `chunk_overlap=10`, approximately 10 characters are carried forward, subject to splitter boundary behavior.

### 6.2 Code

```python
from langchain_text_splitters import CharacterTextSplitter

text = (
    "Machine learning uses data to learn patterns. "
    "An algorithm is a sequence of steps. "
    "RAG uses retrieved context to answer questions."
)

for overlap in (0, 10, 20):
    splitter = CharacterTextSplitter(
        chunk_size=60,
        chunk_overlap=overlap,
        separator=" "
    )
    chunks = splitter.split_text(text)

    print(f"\nOVERLAP = {overlap}")
    for i, chunk in enumerate(chunks, start=1):
        print(i, repr(chunk))
```

- `chunk_overlap=0`: no intentional overlap.
- `chunk_overlap=10`: a small amount of text is repeated.
- `chunk_overlap=20`: more context repeated; storage and chunk count may increase.

### 6.3 Where the overlap appears
- The first chunk has overlapping text at its **end**.
- Middle chunks can have overlap at both **start and end**.
- The final chunk generally has overlap at its start only.
- In the character-splitting demo, overlap of 10 characters helped the cut word `algorithm` appear complete in the next chunk.
- With 20 characters of overlap on ~100-character chunks, the sample's chunk count rose from 8 to 9. In a middle chunk, about 20 characters at the start plus 20 at the end may be repeated, approaching **40%** of the chunk.

These are observations from the lecture's sample, not universal counts.

🟣 **Memory trick:** Overlap is a **bridge** between chunks.

🟠 **Important:** Overlap preserves continuity but creates redundancy and extra embedding work. It is a fix for boundary loss, not a substitute for sensible splitting. Prefer word or structural boundaries when possible.

---

## 7. Token-based splitting with `tiktoken`

### 7.0 Clarification: what this section really is

🔵 **Token-based splitting is NOT a 6th strategy.** It is a different **way of measuring length**.

| Question | Answer |
|---|---|
| What does a *strategy* decide? | **Where** to cut (separators, headings, meaning shifts) |
| What does `tiktoken` change? | **How `chunk_size` and `chunk_overlap` are counted** (tokens instead of characters) |
| Where does it apply? | Only to splitters that have a `chunk_size` and a length function |

🟣 **Memory trick:** Strategy = *where to cut*. Token measuring = *how to count*. They are separate dials.

### 7.1 Which strategies can use it?

| Strategy / splitter | Works with token-based measuring? | How |
|---|---|---|
| 1. Length-based (`CharacterTextSplitter`) | ✅ Yes | `CharacterTextSplitter.from_tiktoken_encoder(...)` |
| 2. Recursive (`RecursiveCharacterTextSplitter`) | ✅ Yes (very common) | `RecursiveCharacterTextSplitter.from_tiktoken_encoder(...)` |
| 3a. Structure-based: code (`from_language`) | ⚠️ Partly | `from_language()` measures in characters by default. To count tokens, build the splitter with `from_tiktoken_encoder(separators=...)` using language separators, or pass a token-counting `length_function` |
| 3b. Structure-based: Markdown headers | ❌ Not directly | `MarkdownHeaderTextSplitter` has no `chunk_size`. Use a **second pass** with a token-based recursive splitter |
| 3c. Structure-based: JSON | ❌ No | `max_chunk_size` is based on serialized JSON size, not tokens. Verify token counts afterward if needed |
| 4. Semantic (`SemanticChunker`) | ❌ No | Boundaries come from embedding similarity, not size. Add a token-based second pass if you need a size cap |
| 5. LLM-based | ❌ Not built in | The LLM decides boundaries. You can state a token target in the prompt, then **verify** with `tiktoken` |
| Dedicated `TokenTextSplitter` | ✅ (it is token-based by design) | Cuts strictly by token count, with no separator hierarchy |

🟠 **Rule of thumb:** If a splitter has `chunk_size`, you can usually measure it in tokens. If its boundaries come from structure or meaning, token counting is a **second-pass or verification step**.

### 7.2 Characters vs. tokens
LLMs process **tokens**, not simply characters or words. A token can be a short word, part of a longer word, punctuation, or another unit. Equal character lengths do not mean equal token counts. `tiktoken` is a tokenizer library commonly used with OpenAI models.

Install:

```bash
uv add tiktoken
```

### 7.3 Example A: length-based + tokens

```python
from langchain_text_splitters import CharacterTextSplitter

token_splitter = CharacterTextSplitter.from_tiktoken_encoder(
    encoding_name="cl100k_base",
    chunk_size=50,       # measured in TOKENS
    chunk_overlap=5      # measured in TOKENS
)

chunks = token_splitter.split_text(text)

for i, chunk in enumerate(chunks, start=1):
    print(i, chunk)
```

### 7.4 Example B: recursive + tokens (most common in practice)

```python
from langchain_text_splitters import RecursiveCharacterTextSplitter

splitter = RecursiveCharacterTextSplitter.from_tiktoken_encoder(
    encoding_name="cl100k_base",
    chunk_size=200,      # TOKENS
    chunk_overlap=20     # TOKENS
)

chunks = splitter.split_text(text)
```

This keeps the recursive hierarchy (paragraph → line → word → character) and only changes how size is counted.

### 7.5 Example C: Markdown headers, then token-based size control

```python
from langchain_text_splitters import (
    MarkdownHeaderTextSplitter,
    RecursiveCharacterTextSplitter,
)

header_splitter = MarkdownHeaderTextSplitter(
    headers_to_split_on=[("#", "H1"), ("##", "H2")],
)
section_docs = header_splitter.split_text(markdown_text)

token_size_splitter = RecursiveCharacterTextSplitter.from_tiktoken_encoder(
    encoding_name="cl100k_base",
    chunk_size=200,      # TOKENS
    chunk_overlap=20
)

final_docs = token_size_splitter.split_documents(section_docs)
```

### 7.6 Example D: verifying token counts for ANY strategy

Works as a check on the output of semantic, LLM-based, JSON or any other splitter:

```python
import tiktoken

enc = tiktoken.get_encoding("cl100k_base")

for i, chunk in enumerate(chunks, start=1):
    text_to_count = chunk.page_content if hasattr(chunk, "page_content") else chunk
    print(f"Chunk {i}: {len(enc.encode(text_to_count))} tokens")
```

### 7.7 Key differences from character measuring
- `chunk_size` and `chunk_overlap` are measured in **tokens**.
- The default separator for `CharacterTextSplitter` is generally a paragraph break (`"\n\n"`) unless you supply another.
- The token count controls the **length measure**; the separator controls **preferred cut locations**.

🟠 **Do not confuse token measurement with token-boundary splitting:** using a tokenizer to measure length does not mean every cut is made at a token boundary.

🔴 **Tokenizer must match your model.** `cl100k_base` suits many OpenAI models; other models use different tokenizers, so counts are approximate for them.

---

## 8. Splitting `Document` objects with `split_documents`

### 8.0 Clarification: what this section really is

🔵 **`split_documents` is NOT a strategy either.** It is a **method (input/output type)** that tells a splitter to accept `Document` objects and keep their metadata.

| Question | Answer |
|---|---|
| What does a *strategy* decide? | Where to cut |
| What does `split_documents` decide? | **What goes in and out**: `list[Document]` → `list[Document]`, with metadata carried forward |

🟣 **Memory trick:** Strategy = *the scissors*. Method = *the shape of the box you hand to the scissors*.

### 8.1 Which strategies can use it?

| Strategy / splitter | `split_documents` available? | Notes |
|---|---|---|
| 1. Length-based (`CharacterTextSplitter`) | ✅ Yes | Standard |
| 2. Recursive (`RecursiveCharacterTextSplitter`) | ✅ Yes | Standard |
| 3a. Code (`from_language`) | ✅ Yes | It is still a `RecursiveCharacterTextSplitter` |
| 3b. Markdown headers (`MarkdownHeaderTextSplitter`) | ❌ Not the main path | It takes a **string** via `split_text()` and returns `Document`s with header metadata. For loaded `Document`s, split each `page_content`, or use it first and then call `split_documents` on a second-pass splitter |
| 3c. JSON (`RecursiveJsonSplitter`) | ❌ Not the main path | Uses `split_json()` on a dict. It also offers `create_documents` / `split_text` in some versions |
| 4. Semantic (`SemanticChunker`) | ✅ In common versions | Verify in your installed `langchain-experimental` |
| 5. LLM-based | Depends on your code | No built-in method; your pipeline must carry metadata itself |
| `TokenTextSplitter` | ✅ Yes | Same base interface |

🟠 **Rule of thumb:** `split_documents` is available on the general-purpose text splitters (length-based, recursive, language-aware). Format-specific splitters (Markdown header, JSON) have their own entry points. Always confirm with `help(YourSplitter)` for your installed version.

### 8.2 Why use it
Real loaders (PDF, web, etc.) return `Document` objects. `split_documents()` splits them **and keeps metadata** (`source`, `page`, etc.) on every chunk.

### 8.3 Example A: length-based + `split_documents`

```python
from langchain_core.documents import Document
from langchain_text_splitters import CharacterTextSplitter

text = (
    "RAG retrieves information from an external knowledge base. "
    "Retrieved passages are added to the prompt. "
    "The LLM uses those passages to answer the question."
)

documents = [
    Document(
        page_content=text,
        metadata={"source": "rag_notes.pdf", "page": 5}
    )
]

splitter = CharacterTextSplitter(
    chunk_size=80,
    chunk_overlap=15,
    separator=" "
)

chunk_documents = splitter.split_documents(documents)

for i, doc in enumerate(chunk_documents, start=1):
    print(f"\nChunk {i}: {doc.page_content}")
    print("Metadata:", doc.metadata)
```

### 8.4 Example B: recursive + `split_documents`

```python
from langchain_text_splitters import RecursiveCharacterTextSplitter

splitter = RecursiveCharacterTextSplitter(chunk_size=150, chunk_overlap=20)
chunk_documents = splitter.split_documents(documents)
```

### 8.5 Example C: code splitter + `split_documents`

```python
from langchain_core.documents import Document
from langchain_text_splitters import RecursiveCharacterTextSplitter, Language

code_docs = [Document(page_content=python_code, metadata={"source": "calc.py"})]

splitter = RecursiveCharacterTextSplitter.from_language(
    language=Language.PYTHON, chunk_size=120, chunk_overlap=20
)
chunk_documents = splitter.split_documents(code_docs)
```

### 8.6 Example D: both add-ons together (tokens + `Document`s)

```python
splitter = RecursiveCharacterTextSplitter.from_tiktoken_encoder(
    encoding_name="cl100k_base", chunk_size=200, chunk_overlap=20
)
chunk_documents = splitter.split_documents(documents)
```

This is the typical production pattern: **recursive strategy** (where to cut) + **tokens** (how to count) + **`split_documents`** (what shape goes in/out).

### 8.7 Output characteristics
- A list of `Document` objects, usually one per chunk.
- Each chunk has its own `page_content`.
- Original metadata is carried forward.
- Overlap repeats some text from the previous chunk.
- Chunks can be passed to the embedding model.

🟢 **Practice:** Check that every chunk still has `source` and `page` metadata. Important for citations and debugging.

---

## 8.8 Summary: strategy vs. add-ons (the 3 independent dials)

```text
Dial 1: STRATEGY        -> WHERE to cut        (length / recursive / structure / semantic / LLM)
Dial 2: LENGTH MEASURE  -> HOW to count size   (characters  OR  tokens via tiktoken)
Dial 3: METHOD          -> WHAT shape in/out   (split_text / create_documents / split_documents)
```

| Combination | Example |
|---|---|
| Recursive + characters + `split_documents` | `RecursiveCharacterTextSplitter(...).split_documents(docs)` |
| Recursive + tokens + `split_documents` | `RecursiveCharacterTextSplitter.from_tiktoken_encoder(...).split_documents(docs)` |
| Markdown headers → recursive + tokens | `MarkdownHeaderTextSplitter` then `from_tiktoken_encoder(...).split_documents(...)` |
| Semantic + token **check** | `SemanticChunker(...)` then count with `tiktoken` |

---

## 9. Length-based splitting: advantages and disadvantages

### Advantages

| Advantage | Why |
|---|---|
| Simple | Configure size, overlap and separator |
| Predictable target | Chunks often approach the target size (not guaranteed) |
| Fast | Mostly simple string operations |
| Consistent memory profile | Similar-sized chunks use similar memory |
| Easy to batch / parallelize | Chunks can be processed independently |

### Disadvantages
1. **Can cut mid-word or mid-sentence.** The LLM receives incomplete thoughts.
2. **Ignores structure and meaning.** The end of one topic can mix with the start of another.
3. **Can separate related information.** A definition and its explanation may land in different chunks.
4. **Overlap creates redundancy.** For 100-character chunks and 20 characters of overlap, neighbours repeat text and increase total processing.
5. **Poor for varied structure.** A fixed cut might mix unrelated textbook topics.

The PDF's overlap example: overlap 20 on ~100-character chunks increased the sample from 8 to 9 chunks, and a middle chunk can be ~40% repeated text. This is why overlap should be **tuned, not maximized**.

🔴 **Mentor's rule — keep one coherent context per chunk.** A chunk that mixes unrelated topics is poor even if its length is perfect.

---

## 10. Strategy 2 — Recursive splitting (text-structure-based)

### 10.1 The idea: smart scissors
Length-based splitting uses one boundary rule. **Recursive splitting** tries a hierarchy of separators, starting with larger text units and moving smaller only when necessary.

Default hierarchy:
1. Paragraphs: `"\n\n"`
2. Lines: `"\n"`
3. Words: `" "`
4. Characters: `""`

Implemented by `RecursiveCharacterTextSplitter`. Recommended starting point for many ordinary text documents.

🟣 **Analogy:** Smart scissors first try paragraph boundaries. If a paragraph is too long, they try lines; then words; finally characters.

🔴 **Important:** The default recursive splitter does **not** explicitly recognize sentence punctuation. It may happen to preserve sentences, but sentence detection is not in its default hierarchy.

### 10.2 How recursion and merging work

Suppose `chunk_size=100` and two paragraphs are 200 and 150 characters:

1. Split at paragraph boundaries.
2. If pieces are too long, try line breaks.
3. If lines are still too long, try spaces.
4. If a piece is still too long, fall back to characters.
5. **Merge** smaller neighbouring pieces greedily while respecting the target and overlap.

LangChain performs the merge internally; you normally do not call it yourself.

### 10.3 Worked example from the lecture

Input:

```text
Hi how are you
My name is Rahul

I am teaching RAG

We are Learning about RAG
```

With `chunk_size=10`, `chunk_overlap=0`, the sample output was:

```python
[
    "Hi how are",
    "you",
    "My name",
    "is Rahul",
    "I am",
    "teaching",
    "RAG",
    "We are",
    "Learning",
    "about RAG"
]
```

Why does `you` remain alone rather than joining `My name`? Merging respects the original line boundaries. Separators may be attached to the following piece and count toward its length, which also explains why `"My name is"` may not fit once a leading newline is counted.

### 10.4 Code: `RecursiveCharacterTextSplitter`

```python
from langchain_text_splitters import RecursiveCharacterTextSplitter

text = """
Artificial intelligence is a field of computer science.
Machine learning learns patterns from data.

Natural language processing works with language.
Computer vision helps systems interpret images.

AI affects healthcare, finance, education, and society.
"""

splitter = RecursiveCharacterTextSplitter(
    chunk_size=150,
    chunk_overlap=20
    # No separators passed: use the default hierarchy.
)

chunks = splitter.split_text(text)

for i, chunk in enumerate(chunks, start=1):
    print(f"\n--- Chunk {i} | {len(chunk)} characters ---")
    print(chunk)
```

**With `Document` objects:**

```python
from langchain_core.documents import Document
from langchain_text_splitters import RecursiveCharacterTextSplitter

texts = [text, "A second document about retrieval augmented generation."]
documents = [
    Document(page_content=t, metadata={"source": f"doc_{i}.txt"})
    for i, t in enumerate(texts, start=1)
]

splitter = RecursiveCharacterTextSplitter(
    chunk_size=150,
    chunk_overlap=20
)

chunks = splitter.split_documents(documents)
print("Number of chunks:", len(chunks))
print(chunks[0].metadata)
```

**Overlap rule of thumb:** start at about **10% of `chunk_size`**. If context is lost, try moving toward **20%** and evaluate. These are heuristics, not universal rules.

### 10.5 What the lecture observed
- One sample chunk contained one sentence (~74 characters); another had two sentences (~142 characters, close to the 150 target).
- The default splitter did not explicitly recognize sentence punctuation.
- Words were not cut in that sample.
- Chunk sizes varied even though `chunk_size=150`.

### 10.6 Advantages
- Good balance between chunk size and coherence.
- Tries paragraphs, lines, words, then characters.
- Recommended for many use cases.
- Supports custom separator lists for different formats.

### 10.7 Disadvantages
- Chunk sizes vary.
- Different chunk lengths can mean uneven embedding time and require context-window checks.
- A semantic unit longer than the target may still be split.
- Markdown and code may need format-specific separators.
- Highly structured documents may need a structure-aware approach.

🟢 **Best starting point:** Begin with `RecursiveCharacterTextSplitter`, inspect output, specialize only if necessary.

### 10.8 Length-based vs Recursive (common confusion)

| | Length-based | Recursive |
|---|---|---|
| Separators | One | A hierarchy |
| Behaviour | Cuts where size says | Prefers big natural breaks, falls back to smaller |
| Quality | Lower | Higher |

---

## 11. Strategy 3 — Document-structure-based splitting

### 11.1 The idea
Uses the structure already in a file. It is related to recursive splitting: the algorithm can still be recursive, but the **separator hierarchy changes** to match the format.

| Format | Structure used |
|---|---|
| HTML | Tags such as `<head>` and `<div>` |
| Markdown | Heading levels `#` through `######` |
| JSON | Nested objects and arrays |
| Source code | Classes, function definitions, conditions and loops |

Goal: keep logical units together (for example, a Markdown H1 section divided by H2 subsections, or code split around classes and functions).

The PDF lists HTML tags as possible boundaries; its code demonstrations focus on **Python code, JSON and Markdown**. A structure-aware hierarchy is a preference, not a guarantee: a very long function, section or JSON value can still be split or remain oversized.

🟣 **Analogy:** Organize a book by chapters and sections rather than cutting every thousand characters.

### 11.2 A. Splitting Python code

Use `RecursiveCharacterTextSplitter.from_language()` with the `Language` enum.

```python
from langchain_text_splitters import RecursiveCharacterTextSplitter, Language

python_code = """
class Calculator:
    def add(self, a, b):
        return a + b

    def multiply(self, a, b):
        return a * b

def main():
    calc = Calculator()
    print(calc.add(2, 3))
"""

splitter = RecursiveCharacterTextSplitter.from_language(
    language=Language.PYTHON,
    chunk_size=120,
    chunk_overlap=20,
)
chunks = splitter.split_text(python_code)

for i, chunk in enumerate(chunks, start=1):
    print(f"--- Chunk {i} ---")
    print(chunk)
```

**What this code does**
1. `from_language()` chooses separators intended for Python syntax.
2. The splitter prefers code boundaries (class/function-related separators) before falling back to less meaningful boundaries.
3. `chunk_size=120` is a target measured in characters, not a guarantee every function stays intact.
4. `chunk_overlap=20` repeats a small amount of text between neighbouring chunks.

🔴 Language-aware splitting is **not a full Python parser**. A large function can still be divided. Syntax-sensitive applications may need AST-based parsing or custom logic.

### 11.3 B. Splitting JSON

JSON is hierarchical: an object can contain nested objects, arrays and values. A structure-aware JSON splitter tries to keep that hierarchy meaningful instead of slicing serialized JSON at arbitrary positions.

```python
from langchain_text_splitters import RecursiveJsonSplitter

data = {
    "company": "ExampleCo",
    "departments": [
        {"name": "Engineering", "employees": 120},
        {"name": "Support", "employees": 45},
    ],
    "location": {
        "city": "Atlanta",
        "state": "Georgia",
    },
}

splitter = RecursiveJsonSplitter(max_chunk_size=120)
json_chunks = splitter.split_json(json_obj=data)

for i, chunk in enumerate(json_chunks, start=1):
    print(f"--- JSON chunk {i} ---")
    print(chunk)
```

**JSON-specific options**
- `max_chunk_size`: target/limit used when dividing the JSON structure. Check your installed version for exact behavior.
- `convert_lists`: in versions that expose it, controls whether lists are transformed into a representation that can be split more conveniently.
- `min_chunk_size`: in some versions, discourages extremely small chunks. Confirm the constructor signature in your version.

Do not assume every option exists in every release. If an argument raises `TypeError`, inspect the docs or run `help(RecursiveJsonSplitter)`.

🔴 **JSON caution:** If a chunk is converted to text for embedding, preserve identifying context. A fragment such as `"employees": 45` is less useful than one carrying its path, e.g. `departments[1].employees = 45`. For production RAG, store JSON paths and source IDs as metadata.

### 11.4 C. Splitting Markdown by headings

Markdown headings give a natural hierarchy. `MarkdownHeaderTextSplitter` attaches heading names to each resulting document's metadata so retrieval keeps section context.

```python
from langchain_text_splitters import MarkdownHeaderTextSplitter

markdown_text = """
# Installation
Install Python and create a virtual environment.

## Windows
Use PowerShell to create and activate the environment.

## macOS
Use the terminal to create and activate the environment.

# Troubleshooting
Check the error message before reinstalling packages.
"""

headers_to_split_on = [
    ("#", "Header 1"),
    ("##", "Header 2"),
    ("###", "Header 3"),
]

splitter = MarkdownHeaderTextSplitter(
    headers_to_split_on=headers_to_split_on,
    strip_headers=False,
)

documents = splitter.split_text(markdown_text)

for doc in documents:
    print("METADATA:", doc.metadata)
    print("CONTENT:", doc.page_content)
    print()
```

**Why header metadata helps.** A chunk from the Windows section can carry:

```python
{"Header 1": "Installation", "Header 2": "Windows"}
```

This helps answer "How do I install it on Windows?" because the chunk retains its path through the document's structure.

🔴 **Important distinction:** `MarkdownHeaderTextSplitter` groups text by headings; it does not necessarily enforce a small maximum chunk length. A long section may need a **second pass** with `RecursiveCharacterTextSplitter`:

```python
from langchain_text_splitters import (
    MarkdownHeaderTextSplitter,
    RecursiveCharacterTextSplitter,
)

header_splitter = MarkdownHeaderTextSplitter(
    headers_to_split_on=[("#", "H1"), ("##", "H2")],
)

section_docs = header_splitter.split_text(markdown_text)

size_splitter = RecursiveCharacterTextSplitter(
    chunk_size=300,
    chunk_overlap=40,
)

final_docs = size_splitter.split_documents(section_docs)
```

In a two-stage pipeline, inspect the resulting `Document` objects to confirm heading metadata is preserved in your installed version.

### 11.5 Advantages and disadvantages

**Advantages**
- Preserves headings, code blocks, functions or nested data better than arbitrary cuts.
- Makes chunks more interpretable to the embedding model and the LLM.
- Improves traceability: heading names or data paths can be saved as metadata.
- Useful for technical docs, source code, config files and structured reports.

**Disadvantages**
- Requires knowing the file format and choosing the right splitter.
- A section or function may be too long and need a second splitting pass.
- Incorrect or inconsistent source formatting reduces the benefit.
- Does not automatically guarantee semantic quality or valid executable code.

🟢 **Use it when:** the source has meaningful structure that answers should preserve (Markdown docs, code repositories, JSON records).

---

## 12. Strategy 4 — Semantic similarity-based chunking

### 12.1 The idea: split when the subject changes
Fixed-size splitting asks, "Have I reached the size limit?" Semantic chunking asks, "Has the meaning or topic changed enough that this is a sensible place to start a new chunk?"

> A retriever searches a vector database for relevant passages.
> The embedding model converts text into vectors.
> **The weather in Atlanta is warm in summer.**
> A reranker reorders retrieved passages by relevance.

Sentences 1, 2 and 4 are about a RAG retrieval pipeline. The weather sentence is a topic change and a candidate boundary.

**A semantic chunker typically:**
1. Breaks a document into small units, often sentences.
2. Embeds those units.
3. Compares embeddings for neighbouring units or groups.
4. Finds large changes in similarity/distance.
5. Creates chunks around those changes, subject to its configuration.

The exact algorithm varies by library. Some compare neighbouring sentence groups and use a threshold or percentile to decide boundaries.

### 12.2 Why embeddings are used
An embedding maps text to a numerical vector. Similar meanings often have vectors that are closer under a chosen distance or similarity measure.

- Similar neighbouring sentences → likely keep together.
- A sharp semantic shift → possible boundary.
- Similarity is a **signal**, not a perfect understanding of text.

Two sentences can be related but use different vocabulary, or use similar words while discussing different things. So semantic chunking needs evaluation on your own documents and questions.

### 12.3 Conceptual pseudocode

```python
# Conceptual pseudocode: exact APIs depend on the semantic chunking library.
sentences = split_into_sentences(document_text)
vectors = embedding_model.embed_documents(sentences)

boundaries = []
for i in range(1, len(sentences)):
    similarity = cosine_similarity(vectors[i - 1], vectors[i])

    if similarity < similarity_threshold:
        boundaries.append(i)  # Start a new chunk here

chunks = group_sentences_using_boundaries(sentences, boundaries)
```

This illustrates the idea only. Functions such as `split_into_sentences`, `cosine_similarity` and `group_sentences_using_boundaries` must be implemented or supplied by a library.

### 12.4 Example with LangChain Experimental

LangChain has provided semantic chunking through the experimental package in some versions. APIs change, so install and verify a compatible version first.

```python
# Example API used by some LangChain Experimental versions.
# Install the compatible langchain-experimental and embedding-provider packages first.

from langchain_experimental.text_splitter import SemanticChunker
from langchain_openai import OpenAIEmbeddings

embeddings = OpenAIEmbeddings(model="text-embedding-3-small")

splitter = SemanticChunker(
    embeddings,
    breakpoint_threshold_type="percentile",
    breakpoint_threshold_amount=90,
)

documents = splitter.create_documents([document_text])

for i, doc in enumerate(documents, start=1):
    print(f"--- Semantic chunk {i} ---")
    print(doc.page_content)
```

If imports or parameters differ, check your installed package docs. Semantic chunking is not part of every minimal LangChain install, and the code may need API credentials for the embedding provider.

### 12.5 Thresholds and trade-offs
- **More sensitive boundary detection:** more, smaller chunks.
- **Less sensitive boundary detection:** fewer, larger chunks.
- **Sentence grouping size:** comparing groups reduces noisy decisions but may hide short topic changes.
- **Embedding model:** affects the similarity signal and therefore the boundaries.

No universal threshold exists. Evaluate with representative queries and a small labelled set of "which passage should answer this question?" examples.

### 12.6 Advantages and disadvantages

**Advantages**
- Keeps a coherent idea together even when sentence lengths vary.
- Places boundaries at topic changes instead of arbitrary character counts.
- Useful for long prose, articles and reports with uneven paragraph structure.

**Disadvantages**
- More expensive than simple splitting (embeddings must be computed).
- Quality depends on the embedding model, sentence segmentation and threshold.
- Unpredictable chunk lengths; a later size limit may still be needed.
- A boundary may separate a statement from its qualification, definition or example.

🟣 **Memory trick:** Recursive splitting respects *format boundaries*; semantic chunking tries to respect *meaning boundaries*.

---

## 13. Strategy 5 — LLM-based chunking

### 13.1 The idea
Instead of character counts, separators or embedding similarity, ask a language model to identify logical units. A prompt might ask the model to:
- Preserve headings and their related paragraphs.
- Keep a question with its answer.
- Keep a claim with its supporting explanation.
- Avoid splitting code blocks or tables.
- Return chunks in original order without omitting content.

The model may return boundaries, grouped passages or structured JSON. Your pipeline should validate the result before storing it.

### 13.2 Simple prompt example

```text
You are splitting a document for a retrieval-augmented generation (RAG) system.

Split the document into coherent, self-contained chunks.

Rules:
1. Preserve the original meaning and wording as much as possible.
2. Keep a heading with the paragraphs it introduces.
3. Keep definitions with their explanations when practical.
4. Do not omit facts, examples, warnings, table rows, or code.
5. Preserve the original order.
6. Aim for approximately 300–500 tokens per chunk, but do not split a sentence
   or a table row merely to hit the target.
7. Return JSON: a list of objects with "chunk_id", "heading", and "text".

DOCUMENT:
{document_text}
```

This is a prompt template, not a production pipeline. The application must insert the document safely, call a model, parse the output and validate it.

### 13.3 Safer implementation pattern
Rather than trusting a model to rewrite the source, ask it to identify **boundaries or source spans**, then rebuild chunks from the original text. This reduces accidental paraphrasing or omissions.

**Robust workflow:**
1. Divide a very large document into manageable sections.
2. Ask the LLM to suggest boundaries or source-span IDs.
3. Validate that all spans are present, ordered and non-overlapping (unless overlap is intentional).
4. Check chunk length and required metadata.
5. Compare combined chunks against the source to detect missing content.
6. Store validated chunks and their source references.

If the LLM returns free-form rewritten text, exact source coverage becomes harder to verify.

### 13.4 Advantages and disadvantages

**Advantages**
- Recognizes complex relationships that simple rules miss.
- Keeps explanations, examples, definitions and headings together.
- Adaptable to specialised documents (policies, technical manuals).

**Disadvantages**
- Adds model-call latency and cost.
- May omit, duplicate, reorder or rewrite source content.
- Results can vary between runs or models.
- Requires output parsing, error handling and quality checks.
- Sending private documents to an external model may raise privacy, security or compliance concerns.

🟠 **Production rule:** Never assume LLM-produced chunks are complete just because the response looks well formatted. Validate coverage, order, size, metadata and source traceability.

### 13.5 When it is worth considering
When documents are complex and coherence matters more than ingestion speed or cost (long policy documents, complex manuals, documents with relationships hard to express as separators).

For ordinary PDFs, web pages and text files, first build a strong baseline with recursive splitting and appropriate document structure. Add LLM-based chunking only if evaluation shows a real improvement.

---

# PART C — CHOOSING A STRATEGY

---

## 14. Strategy comparison, decision table and decision flow

### 14.1 Side-by-side comparison

| Strategy | Main signal for a boundary | Good fit | Main risk |
|---|---|---|---|
| Length-based | Character or token target | Simple text, baseline experiments | May cut through meaning |
| Recursive | Hierarchy of separators | General text | Not true semantic understanding |
| Document-structure-based | Headings, code syntax, JSON hierarchy | Manuals, code, structured data | Needs correct format handling |
| Semantic | Embedding similarity change | Topic-changing prose | Extra cost, unstable chunk sizes |
| LLM-based | Model's interpretation | Complex documents | Cost, variability, possible omissions |

### 14.2 Decision table

| Your source or requirement | Good starting strategy | Why | Watch out for |
|---|---|---|---|
| Plain text, simple notes | `CharacterTextSplitter` or recursive | Easy baseline | Arbitrary boundaries can break ideas |
| General PDFs, articles, mixed prose | `RecursiveCharacterTextSplitter` | Tries increasingly smaller separators | Does not fully understand meaning |
| Markdown manuals | `MarkdownHeaderTextSplitter`, then recursive | Preserves heading context | Long sections may remain too large |
| Python or other source code | Language-aware recursive splitter | Prefers code-specific boundaries | Not a full syntax parser |
| Nested JSON | JSON-aware splitter | Uses object/array structure | Preserve field paths and IDs |
| Prose with frequent topic changes | Semantic chunking | Embedding similarity finds topic shifts | Extra cost and variable sizes |
| Complex docs where coherence is critical | LLM-based chunking | Reasons about relationships | Higher cost; validate output |
| Tables | Extract rows and headers deliberately; chunk by row groups | Keeps values with column names | Generic splitting detaches values from headers |
| Very large documents | Section-first splitting, then size control | Controls size, preserves structure | Needs reliable section detection |

### 14.3 Practical decision flow

```text
1. Is the source structured?
     YES -> format-aware splitter (Markdown, code, JSON) or table-specific extraction
     NO  -> recursive splitting
2. Are chunks breaking important ideas?
     Inspect real examples first. Then try better separators,
     heading-aware splitting, or a different size/overlap.
3. Are topic boundaries still poor?
     Test semantic chunking on representative documents.
4. Do complex relationships remain lost?
     Consider LLM-based boundary detection, with strict validation.
5. Does evaluation improve?
     Keep the complex strategy only if measured retrieval or
     answer quality justifies the added cost.
```

### 14.4 Do not choose by fashion
A sophisticated strategy is not automatically better. The right choice depends on:
- Source format and document quality.
- Types of user questions.
- Embedding model and retrieval method.
- Chunk-size limits and the LLM context window.
- Ingestion volume, latency and cost.
- Need for citations, metadata and traceability.

A simple baseline with carefully tuned size and overlap may outperform a complex strategy not tuned for the dataset.

### 14.5 How to evaluate chunking in a RAG system
Create a small evaluation set of real questions with expected source passages. Compare candidate strategies using the same documents, embedding model, retrieval settings and queries.

- **Retrieval recall:** does the correct evidence appear in the top-k chunks?
- **Precision of retrieved context:** how much retrieved material is relevant?
- **Answer correctness and faithfulness:** does the answer match the source evidence?
- **Citation quality:** can the answer be traced to the correct document and section?
- **Chunk statistics:** number of chunks, average/maximum size, overlap, unusually small fragments.
- **Operational cost:** ingestion time, embedding calls, LLM calls, storage.

Change **one major variable at a time**. If chunking, embedding model, top-k and reranking all change together, you cannot tell what helped.

---

# PART D — RULES AND REVISION

---

## 15. Golden rules for chunking

Practical rules of thumb. Treat numbers as starting points, not universal laws.

**Rule 1 — Start with a baseline.** Use a recursive splitter for general text, then inspect its output. Don't begin with the most expensive method unless the format clearly requires it.

**Rule 2 — Preserve meaning, not just length.** Prefer paragraph, heading, sentence, function or row boundaries when they represent complete ideas. A chunk with a perfect character count but a broken definition is a poor retrieval unit.

**Rule 3 — Tune chunk size for the task.**
- Smaller chunks make retrieval more specific.
- Larger chunks preserve broader context.
- Very small chunks lose definitions and surrounding explanation.
- Very large chunks contain irrelevant text and reduce precision.

Test multiple sizes on representative queries.

**Rule 4 — Use overlap deliberately.** Overlap helps when a key definition or sentence crosses a boundary. Too much duplicates content, increases embedding and storage cost, and can fill top-k with near-duplicate chunks. Start modest and measure.

**Rule 5 — Preserve metadata.** Consider at minimum:
- `source` or file identifier
- page number or section
- heading/path
- chunk index
- stable document ID and version

Metadata helps with citations, filtering, debugging and re-ingestion.

**Rule 6 — Treat tables as tables.** Preserve column headers with rows. For long tables, chunk by row groups and repeat the header in each chunk when needed. If a PDF table is extracted as scrambled text, fix extraction before applying a generic splitter.

**Rule 7 — Treat code as code.** Use language-aware splitting. Preserve class/function names and file paths as metadata. If valid syntax is essential, consider syntax-aware parsing and test the chunks.

**Rule 8 — Check the real token count.** Character count is not token count. If the target model has a token limit, use an appropriate tokenizer or verify chunk token counts.

**Rule 9 — Inspect edge cases.** Manually inspect:
- First and last chunks of each document.
- Chunks near headings and page breaks.
- Long tables, lists and code blocks.
- Very short or unusually long chunks.
- Chunks that lack context or duplicate a neighbour.

**Rule 10 — Keep ingestion reproducible.** Record splitter type, parameters, embedding model and document version. If you change the chunking algorithm, reprocess affected documents consistently; avoid mixing incompatible chunking schemes without tracking versions.

**Rule 11 — Avoid losing source traceability.** Each chunk should point back to its original document and, where possible, page, heading, row or character span. Essential for citations and debugging hallucinations.

**Rule 12 — Measure the end result.** Chunking is part of the retrieval pipeline, not an isolated formatting exercise. Evaluate whether the right evidence is retrieved and whether the final answer is correct and grounded.

🟣 **Memory sentence:** *Preserve structure, keep context, control size, track the source, and test retrieval.*

---

## 16. Common mistakes and quick fixes

| Mistake | Why it hurts | Quick fix |
|---|---|---|
| Choosing chunk size only by habit | Different datasets and questions need different context | Compare a few sizes with the same evaluation questions |
| Setting overlap extremely high | Many near-duplicate vectors | Start modestly and measure |
| Splitting Markdown before considering headings | Chunks lose section meaning | Split by headers first, then size-limit long sections |
| Treating every PDF as clean prose | Tables and multi-column layouts get garbled | Use layout-aware extraction and table-specific handling |
| Using semantic chunking without evaluation | More cost does not guarantee better retrieval | Compare with a recursive baseline |
| Trusting LLM output without checking coverage | Facts or sections may disappear | Validate source spans, ordering and completeness |
| Ignoring metadata | Sources and citations become hard to recover | Preserve document IDs, page/section and chunk index |
| Assuming chunk size is a hard guarantee | Some splitters treat it as a target or vary by version | Inspect actual chunk lengths and token counts |
| Changing many pipeline settings together | Can't tell what caused the improvement | Run controlled comparisons |

---

## 17. Interview questions and answers

**Q1. What is recursive text splitting?**
It splits text using a hierarchy of separators. It tries to preserve larger units first (paragraphs), then falls back to smaller units (lines, spaces, characters) as needed to approach the target size.

**Q2. Why is recursive splitting a common default?**
A useful balance between simplicity and preserving natural boundaries. Works reasonably well for general text without a separate semantic model call.

**Q3. What is the difference between recursive and document-structure-based splitting?**
Recursive splitting usually follows a generic separator hierarchy. Document-structure-based splitting chooses boundaries for a particular format (Python syntax, JSON structure, Markdown headings). A format-aware splitter can itself use recursive logic.

**Q4. How does Markdown header splitting help RAG?**
It groups text around headings and can store heading paths in metadata, giving retrieved passages context about their section. Very long sections may still need another size-based pass.

**Q5. How does JSON splitting differ from plain-text splitting?**
It tries to respect nested objects and arrays instead of cutting serialized text at arbitrary positions. Paths or metadata should still be preserved so a value is understandable outside the full JSON.

**Q6. What is semantic chunking?**
It uses embeddings and similarity changes between neighbouring text units to estimate where the topic changes. It may be more coherent than fixed-size splitting, but costs more and needs tuning.

**Q7. Does semantic chunking guarantee better retrieval?**
No. Embedding quality, thresholds, document style and query types all affect results. Evaluate against a simpler baseline.

**Q8. What is LLM-based chunking?**
It uses a language model to identify or create coherent sections from instructions. It handles complex relationships but may omit or rewrite content and adds latency and cost.

**Q9. How can you reduce omissions in LLM-based chunking?**
Prefer boundary detection over free-form rewriting, preserve source spans, validate ordering and coverage, check chunk sizes, and compare reconstructed content with the original.

**Q10. Which chunking strategy is best for all RAG systems?**
None. Start with a sensible baseline, use source-aware splitting for structured files, and compare alternatives using representative queries and retrieval metrics.

**Q11. What is the role of chunk overlap?**
It repeats a small portion of neighbouring text to preserve context when a relevant idea crosses a boundary. Excessive overlap increases duplication and cost.

**Q12. How would you select a chunking strategy for production?**
Inspect source formats and query types, create a baseline, compare chunk sizes and overlap, evaluate retrieval recall and answer faithfulness, inspect failure cases, and account for ingestion cost and latency.

---

## 18. Quick revision sheet

### Five things to remember
1. **Recursive:** try natural separators from larger to smaller.
2. **Structure-based:** use the source's format, such as headings or code boundaries.
3. **Semantic:** use meaning similarity to estimate topic changes.
4. **LLM-based:** use model reasoning, but validate completeness.
5. **Choosing:** judge chunking by retrieval and answer quality, not by how sophisticated the algorithm sounds.

### One-line memory for each strategy
| Strategy | Analogy |
|---|---|
| Length-based | Ruler (measures) |
| Recursive | Smart scissors (big units first) |
| Structure-based | Book chapters (follows the format) |
| Semantic | Topic detector (cuts when meaning shifts) |
| LLM-based | Human editor (reads and decides) |

### Method type cheat sheet
- `split_text`: `str` → `list[str]`
- `create_documents`: `list[str]` → `list[Document]`
- `split_documents`: `list[Document]` → `list[Document]`

---

## 19. Source section map (PDF 3.1–3.15)

| PDF section | Topic | Where in this guide |
|---|---|---|
| 3.1 | Where text splitters fit in the RAG pipeline | Section 1 |
| 3.2 | What text splitters are | Section 2 |
| 3.3 | Why text splitting is needed | Section 3 |
| 3.4 | LangChain splitter methods and mixing methods | Section 4 |
| 3.5 | Strategy 1: length-based splitting | Section 5 |
| 3.6 | Chunk overlap | Section 6 |
| 3.7 | Token-based splitting with `tiktoken` (add-on, see compatibility table) | Section 7 |
| 3.8 | Splitting `Document` objects (add-on, see compatibility table) | Section 8 |
| 3.9 | Length-based: advantages and disadvantages | Section 9 |
| 3.10 | Strategy 2: recursive splitting | Section 10 |
| 3.11 | Strategy 3: document-structure-based | Section 11 |
| 3.12 | Strategy 4: semantic chunking | Section 12 |
| 3.13 | Strategy 5: LLM-based chunking | Section 13 |
| 3.14 | Choosing a chunking strategy | Section 14 |
| 3.15 | Golden rules | Section 15 |

---

## Version and API note
LangChain APIs and package boundaries change over time. Install the packages appropriate to your environment, inspect constructor signatures when an example fails, and pin tested versions in production. The snippets illustrate the strategies; run them against your installed package versions before using them in a pipeline.
