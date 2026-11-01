# 📘 Advanced RAG — Chapter 3: Text Splitters
## Organized, beginner-friendly study notes with Python code

> **Visual guide:** 🔵 Core concept · 🟠 Important point · 🟣 Memory trick · 🔴 Caution · 🟢 Practice

---

# 🧾 Coverage map — original PDF sections 3.1–3.15

This guide follows the full Chapter 3 sequence in the source PDF (printed pages 43–70):

| PDF section | Topic in this guide |
|---|---|
| 3.1 | Where text splitters fit in the RAG pipeline |
| 3.2 | What text splitters are |
| 3.3 | Why text splitting is needed |
| 3.4 | LangChain splitter methods and mixing methods |
| 3.5 | Strategy 1: length-based splitting |
| 3.6 | Chunk overlap |
| 3.7 | Token-based splitting with `tiktoken` |
| 3.8 | Splitting `Document` objects with `split_documents()` |
| 3.9 | Length-based splitting: advantages and disadvantages |
| 3.10 | Strategy 2: recursive splitting |
| 3.11 | Strategy 3: document-structure-based splitting |
| 3.12 | Strategy 4: semantic similarity-based chunking |
| 3.13 | Strategy 5: LLM-based chunking |
| 3.14 | Choosing a chunking strategy |
| 3.15 | Golden rules |

The explanations below are reorganized for learning, but the original section topics are represented in this map.

# 🧭 Chapter map

1. Where text splitters fit in RAG
2. What chunking is and why it matters
3. LangChain splitter methods
4. Strategy 1 — Length-based splitting
5. Chunk overlap
6. Token-based splitting with `tiktoken`
7. Splitting `Document` objects
8. Length-based splitting: pros and cons
9. Strategy 2 — Recursive splitting
10. Strategy 3 — Document-structure-based splitting: Python, JSON, Markdown
11. Strategy 4 — Semantic chunking
12. Strategy 5 — LLM-based chunking
13. Strategy comparison and selection
14. Golden rules, interview questions, practice, revision sheet

---

# 1. Where text splitters fit in the RAG pipeline

## 1.1 The story: preparing a library for a smart librarian

Imagine building an assistant that answers questions from PDFs, text files, web pages, and reports.

- **Document loaders** load source files and extract their text.
- **Text splitters** divide the loaded text into smaller passages called **chunks**.
- **Embedding models** convert chunks into vectors (lists of numbers representing meaning).
- A **vector database** stores the vectors and related chunk information.
- At query time, the retriever searches for relevant chunks and gives them to the LLM as context.

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
     Text splitters
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
| **Knowledge base** | Indexed representation used for retrieval | A vector database containing chunk embeddings and metadata |

## 1.2 Quick recap: loaders and `Document` objects

A loader generally:
1. Loads a source into memory (disk, database, cloud drive, website, etc.).
2. Parses its format and extracts text.
3. Returns a consistent LangChain `Document`.

A `Document` has two important attributes:

- `page_content`: extracted text.
- `metadata`: key-value information such as source path, page number, file type, or file size.

```python
from langchain_core.documents import Document

doc = Document(
    page_content="RAG combines retrieval with language generation.",
    metadata={"source": "rag_notes.pdf", "page": 12}
)

print(doc.page_content)
print(doc.metadata)
```

🟠 **Important:** Keep metadata when splitting. It allows a retrieved chunk to be traced back to its source and page.

LangChain loaders and splitters follow common interface patterns, so you can often swap a loader or splitter without rewriting the entire pipeline.

---

# 2. What are text splitters?

A **text splitter** divides a large string or loaded `Document` into smaller chunks. In LangChain, “text splitting” and “chunking” mean the same general process.

A good chunk is:
- **Coherent:** contains one related idea.
- **Small enough:** retrieval can find a precise passage.
- **Large enough:** has enough context to make sense.
- **Traceable:** retains source metadata.
- **Usable downstream:** can be embedded and fit within the final prompt budget.

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

# 3. Why do we need to split text?

## 3.1 Context-window limits

An LLM can only process a limited amount of input at one time. This limit is its **context window**. A document might contain a million tokens while the model accepts far fewer. Too much text can exceed the limit or leave little room for the question and answer. Chunking lets the retriever select a smaller, relevant passage.

The actual context limit depends on the model, and the final prompt still needs a budget.

## 3.2 The “lost in the middle” problem

Even when a long prompt fits, a model may use information near the beginning and end more reliably than details buried in the middle. Smaller, focused retrieved passages can help, although chunking does not eliminate this problem entirely.

## 3.3 The most important reason: better retrieval selectivity

Imagine a knowledge source containing **5 text files** and a retriever configured to return the top 5 results.

### Without chunking
- The loader returns 5 `Document` objects — one per file.
- If each document is represented by a 512-dimensional embedding, the embedding array has shape **5 × 512**.
- Retrieving top 5 out of 5 returns every file. Search cannot select a small passage within a file.
- The entire files may be placed into context, including irrelevant material.

### With chunking
- Split each file into about 10 chunks: **50 chunks**.
- Embeddings have shape **50 × 512**.
- Retrieving top 5 out of 50 can select the most relevant passages.
- If each file has about 1,000 lines, the unchunked context could contain 5,000 lines. Five retrieved chunks of about 100 lines each could contain around 500 lines.

| Setup | Searchable vectors | What top-5 retrieval can do |
|---|---:|---|
| Five whole files | 5 | Returns all five files |
| Five files, ten chunks each | 50 | Selects five passages from many candidates |

🔵 **Key takeaway:** Chunking makes retrieval selective by giving similarity search smaller candidates instead of whole documents.

## 3.4 Does page-wise PDF loading make splitting unnecessary?

No. A PDF loader such as `PyPDFLoader` may return one `Document` per page, but a page is not always a good chunk.

Further splitting helps because:
1. **A page may contain multiple topics.** Splitting can create more focused passages.
2. **Embeddings compress text.** Encoding 10,000 words into 512 numbers is heavier compression than encoding 1,000 words into 512 numbers, so smaller coherent passages may preserve useful topic signals better.
3. **Embedding can be batched or parallelized.** Many chunks can be processed in batches, whereas one huge document produces only one vector.

🔴 **Caution:** Smaller is not always better. Tiny chunks can lose context; choose sizes by testing retrieval quality.

---

# 4. The three common LangChain splitter methods

| Method | Input | Output | When to use |
|---|---|---|---|
| `split_text(text)` | One string | List of strings | You have plain text |
| `split_documents(documents)` | List of `Document` objects | List of smaller `Document` objects | You loaded documents and need metadata preserved |
| `create_documents(texts)` | List of strings | List of `Document` objects | You want to wrap strings as documents |

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

🟠 **Remember:** Choose the method according to the input you have and what the next pipeline step needs.


## 4.1 Mixing splitter methods in sequence

The PDF also explains that you can run two splitters one after another. This is useful when the first splitter creates text pieces and the next step needs `Document` objects.

**Example flow:**

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

**Remember the input/output types:**

- `split_text`: `str` → `list[str]`
- `create_documents`: `list[str]` → `list[Document]`
- `split_documents`: `list[Document]` → `list[Document]`

Use a sequence like this only when it solves a real pipeline need; you do not have to use all three methods for every document.

---

# 5. Strategy 1 — Length-based splitting

## 5.1 The idea

Length-based splitting cuts text using a configured length and a chosen separator. It is simple and fast but does not understand meaning.

### Core parameters

| Parameter | Meaning |
|---|---|
| `chunk_size` | Target chunk length measured by the chosen length function |
| `length_function` | Measures length; `len` counts characters |
| `separator` | Preferred place to split |
| `chunk_overlap` | Text repeated between neighbouring chunks |

### Common separators

| Separator | Preferred boundary |
|---|---|
| `""` | Character |
| `" "` | Space / word |
| `"\n"` | Line |
| `"\n\n"` | Paragraph |

### How size and separator interact
- If a suitable separator occurs before the target length, the splitter can split there and create a shorter chunk.
- If the separator is too coarse — for example, paragraph breaks are far apart — a chunk may exceed `chunk_size`.
- Some configurations issue warnings when chunks exceed the target. Exact behavior depends on the splitter and version.

🔴 **Important:** `chunk_size` is not always a strict guarantee that every chunk will be below the limit. The separator and implementation matter.

## 5.2 Code: `CharacterTextSplitter`

Install the package if needed:

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
    print(f"\\n--- Chunk {i} | {len(chunk)} characters ---")
    print(chunk)
```

A helper function makes experiments easy:

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
print(make_chunks(text, 250, "\\n\\n"))
```

The lecture also used `termcolor` to print chunks in different colours, with indices and lengths, to make boundaries easier to see. This is a display aid only; it does not change splitting.


Optional terminal display (install `termcolor` first with `uv add termcolor`):

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

This makes it easier to see where chunks begin and end. The colours are only for terminal readability; they do not affect the splitter or embedding quality.

## 5.3 Experiments and observations

| Setting | Observation in the lecture's sample | Lesson |
|---|---|---|
| `chunk_size=100`, `separator=""` | About 8 chunks; words such as “algorithm” and “computer” were cut | Character cuts can damage context |
| `chunk_size=300`, `separator=""` | About 3 chunks | Larger size generally means fewer chunks |
| `chunk_size=50`, `separator=""` | More chunks | Smaller size generally means more chunks |
| `chunk_size=100`, `separator=" "` | About 8 chunks, examples around 92 and 95 characters | Word boundaries are more readable |
| `chunk_size=100`, `separator="\\n\\n"` | Warnings and chunks of 217 and 227 characters | Paragraph breaks can be too far apart |
| `chunk_size=250`, `separator="\\n\\n"` | Same 3 paragraph-based chunks, around 217, 227 and 263 characters | Tune size and separator together; a coarse separator can still yield an oversized chunk |

A trailing space at a cut point may be dropped, so a displayed “100-character” chunk can show 99 characters.

🟠 **Mentor's rule — tune `chunk_size` and `separator` together.** If you see an oversized chunk, check whether your separator occurs frequently enough.


The PDF's experiment table is worth memorizing:

| Configuration | Lecture's observed result |
|---|---|
| `chunk_size=100`, `separator=""` | 8 chunks; words such as “algorithm” and “computer” were cut |
| `chunk_size=300`, `separator=""` | 3 chunks |
| `chunk_size=50`, `separator=""` | More chunks than with size 100 |
| `chunk_size=100`, `separator=" "` | 8 chunks, examples around 92 and 95 characters; word boundaries were respected |
| `chunk_size=100`, `separator="\n\n"` | Warnings and chunks of 217 and 227 characters because paragraph breaks were too far apart |
| `chunk_size=250`, `separator="\n\n"` | Same 3 paragraph chunks, about 217, 227, and 263 characters; the last slightly exceeded 250 |

The key lesson is not to memorize the numbers as universal behavior. Memorize the relationship: **the separator decides where a split is allowed; `chunk_size` guides the target, and coarse separators can produce oversized chunks.**

---

# 6. Chunk overlap

## 6.1 Why overlap is needed

A character cut might split `algorithm` into `algo` and `rithm`, or separate a definition from its explanation. **Chunk overlap** repeats some text from the end of one chunk at the beginning of the next.

```text
Chunk 1: ... machine learning algorithm | retrieval ...
Chunk 2:                 algorithm | retrieval works ...
                         <--- repeated text --->
```

With `chunk_overlap=10`, approximately 10 characters are carried forward, subject to splitter boundary behavior.

## 6.2 Code example

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

    print(f"\\nOVERLAP = {overlap}")
    for i, chunk in enumerate(chunks, start=1):
        print(i, repr(chunk))
```

- `chunk_overlap=0`: no intentional overlap.
- `chunk_overlap=10`: a small amount of text is repeated.
- `chunk_overlap=20`: more context is repeated; storage and chunk count may increase.

🟣 **Memory trick:** Overlap is a **bridge** between chunks.


The lecture also demonstrates how to inspect the overlap visually:

- The first chunk has overlapping text at its **end**.
- Middle chunks can have overlap at both their **start and end**.
- The final chunk generally has overlap at its start but has no following chunk to overlap with.
- In the character-splitting demo, overlap of 10 characters helps the cut word `algorithm` appear complete in the next chunk.
- With 20 characters of overlap on roughly 100-character chunks, the example's chunk count increased from 8 to 9. In a middle chunk, about 20 characters at the start plus 20 at the end may be repeated, approaching 40% of the chunk.

These are observations from the lecture's sample, not universal counts. Actual results depend on text and splitter settings.

🟠 **Important:** Overlap can help preserve continuity, but it creates redundancy and extra embedding work. It is a fix for boundary loss, not a substitute for sensible splitting. Prefer word or structural boundaries when possible.

---

# 7. Token-based splitting with `tiktoken`

## 7.1 Characters vs. tokens

LLMs process text as **tokens**, not simply characters or words. A token can be a whole short word, part of a longer word, punctuation, or another text unit. Equal character lengths do not necessarily mean equal token counts.

`tiktoken` is a tokenizer library commonly used with OpenAI models.

Install:

```bash
uv add tiktoken
```

Example:

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

Key differences:
- `chunk_size` and `chunk_overlap` are measured in tokens.
- The default separator for `CharacterTextSplitter` is generally a paragraph break (`"\\n\\n"`) unless you supply another separator.
- The token count controls the length measure, while the separator controls preferred cut locations.

🟠 **Do not confuse token measurement with token-boundary splitting:** using a tokenizer to measure length does not mean every cut is made at a token boundary.

---

# 8. Splitting `Document` objects with `split_documents`

Real loaders commonly return `Document` objects. Use `split_documents()` to split them while retaining metadata.

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
    print(f"\\nChunk {i}: {doc.page_content}")
    print("Metadata:", doc.metadata)
```

Output characteristics:
- A list of `Document` objects, usually one per chunk.
- Each chunk has its own `page_content`.
- Original metadata is carried forward.
- Overlap repeats some text from the previous chunk.
- The chunks can be passed to the embedding model.

🟢 **Practice:** Check that every chunk still has `source` and `page` metadata. This is important for citations and debugging.

---

# 9. Length-based splitting: advantages and disadvantages

## Advantages

| Advantage | Why |
|---|---|
| Simple | Configure size, overlap and separator |
| Predictable target | Chunks often approach the target size, though not guaranteed |
| Fast | Mostly simple string operations |
| Consistent memory profile | Similar-sized chunks use similar memory |
| Easy to batch / parallelize | Chunks can be processed independently |

## Disadvantages

1. **Can cut mid-word or mid-sentence.** The LLM receives incomplete thoughts.
2. **Ignores structure and meaning.** The end of one topic can be mixed with the start of another.
3. **Can separate related information.** A definition and its explanation may be in different chunks.
4. **Overlap creates redundancy.** For 100-character chunks and 20 characters of overlap, neighbouring chunks repeat text and increase total processing.
5. **Poor for varied structure.** A fixed cut might mix unrelated textbook topics.


The PDF's overlap example notes that using overlap 20 on approximately 100-character chunks increased the sample from 8 to 9 chunks. Since a middle chunk can repeat about 20 characters at its beginning and 20 at its end, roughly 40% of that chunk can be repeated text. This is why overlap should be tuned rather than maximized.

🔴 **Mentor's rule — keep one coherent context per chunk.** A chunk that mixes unrelated topics is poor even if its length is perfect.

---

# 10. Strategy 2 — Text-structure-based splitting (recursive splitting)

## 10.1 The idea: smart scissors

Length-based splitting uses one boundary rule. **Recursive splitting** tries a hierarchy of separators, starting with larger text units and moving to smaller units only when necessary.

Default hierarchy:

1. Paragraphs: `"\\n\\n"`
2. Lines: `"\\n"`
3. Words: `" "`
4. Characters: `""`

It is implemented by `RecursiveCharacterTextSplitter` and is a recommended starting point for many ordinary text documents.

🟣 **Analogy:** Smart scissors first try paragraph boundaries. If a paragraph is too long, they try lines; if a line is too long, words; finally, characters.

🔴 **Important:** The default recursive splitter does **not** explicitly recognize sentence punctuation. It may happen to preserve sentences, but sentence detection is not its default hierarchy.

## 10.2 How recursion and merging work

Suppose `chunk_size=100`, and two paragraphs are 200 and 150 characters long:

1. Split at paragraph boundaries.
2. If the pieces are too long, try line breaks.
3. If lines are still too long, try spaces.
4. If an individual piece is still too long, the splitter can fall back to characters.
5. Merge smaller neighbouring pieces greedily while respecting the target and overlap.

LangChain performs the merge internally; you normally do not call its internal merge function yourself.

### Worked example from the lecture

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

Why does `you` remain alone rather than joining `My name`? In this example, merging respects the original line boundaries. Separators may be attached to the following piece and count toward its length, which also explains why `"My name is"` may not fit once a leading newline is counted.

## 10.3 Code: `RecursiveCharacterTextSplitter`

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
    print(f"\\n--- Chunk {i} | {len(chunk)} characters ---")
    print(chunk)
```

For `Document` objects:

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

A reasonable starting overlap is about **10% of `chunk_size`**. If context is lost, try moving toward **20%** and evaluate. These are heuristics, not universal rules.

## 10.4 What the lecture observed

- One sample chunk happened to contain one sentence (~74 characters); another contained two sentences (~142 characters, close to a 150-character target).
- The default splitter did not explicitly recognize sentence punctuation.
- Words were not cut in that sample.
- Chunk sizes varied even though `chunk_size=150`.

## 10.5 Advantages

- Good balance between chunk size and coherence.
- Tries paragraphs, lines, words, then characters.
- Recommended for many use cases.
- Supports custom separator lists for different formats.

## 10.6 Disadvantages

- Chunk sizes vary.
- Different chunk lengths can mean uneven embedding time and require context-window checks.
- A semantic unit longer than the target may still be split.
- Markdown and code may need format-specific separators.
- Highly structured documents may need a structure-aware approach.

🟢 **Best starting point:** Begin with `RecursiveCharacterTextSplitter`, inspect its output, and specialize only if necessary.

---

# 11. Strategy 3 — Document-structure-based splitting

## 11.1 The idea

This strategy uses the structure already present in a file. It is related to recursive splitting: the algorithm can still be recursive, but the **separator hierarchy changes** to match the format.

| Format | Structure used |
|---|---|
| HTML | Tags such as `<head>` and `<div>` |
| Markdown | Heading levels `#` through `######` |
| JSON | Nested objects and arrays |
| Source code | Classes, function definitions, conditions and loops |

The goal is to keep logical units together — for example, a Markdown H1 section divided by H2 subsections, or code split around classes and functions.

The PDF lists HTML tags such as `<head>` and `<div>` as possible structural boundaries. Its code demonstrations focus on Python source code, JSON, and Markdown. A structure-aware separator hierarchy is a preference rather than a guarantee: a very long function, section, or JSON value can still be split or remain oversized.

🟣 **Analogy:** Organize a book by chapters and sections rather than cutting it every thousand characters.

## 11.2 A. Splitting Python code

Use `RecursiveCharacterTextSplitter.from_language()` with the `Language` enum.

```python
from langchain_text_splitters import RecursiveCharacterTextSplitter, Language

python_code =

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

### What this code does

1. `from_language()` chooses separators intended for Python syntax.
2. The splitter prefers code boundaries (such as class/function-related separators) before falling back to less meaningful boundaries.
3. `chunk_size=120` sets a target size, measured in characters by this splitter configuration—not a guarantee that every function stays intact.
4. `chunk_overlap=20` repeats a small amount of text between neighboring chunks to preserve context.

**Important:** Language-aware splitting is not a full Python parser. A large function can still be divided, and syntax-sensitive applications may need AST-based parsing or custom logic.

## 11.3 B. Splitting JSON

JSON is hierarchical: an object can contain nested objects, arrays, and values. A structure-aware JSON splitter tries to keep that hierarchy meaningful instead of slicing the serialized JSON at arbitrary character positions.

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

### Understand the JSON-specific options

- `max_chunk_size`: a target/limit used by the splitter when deciding how to divide the JSON structure. Check the installed package version for exact behavior.
- `convert_lists`: in versions that expose this option, it controls whether lists are transformed into a representation that can be split more conveniently.
- `min_chunk_size`: available in some implementations/versions to discourage extremely small chunks. Confirm the constructor signature in your installed version.

Do not assume every option exists in every LangChain release. If an argument raises `TypeError`, inspect the installed version's documentation or run `help(RecursiveJsonSplitter)`.

### JSON caution

If a chunk is converted to text for embedding, preserve identifying context. A fragment such as `"employees": 45` is less useful than one that also carries its path, for example `departments[1].employees = 45`. For production RAG, consider storing JSON paths and source IDs as metadata.

## 11.4 C. Splitting Markdown by headings

Markdown headings provide a natural hierarchy. `MarkdownHeaderTextSplitter` can attach heading names to each resulting document's metadata, helping retrieval preserve section context.

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

### Why header metadata helps

A chunk from the Windows section can carry metadata such as:

```python
{"Header 1": "Installation", "Header 2": "Windows"}
```

This helps answer “How do I install it on Windows?” because the chunk retains the path through the document's structure.

**One important distinction:** `MarkdownHeaderTextSplitter` groups text by headings; it does not necessarily enforce a small maximum chunk length. A long section may need a second pass with `RecursiveCharacterTextSplitter`.

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

In a two-stage pipeline, test whether heading metadata is preserved by your installed version and pipeline. Inspect the resulting `Document` objects rather than assuming metadata behavior.

## 11.5 Advantages and disadvantages of structure-based splitting

**Advantages**
- Preserves headings, code blocks, functions, or nested data better than arbitrary cuts.
- Can make chunks more interpretable to both the embedding model and the LLM.
- Often improves traceability because heading names or data paths can be saved as metadata.
- Useful for technical documentation, source code, configuration files, and structured reports.

**Disadvantages**
- Requires knowing the file format and selecting the appropriate splitter.
- A section or function may be too long and still need a second splitting pass.
- Incorrect or inconsistent source formatting can reduce the benefit.
- Structure-aware splitting does not automatically guarantee semantic quality or valid executable code.

🟢 **Use it when:** the source has meaningful structure that answers should preserve—especially Markdown documentation, code repositories, and JSON records.

---

# 12. Strategy 4 — Semantic similarity-based chunking

## 12.1 The idea: split when the subject changes

Fixed-size splitting asks, “Have I reached the size limit?” Semantic chunking asks, “Has the meaning or topic changed enough that this is a sensible place to start a new chunk?”

For example:

> A retriever searches a vector database for relevant passages.  
> The embedding model converts text into vectors.  
> **The weather in Atlanta is warm in summer.**  
> A reranker reorders retrieved passages by relevance.

The first, second, and fourth sentences concern a RAG retrieval pipeline. The weather sentence is a topic change and is a candidate boundary.

A semantic chunker typically:
1. Breaks a document into small units, often sentences.
2. Embeds those units.
3. Compares embeddings for neighboring units or groups.
4. Finds large changes in similarity/distance.
5. Creates chunks around those changes, subject to its configuration.

The exact algorithm varies by library. Some implementations compare neighboring sentence groups and use a threshold or percentile to decide where a boundary belongs.

## 12.2 Why embeddings are used

An embedding maps text to a numerical vector. Similar meanings often have vectors that are closer under a chosen distance or similarity measure.

- Similar neighboring sentences → likely keep together.
- A sharp semantic shift → possible boundary.
- Similarity is a signal, not a perfect understanding of the text.

Two sentences can be related but use different vocabulary; two sentences can also use similar words while discussing different things. Semantic chunking therefore needs evaluation on your actual documents and questions.

## 12.3 Conceptual example

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

This illustrates the idea, not a ready-to-run implementation: functions such as `split_into_sentences`, `cosine_similarity`, and `group_sentences_using_boundaries` must be implemented or supplied by a library.

## 12.4 Example with LangChain Experimental

LangChain has provided semantic chunking through the experimental package in certain versions. APIs can change, so install and verify a compatible version before relying on this example.

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

If imports or parameters differ in your environment, check the installed package documentation. Semantic chunking is not part of every minimal LangChain installation, and the code may require API credentials for the chosen embedding provider.

## 12.5 Thresholds and trade-offs

- **More sensitive boundary detection:** tends to create more, smaller chunks.
- **Less sensitive boundary detection:** tends to create fewer, larger chunks.
- **Sentence grouping size:** comparing groups of sentences can reduce noisy decisions but may hide short topic changes.
- **Embedding model:** affects the similarity signal and therefore the boundaries.

There is no universally correct threshold. Evaluate alternatives using representative queries and a small labeled set of “which passage should answer this question?” examples.

## 12.6 Advantages and disadvantages

**Advantages**
- Can keep a coherent idea together even when sentence lengths vary.
- Can place boundaries where the topic changes instead of at an arbitrary character count.
- Useful for long prose, articles, and reports with uneven paragraph structure.

**Disadvantages**
- Usually more computationally expensive than simple character splitting because embeddings must be calculated.
- Boundary quality depends on the embedding model, sentence segmentation, and threshold.
- Can produce unpredictable chunk lengths; a later size limit may still be necessary.
- A semantic boundary may separate a statement from a qualification, definition, or example that is needed to understand it.

🟣 **Memory trick:** Recursive splitting respects *format boundaries*; semantic chunking tries to respect *meaning boundaries*.

---

# 13. Strategy 5 — LLM-based chunking

## 13.1 The idea: ask a language model to identify coherent sections

Instead of relying only on character counts, separators, or embedding similarity, LLM-based chunking asks a language model to identify logical units in the document.

A prompt might ask the model to:
- Preserve headings and their related paragraphs.
- Keep a question with its answer.
- Keep a claim with its supporting explanation.
- Avoid splitting code blocks or tables.
- Return chunks in the original order without omitting content.

The model may return boundaries, grouped passages, or structured JSON. Your pipeline should validate the result before storing it.

## 13.2 Simple prompt example

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

This is a prompt template, not a complete production pipeline. The application must insert the document safely, call a model, parse the output, and validate it.

## 13.3 Safer implementation pattern

Rather than trusting a model to rewrite the source, a safer design is to ask it to identify boundaries or source spans, then reconstruct the chunks from the original text. This reduces accidental paraphrasing or omissions.

A robust workflow is:

1. Divide a very large document into manageable sections.
2. Ask the LLM to suggest boundaries or source-span IDs.
3. Validate that all spans are present, ordered, and non-overlapping unless overlap is intentional.
4. Check chunk length and required metadata.
5. Compare the combined chunks against the source to detect missing content.
6. Store the validated chunks and their source references.

If the LLM returns free-form rewritten text, exact source coverage becomes harder to verify.

## 13.4 Advantages and disadvantages

**Advantages**
- Can recognize complex relationships that simple rules miss.
- Can keep explanations, examples, definitions, and headings together.
- Can be adapted to specialized documents, such as policies or technical manuals.

**Disadvantages**
- Adds model-call latency and cost.
- May omit, duplicate, reorder, or rewrite source content.
- Results can vary between runs or models.
- Requires output parsing, error handling, and quality checks.
- Sending private documents to an external model may raise privacy, security, or compliance concerns.

🟠 **Production rule:** Never assume LLM-produced chunks are complete just because the response looks well formatted. Validate coverage, order, size, metadata, and source traceability.

## 13.5 When LLM-based chunking is worth considering

Consider it when documents are complex and coherence matters more than ingestion speed or cost—for example, long policy documents, complex manuals, or documents with relationships that are hard to express as separators.

For ordinary PDFs, web pages, and text files, first establish a strong baseline with recursive splitting and appropriate document structure. Only add LLM-based chunking if evaluation shows a real improvement.

---

# 14. Choosing a chunking strategy

## 14.1 Decision table

| Your source or requirement | Good starting strategy | Why | Watch out for |
|---|---|---|---|
| Plain text, simple notes | `CharacterTextSplitter` or recursive splitter | Easy baseline | Arbitrary boundaries can break ideas |
| General PDFs, articles, mixed prose | `RecursiveCharacterTextSplitter` | Tries increasingly smaller separators | Does not fully understand meaning |
| Markdown manuals | `MarkdownHeaderTextSplitter`, often followed by recursive splitting | Preserves heading context | Long sections may remain too large |
| Python or other source code | Language-aware recursive splitter | Prefers code-specific boundaries | Not a full syntax parser |
| Nested JSON | JSON-aware splitter | Uses object/array structure | Preserve field paths and IDs |
| Prose with frequent topic changes | Semantic chunking | Uses embedding similarity to find likely topic shifts | Extra embedding cost and variable sizes |
| Complex documents where coherence is critical | LLM-based chunking | Can reason about relationships and section meaning | Higher cost and must validate output |
| Tables | Extract rows and headers deliberately; chunk by rows/groups | Keeps values associated with column names | Generic text splitting can detach values from headers |
| Very large documents | Hierarchical/section-first splitting, then size control | Controls size while preserving structure | Needs reliable section detection |

## 14.2 A practical decision flow

1. **Is the source structured?**
   - Yes: start with a format-aware splitter (Markdown, code, JSON) or table-specific extraction.
   - No: start with recursive splitting.
2. **Are chunks breaking important ideas?**
   - Inspect actual examples before changing anything.
   - Try better separators, heading-aware splitting, or a different size/overlap.
3. **Are topic boundaries still poor?**
   - Test semantic chunking on representative documents.
4. **Does the document have complex relationships that rules and semantic boundaries cannot preserve?**
   - Consider LLM-based boundary detection, with strict validation.
5. **Does evaluation improve?**
   - Keep the more complex strategy only if measured retrieval or answer quality justifies the added cost.

## 14.3 Do not choose by fashion

A sophisticated strategy is not automatically better. The right strategy depends on:
- Source format and document quality.
- Types of user questions.
- Embedding model and retrieval method.
- Chunk-size limits and the LLM context window.
- Ingestion volume, latency, and cost.
- Need for citations, metadata, and source traceability.

A simple baseline with carefully chosen chunk size and overlap may outperform a complex strategy that has not been tuned for the dataset.

## 14.4 How to evaluate chunking in a RAG system

Create a small evaluation set of real questions and their expected source passages. Compare candidate strategies using the same documents, embedding model, retrieval settings, and queries.

Useful checks include:
- **Retrieval recall:** does the correct evidence appear in the retrieved top-k chunks?
- **Precision of retrieved context:** how much retrieved material is relevant?
- **Answer correctness and faithfulness:** does the final answer match the source evidence?
- **Citation quality:** can the answer be traced to the correct document and section?
- **Chunk statistics:** number of chunks, average/maximum size, overlap, and unusually small fragments.
- **Operational cost:** ingestion time, embedding calls, LLM calls, and storage.

Change one major variable at a time. If chunking, embedding model, top-k, and reranking are all changed together, it becomes difficult to identify what helped.

---

# 15. Golden rules for chunking

These are practical rules of thumb. Treat the numbers as starting points, not universal laws.

## Rule 1 — Start with a baseline

Use a recursive splitter for general text, then inspect its output. Do not begin with the most expensive method unless the document format clearly requires it.

## Rule 2 — Preserve meaning, not just length

Prefer paragraph, heading, sentence, function, or row boundaries when they represent complete ideas. A chunk with a perfect character count but a broken definition may be a poor retrieval unit.

## Rule 3 — Tune chunk size for the task

- Smaller chunks can make retrieval more specific.
- Larger chunks can preserve broader context.
- Very small chunks can lose definitions and surrounding explanation.
- Very large chunks can contain irrelevant text and reduce retrieval precision.

Test multiple sizes on representative queries rather than relying on one universal value.

## Rule 4 — Use overlap deliberately

Overlap helps when a key definition or sentence crosses a boundary. Too much overlap duplicates content, increases embedding and storage costs, and can fill top-k results with near-duplicate chunks. Start with modest overlap and measure whether it helps.

## Rule 5 — Preserve metadata

At minimum, consider:
- `source` or file identifier
- page number or section
- heading/path
- chunk index
- stable document ID and version

Metadata helps with citations, filtering, debugging, and re-ingestion. Keep it attached to each chunk.

## Rule 6 — Treat tables as tables

Preserve column headers with rows. For long tables, chunk by row groups and repeat the header in each chunk when needed. If a PDF table is extracted into scrambled text, fix table extraction before applying a generic splitter.

## Rule 7 — Treat code as code

Use language-aware splitting when useful. Preserve class/function names and file paths as metadata. If valid syntax is essential, consider syntax-aware parsing and test the chunks.

## Rule 8 — Check the real token count

Character count is not token count. If the target model has a token limit, use an appropriate tokenizer or verify the resulting chunk token counts before sending them to the model.

## Rule 9 — Inspect edge cases

Manually inspect:
- The first and last chunks of each document.
- Chunks near headings and page breaks.
- Long tables, lists, and code blocks.
- Very short or unusually long chunks.
- Chunks that lack context or duplicate a neighbor.

## Rule 10 — Keep ingestion reproducible

Record the splitter type, parameters, embedding model, and document version. If you change the chunking algorithm, reprocess affected documents consistently; avoid mixing incompatible chunking schemes without tracking versions.

## Rule 11 — Avoid losing source traceability

Each chunk should point back to its original document and, where possible, page, heading, row, or character span. This is essential for citations and debugging hallucinations.

## Rule 12 — Measure the end result

Chunking is part of the retrieval pipeline, not an isolated formatting exercise. Evaluate whether the right evidence is retrieved and whether the final answer is correct and grounded.

🟣 **Memory sentence:** *Preserve structure, keep context, control size, track the source, and test retrieval.*

---

# 16. Common mistakes and quick fixes

| Mistake | Why it hurts | Quick fix |
|---|---|---|
| Choosing chunk size only by habit | Different datasets and questions need different context | Compare a few sizes with the same evaluation questions |
| Setting overlap extremely high | Creates many near-duplicate vectors | Start modestly and measure retrieval impact |
| Splitting Markdown before considering headings | Chunks may lose section meaning | Split by headers first, then size-limit long sections |
| Treating every PDF as clean prose | Tables and multi-column layouts may be garbled | Use layout-aware extraction and table-specific handling |
| Using semantic chunking without evaluation | More cost does not guarantee better retrieval | Compare it with a recursive baseline |
| Trusting LLM output without checking coverage | Facts or sections may disappear | Validate source spans, ordering, and completeness |
| Ignoring metadata | Sources and citations become difficult to recover | Preserve document IDs, page/section, and chunk index |
| Assuming chunk size is a hard guarantee | Some splitters treat it as a target or behave differently by version | Inspect actual chunk lengths and token counts |
| Changing many pipeline settings together | You cannot tell what caused the improvement | Run controlled comparisons |

---

# 17. Interview questions and answers — Sections 3.10–3.15

### Q1. What is recursive text splitting?
It splits text using a hierarchy of separators. It tries to preserve larger units first, such as paragraphs, then falls back to smaller units such as lines, spaces, or characters when needed to approach the target size.

### Q2. Why is recursive splitting a common default?
It is a useful balance between simplicity and preserving natural boundaries. It works reasonably well for many general-text documents without requiring a separate semantic model call.

### Q3. What is the difference between recursive and document-structure-based splitting?
Recursive splitting usually follows a generic separator hierarchy. Document-structure-based splitting chooses boundaries for a particular format, such as Python syntax, JSON structure, or Markdown headings. A format-aware splitter can itself use recursive logic.

### Q4. How does Markdown header splitting help RAG?
It groups text around headings and can store heading paths in metadata. That gives retrieved passages context about which section they came from. Very long sections may still need another size-based splitting pass.

### Q5. How does JSON splitting differ from plain-text splitting?
JSON splitting attempts to respect nested objects and arrays instead of cutting serialized text at arbitrary character positions. Metadata or paths should still be preserved so a value remains understandable outside the full JSON document.

### Q6. What is semantic chunking?
It uses embeddings and similarity changes between neighboring text units to estimate where the topic changes. It may create more coherent chunks than fixed-size splitting, but it costs more and needs tuning.

### Q7. Does semantic chunking guarantee better retrieval?
No. Embedding quality, thresholds, document style, and query types all affect results. Evaluate it against a simpler baseline.

### Q8. What is LLM-based chunking?
It uses a language model to identify or create coherent sections based on instructions. It can handle complex relationships, but it may omit or rewrite content and adds latency and cost.

### Q9. How can you reduce omissions in LLM-based chunking?
Prefer boundary detection over free-form rewriting, preserve source spans, validate ordering and coverage, check chunk sizes, and compare reconstructed content with the original source.

### Q10. Which chunking strategy is best for all RAG systems?
There is no single best strategy. Start with a sensible baseline, use source-aware splitting for structured files, and compare alternatives using representative queries and retrieval metrics.

### Q11. What is the role of chunk overlap?
Overlap repeats a small portion of neighboring text to preserve context when a relevant idea crosses a boundary. Excessive overlap increases duplication and cost.

### Q12. How would you select a chunking strategy for a production RAG application?
Inspect source formats and query types, create a baseline, compare chunk sizes and overlap, evaluate retrieval recall and answer faithfulness, inspect failure cases, and account for ingestion cost and latency.

---

# 18. Quick revision sheet

| Strategy | Main signal for a boundary | Good fit | Main risk |
|---|---|---|---|
| Length-based | Character or token target | Simple text and baseline experiments | May cut through meaning |
| Recursive | Hierarchy of separators | General text | Not true semantic understanding |
| Document-structure-based | Headings, code syntax, JSON hierarchy, other format boundaries | Manuals, code, structured data | Needs correct format handling |
| Semantic similarity-based | Embedding similarity/distance change | Topic-changing prose | Extra embedding cost and unstable chunk sizes |
| LLM-based | Model's interpretation of coherent units | Complex documents | Cost, variability, and possible omissions |

## Five things to remember

1. **Recursive:** try natural separators from larger to smaller.
2. **Structure-based:** use the source's format, such as headings or code boundaries.
3. **Semantic:** use meaning similarity to estimate topic changes.
4. **LLM-based:** use model reasoning, but validate completeness.
5. **Choosing:** judge chunking by retrieval and answer quality—not by how sophisticated the algorithm sounds.

---

## Version and API note

LangChain APIs and package boundaries change over time. Install the packages appropriate to your environment, inspect constructor signatures when an example fails, and pin tested versions in production. The snippets in these notes illustrate the strategies; run them against your installed package versions before using them in a pipeline.

## Final coverage note

This revision fills out the previously truncated material for document-structure-based splitting, semantic chunking, LLM-based chunking, strategy selection, and golden rules. It is a learning guide with explanatory examples; it should not be treated as a verbatim reproduction of every diagram, sentence, or code cell in the source lecture.
