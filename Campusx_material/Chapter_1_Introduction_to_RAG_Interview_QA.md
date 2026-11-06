# Introduction to RAG (Chapter 1) - Interview Questions and Answers

> Based on Chapter 1: Introduction to RAG from <File>2 Advanced_RAG_Chapters_1_2_Visual_Formatting.md</File>

## 1. What is RAG?
**Answer:** RAG (Retrieval-Augmented Generation) is an application pattern that retrieves relevant external information, adds it to the LLM prompt, and generates an answer using that context.

## 2. Why is RAG needed?
**Answer:** RAG addresses common LLM limitations such as stale knowledge, hallucinations, lack of citations, and inability to access private organizational data.

## 3. What are the three stages of RAG?
**Answer:**
1. Retrieval
2. Augmentation
3. Generation

**Memory Trick:** Find → Add → Answer.

## 4. How do LLMs work?
**Answer:** LLMs use Transformer architecture, self-attention mechanisms, and large-scale pre-training to learn language patterns and generate text through next-token prediction.

## 5. What is pre-training?
**Answer:** Pre-training is the initial training phase where an LLM learns from massive datasets such as books, websites, articles, blogs, and code repositories.

## 6. What is parametric knowledge?
**Answer:** Parametric knowledge is the knowledge implicitly stored within the learned parameters (weights and biases) of a model rather than in an external database.

## 7. What is next-token prediction?
**Answer:** It is the process where a language model predicts the most probable next token based on previous tokens and context.

## 8. Why can LLMs hallucinate?
**Answer:** Because LLMs are optimized to generate probable text rather than verify factual accuracy.

## 9. What are the four major problems of plain LLMs?
**Answer:**
- Stale knowledge
- Hallucinations
- Missing citations/source attribution
- No access to private organizational data

## 10. What is stale knowledge?
**Answer:** Knowledge becomes stale when facts in the model's training data become outdated after the training cutoff date.

## 11. What is hallucination?
**Answer:** Hallucination occurs when an LLM generates incorrect, invented, or unsupported information while sounding confident.

## 12. How does RAG reduce hallucinations?
**Answer:** By supplying relevant evidence from external sources during answer generation. However, it cannot completely eliminate hallucinations.

## 13. What is source attribution?
**Answer:** Source attribution means providing references, documents, pages, or URLs that support generated answers.

## 14. Why don't LLMs have access to private company data by default?
**Answer:** Because organizational data is not part of the model's original training dataset and must be explicitly connected.

## 15. What is in-context learning?
**Answer:** In-context learning allows a model to use information supplied within the prompt without modifying model parameters.

## 16. What are the advantages of in-context learning?
**Answer:**
- Easy to implement
- No retraining required
- Knowledge updates can be made by changing context

## 17. What are the limitations of in-context learning?
**Answer:**
- Context window limitations
- Expensive with long prompts
- Too much irrelevant information reduces effectiveness

## 18. What is fine-tuning?
**Answer:** Fine-tuning is additional training on targeted datasets to adjust model behavior, style, format, or task performance.

## 19. Difference between RAG and Fine-Tuning?
**Answer:**
| RAG | Fine-Tuning |
|------|-------------|
| Changes input context | Changes model behavior |
| Uses external knowledge | Learns from new training |
| Easy updates | Requires retraining |
| Best for changing facts | Best for behavior customization |

## 20. What is a context window?
**Answer:** The maximum amount of text/tokens an LLM can process at one time.

## 21. What is the 'Lost in the Middle' problem?
**Answer:** Important information placed in the middle of a long context may receive less attention compared to information at the beginning or end.

## 22. Why was RAG created?
**Answer:** RAG was created to efficiently provide relevant external knowledge to LLMs without stuffing entire knowledge bases into prompts.

## 23. Give the formula for RAG.
**Answer:**
In-Context Learning + Query-Specific External Knowledge = RAG

## 24. What happens during Retrieval?
**Answer:** Relevant information is searched and selected from an external knowledge source.

## 25. What happens during Augmentation?
**Answer:** Retrieved information is inserted into the prompt alongside the user's question.

## 26. What happens during Generation?
**Answer:** The LLM generates the final answer using the augmented prompt.

## 27. Why is retrieval important in RAG?
**Answer:** Retrieval determines which evidence reaches the model. Poor retrieval often leads to poor answers.

## 28. What is filtered context?
**Answer:** Only the most relevant information is provided to the LLM instead of the entire knowledge base.

## 29. What are the benefits of filtered context?
**Answer:**
- Reduces token usage
- Lowers cost
- Improves relevance
- Fits inside context windows
- Supports source traceability

## 30. What is a knowledge source?
**Answer:** The original collection of information such as PDFs, websites, databases, documents, or internal repositories.

## 31. What is chunking?
**Answer:** Chunking is the process of splitting large documents into smaller pieces for efficient retrieval.

## 32. Why is chunking necessary?
**Answer:** Large documents are difficult to search and pass directly to an LLM. Smaller chunks improve retrieval precision.

## 33. What is chunk overlap?
**Answer:** Shared text between consecutive chunks that preserves context across chunk boundaries.

## 34. What happens if chunks are too small?
**Answer:** Important contextual information can be lost.

## 35. What happens if chunks are too large?
**Answer:** Retrieval precision decreases because irrelevant information is included.

## 36. What is an embedding?
**Answer:** A numerical vector representation of text that captures semantic meaning.

## 37. Why are embeddings used in RAG?
**Answer:** They help compare the semantic similarity between user queries and stored documents.

## 38. What is a vector store?
**Answer:** A database optimized for storing embeddings and performing similarity searches.

## 39. What information is typically stored in a vector store?
**Answer:**
- Embedding vectors
- Text chunks
- IDs
- Metadata

## 40. Why is metadata important?
**Answer:** Metadata supports filtering, citations, source tracking, updates, and access control.

## 41. What is query embedding?
**Answer:** Converting a user's question into a vector using the same embedding model used for documents.

## 42. What is similarity search?
**Answer:** A process that finds document vectors closest to the query vector.

## 43. Name common similarity metrics.
**Answer:**
- Cosine Similarity
- Euclidean Distance
- Dot Product

## 44. What is top-k retrieval?
**Answer:** A retrieval approach that returns the top K most relevant chunks.

## 45. What happens if k is very small?
**Answer:** Important evidence may be missed.

## 46. What happens if k is very large?
**Answer:** Noise and irrelevant information increase.

## 47. What is context assembly?
**Answer:** The process of organizing retrieved chunks and metadata into a prompt for the LLM.

## 48. What are the two major paths in a RAG system?
**Answer:**
- Indexing Path
- Query-Time Path

## 49. What happens during indexing?
**Answer:**
1. Collect documents
2. Load documents
3. Chunk data
4. Generate embeddings
5. Store vectors and metadata

## 50. What happens during query time?
**Answer:**
1. Receive query
2. Create query embedding
3. Retrieve relevant chunks
4. Assemble context
5. Generate answer

## 51. Does RAG completely solve stale knowledge?
**Answer:** No. The external source must be updated and retrieval must successfully find the updated content.

## 52. Does RAG eliminate hallucinations?
**Answer:** No. It reduces them but cannot guarantee factual correctness.

## 53. What are common RAG failure points?
**Answer:**
- Bad source data
- Poor document parsing
- Poor chunking
- Weak embeddings
- Retrieval failures
- Poor prompt construction
- Generation errors
- Missing evaluation
- Security failures

## 54. Why should indexing and query paths be kept separate?
**Answer:** It simplifies scalability, caching, monitoring, and incremental updates.

## 55. What is the most important lesson about RAG?
**Answer:** RAG provides relevant evidence to the model but does not guarantee truth. High-quality sources, retrieval, prompting, and evaluation are all critical.

---

# Quick Interview Revision

### Remember These Keywords
- RAG = Retrieval + Augmentation + Generation
- LLM Issues = Stale Knowledge + Hallucination + Missing Sources + Private Data Gap
- Pipeline = Source → Load → Split → Embed → Store → Retrieve → Augment → Generate
- Embedding = Text → Vector
- Vector Store = Storage + Similarity Search
- Golden Rule = Retrieval improves answers but does not guarantee correctness.
