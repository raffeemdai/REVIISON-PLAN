# <span style="color: #374151;">Advanced RAG --- Chapters 1 & 2 Complete Study Notes</span>

## <span style="color: #2457A7;">Based on `Campusx_Advanced_RAG_Complete_2026.pdf`</span>

> **Purpose:** A beginner-friendly, revision-ready companion to Chapters
> 1 and 2. It covers the chapter sections, concepts, examples, key
> distinctions, practical syntax, interview questions, revision prompts,
> and memory aids. Explanations are rewritten in simple language; code
> is illustrative and may need package/version adjustments in your
> environment.

------------------------------------------------------------------------

## <span style="color: #2B6CB0;">How to use these notes</span>

1.  Read the story first to understand *why* a concept exists.
2.  Study the theory and tables.
3.  Run the Python examples with a small sample file.
4.  Answer the interview questions without looking.
5.  Use the last-minute revision sheet before an interview.

### <span style="color: #4C6B9A;">The big picture</span>

Imagine an organization has a very capable new employee (the LLM). The
employee speaks fluently and has broad knowledge, but their training may
be old, they may confidently make mistakes, and they cannot
automatically read your private company files. RAG gives that employee a
**searchable library**, finds the few pages relevant to the current
question, places those pages beside the question, and asks the employee
to answer using them.

------------------------------------------------------------------------

# <span style="color: #0B7285;">Chapter 1 --- Introduction to RAG</span>

## <span style="color: #2B6CB0;">Chapter map</span>

This chapter builds the idea from first principles:

1.  How LLMs work
2.  Four problems users face with plain LLMs
3.  Adding external knowledge
4.  In-context learning and its limits
5.  Why RAG was born
6.  Retrieval, augmentation, generation
7.  The complete RAG pipeline and its components
8.  Which problems RAG helps solve---and where it still fails
9.  Golden rules

------------------------------------------------------------------------

## <span style="color: #2457A7;">1.1 How we will approach RAG</span>

The learning path is **Why → What → How**.

-   **Why:** What goes wrong when we use a plain LLM?
-   **What:** What is RAG, and what does the name mean?
-   **How:** What components implement a RAG pipeline, and how does data
    flow between them?

The important learning habit is to derive the solution from the problem
rather than memorize a definition.

<span style="color: #0F766E;"><strong>Story:</strong></span> Before installing a library in a kitchen, first identify what
problem the kitchen has. Likewise, before learning vector databases and
retrievers, understand why an LLM needs outside information.

------------------------------------------------------------------------

## <span style="color: #2457A7;">1.2 How LLMs work</span>

### <span style="color: #4C6B9A;">Why LLMs feel powerful</span>

Modern chat assistants can answer questions about programming, finance,
science, health, writing, and many other domains. Much of this
capability comes from:

-   **Transformer architecture:** a neural-network architecture used by
    many modern LLMs.
-   **Self-attention:** lets the model calculate relationships between
    tokens in the input, helping it use context.
-   **Large-scale training:** the model learns patterns from a very
    large training corpus.

Architecture alone is not enough; the training data and training process
are crucial.

### <span style="color: #4C6B9A;">What does "Large Language Model" mean?</span>

  -----------------------------------------------------------------------
  Term                                Simple meaning
  ----------------------------------- -----------------------------------
  Language                            Training on large amounts of text
                                      lets the model learn statistical
                                      patterns in language and generate
                                      text.

  Model                               A learned AI/deep-learning function
                                      with parameters.

  Large                               A large number of learned
                                      parameters---weights and
                                      biases---often billions or more.
  -----------------------------------------------------------------------

### <span style="color: #4C6B9A;">Pre-training</span>

**Pre-training** is the broad initial training stage in which a model
learns from a huge corpus: articles, books, websites, blogs, reviews,
code, and other text sources.

It is expensive because it can require: - Weeks or months of training
for large models. - Many GPUs and distributed compute. - Data cleaning
and preprocessing by skilled teams. - Storage for model weights,
optimizer state, gradients, checkpoints, and updates. - Substantial
electricity and infrastructure.

### <span style="color: #4C6B9A;">What is stored in the model parameters?</span>

The model does **not** store a neat, searchable database of facts. Its
parameters encode learned patterns and statistical relationships in
language. When prompted, it uses those learned patterns to predict a
useful continuation.

This is called **parametric knowledge**: knowledge represented
implicitly in model parameters.

### <span style="color: #4C6B9A;">Next-token generation</span>

At a high level, an autoregressive language model predicts a
distribution over the next token, selects a token according to its
decoding settings, appends it to the sequence, and repeats until it
stops.

Example: - Prompt: `The capital of France is` - The model assigns
probabilities to possible next tokens. - `Paris` may receive a high
probability. - The model emits a token and continues.

A model can generate a plausible continuation without checking whether
it is factually true. That distinction becomes important when we study
hallucinations.

### <span style="color: #4C6B9A;">Important points</span>

-   The LLM is the **generator**, not automatically a live search
    engine.
-   Training knowledge can become stale after the training data cutoff.
-   Model parameters are not equivalent to a conventional database.
-   A fluent answer is not proof of a correct answer.

<span style="color: #7C3AED;"><strong>Memory trick:</strong></span> **T-P-N** --- **T**ransformer processes context,
**P**re-training learns patterns, **N**ext-token prediction generates
text.

------------------------------------------------------------------------

## <span style="color: #2457A7;">1.3 Four problems users face with plain LLMs</span>

### <span style="color: #4C6B9A;">Problem 1: Knowledge cutoff / stale knowledge</span>

The model's training knowledge is tied to the data and updates used
during training. It may not know about events, product changes,
policies, or releases that happened later.

<span style="color: #2563EB;"><strong>Example:</strong></span> Ask a model trained before a new software release to
explain a feature introduced yesterday. Without a current source, it may
not know the change.

### <span style="color: #4C6B9A;">Problem 2: Hallucination</span>

A model can produce confident-sounding but incorrect, invented, or
unsupported content. It is optimized to generate likely text, not to
guarantee that every statement is true.

<span style="color: #2563EB;"><strong>Example:</strong></span> It invents a nonexistent research paper or gives an API
parameter that does not exist.

### <span style="color: #4C6B9A;">Problem 3: No source attribution by default</span>

A plain answer may not show which document, page, or evidence supports
each claim. A user cannot easily verify the answer.

### <span style="color: #4C6B9A;">Problem 4: No access to private or organization-specific data by default</span>

The model generally does not automatically know your private company
policies, internal reports, local files, or recently uploaded documents.
Those sources must be made available to the application.

### <span style="color: #4C6B9A;">Summary table</span>

  -----------------------------------------------------------------------
  Problem                 Why it occurs           What an application may
                                                  need
  ----------------------- ----------------------- -----------------------
  Stale knowledge         Training data is not    Current external
                          continuously updated    sources

  Hallucination           Plausible generation is Relevant evidence,
                          not guaranteed truth    checks, and careful
                                                  prompting

  Missing citations       Plain generation may    Source metadata and
                          not track evidence      citation logic
                          sources                 

  Private-data gap        The model has no        A controlled way to
                          automatic access to     retrieve authorized
                          private files           data
  -----------------------------------------------------------------------

<span style="color: #B91C1C;"><strong>Interview distinction:</strong></span> RAG can reduce these problems, but it does
**not** guarantee that hallucinations disappear, every source is cited,
or all retrieved information is correct.

------------------------------------------------------------------------

## <span style="color: #2457A7;">1.4 Finding the solution: add external knowledge</span>

If the model lacks a fact, one option is to supply that fact in the
prompt at answer time. This is external knowledge.

Example: - Question: "What is our company's parental-leave policy?" -
External context: the relevant approved policy excerpt. - Prompt: ask
the model to answer the question using the supplied excerpt.

The model can now use information that was not part of its original
training.

This leads to **in-context learning**: the model uses information and
examples supplied in its input to produce the response, without changing
its trained parameters.

------------------------------------------------------------------------

## <span style="color: #2457A7;">1.5 Ways to provide additional knowledge</span>

### <span style="color: #4C6B9A;">Option A: In-context learning (provide context in the prompt)</span>

Add the relevant text to the prompt together with the question.

**Advantages** - Simple to understand and implement. - No model
retraining required. - Updating the supplied knowledge can be as easy as
changing the input.

**Limitations** - The application must identify what text to supply. -
Large context can exceed model limits. - Too much irrelevant context can
reduce the usefulness of the answer.

### <span style="color: #4C6B9A;">Option B: Fine-tuning</span>

Fine-tuning continues training a pre-trained model on a targeted
dataset. Parameter-efficient methods such as **PEFT**, **LoRA**, and
**QLoRA** can update a smaller portion of the effective model parameters
rather than retraining everything.

Fine-tuning is often useful for changing behavior, style, format, task
performance, or domain-specific response patterns. It is not usually the
simplest way to keep a large collection of frequently changing facts
current.

**Costs and trade-offs** - Requires expertise and compute; often GPUs. -
Is more involved than simply adding context. - New facts may require
additional data and training rather than a simple document update.

### <span style="color: #4C6B9A;">Decision</span>

For frequently changing factual knowledge, supplying relevant external
context is often the simpler starting point. Fine-tuning and RAG solve
different problems and can sometimes be combined.

<span style="color: #7C3AED;"><strong>Memory trick:</strong></span> **Context changes the input; fine-tuning changes the
model's learned behavior.**

------------------------------------------------------------------------

## <span style="color: #2457A7;">1.6 Limits of in-context learning</span>

Imagine asking about a 1,000-page company handbook. Simply pasting the
whole handbook into every prompt is not a good general solution.

### <span style="color: #4C6B9A;">Limit 1: Context window</span>

A model can process only a limited amount of input at one time. This
maximum is called its **context window** and is measured in tokens.
Exact sizes depend on the model; some support hundreds of thousands of
tokens and some support around a million, but none should be treated as
unlimited.

If the prompt plus documents exceeds the limit, the full input cannot be
processed as intended.

### <span style="color: #4C6B9A;">Limit 2: "Lost in the middle"</span>

Even when a long prompt fits, the model may use information near the
beginning or end more effectively than information buried in the middle.
Important content can be overlooked when the context is excessively
long.

### <span style="color: #4C6B9A;">The book-versus-chapter analogy</span>

If you have a test on one chapter, studying that chapter is usually more
effective than being handed an entire book and being told to memorize
every page. Similarly, a model often works better when the prompt
contains the relevant evidence rather than a huge pile of unrelated
text.

### <span style="color: #4C6B9A;">Resulting design idea</span>

1.  Store the complete knowledge collection outside the prompt.
2.  For each user question, find the relevant pieces.
3.  Send only those pieces to the LLM.
4.  Generate the answer using that focused context.

This is the key motivation behind RAG.

------------------------------------------------------------------------

## <span style="color: #2457A7;">1.7 Birth of RAG</span>

The core idea can be summarized as:

**In-context learning + filtered, query-specific external knowledge =
Retrieval-Augmented Generation.**

-   **Retrieval:** find relevant information.
-   **Augmentation:** add that information to the prompt.
-   **Generation:** use the LLM to write the response.

RAG does not normally require changing the LLM's parameters whenever the
knowledge base changes. Instead, the application updates the external
knowledge collection and retrieves current, relevant pieces at query
time.

------------------------------------------------------------------------

## <span style="color: #2457A7;">1.8 The three jobs of RAG</span>

  -----------------------------------------------------------------------
  Name                    Job                     Typical component
  ----------------------- ----------------------- -----------------------
  Retrieval               Find relevant knowledge Retriever / retrieval
                          for the current         pipeline
                          question                

  Augmentation            Combine the question    Context assembly / RAG
                          and retrieved evidence  chain
                          into a useful prompt    

  Generation              Produce the final       LLM
                          answer from the         
                          augmented prompt        
  -----------------------------------------------------------------------

### <span style="color: #4C6B9A;">Mini-example</span>

Question: "How many days of annual leave do employees receive?"

1.  **Retrieve:** Search approved HR documents for the annual-leave
    policy.
2.  **Augment:** Put the relevant policy passage and the user's question
    into a prompt.
3.  **Generate:** Ask the LLM to answer using the passage.

<span style="color: #7C3AED;"><strong>Memory trick:</strong></span> **R-A-G = Find → Add → Answer.**

------------------------------------------------------------------------

## <span style="color: #2457A7;">1.9 Retrieval</span>

Retrieval finds the information that best matches the user's current
question in an external knowledge base.

A typical system may: - Store document chunks and their metadata. -
Represent text as vectors for semantic comparison. - Convert the user's
question into a compatible query vector. - Rank candidate chunks and
return the most relevant ones.

Retrieval is not the final answer. Its output is evidence---usually
document objects containing text and metadata.

The quality of the final answer depends heavily on the quality of the
retrieved evidence.

------------------------------------------------------------------------

## <span style="color: #2457A7;">1.10 Augmentation</span>

Augmentation means inserting retrieved information into the LLM input in
a controlled format.

A simple conceptual prompt is:

``` text
You are a helpful assistant. Answer using the context below.
If the context does not contain the answer, say that you do not know.

Context:
{retrieved_context}

Question:
{user_question}

Answer:
```

The exact prompt depends on the application. A good prompt can tell the
model: - Use the supplied evidence. - Do not invent missing details. -
State when the evidence is insufficient. - Include citations or source
references when available.

Augmentation is more than blindly concatenating text: a practical system
may format sources, remove duplicates, respect token budgets, and
preserve metadata for citations.

------------------------------------------------------------------------

## <span style="color: #2457A7;">1.11 Why filtered context makes RAG powerful</span>

Filtered context helps in several ways: - It keeps input within the
context window. - It reduces irrelevant information. - It can make the
evidence easier for the LLM to use. - It can reduce unnecessary prompt
tokens and cost. - It supports traceability when source metadata is
preserved.

**Caution:** "Relevant" is not always the same as "correct." Retrieval
can return misleading, stale, incomplete, or malicious content. RAG
needs evaluation and security controls as well as retrieval.

------------------------------------------------------------------------

## <span style="color: #2457A7;">1.12 The RAG pipeline: components and data flow</span>

The chapter presents the RAG pipeline as a set of connected components.
The first part prepares the knowledge base; the second part answers each
query.

### <span style="color: #4C6B9A;">Indexing / knowledge preparation (usually done before a question arrives)</span>

1.  **Knowledge source:** raw PDFs, text files, web pages, databases, or
    other authorized material.
2.  **Document loaders:** parse different source formats into a common
    document representation.
3.  **Chunking:** split long documents into manageable pieces.
4.  **Embedding:** turn chunks into numeric vectors.
5.  **Vector store / knowledge base:** store vectors, text, IDs, and
    metadata so relevant chunks can be searched.

### <span style="color: #4C6B9A;">Query-time path</span>

6.  **Query embedding:** convert the user's question into a vector using
    the compatible embedding model.
7.  **Similarity search / retrieval:** compare the query vector with
    stored vectors and select candidates.
8.  **Augmentation / context assembly:** format retrieved chunks
    alongside the original question.
9.  **Generation:** the LLM generates a response from the augmented
    prompt.

### <span style="color: #4C6B9A;">Diagram</span>

``` text
                  INDEXING / PREPARATION
Raw sources → Loaders → Chunks → Embeddings → Vector store
                                                ▲
                                                │ similarity search
User question → Query embedding ────────────────┘
                         │
                         ▼
             Retrieved text + metadata
                         │
                         ▼
             Context assembly / prompt
                         │
                         ▼
                    LLM generation
                         │
                         ▼
                  Answer (+ sources)
```

### <span style="color: #4C6B9A;">A critical distinction</span>

The **document-ingestion path** can run when data is added or updated.
The **query path** runs for each user question. Keeping these paths
separate helps design incremental updates, caching, and monitoring.

------------------------------------------------------------------------

## <span style="color: #2457A7;">1.13 Component 1: knowledge source and document loaders</span>

A knowledge source is the raw collection from which the application
obtains information. Examples include local files, shared drives, cloud
storage, web pages, and authorized enterprise repositories.

Document loaders read and parse each format. Their purpose is to turn
heterogeneous inputs into a common representation that later components
can process.

Examples: - Text loader for plain text. - CSV loader for tabular rows. -
PDF loader for PDF pages. - JSON loader for structured data. - Web
loader for HTML pages.

Loaders are covered in detail in Chapter 2.

------------------------------------------------------------------------

## <span style="color: #2457A7;">1.14 Component 2: chunking</span>

Large documents are divided into smaller units called **chunks**.

Why chunk? - Whole documents may be too large for retrieval or
prompting. - A small chunk can focus on one subject. - Search can return
only the relevant section rather than an entire file.

A chunk should be large enough to preserve meaning but small enough to
retrieve precisely.

Important parameters: - **Chunk size:** the target maximum size for a
chunk, measured by characters, tokens, or another splitter-specific
unit. - **Chunk overlap:** the amount of text shared between neighboring
chunks. Overlap helps preserve context at boundaries. - **Splitting
strategy:** character, separator-aware, sentence-aware, structure-aware,
or other techniques.

Trade-offs: - Chunks too small can lose context. - Chunks too large can
contain irrelevant information and reduce retrieval precision. - Too
much overlap increases storage and duplicated retrieval results.

<span style="color: #2563EB;"><strong>Example:</strong></span> A 10-page document split into approximately three chunks
per page might create around 30 chunk objects. The actual count depends
on text length and the splitting rules.

------------------------------------------------------------------------

## <span style="color: #2457A7;">1.15 Component 3: embedding</span>

An **embedding model** converts text into a numeric vector intended to
represent useful semantic features.

Example (illustrative only):

``` text
"How do I reset my password?" → [0.12, -0.08, 0.44, ...]
```

The vector is not a human-readable summary. It is a numerical
representation that can be compared with other vectors.

For semantic retrieval: - Embed document chunks during indexing. - Embed
the query at query time using the compatible model. - Compare the query
vector with document vectors.

Use a consistent embedding model and configuration for the indexed
documents and queries unless the system explicitly supports compatible
alternatives.

------------------------------------------------------------------------

## <span style="color: #2457A7;">1.16 Component 4: vector store (knowledge base)</span>

A vector store persists vectors and provides indexing/search operations.
A practical record often contains: - An identifier. - An embedding
vector. - Original text or a reference to it. - Metadata such as source,
page, title, date, or permissions.

The vector store is not the LLM and is not itself the complete RAG
pipeline. It is one component of the knowledge and retrieval layer.

Metadata matters for: - Identifying the source of a passage. - Filtering
by department, date, document type, or access rights. - Building
citations. - Removing or updating records when source documents change.

------------------------------------------------------------------------

## <span style="color: #2457A7;">1.17 Component 5: query embedding and similarity search</span>

At query time, the question is embedded into a vector. The vector store
compares it with document vectors using a supported similarity or
distance metric.

Common metrics: - **Cosine similarity:** compares vector direction;
larger values often indicate greater similarity when used as a
similarity score. - **Euclidean distance:** straight-line distance;
smaller distance indicates closer vectors. - **Dot product:** combines
vector direction and magnitude; interpretation depends on the embedding
model and normalization.

Do not compare raw scores across different metrics or systems without
checking their definitions. A higher cosine similarity and a lower
Euclidean distance can both mean "closer," but the score direction
differs.

------------------------------------------------------------------------

## <span style="color: #2457A7;">1.18 Component 6: retrieval</span>

The retrieval step chooses a set of candidate documents, often the top
*k* results.

-   **k** is the number of results requested.
-   Too small a k may omit needed evidence.
-   Too large a k can add noise, consume tokens, and make the answer
    less focused.

Retrieval may use semantic search, keyword search, filters, thresholds,
or a combination. Basic similarity search is only one possible strategy;
later chapters study more advanced methods.

------------------------------------------------------------------------

## <span style="color: #2457A7;">1.19 Component 7: augmentation / context assembly</span>

The retrieved documents are formatted into context for the model.

A practical context assembly stage can: - Select useful chunks. -
Preserve source metadata. - Remove duplicate or redundant chunks. - Fit
the context into a token budget. - Separate individual sources
clearly. - Keep the original user question intact.

This is the point where retrieved evidence becomes part of the prompt.

------------------------------------------------------------------------

## <span style="color: #2457A7;">1.20 Component 8: generation</span>

The LLM receives the user question plus retrieved context and produces
the answer.

A well-designed system asks the LLM to ground its answer in the provided
context and acknowledge when the context is insufficient. The
application may also return source names, page numbers, URLs, or
quotations alongside the response.

Generation can still fail if: - The retrieved material is wrong or
incomplete. - The prompt is unclear. - The model misinterprets the
evidence. - The answer is not checked for factual grounding.

------------------------------------------------------------------------

## <span style="color: #2457A7;">1.21 Complete data flow</span>

### <span style="color: #4C6B9A;">Indexing time</span>

1.  Collect source documents.
2.  Load and parse them.
3.  Split them into chunks.
4.  Generate embeddings.
5.  Store vectors and associated text/metadata.

### <span style="color: #4C6B9A;">Query time</span>

1.  Receive the user's question.
2.  Embed the question.
3.  Search for relevant chunks.
4.  Rank and select the chunks.
5.  Assemble context with the question.
6.  Ask the LLM to generate an answer.
7.  Return the answer, ideally with source evidence.

### <span style="color: #4C6B9A;">Update time</span>

When source content changes, a production system needs an
ingestion/update policy: - Detect new or changed documents. - Reprocess
affected content. - Recreate chunks and embeddings as necessary. -
Upsert new records and remove obsolete records. - Track source versions
to avoid stale or duplicate content.

Incremental ingestion is a production design implication of the
pipeline; exact implementation depends on the source and vector store.

------------------------------------------------------------------------

## <span style="color: #2457A7;">1.22 Do the four problems get solved?</span>

  -----------------------------------------------------------------------
  Original problem        How RAG helps           Remaining limitation
  ----------------------- ----------------------- -----------------------
  Stale knowledge         Can retrieve newly      Only if the source is
                          added/current external  updated and retrieval
                          content                 finds it

  Hallucination           Supplies evidence and   The model can still
                          can instruct the model  misread, overstate, or
                          to ground answers       ignore evidence

  No attribution          Metadata can preserve   Citations must be
                          source/page/URL for     correctly connected to
                          citations               claims

  No private data access  Can retrieve authorized Requires access
                          private documents       controls, secure
                                                  ingestion, and
                                                  permission-aware
                                                  retrieval
  -----------------------------------------------------------------------

**Main lesson:** RAG is a way to provide the model with relevant
evidence, not a guarantee of truth.

------------------------------------------------------------------------

## <span style="color: #2457A7;">1.23 Where RAG fails</span>

Common failure points include:

1.  **Bad source data:** incorrect, outdated, duplicated, or incomplete
    documents enter the knowledge base.
2.  **Poor parsing:** a loader loses table structure, misses image text,
    or garbles a PDF.
3.  **Poor chunking:** key context is split apart or a chunk combines
    unrelated topics.
4.  **Weak embeddings or mismatch:** query and documents are not
    represented compatibly.
5.  **Bad retrieval:** the relevant passage is not returned or noisy
    passages dominate.
6.  **Poor context assembly:** useful evidence is truncated, duplicated,
    or mixed with irrelevant content.
7.  **Generation failure:** the LLM gives an unsupported answer despite
    having evidence.
8.  **Missing evaluation:** the team never measures retrieval relevance
    or answer faithfulness.
9.  **Security failures:** sensitive data or malicious instructions can
    be ingested or retrieved without proper controls.

RAG quality is end-to-end: every stage matters.

------------------------------------------------------------------------

## <span style="color: #2457A7;">1.24 Golden rules</span>

-   Understand the reason for RAG before memorizing its components.
-   Keep indexing and query-time work conceptually separate.
-   Retrieve relevant chunks, not every document.
-   Preserve metadata so evidence can be traced to its source.
-   Use compatible embeddings for query and document vectors.
-   Treat retrieval output as evidence to evaluate---not automatically
    as truth.
-   RAG reduces some risks; it does not eliminate hallucinations.
-   Test loaders and retrieval on the real document types in your
    project.
-   Protect private data with authorization-aware ingestion and
    retrieval.
-   Measure the system rather than relying on a few impressive demos.

### <span style="color: #4C6B9A;">Chapter 1 one-minute recap</span>

A plain LLM learns language patterns during training, but it may be
outdated, hallucinate, lack citations, and lack access to private data.
In-context learning lets us provide outside knowledge, but stuffing an
entire library into a prompt is limited by context size and
lost-in-the-middle effects. RAG solves this by retrieving relevant
external information, augmenting the prompt with it, and generating an
answer. Its pipeline includes sources, loaders, chunks, embeddings, a
vector store, query embedding, retrieval, context assembly, and
generation.

------------------------------------------------------------------------

# <span style="color: #0B7285;">Chapter 2 --- Document Loaders</span>

## <span style="color: #2B6CB0;">Chapter map</span>

This chapter explains how raw information becomes usable documents for a
RAG pipeline:

1.  Why retrieval is needed
2.  Why loaders are needed
3.  What a loader does
4.  The common `Document` object
5.  Loader choice by file type
6.  LangChain setup and the universal loading pattern
7.  Text, PDF, CSV, JSON, and web loaders
8.  PDF parsing, images, tables, and metadata
9.  Recursive crawling
10. `load()` versus `lazy_load()`
11. Golden rules

------------------------------------------------------------------------

## <span style="color: #2457A7;">2.1 RAG refresher: why we need retrieval</span>

The four problems from Chapter 1 are the motivation: - The model may not
know recent information. - It may hallucinate. - It may not show its
sources. - It may not know private or internal information.

A prompt can include external knowledge, but sending the entire library
is inefficient and may exceed the context window. Retrieval selects
relevant pieces for the question.

<span style="color: #0F766E;"><strong>Story:</strong></span> A library can contain millions of pages. A librarian does not
hand the visitor every book; the librarian finds the few useful pages.
Document loaders are the intake staff who read the different kinds of
books and prepare their content for the rest of the system.

------------------------------------------------------------------------

## <span style="color: #2457A7;">2.2 Why document loaders are needed</span>

### <span style="color: #4C6B9A;">The goal</span>

A RAG system may use documents in many formats: - Plain text and
Markdown. - PDF. - CSV. - JSON. - Web pages. - Other enterprise sources.

Each format stores information differently. The pipeline needs a
consistent representation so later components do not need a completely
different workflow for every format.

### <span style="color: #4C6B9A;">Knowledge source versus vector store</span>

  -----------------------------------------------------------------------
  Aspect                  Knowledge source        Vector store
  ----------------------- ----------------------- -----------------------
  Contains                Original files/content  Searchable vectors plus
                                                  associated records

  Role                    The source of truth /   Supports retrieval by
                          raw material            vector similarity and
                                                  other supported filters

  Analogy                 The original library or A searchable
                          file cabinet            catalog/index

  Example                 PDF policy on disk      Stored chunk embeddings
                                                  with page/source
                                                  metadata
  -----------------------------------------------------------------------

The vector store is not a replacement for managing original source
files. A robust ingestion process should know where a record came from
and how it relates to the source.

### <span style="color: #4C6B9A;">The format problem</span>

A PDF is not read in the same way as a CSV. A CSV has rows and columns;
JSON has nested keys and values; HTML has tags and page structure. Each
parser must understand the format well enough to extract useful content.

A **document loader** hides much of this format-specific work behind a
common interface. Downstream components can then work with the returned
documents rather than every original file format.

------------------------------------------------------------------------

## <span style="color: #2457A7;">2.3 What are document loaders?</span>

A document loader is a pipeline component that: 1. Connects to or opens
a source. 2. Reads its content. 3. Parses/extracts text or other
supported information. 4. Returns one or more standardized `Document`
objects, usually with metadata.

### <span style="color: #4C6B9A;">Why parsing matters</span>

  -----------------------------------------------------------------------
  Source format                       What parsing needs to understand
  ----------------------------------- -----------------------------------
  TXT                                 Plain text bytes and character
                                      encoding

  PDF                                 Page content, layout, and sometimes
                                      scanned images

  CSV                                 Rows, columns, headers, and values

  JSON                                Nested structure, keys, arrays, and
                                      the desired fields

  Web page                            HTML structure, page title, visible
                                      text, links, and metadata
  -----------------------------------------------------------------------

A loader may successfully open a file yet still extract poor content.
For example, a scanned PDF may contain page images rather than
selectable text. OCR or a multimodal model may be needed. Similarly,
extracting text from a table does not guarantee that the table's
row/column structure is preserved.

<span style="color: #0F766E;"><strong>Key idea:</strong></span> File loading and high-quality semantic extraction are not
the same thing.

------------------------------------------------------------------------

## <span style="color: #2457A7;">2.4 The unified output: the `Document` object</span>

LangChain loaders commonly return a list of `Document` objects. Each
object has two central fields:

  -----------------------------------------------------------------------
  Field                   Type                    Meaning
  ----------------------- ----------------------- -----------------------
  `page_content`          string                  The extracted
                                                  text/content for that
                                                  document unit

  `metadata`              dictionary              Information about its
                                                  origin and context
  -----------------------------------------------------------------------

Metadata may include: - `source`: file path or URL. - `page`: page index
or page number. - `row`: row index for tabular data. - `title`, author,
creation/modification dates, or other parser-specific values.

Example:

``` python
from langchain_core.documents import Document

doc = Document(
    page_content="Employees receive 15 days of annual leave.",
    metadata={"source": "hr_policy.pdf", "page": 4}
)

print(doc.page_content)
print(doc.metadata)
```

### <span style="color: #4C6B9A;">Why a unified format is useful</span>

A splitter can accept `Document` objects regardless of whether they came
from a PDF, text file, or web page. When it splits a document, metadata
is normally carried forward to the resulting chunks, which helps
preserve source traceability.

### <span style="color: #4C6B9A;">Important details</span>

-   A loader may return one `Document` for a whole file, one per page,
    one per row, or another logical unit.
-   The number of documents returned depends on the loader and its
    configuration.
-   Never assume all loaders return one object per file.
-   Inspect the first document's metadata before building filtering or
    citation logic.

------------------------------------------------------------------------

## <span style="color: #2457A7;">2.5 One loader per file type</span>

The typical approach is to choose a loader appropriate to the source:

  -----------------------------------------------------------------------
  Input                   Common LangChain loader Typical unit returned
  ----------------------- ----------------------- -----------------------
  Plain text              `TextLoader`            Often one document per
                                                  file

  PDF                     `PyPDFLoader`,          Often one document per
                          `PDFMinerLoader`,       page
                          `PDFPlumberLoader`,     
                          other specialized       
                          loaders                 

  CSV                     `CSVLoader`             Often one document per
                                                  row

  JSON                    `JSONLoader`            Depends on the
                                                  configured extraction
                                                  expression

  HTML/web                `WebBaseLoader`         Often one document per
                                                  URL

  Recursive website crawl `RecursiveUrlLoader`    One document per
                                                  fetched page, subject
                                                  to crawl rules
  -----------------------------------------------------------------------

These are typical behaviors, not universal guarantees. Confirm the
loader's current documentation and inspect its output.

------------------------------------------------------------------------

## <span style="color: #2457A7;">2.6 Summary: loader responsibility</span>

A loader is responsible for turning a source into documents. It is not
responsible for all the later RAG work.

``` text
Source file → Loader → Document objects → Text splitter → Chunks
            → Embedding model → Vectors → Vector store
```

The loader should not be confused with: - **Text splitter:** divides
loaded content into smaller chunks. - **Embedding model:** converts text
into vectors. - **Vector store:** persists and searches vector
records. - **Retriever:** selects documents for a query. - **LLM:**
generates the final answer.

------------------------------------------------------------------------

## <span style="color: #2457A7;">2.7 Hands-on setup: loading files in LangChain</span>

The course's practical pattern uses a project folder with a
knowledge-source directory. A simple example:

``` text
my-rag-project/
├── knowledge-source/
│   ├── sample.txt
│   ├── products.csv
│   ├── policy.pdf
│   └── sample.json
├── main.py
└── pyproject.toml
```

### <span style="color: #4C6B9A;">Install packages</span>

For a `uv`-managed project, add the packages required by the loaders you
use. Common packages may include:

``` bash
uv add langchain-community langchain-core
uv add pypdf pdfminer.six pdfplumber
```

Some integrations require extra dependencies. Install only what you need
and check the current LangChain documentation for version-specific
package names.

If using a `requirements.txt` file:

``` bash
uv pip install -r requirements.txt
```

### <span style="color: #4C6B9A;">Universal loading pattern</span>

``` python
loader = SomeLoader(path_or_source)
documents = loader.load()

print(type(documents))          # commonly list
print(len(documents))           # number of Document objects
print(type(documents[0]))       # commonly Document
print(documents[0].page_content)
print(documents[0].metadata)
```

The class and constructor parameters change with the source type, but
the basic pattern is consistent.

### <span style="color: #4C6B9A;">Helpful debugging</span>

``` python
from pprint import pprint

print("Number of documents:", len(documents))
if documents:
    print("First content preview:")
    print(documents[0].page_content[:500])
    print("First metadata:")
    pprint(documents[0].metadata)
```

------------------------------------------------------------------------

## <span style="color: #2457A7;">2.8 TextLoader</span>

### <span style="color: #4C6B9A;">What it does</span>

`TextLoader` reads a plain text file and returns document content with
source metadata. In the course's example, the file produces one
`Document`.

### <span style="color: #4C6B9A;">Syntax and example</span>

``` python
from pathlib import Path
from pprint import pprint
from langchain_community.document_loaders import TextLoader

file_path = Path("knowledge-source/sample.txt")

print(file_path.exists())  # Check that the path is correct

loader = TextLoader(file_path)
documents = loader.load()

print(type(documents))           # list
print(len(documents))            # commonly 1
print(type(documents[0]))         # Document
print(documents[0].page_content)  # extracted text
pprint(documents[0].metadata)     # includes source information
```

### <span style="color: #4C6B9A;">Parameters and behavior</span>

-   **File path:** identifies the file to read. A string or path object
    may be accepted, depending on the installed version.
-   **Encoding:** text files can use different encodings. If decoding
    fails, specify the correct encoding supported by the loader.
-   **Metadata:** the source path helps identify where the content came
    from.

### <span style="color: #4C6B9A;">Common errors</span>

-   Wrong working directory or file path.
-   File does not exist.
-   Incorrect encoding.
-   Assuming a loader returned many documents when it returned one.

<span style="color: #047857;"><strong>Practice:</strong></span> Change the sample file's contents, rerun the loader, and
inspect `page_content` and `metadata`.

------------------------------------------------------------------------

## <span style="color: #2457A7;">2.9 The loader interface (from the documentation)</span>

The loader interface provides common ways to retrieve documents. The
exact available methods vary by loader and version.

  -----------------------------------------------------------------------
  Method                  General behavior        When to use
  ----------------------- ----------------------- -----------------------
  `load()`                Returns the loaded      Convenient for small or
                          documents as a list     moderate sources

  `lazy_load()`           Yields documents        Useful when sources are
                          incrementally as a      large and supported by
                          generator/iterator      the loader

  Async variants, such as Allow asynchronous      Useful in async
  `aload()` or            loading                 applications and
  `alazy_load()` where                            I/O-heavy workflows
  implemented                                     
  -----------------------------------------------------------------------

### <span style="color: #4C6B9A;">Example</span>

``` python
documents = loader.load()

for doc in documents:
    print(doc.metadata, doc.page_content[:100])
```

### <span style="color: #4C6B9A;">Important caution</span>

Not every loader supports every method equally. Consult the installed
loader's documentation. The difference between synchronous and
asynchronous APIs is about how work is scheduled; it does not magically
improve parsing quality.

------------------------------------------------------------------------

## <span style="color: #2457A7;">2.10 PDF loaders</span>

PDF is challenging because it can contain selectable text, scanned page
images, tables, diagrams, multi-column layouts, and mixed content.
Different libraries use different parsers and trade off text quality,
metadata, image handling, and speed.

### <span style="color: #4C6B9A;">Loader comparison</span>

  -----------------------------------------------------------------------
  Loader                  Typical strength        Limitation / caution
  ----------------------- ----------------------- -----------------------
  `PyPDFLoader`           Straightforward         Complex layouts,
                          page-wise text          tables, and image text
                          extraction              may be missed or poorly
                                                  represented

  `PDFMinerLoader`        Can support image       May be slower when OCR
                          extraction/OCR          is involved; output
                          configurations and text order/quality must be
                          extraction              checked

  `PDFPlumberLoader`      Often provides richer   Extracted text can have
                          PDF metadata and        spacing/layout
                          page-by-page output     problems; not
                                                  automatically a
                                                  structured table or
                                                  image-description
                                                  solution

  Mathpix-related loader  Specialized for PDFs    Requires its relevant
                          with mathematical       setup/service and
                          notation                should be selected when
                                                  math extraction is
                                                  important
  -----------------------------------------------------------------------

The best loader depends on the PDF. Test it on representative pages
before processing a large collection.

### <span style="color: #4C6B9A;">A. PyPDFLoader: page mode</span>

``` python
from pathlib import Path
from pprint import pprint
from langchain_community.document_loaders.pdf import PyPDFLoader

file_path = Path("knowledge-source/policy.pdf")

loader = PyPDFLoader(file_path.as_posix(), mode="page")
documents = loader.load()

print("Pages/documents:", len(documents))
print(documents[0].page_content)
pprint(documents[1].metadata)
```

-   `Path.as_posix()` converts a path to slash-separated form, useful
    for passing paths consistently.
-   In `mode="page"`, the loader commonly returns one `Document` per PDF
    page.
-   Page metadata can include the source and page index, along with
    parser metadata.
-   Printed PDF page numbers and zero-based programmatic indexes may
    differ. Inspect the metadata rather than assuming the index equals
    the printed page label.

### <span style="color: #4C6B9A;">B. Page mode versus single mode</span>

A PDF loader may support different modes: - **Page mode:** keeps pages
as separate documents and can preserve page-level metadata. - **Single
mode:** combines the PDF into one document. Page-specific metadata may
no longer be available in the same way.

Use page mode when citations, page filtering, or page-level retrieval
matter. Use single mode only when combining the whole file is
appropriate.

### <span style="color: #4C6B9A;">C. Extracting images with PyPDFLoader (limited)</span>

Some versions/configurations allow image extraction, for example:

``` python
from langchain_community.document_loaders import PyPDFLoader
from langchain_community.document_loaders.parsers import RapidOCRBlobParser

loader = PyPDFLoader(
    "knowledge-source/policy.pdf",
    mode="page",
    extract_images=True,
    images_parser=RapidOCRBlobParser(),
    images_inner_format="html-img",
)

documents_with_images = loader.load()
```

This is an illustrative pattern; availability and constructor options
can change by package version. OCR libraries must be installed. The
course notes show that this approach may fail to recover text from some
images, so do not assume that enabling `extract_images=True` captures
every diagram.

Possible image-text approaches: 1. A multimodal LLM that can describe or
read the image. 2. OCR, such as RapidOCR or Tesseract, for text
contained in images. 3. A different PDF parser when the first parser
fails.

OCR can extract visible words but may not understand the meaning of a
diagram. A multimodal model may provide a description, but its output
should also be checked.

### <span style="color: #4C6B9A;">D. PDFMinerLoader for images and tables</span>

Illustrative pattern:

``` python
from pathlib import Path
from langchain_community.document_loaders import PDFMinerLoader
from langchain_community.document_loaders.parsers import RapidOCRBlobParser

file_path = Path("knowledge-source/policy.pdf")

loader = PDFMinerLoader(
    file_path.as_posix(),
    mode="page",
    extract_images=True,
    images_parser=RapidOCRBlobParser(),
    images_inner_format="html-img",
)

documents = loader.load()
print(len(documents))
print(documents[0].page_content)
```

-   In the course's example, image-derived text from a diagram was
    appended after the ordinary page text.
-   Image-derived text was marked with HTML image tags.
-   OCR processing can make this slower.
-   A parser may recover text that another loader misses; results vary
    by file and parser.

### <span style="color: #4C6B9A;">E. Marking extracted image text: HTML versus Markdown</span>

  -----------------------------------------------------------------------
  Format                              Behavior
  ----------------------------------- -----------------------------------
  `html-img`                          Image-derived text is wrapped in
                                      HTML-like image tags

  Markdown image format               Image-derived text is represented
                                      with Markdown image syntax
  -----------------------------------------------------------------------

HTML markers can help distinguish OCR-derived text from ordinary PDF
text because normal text is less likely to contain the same HTML tags.
Clear markers are useful when downstream code needs to tell content
sources apart.

### <span style="color: #4C6B9A;">F. Tables in PDFs</span>

A PDF parser may extract text from a table near its original position in
`page_content`. That does **not** guarantee that rows and columns were
reconstructed into a structured table.

For example, the parser may return the words from a table but lose the
relationship between each value and its column header. If the RAG
application must answer numeric or row-specific questions, verify the
extraction carefully and consider a table-aware extraction process.

### <span style="color: #4C6B9A;">G. Filter documents by metadata</span>

If the loader returns one document per page, metadata can be used to
select a page. The course's example uses a zero-based page index; a
printed page 8 corresponds to index 7 when indexing starts at zero.

``` python
for doc in documents:
    if doc.metadata.get("page") == 7:
        print(doc.page_content)
        break
```

First inspect metadata:

``` python
pprint(documents[0].metadata)
```

Do not assume every PDF loader uses the same metadata keys or numbering
convention.

### <span style="color: #4C6B9A;">H. PDFPlumberLoader</span>

``` python
from pathlib import Path
from langchain_community.document_loaders import PDFPlumberLoader

file_path = Path("knowledge-source/policy.pdf")
loader = PDFPlumberLoader(file_path.as_posix())
documents = loader.load()

print(len(documents))
print(documents[0].page_content)
pprint(documents[0].metadata)
```

Typical behavior described in the course: - Returns page-by-page
documents. - Provides richer metadata such as source, file path, page,
total pages, author, creation date, creator, keywords, modified date,
producer, subject, title, and trapped status, depending on the file. -
Extracted words may run together or have missing spaces. - In the
demonstrated examples, it did not directly provide structured tables or
image descriptions.

### <span style="color: #4C6B9A;">PDF loader decision guide</span>

Choose based on the requirement: - Clean selectable text → test
`PyPDFLoader` and alternatives. - Page-level source references → use a
page-wise mode and preserve metadata. - Scanned pages or text inside
diagrams → OCR or multimodal processing. - Tables with row/column
semantics → use table-aware extraction and validate output. - Rich
metadata → test `PDFPlumberLoader`. - Mathematical notation → consider a
math-specialized option.

**Golden rule for PDFs:** Never judge a loader from the fact that it ran
without errors. Inspect the extracted text, tables, images, page
metadata, and reading order on real examples.

------------------------------------------------------------------------

## <span style="color: #2457A7;">2.11 CSVLoader</span>

### <span style="color: #4C6B9A;">What it does</span>

CSV files contain rows and columns. `CSVLoader` commonly converts each
row into a `Document`. In the course example, a 1,000-row CSV produces
1,000 documents.

### <span style="color: #4C6B9A;">Basic example</span>

``` python
from pathlib import Path
from pprint import pprint
from langchain_community.document_loaders.csv_loader import CSVLoader

file_path = Path("knowledge-source/products.csv")
loader = CSVLoader(file_path=file_path)

documents = loader.load()

print("Rows/documents:", len(documents))
print(documents[0].page_content)
pprint(documents[0].metadata)
```

A row's `page_content` commonly contains column names and values, and
metadata often includes the source path and row index.

### <span style="color: #4C6B9A;">Customizing what goes into content and metadata</span>

CSVLoader supports configuration such as: - **`source_column`:** use a
selected column's value as the source metadata. -
**`metadata_columns`:** keep specified columns in metadata rather than
putting all their information into the main content. - **Encoding / CSV
parsing options:** depending on the installed version, use the
appropriate options for file encoding and CSV dialect.

Illustrative example:

``` python
from langchain_community.document_loaders.csv_loader import CSVLoader

loader = CSVLoader(
    file_path="knowledge-source/products.csv",
    source_column="Industry",
    metadata_columns=["Industry", "Product_ID"],
)

documents = loader.load()

print(documents[0].page_content)
print(documents[0].metadata)
```

Check the column names against the actual CSV header. The exact
available constructor parameters depend on the installed version.

### <span style="color: #4C6B9A;">Why metadata columns matter</span>

Suppose a row describes a product. Product description is useful for
semantic search; product ID, industry, region, or category may be more
useful as metadata filters or source identifiers. Separating these roles
can improve retrieval and citations.

### <span style="color: #4C6B9A;">Common CSV pitfalls</span>

-   Incorrect header or column names.
-   Wrong delimiter or encoding.
-   Blank or malformed rows.
-   Treating a row index as a permanent business identifier.
-   Losing the association between a value and its column name.

------------------------------------------------------------------------

## <span style="color: #2457A7;">2.12 JSONLoader</span>

### <span style="color: #4C6B9A;">What it does</span>

JSON is hierarchical. A loader needs to know which part of the JSON
structure should become each document.

For example:

``` json
{
  "products": [
    {"id": "P-01", "name": "Notebook", "description": "Reusable paper notebook"},
    {"id": "P-02", "name": "Pen", "description": "Blue ink pen"}
  ]
}
```

A JSON loader can extract each product separately instead of placing the
entire JSON file into one document.

### <span style="color: #4C6B9A;">Basic pattern</span>

LangChain's JSON loader commonly uses a **`jq_schema`** expression to
select data, plus a content extraction strategy. Example:

``` python
from langchain_community.document_loaders.json_loader import JSONLoader

loader = JSONLoader(
    file_path="knowledge-source/sample.json",
    jq_schema=".products[]",
    text_content=False,
)

documents = loader.load()

print("Documents:", len(documents))
print(documents[0].page_content)
print(documents[0].metadata)
```

The exact `jq_schema`, `text_content`, and metadata configuration must
match the actual JSON structure and installed loader version. Some
setups require the `jq` dependency.

### <span style="color: #4C6B9A;">What to understand</span>

-   JSON keys and arrays form a hierarchy.
-   The extraction expression decides which object or field is selected.
-   A document may represent one object, one array item, or another
    selected unit.
-   Metadata can preserve IDs or source information that should not be
    lost.
-   Validate the output: a loader can parse valid JSON yet select the
    wrong fields.

<span style="color: #047857;"><strong>Practice:</strong></span> Change `.products[]` to select one field, then inspect how
the number and content of documents change.

------------------------------------------------------------------------

## <span style="color: #2457A7;">2.13 WebBaseLoader</span>

### <span style="color: #4C6B9A;">What it does</span>

`WebBaseLoader` retrieves a web page and parses its HTML into text and
metadata. It is useful when a website or selected URLs are a source of
knowledge.

### <span style="color: #4C6B9A;">Single URL example</span>

``` python
from langchain_community.document_loaders import WebBaseLoader

url = "https://example.com/"
loader = WebBaseLoader(web_paths=[url])
documents = loader.load()

print(len(documents))
print(documents[0].page_content[:1000])
print(documents[0].metadata)
```

### <span style="color: #4C6B9A;">Multiple URLs</span>

``` python
from langchain_community.document_loaders import WebBaseLoader

urls = [
    "https://example.com/page-one",
    "https://example.com/page-two",
]

loader = WebBaseLoader(web_paths=urls)
documents = loader.load()

print("Documents:", len(documents))
for doc in documents:
    print(doc.metadata.get("source"))
    print(doc.page_content[:200])
```

The course's example uses one document per URL. Metadata may include the
source URL, title, and page-related fields.

### <span style="color: #4C6B9A;">Important cautions</span>

-   A web page may require JavaScript rendering or authentication that a
    simple HTML loader does not handle.
-   Menus, footers, cookie notices, and unrelated content may be
    extracted.
-   A URL may change or disappear.
-   Follow site terms, robots rules, authorization requirements, and
    your organization's policies.
-   Keep the source URL in metadata for provenance and updates.
-   Check whether the extracted content is actually the main article
    rather than navigation or boilerplate.

------------------------------------------------------------------------

## <span style="color: #2457A7;">2.14 RecursiveUrlLoader</span>

### <span style="color: #4C6B9A;">What it does</span>

`RecursiveUrlLoader` can start from a URL and follow eligible links
recursively, subject to its crawl rules. This is useful for ingesting a
documentation section rather than manually listing every page.

Illustrative pattern:

``` python
from langchain_community.document_loaders import RecursiveUrlLoader

loader = RecursiveUrlLoader(
    url="https://example.com/docs/",
    max_depth=2,
)

documents = loader.load()
print("Pages/documents:", len(documents))

for doc in documents[:3]:
    print(doc.metadata)
    print(doc.page_content[:300])
```

Constructor parameters and defaults can differ by version; confirm the
current API before running.

### <span style="color: #4C6B9A;">How it differs from WebBaseLoader</span>

  -----------------------------------------------------------------------
  WebBaseLoader                       RecursiveUrlLoader
  ----------------------------------- -----------------------------------
  Fetches the URL(s) supplied         Can discover and follow links from
                                      a starting URL

  Good for a known list of pages      Good for a bounded crawl of linked
                                      pages

  Scope is mostly the provided URLs   Scope depends on recursion depth
                                      and URL/link filtering
  -----------------------------------------------------------------------

### <span style="color: #4C6B9A;">Crawl safeguards</span>

-   Set a suitable depth or page limit where supported.
-   Restrict the crawl to the intended domain/path.
-   Avoid crawling login areas or private pages without authorization.
-   Be mindful of rate limits and server load.
-   Deduplicate URLs and documents.
-   Record crawl time and source URLs so content can be refreshed.
-   Check whether links lead to irrelevant, duplicated, or unbounded
    content.

------------------------------------------------------------------------

## <span style="color: #2457A7;">2.15 `load()` versus `lazy_load()`</span>

### <span style="color: #4C6B9A;">`load()`</span>

-   Loads the documents and returns them as a list.
-   The whole returned list is held in memory.
-   Convenient for small sources and simple scripts.
-   Easy to inspect, index, slice, and pass to the next step.

### <span style="color: #4C6B9A;">`lazy_load()`</span>

-   Returns a generator/iterator that yields documents incrementally.
-   The application can process a document before asking for the next
    one.
-   Useful for large sources when the loader supports lazy loading.
-   Can reduce peak memory usage, depending on the loader and how
    downstream code is written.

### <span style="color: #4C6B9A;">Comparison</span>

  -----------------------------------------------------------------------
  Aspect                  `load()`                `lazy_load()`
  ----------------------- ----------------------- -----------------------
  Return behavior         List                    Generator/iterator

  Memory pattern          All returned documents  Documents can be
                          are held in a list      processed incrementally

  Typical use             Small/medium sources    Large or
                                                  streaming-style
                                                  processing

  Ease of inspection      Very easy to            Usually iterate over it
                          index/slice             

  Caveat                  May use significant     Not all loaders support
                          memory for large input  it; lazy downstream
                                                  processing is required
  -----------------------------------------------------------------------

### <span style="color: #4C6B9A;">Example</span>

``` python
# <span style="color: #374151;">Eager loading</span>
documents = loader.load()
for doc in documents:
    process_document(doc)

# <span style="color: #374151;">Lazy loading, if this loader supports it</span>
for doc in loader.lazy_load():
    process_document(doc)
```

A simple placeholder processor:

``` python
def process_document(doc):
    print(doc.metadata.get("source"), len(doc.page_content))
```

**Important:** `lazy_load()` is not automatically faster in every
situation. It changes how results are yielded and can reduce memory
pressure. The actual speed depends on I/O, parsing, network behavior,
and downstream work.

------------------------------------------------------------------------

## <span style="color: #2457A7;">2.16 Golden rules for document loaders</span>

1.  **Select a loader for the source format and your extraction goal.**
2.  **Inspect output immediately:** print document count, a content
    preview, and metadata.
3.  **Do not assume one file equals one document.** A PDF may produce
    one document per page; a CSV may produce one per row.
4.  **Preserve metadata.** Source, page, row, URL, and identifiers
    enable traceability and filtering.
5.  **Validate parsing quality.** Successful execution does not mean
    tables, images, reading order, or all text were extracted correctly.
6.  **Test real examples.** PDF parsing results vary with layout and
    parser.
7.  **Use OCR or multimodal processing when necessary** for scanned
    pages or text embedded in images.
8.  **Do not confuse extracted table text with a structured table.**
9.  **Choose `load()` or `lazy_load()` based on source size and loader
    support.**
10. **Keep the source of truth.** The vector store should not make the
    original document untraceable.
11. **Plan for updates and deletions.** Changed or removed source
    content must be reflected in the index.
12. **Respect permissions and website policies.** A loader should only
    ingest content the application is authorized to use.

### <span style="color: #4C6B9A;">Chapter 2 one-minute recap</span>

Document loaders ingest raw sources and parse them into standardized
`Document` objects with `page_content` and `metadata`. Different formats
need different parsers: text, PDF, CSV, JSON, and web loaders have
different output units and limitations. PDF extraction is especially
tricky for images, tables, and layouts. Inspect every loader's output,
preserve metadata, and use lazy loading when supported and appropriate
for large sources.

------------------------------------------------------------------------

# <span style="color: #374151;">Python practice lab</span>

These exercises progress from simple loading to a small ingestion
preparation pipeline. Use your own sample files and install only the
relevant packages.

## <span style="color: #2457A7;">Lab 1 --- Create sample files</span>

Create `knowledge-source/sample.txt`:

``` text
RAG stands for Retrieval-Augmented Generation.
A retriever finds relevant evidence for a user question.
An LLM generates an answer from the question and supplied context.
```

Create `knowledge-source/products.csv`:

``` csv
product_id,name,description
P-01,Notebook,Reusable paper notebook
P-02,Pen,Blue ink pen for writing
P-03,Water Bottle,Reusable stainless steel bottle
```

Create `knowledge-source/sample.json`:

``` json
{
  "products": [
    {"id": "P-01", "name": "Notebook", "description": "Reusable paper notebook"},
    {"id": "P-02", "name": "Pen", "description": "Blue ink pen for writing"}
  ]
}
```

## <span style="color: #2457A7;">Lab 2 --- Load and inspect text</span>

``` python
from langchain_community.document_loaders import TextLoader

loader = TextLoader("knowledge-source/sample.txt")
docs = loader.load()

print("Document count:", len(docs))
print("Content:", docs[0].page_content)
print("Metadata:", docs[0].metadata)
```

Questions: 1. Why is the result a list? 2. What is `page_content`? 3.
Why should you preserve `metadata`?

## <span style="color: #2457A7;">Lab 3 --- Load CSV rows</span>

``` python
from langchain_community.document_loaders.csv_loader import CSVLoader

loader = CSVLoader("knowledge-source/products.csv")
docs = loader.load()

print("Rows:", len(docs))
for doc in docs:
    print(doc.page_content)
    print(doc.metadata)
```

Expected observation: a small CSV with three data rows will commonly
yield three documents. Verify the result in your installed version.

## <span style="color: #2457A7;">Lab 4 --- Load JSON objects</span>

``` python
from langchain_community.document_loaders.json_loader import JSONLoader

loader = JSONLoader(
    file_path="knowledge-source/sample.json",
    jq_schema=".products[]",
    text_content=False,
)
docs = loader.load()

print("Products:", len(docs))
for doc in docs:
    print(doc.page_content)
    print(doc.metadata)
```

If you see a dependency or schema error, check the JSONLoader
documentation and ensure the JSON query dependency and constructor
settings match your environment.

## <span style="color: #2457A7;">Lab 5 --- Load a PDF page by page</span>

``` python
from langchain_community.document_loaders import PyPDFLoader

loader = PyPDFLoader("knowledge-source/policy.pdf", mode="page")
docs = loader.load()

print("Pages:", len(docs))
for i, doc in enumerate(docs[:3]):
    print(f"\nDocument {i}")
    print("Metadata:", doc.metadata)
    print(doc.page_content[:500])
```

Inspect whether text, headings, tables, and page references were
extracted correctly.

## <span style="color: #2457A7;">Lab 6 --- Write a generic inspection helper</span>

``` python
def inspect_documents(documents, preview_chars=250):
    print(f"Number of documents: {len(documents)}")

    for index, doc in enumerate(documents[:5]):
        print(f"\n--- Document {index} ---")
        print("Metadata:", doc.metadata)
        print("Content preview:")
        print(doc.page_content[:preview_chars])

# <span style="color: #374151;">Example:</span>
# <span style="color: #374151;">inspect_documents(docs)</span>
```

## <span style="color: #2457A7;">Lab 7 --- Compare eager and lazy loading</span>

``` python
from langchain_community.document_loaders import TextLoader

loader = TextLoader("knowledge-source/sample.txt")

# <span style="color: #374151;">Eager: returns a list</span>
docs = loader.load()
print(type(docs), len(docs))

# <span style="color: #374151;">Lazy: use only if supported by this loader/version</span>
for doc in loader.lazy_load():
    print(doc.metadata, len(doc.page_content))
```

## <span style="color: #2457A7;">Lab 8 --- Prepare documents for the next stage</span>

This exercise does not yet generate embeddings or use a vector database.
It checks that the loaded content is ready for splitting.

``` python
def validate_documents(documents):
    valid = []
    for doc in documents:
        if not isinstance(doc.page_content, str):
            print("Skipping: content is not a string")
            continue

        if not doc.page_content.strip():
            print("Skipping empty content:", doc.metadata)
            continue

        # Preserve source metadata for downstream chunks and citations.
        valid.append(doc)

    return valid

# <span style="color: #374151;">documents = loader.load()</span>
# <span style="color: #374151;">clean_documents = validate_documents(documents)</span>
# <span style="color: #374151;">print("Valid documents:", len(clean_documents))</span>
```

------------------------------------------------------------------------

# <span style="color: #374151;">Interview preparation --- Chapters 1 and 2</span>

## <span style="color: #2457A7;">Core questions and answers</span>

### <span style="color: #4C6B9A;">1. What is RAG?</span>

RAG is an application pattern that retrieves relevant external
information, adds it to the LLM prompt, and generates an answer using
that context.

### <span style="color: #4C6B9A;">2. Why is RAG useful?</span>

It can make current or private knowledge available at query time,
provide evidence for answers, and reduce reliance on the model's
internal training knowledge alone.

### <span style="color: #4C6B9A;">3. What are the three stages represented by RAG?</span>

Retrieval, augmentation, and generation.

### <span style="color: #4C6B9A;">4. What is parametric knowledge?</span>

Knowledge represented implicitly in the model's learned parameters
rather than stored as a directly searchable external document database.

### <span style="color: #4C6B9A;">5. What is in-context learning?</span>

The model uses information or examples supplied in the current input
without updating its model parameters.

### <span style="color: #4C6B9A;">6. Why not paste the entire knowledge base into every prompt?</span>

The input can exceed the context window, be expensive, and contain
irrelevant content that the model may overlook or misuse.

### <span style="color: #4C6B9A;">7. What is the "lost in the middle" problem?</span>

A model may fail to use information buried in the middle of a long
context as effectively as information placed in more salient positions.

### <span style="color: #4C6B9A;">8. Does RAG eliminate hallucinations?</span>

No. It supplies evidence, but retrieval can be poor and the LLM can
still misinterpret or ignore the evidence.

### <span style="color: #4C6B9A;">9. What is the difference between a knowledge source and a vector store?</span>

The source contains original material; the vector store indexes vector
representations and associated records for retrieval.

### <span style="color: #4C6B9A;">10. What is a document loader?</span>

A component that reads/parses a source and returns standardized document
objects.

### <span style="color: #4C6B9A;">11. What is a LangChain `Document`?</span>

A common object with `page_content` for extracted content and `metadata`
for source/context information.

### <span style="color: #4C6B9A;">12. Why is metadata important?</span>

It enables source attribution, page/row references, filtering,
permission-aware retrieval, debugging, and updates.

### <span style="color: #4C6B9A;">13. Does every loader return one document per file?</span>

No. It depends on the loader and configuration. A PDF loader may return
one per page; a CSV loader may return one per row.

### <span style="color: #4C6B9A;">14. What is the difference between loading and chunking?</span>

Loading parses the source into document objects. Chunking divides those
documents into smaller retrieval units.

### <span style="color: #4C6B9A;">15. What is the difference between chunking and embedding?</span>

Chunking decides the text units; embedding converts each unit into a
vector.

### <span style="color: #4C6B9A;">16. What does a vector store do?</span>

It stores vectors and associated records and supports similarity search
and related retrieval operations.

### <span style="color: #4C6B9A;">17. Why is PDF loading difficult?</span>

PDFs can combine selectable text, scanned images, diagrams, tables, and
complex layouts. A parser may extract text but lose structure or miss
image content.

### <span style="color: #4C6B9A;">18. Does extracted table text mean the table is structured?</span>

No. The parser may extract words while losing row/column relationships.

### <span style="color: #4C6B9A;">19. When might OCR be needed?</span>

When important text is present in scanned pages or images and is not
available as selectable PDF text.

### <span style="color: #4C6B9A;">20. What is the difference between `load()` and `lazy_load()`?</span>

`load()` returns a list of all documents; `lazy_load()` yields documents
incrementally when supported, which can help reduce peak memory use.

### <span style="color: #4C6B9A;">21. When would you use WebBaseLoader?</span>

When you already know which web URLs to load.

### <span style="color: #4C6B9A;">22. When would you use RecursiveUrlLoader?</span>

When you want to start at a page and follow eligible links within a
bounded crawl.

### <span style="color: #4C6B9A;">23. What happens if the loader returns poor text?</span>

The downstream chunks, embeddings, retrieval results, and answers may
all be poor. Inspect and validate extraction before indexing.

### <span style="color: #4C6B9A;">24. How does RAG support citations?</span>

Loaders and ingestion preserve source metadata; the application carries
it through chunks and retrieval, then connects the answer to the
returned sources.

### <span style="color: #4C6B9A;">25. What is the difference between fine-tuning and RAG?</span>

Fine-tuning changes learned model behavior through further training; RAG
provides external context at query time. They address different needs
and can be combined.

------------------------------------------------------------------------

# <span style="color: #374151;">Common mistakes to avoid</span>

-   Saying that the LLM "searches the vector database by itself." The
    application orchestrates retrieval and passes results to the model.
-   Treating the vector store as the original source of truth.
-   Assuming RAG always produces factual answers.
-   Assuming every PDF loader extracts images, diagrams, and tables
    correctly.
-   Losing page numbers or URLs during preprocessing.
-   Forgetting that query and document embeddings should be compatible.
-   Using chunks so large that retrieval returns lots of irrelevant
    text.
-   Using chunks so small that they lose the context needed to interpret
    a fact.
-   Assuming `load()` and `lazy_load()` are supported identically by
    every loader.
-   Crawling websites without appropriate scope, permissions, or rate
    limits.
-   Treating a successful load as proof of extraction quality.

------------------------------------------------------------------------

# <span style="color: #374151;">Memory tricks</span>

  -----------------------------------------------------------------------
  Concept                             Memory trick
  ----------------------------------- -----------------------------------
  RAG                                 **Find → Add → Answer**

  LLM problems                        **S-H-S-P**: Stale knowledge,
                                      Hallucination, Source missing,
                                      Private data missing

  Pipeline                            **Source → Load → Split → Embed →
                                      Store → Retrieve → Augment →
                                      Generate**

  `Document` object                   **C + M**: Content
                                      (`page_content`) + Metadata

  Loader vs splitter                  Loader reads; splitter cuts

  Splitter vs embedding               Splitter makes text pieces;
                                      embedding makes vectors

  Knowledge source vs vector store    Original library vs searchable
                                      index

  PDF quality check                   **Text, Tables, Images, Metadata,
                                      Order**

  `load()` vs `lazy_load()`           Load all vs yield as you go

  RAG limitation                      Evidence helps, but does not
                                      guarantee truth
  -----------------------------------------------------------------------

------------------------------------------------------------------------

# <span style="color: #374151;">Final revision sheets</span>

## <span style="color: #2457A7;">Chapter 1: 10 things to remember</span>

1.  LLMs learn patterns from large-scale training; they do not
    automatically have live access to every source.
2.  Plain LLMs can have stale knowledge, hallucinate, lack citations,
    and lack private data.
3.  In-context learning supplies information in the prompt without
    changing model parameters.
4.  Fine-tuning changes model behavior through further training; it is
    not the default solution for every changing fact.
5.  Context windows are limited, and long prompts can suffer from
    lost-in-the-middle effects.
6.  RAG = retrieval + augmentation + generation.
7.  Indexing and query-time retrieval are different parts of the
    workflow.
8.  Loaders, splitters, embedding models, vector stores, retrievers, and
    LLMs have different jobs.
9.  Metadata is essential for traceability and citations.
10. RAG reduces some risks but does not guarantee correctness.

## <span style="color: #2457A7;">Chapter 2: 10 things to remember</span>

1.  Loaders parse heterogeneous sources into a common document format.
2.  `page_content` holds extracted text; `metadata` describes source and
    context.
3.  Different loaders return different numbers of documents.
4.  Inspect the returned count, content, and metadata before indexing.
5.  PDF parsing quality varies with layout and parser.
6.  OCR can help with scanned images, but does not guarantee semantic
    understanding.
7.  Extracted table text is not necessarily a structured table.
8.  CSV row metadata and JSON extraction rules should be selected
    deliberately.
9.  Web crawling needs scope, permission, and rate-limit safeguards.
10. Use lazy loading when appropriate and supported; preserve source
    information throughout the pipeline.

## <span style="color: #2457A7;">Self-test checklist</span>

Before moving to Chapter 3, make sure you can:

-   [ ] Explain RAG without reading a definition.
-   [ ] Draw the indexing path and query path.
-   [ ] Explain the four problems of a plain LLM.
-   [ ] Explain the context-window and lost-in-the-middle limitations.
-   [ ] Distinguish retrieval, augmentation, and generation.
-   [ ] Explain what a `Document` object contains.
-   [ ] Load a TXT, CSV, PDF, JSON, and web page using the appropriate
    loader.
-   [ ] Inspect metadata and explain why it matters.
-   [ ] Describe why PDFs need careful validation for tables and images.
-   [ ] Explain `load()` versus `lazy_load()`.
-   [ ] Identify where poor source extraction can damage the final RAG
    answer.

------------------------------------------------------------------------

## <span style="color: #2457A7;">Source and scope note</span>

Prepared from the first two chapters of
`Campusx_Advanced_RAG_Complete_2026.pdf`: - **Chapter 1 --- Introduction
to RAG** (printed starting page 5) - **Chapter 2 --- Document Loaders**
(printed starting page 23 and ending before Chapter 3)

The notes cover the chapter's numbered sections, key concepts, loader
families, examples, limitations, practical code patterns, and
chapter-level review. Code is rewritten for learning and may require
adjustments for the exact versions installed. Always check the current
library documentation before deploying code.
