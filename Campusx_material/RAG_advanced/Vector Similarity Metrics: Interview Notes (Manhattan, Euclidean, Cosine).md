# Manhattan vs Euclidean vs Cosine

Three ways to measure how close two vectors are. In RAG, they decide which chunks count as "similar" to the user's question.

---

## 1. Theory

### Manhattan distance (L1)
Add up the absolute differences along each dimension. Think of walking along city blocks where you can't cut diagonally.

```
d = |x1 - y1| + |x2 - y2| + ... + |xn - yn|
```

### Euclidean distance (L2)
The straight-line distance between two points, the one you would measure with a ruler.

```
d = sqrt( (x1 - y1)^2 + (x2 - y2)^2 + ... + (xn - yn)^2 )
```

### Cosine similarity
Ignores length and measures only the **angle** between two vectors, meaning whether they point in the same direction.

```
cosine(A, B) = (A . B) / (|A| x |B|)
```

- **A . B** (dot product) = a1*b1 + a2*b2 + ...
- **|A|** (magnitude) = sqrt(a1^2 + a2^2 + ...)

| Value | Meaning |
|---|---|
| 1 | Same direction (very similar) |
| 0 | Unrelated |
| -1 | Opposite |

Cosine **distance** = 1 - cosine similarity.

---

## 2. Worked Example

**A = (1, 2)** and **B = (4, 6)**

### Manhattan
```
|1 - 4| + |2 - 6| = 3 + 4 = 7
```

### Euclidean
```
sqrt( (1 - 4)^2 + (2 - 6)^2 )
= sqrt( 9 + 16 )
= sqrt(25)
= 5
```

### Cosine

**Step 1: dot product**
```
A . B = (1 x 4) + (2 x 6) = 4 + 12 = 16
```

**Step 2: magnitudes**
```
|A| = sqrt(1^2 + 2^2) = sqrt(5)  = 2.236
|B| = sqrt(4^2 + 6^2) = sqrt(52) = 7.211
```

**Step 3: divide**
```
cosine = 16 / (2.236 x 7.211) = 16 / 16.12 = about 0.99
```

### Result

| Metric | Result | Reading |
|---|---|---|
| Manhattan | **7** | 7 units apart (block-wise) |
| Euclidean | **5** | 5 units apart (straight line) |
| Cosine | **about 0.99** | Almost the same direction |

**Takeaway:** Cosine says the vectors are almost identical because they point the same way. Manhattan and Euclidean say they are far apart because B is longer. For text, direction (meaning) usually matters more than length.

---

## 3. Comparison

| | Manhattan | Euclidean | Cosine |
|---|---|---|---|
| Also called | L1 distance, taxicab | L2 distance | Angular similarity |
| Measures | Sum of per-dimension differences | Straight-line gap | Angle between vectors |
| Affected by vector length? | Yes | Yes | **No** |
| Higher value means | Less similar | Less similar | **More similar** |
| Range | 0 to infinity | 0 to infinity | -1 to 1 |
| Typical use | Sparse or grid-like data | Clustering, image features | **Text embeddings, RAG** |

---

## 4. Python Code

```python
import numpy as np

A = np.array([1, 2])
B = np.array([4, 6])

manhattan = np.sum(np.abs(A - B))
euclidean = np.linalg.norm(A - B)
cosine = np.dot(A, B) / (np.linalg.norm(A) * np.linalg.norm(B))

print(manhattan)   # 7
print(euclidean)   # 5.0
print(cosine)      # 0.9923
```

---

## 5. Which One for RAG?

- **Cosine similarity is the default choice** for text embeddings, because meaning is captured by direction.
- If embeddings are **normalized** (length 1), cosine, dot product, and Euclidean all give the **same ranking** of results. Many embedding models return normalized vectors.
- Manhattan is rarely used in RAG.
- Use the metric your embedding model recommends, and set it when you create the vector store (FAISS, Chroma, Pinecone, and others each have a metric setting).

---

## 6. Interview Questions

**Q1. Difference between Euclidean and Manhattan distance?**
Euclidean is the straight-line distance (square root of squared differences). Manhattan is the sum of absolute differences along each dimension, like moving along a grid.

**Q2. Why is cosine preferred for text embeddings?**
It compares direction, not length. Two texts with similar meaning point the same way even if their vectors differ in magnitude.

**Q3. What does a cosine similarity of 0 mean?**
The vectors are unrelated (perpendicular). 1 means the same direction, -1 means opposite.

**Q4. What is the relation between cosine similarity and cosine distance?**
Cosine distance = 1 - cosine similarity. A smaller distance means more similar.

**Q5. When do cosine and Euclidean give the same ranking?**
When the vectors are normalized to length 1.

**Q6. Which metric would you choose in a RAG system?**
Cosine similarity, unless the embedding model documentation recommends another metric.

---

## 7. Quick Revision

- **Manhattan** = sum of absolute differences (city blocks)
- **Euclidean** = straight-line distance (ruler)
- **Cosine** = angle between vectors (direction only)
- Distance metrics: **smaller = more similar**. Cosine similarity: **larger = more similar**
- Example A = (1, 2), B = (4, 6): Manhattan **7**, Euclidean **5**, Cosine **0.99**
- RAG default: **cosine**
- Normalized vectors: cosine, dot product, and Euclidean rank the same
