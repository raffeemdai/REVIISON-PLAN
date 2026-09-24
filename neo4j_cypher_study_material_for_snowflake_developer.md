# Neo4j Cypher Study Material for a Snowflake Developer

https://graphacademy.neo4j.com/categories/become-certified#curriculum

refer  https://memgraph.com/blog/cypher-cheat-sheet
https://dev.to/mangesh28/mastering-neo4j-cypher-a-practical-guide-to-graph-query-language-448k


APOC:

https://neo4j.com/blog/developer/intro-user-defined-procedures-apoc/

https://neo4j.com/videos/?query=apoc

https://www.classcentral.com/classroom/youtube-neo4j-apoc-utility-library-howto-series-61357/61e627307ce8b

https://neo4j.com/labs/apoc/

https://www.youtube.com/playlist?list=PL9Hl4pk2FsvXEww23lDX_owoKoqqBQpdq

Graph data science:
https://graphacademy.neo4j.com/courses/gds-fundamentals

. GraphGists (community-contributed, hands-on tutorials with live/interactive graphs)


# Neo4j Fundamentals — Notes

# Neo4j Fundamentals — Notes

## 1. What is Neo4j?

- **Neo4j** is a **graph database** — data is stored as a graph made of **nodes** and **relationships**.
- Graph databases shine when the **connections between data** matter as much as the data itself.
- Neo4j uses a **labelled property graph** model.

### Core building blocks

| Element | What it is | Example |
|---|---|---|
| **Node** | A circle/vertex — usually represents an object or entity | A person, a company, a location |
| **Label** | Categorizes a node (what "type" it is) | `Person`, `Company`, `Location` |
| **Relationship** | A line/edge — describes how two nodes are connected | `WORKS_AT`, `FOUNDED_IN` |
| **Property** | Key–value data stored on nodes or relationships | `first name`, `last name`, `position` |

### Key details

- **Nodes** can have **multiple labels** (e.g. Michael is both `Person` and `Employee`).
- Labels are usually a single noun (`Person`, `Product`, `Event`) and let you **distinguish/filter** between different types of nodes.
- **Relationships** always have:
  - a **type** (e.g. `WORKS_AT`)
  - a **start node and an end node**
  - a **direction** (e.g. "Michael works at Neo4j" ≠ "Neo4j works at Michael")
- A node can have **multiple relationships**, and two relationships can represent a bidirectional connection (e.g. Michael `LOVES` Sarah / Sarah `LOVES` Michael) — you can't assume one direction implies the other.
- **Properties** have a data type (integer, boolean, string, list, etc.) and can act as unique identifiers (keys) for a node label.
- Nodes/relationships of the same type **don't need identical properties** — Neo4j is **schemaless**.

> **Takeaway:** Neo4j gives equal priority to *data* and *relationships*, unlike traditional databases.

### Example: nodes, labels, relationships & properties together

```
(Michael:Person:Employee {firstName: "Michael", lastName: "Faraday", born: "1791-09-22"})
    -[:WORKS_AT {position: "Engineer"}]-> (Neo4j:Company {name: "Neo4j", website: "neo4j.com"})

(Neo4j:Company) -[:FOUNDED_IN]-> (Sweden:Location {name: "Sweden", capital: "Stockholm"})

(Sarah:Person {firstName: "Sarah", lastName: "Faraday"})
```

- **Properties can live on relationships too** — here `WORKS_AT` carries a `position` property.
- This confirms nodes/relationships are flexible: `Michael` has `firstName`/`lastName`/`born`, while `Sarah` (also a `Person`) only has `firstName`/`lastName` — no need to match property sets.

### Naming conventions worth remembering

- **Node labels →** singular nouns: `Product`, `Event`, `Account`.
- **Relationship types →** verbs, describing:
  | Kind of connection | Example |
  |---|---|
  | Personal connection | `Person KNOWS Person`, `Person MARRIED_TO Person` |
  | A fact | `Person LIVES_IN Location`, `Person OWNS Car`, `Person RATED Movie` |
  | A hierarchy | `Parent PARENT_OF Child`, `Software DEPENDS_ON Library` |
  | Any generic connection | `Entity CONNECTED_TO Entity` |

---

## 2. Why Graphs? (Relational vs Graph)

- In **relational databases**, relationships are represented via **foreign keys and joins**, computed using indexes.
- Problem: as data grows, the index grows, and **joins get slower** (related to Big O notation) — especially with:
  - many-to-many relationships
  - hierarchical data / trees
  - paths of varying or unknown depth
  - constantly changing datasets

### NoSQL landscape (quick comparison)

| Type | Strength |
|---|---|
| Document stores | Flexibility |
| Wide-column stores | Scalability for large datasets |
| Key-value stores | Simplicity, high performance |
| **Graph databases** | Efficient modeling & querying of relationships |

### How graphs solve it

- When a relationship is created, Neo4j stores a **direct pointer** between the two nodes.
- Reading data means **following pointers in memory** instead of relying on an index.
- Result: **query time stays roughly constant**, regardless of overall database size.

### When to use a graph database

- Understanding relationships between entities
- Self-referencing data (hierarchies)
- Finding relationships of **varying/unknown depth**
- Calculating routes/paths between points in a network

---

## 3. Graphs Are Everywhere (History & Use Cases)

### Origin: Seven Bridges of Königsberg (1736)
- Classic graph theory problem: can you cross all 7 bridges of the city exactly once without retracing steps?
- **Euler** modeled land masses as **nodes** and bridges as **relationships/edges**.
- He proved it was impossible (nodes need an even number of edges for such a path) — this laid the foundation of **graph theory**.

### Real-world use cases

1. **Customer Recommendations**
   - Customers, products, categories connected via `PURCHASED` / `IN_CATEGORY`.
   - Pattern: "customers who bought similar products also bought X" → drives recommendations.

2. **Network & Security**
   - Devices, users, servers connected via `LOGGED_INTO` / `CONNECTED_TO`.
   - Helps detect **suspicious access patterns** (e.g. same user logging in from two places at once).

3. **Fraud Detection**
   - Transactions between accounts (`TRANSFERRED`) can form **cycles** (A → B → C → A).
   - Cycles are hard to spot in tables but obvious in a graph — a strong fraud signal (money laundering).

4. **Supply Chain**
   - Suppliers → Parts → Products (`SUPPLIES`, `USED_IN`).
   - A graph query can instantly show which products are affected if a supplier has a disruption.

5. **Knowledge Graphs & Generative AI**
   - AI agents need three types of memory, all representable as graphs:
     - **Short-term memory** — sequence of conversation messages
     - **Long-term memory** — knowledge graph of facts/entities across sessions
     - **Reasoning memory** — audit trail linking reasoning steps to tool calls/entities
   - Because all three live in one graph database, a single query can trace from a tool call → reasoning step → triggering message → referenced entity.
   - Neo4j + GenAI combines **vector search**, **knowledge graphs**, and **data science**.

---

## 4. Cypher Basics

- **Cypher** is Neo4j's query language for exploring/reading graph data.
- Example dataset used: a **movies graph** with `Person`, `Movie`, `Genre` nodes and relationships like `ACTED_IN`, `DIRECTED`, `IN_GENRE`.

### Example 1 — Find a node by property

```cypher
MATCH (p:Person {name: "Tom Hanks"})
RETURN p
```
- Returns the single `Person` node with `name = "Tom Hanks"`.
- Clicking the node shows its properties: `bio`, `born`, `bornIn`, `name`, etc.
- Double-clicking a node expands its relationships (e.g. `ACTED_IN` → movie nodes).

### Example 2 — Traverse a relationship

```cypher
MATCH (p:Person)-[:ACTED_IN]->(m:Movie {title: "Toy Story"})
RETURN p, m
```
- Finds all people who acted in the movie *Toy Story*.
- Returns a **graph** result: the movie node + all connected `ACTED_IN` relationships/person nodes.

### Example 3 — Traverse a different relationship type

```cypher
MATCH (m:Movie {title: "Toy Story"})-[:IN_GENRE]->(g:Genre)
RETURN m, g
```
- Finds the genres of *Toy Story* (e.g. Adventure, Animation, Children).

### Example 4 — Return tabular data instead of a graph

```cypher
MATCH (m:Movie {title: "Toy Story"})-[:IN_GENRE]->(g:Genre)
RETURN m.title, g.name
```
- Returning **specific properties** (instead of whole nodes) gives back a **table**, not a graph visualization.

---

## 5. Cypher in Neo4j Browser (UI) vs Python

The **Cypher language itself doesn't change** between the two — what changes is *how you send it and handle parameters*.

### In Neo4j Browser (UI)

You type raw Cypher directly and run it as-is:

```cypher
MATCH (p:Person {name: "Tom Hanks"})-[:ACTED_IN]->(m:Movie)
RETURN m.title
```

To use a parameter in the Browser, declare it with `:param`, then reference it with `$`:

```cypher
:param name => "Tom Hanks"

MATCH (p:Person {name: $name})-[:ACTED_IN]->(m:Movie)
RETURN m.title
```

### In Python (using the official `neo4j` driver)

You don't type Cypher into a shell — you send it as a **string** through a driver session, and pass parameters as a **separate dictionary/kwargs** (never string-concatenated, to avoid injection):

```python
from neo4j import GraphDatabase

driver = GraphDatabase.driver("bolt://localhost:7687", auth=("neo4j", "password"))

def get_movies(tx, actor_name):
    query = """
    MATCH (p:Person {name: $name})-[:ACTED_IN]->(m:Movie)
    RETURN m.title AS title
    """
    result = tx.run(query, name=actor_name)
    return [record["title"] for record in result]

with driver.session() as session:
    titles = session.execute_read(get_movies, "Tom Hanks")
    print(titles)

driver.close()
```

### Key differences

| | Neo4j Browser (UI) | Python |
|---|---|---|
| **Query text** | Same Cypher | Same Cypher (as a string) |
| **Running it** | Type & hit run | `session.run(query, **params)` or `tx.run(...)` |
| **Parameters** | `:param x => value` then `$x` | Passed as Python kwargs/dict, referenced as `$x` in the query |
| **Results** | Rendered as graph or table visually | Returned as `Record` objects — access fields like a dict (`record["title"]`) |
| **Connection** | Browser is already connected to a DB | You must open a `Driver`/`Session` yourself (`bolt://` URI + auth) |
| **Transactions** | Handled automatically per query you run | You choose `execute_read` / `execute_write` (or manual transactions) |

> **Takeaway:** The Cypher clauses (`MATCH`, `CREATE`, `RETURN`, `WHERE`, etc.) are identical in both. The only real "syntax" difference is that parameters are always referenced as `$paramName` — but in Python you *define* them as function arguments/dict keys instead of using the Browser's `:param` command.

---

## Quick Recap

- Graph = **nodes** (things) + **relationships** (connections) + **labels** (categories) + **properties** (data).
- Graph databases avoid the join/index slowdown of relational databases by storing direct pointers between related nodes.
- Graphs are useful anywhere relationships matter: recommendations, fraud detection, security, supply chains, AI knowledge graphs.
- **Cypher** = Neo4j's query language, using `MATCH` (find), `RETURN` (output), and arrow syntax `-[:TYPE]->` to describe patterns.

## 1. What is Neo4j?

- **Neo4j** is a **graph database** — data is stored as a graph made of **nodes** and **relationships**.
- Graph databases shine when the **connections between data** matter as much as the data itself.
- Neo4j uses a **labelled property graph** model.

### Core building blocks

| Element | What it is | Example |
|---|---|---|
| **Node** | A circle/vertex — usually represents an object or entity | A person, a company, a location |
| **Label** | Categorizes a node (what "type" it is) | `Person`, `Company`, `Location` |
| **Relationship** | A line/edge — describes how two nodes are connected | `WORKS_AT`, `FOUNDED_IN` |
| **Property** | Key–value data stored on nodes or relationships | `first name`, `last name`, `position` |

### Key details

- **Nodes** can have **multiple labels** (e.g. Michael is both `Person` and `Employee`).
- Labels are usually a single noun (`Person`, `Product`, `Event`) and let you **distinguish/filter** between different types of nodes.
- **Relationships** always have:
  - a **type** (e.g. `WORKS_AT`)
  - a **start node and an end node**
  - a **direction** (e.g. "Michael works at Neo4j" ≠ "Neo4j works at Michael")
- A node can have **multiple relationships**, and two relationships can represent a bidirectional connection (e.g. Michael `LOVES` Sarah / Sarah `LOVES` Michael) — you can't assume one direction implies the other.
- **Properties** have a data type (integer, boolean, string, list, etc.) and can act as unique identifiers (keys) for a node label.
- Nodes/relationships of the same type **don't need identical properties** — Neo4j is **schemaless**.

> **Takeaway:** Neo4j gives equal priority to *data* and *relationships*, unlike traditional databases.

### Example: nodes, labels, relationships & properties together

```
(Michael:Person:Employee {firstName: "Michael", lastName: "Faraday", born: "1791-09-22"})
    -[:WORKS_AT {position: "Engineer"}]-> (Neo4j:Company {name: "Neo4j", website: "neo4j.com"})

(Neo4j:Company) -[:FOUNDED_IN]-> (Sweden:Location {name: "Sweden", capital: "Stockholm"})

(Sarah:Person {firstName: "Sarah", lastName: "Faraday"})
```

- **Properties can live on relationships too** — here `WORKS_AT` carries a `position` property.
- This confirms nodes/relationships are flexible: `Michael` has `firstName`/`lastName`/`born`, while `Sarah` (also a `Person`) only has `firstName`/`lastName` — no need to match property sets.

### Naming conventions worth remembering

- **Node labels →** singular nouns: `Product`, `Event`, `Account`.
- **Relationship types →** verbs, describing:
  | Kind of connection | Example |
  |---|---|
  | Personal connection | `Person KNOWS Person`, `Person MARRIED_TO Person` |
  | A fact | `Person LIVES_IN Location`, `Person OWNS Car`, `Person RATED Movie` |
  | A hierarchy | `Parent PARENT_OF Child`, `Software DEPENDS_ON Library` |
  | Any generic connection | `Entity CONNECTED_TO Entity` |

---

## 2. Why Graphs? (Relational vs Graph)

- In **relational databases**, relationships are represented via **foreign keys and joins**, computed using indexes.
- Problem: as data grows, the index grows, and **joins get slower** (related to Big O notation) — especially with:
  - many-to-many relationships
  - hierarchical data / trees
  - paths of varying or unknown depth
  - constantly changing datasets

### NoSQL landscape (quick comparison)

| Type | Strength |
|---|---|
| Document stores | Flexibility |
| Wide-column stores | Scalability for large datasets |
| Key-value stores | Simplicity, high performance |
| **Graph databases** | Efficient modeling & querying of relationships |

### How graphs solve it

- When a relationship is created, Neo4j stores a **direct pointer** between the two nodes.
- Reading data means **following pointers in memory** instead of relying on an index.
- Result: **query time stays roughly constant**, regardless of overall database size.

### When to use a graph database

- Understanding relationships between entities
- Self-referencing data (hierarchies)
- Finding relationships of **varying/unknown depth**
- Calculating routes/paths between points in a network

---

## 3. Graphs Are Everywhere (History & Use Cases)

### Origin: Seven Bridges of Königsberg (1736)
- Classic graph theory problem: can you cross all 7 bridges of the city exactly once without retracing steps?
- **Euler** modeled land masses as **nodes** and bridges as **relationships/edges**.
- He proved it was impossible (nodes need an even number of edges for such a path) — this laid the foundation of **graph theory**.

### Real-world use cases

1. **Customer Recommendations**
   - Customers, products, categories connected via `PURCHASED` / `IN_CATEGORY`.
   - Pattern: "customers who bought similar products also bought X" → drives recommendations.

2. **Network & Security**
   - Devices, users, servers connected via `LOGGED_INTO` / `CONNECTED_TO`.
   - Helps detect **suspicious access patterns** (e.g. same user logging in from two places at once).

3. **Fraud Detection**
   - Transactions between accounts (`TRANSFERRED`) can form **cycles** (A → B → C → A).
   - Cycles are hard to spot in tables but obvious in a graph — a strong fraud signal (money laundering).

4. **Supply Chain**
   - Suppliers → Parts → Products (`SUPPLIES`, `USED_IN`).
   - A graph query can instantly show which products are affected if a supplier has a disruption.

5. **Knowledge Graphs & Generative AI**
   - AI agents need three types of memory, all representable as graphs:
     - **Short-term memory** — sequence of conversation messages
     - **Long-term memory** — knowledge graph of facts/entities across sessions
     - **Reasoning memory** — audit trail linking reasoning steps to tool calls/entities
   - Because all three live in one graph database, a single query can trace from a tool call → reasoning step → triggering message → referenced entity.
   - Neo4j + GenAI combines **vector search**, **knowledge graphs**, and **data science**.

---

## 4. Cypher Basics

- **Cypher** is Neo4j's query language for exploring/reading graph data.
- Example dataset used: a **movies graph** with `Person`, `Movie`, `Genre` nodes and relationships like `ACTED_IN`, `DIRECTED`, `IN_GENRE`.

### Example 1 — Find a node by property

```cypher
MATCH (p:Person {name: "Tom Hanks"})
RETURN p
```
- Returns the single `Person` node with `name = "Tom Hanks"`.
- Clicking the node shows its properties: `bio`, `born`, `bornIn`, `name`, etc.
- Double-clicking a node expands its relationships (e.g. `ACTED_IN` → movie nodes).

### Example 2 — Traverse a relationship

```cypher
MATCH (p:Person)-[:ACTED_IN]->(m:Movie {title: "Toy Story"})
RETURN p, m
```
- Finds all people who acted in the movie *Toy Story*.
- Returns a **graph** result: the movie node + all connected `ACTED_IN` relationships/person nodes.

### Example 3 — Traverse a different relationship type

```cypher
MATCH (m:Movie {title: "Toy Story"})-[:IN_GENRE]->(g:Genre)
RETURN m, g
```
- Finds the genres of *Toy Story* (e.g. Adventure, Animation, Children).

### Example 4 — Return tabular data instead of a graph

```cypher
MATCH (m:Movie {title: "Toy Story"})-[:IN_GENRE]->(g:Genre)
RETURN m.title, g.name
```
- Returning **specific properties** (instead of whole nodes) gives back a **table**, not a graph visualization.

---

## 5. Cypher in Neo4j Browser (UI) vs Python

The **Cypher language itself doesn't change** between the two — what changes is *how you send it and handle parameters*.

### In Neo4j Browser (UI)

You type raw Cypher directly and run it as-is:

```cypher
MATCH (p:Person {name: "Tom Hanks"})-[:ACTED_IN]->(m:Movie)
RETURN m.title
```

To use a parameter in the Browser, declare it with `:param`, then reference it with `$`:

```cypher
:param name => "Tom Hanks"

MATCH (p:Person {name: $name})-[:ACTED_IN]->(m:Movie)
RETURN m.title
```

### In Python (using the official `neo4j` driver)

You don't type Cypher into a shell — you send it as a **string** through a driver session, and pass parameters as a **separate dictionary/kwargs** (never string-concatenated, to avoid injection):

```python
from neo4j import GraphDatabase

driver = GraphDatabase.driver("bolt://localhost:7687", auth=("neo4j", "password"))

def get_movies(tx, actor_name):
    query = """
    MATCH (p:Person {name: $name})-[:ACTED_IN]->(m:Movie)
    RETURN m.title AS title
    """
    result = tx.run(query, name=actor_name)
    return [record["title"] for record in result]

with driver.session() as session:
    titles = session.execute_read(get_movies, "Tom Hanks")
    print(titles)

driver.close()
```

### Key differences

| | Neo4j Browser (UI) | Python |
|---|---|---|
| **Query text** | Same Cypher | Same Cypher (as a string) |
| **Running it** | Type & hit run | `session.run(query, **params)` or `tx.run(...)` |
| **Parameters** | `:param x => value` then `$x` | Passed as Python kwargs/dict, referenced as `$x` in the query |
| **Results** | Rendered as graph or table visually | Returned as `Record` objects — access fields like a dict (`record["title"]`) |
| **Connection** | Browser is already connected to a DB | You must open a `Driver`/`Session` yourself (`bolt://` URI + auth) |
| **Transactions** | Handled automatically per query you run | You choose `execute_read` / `execute_write` (or manual transactions) |

> **Takeaway:** The Cypher clauses (`MATCH`, `CREATE`, `RETURN`, `WHERE`, etc.) are identical in both. The only real "syntax" difference is that parameters are always referenced as `$paramName` — but in Python you *define* them as function arguments/dict keys instead of using the Browser's `:param` command.

---

## Quick Recap

- Graph = **nodes** (things) + **relationships** (connections) + **labels** (categories) + **properties** (data).
- Graph databases avoid the join/index slowdown of relational databases by storing direct pointers between related nodes.
- Graphs are useful anywhere relationships matter: recommendations, fraud detection, security, supply chains, AI knowledge graphs.
- **Cypher** = Neo4j's query language, using `MATCH` (find), `RETURN` (output), and arrow syntax `-[:TYPE]->` to describe patterns.
## 1. How to Think About Neo4j if You Know Snowflake

If you are coming from Snowflake, the easiest mental model is:

| Snowflake / Relational idea | Neo4j idea |
|---|---|
| Table | Label / group of nodes |
| Row | Node |
| Column | Property |
| Primary/foreign-key join | Relationship |
| JOIN | Graph traversal |
| WHERE | WHERE |
| SELECT | RETURN |
| GROUP BY | Aggregation with RETURN/WITH |
| CTE / intermediate query step | WITH |
| MERGE / UPSERT idea | MERGE |
| Index | Index |
| Unique / not-null constraints | Neo4j constraints |
| Hierarchical joins | Relationship paths |

### Core mental shift

In Snowflake, you often think:

```sql
SELECT ...
FROM customer c
JOIN orders o ON ...
JOIN product p ON ...
```

In Neo4j, you think in a connected pattern:

```cypher
MATCH (c:Customer)-[:PLACED]->(o:Order)-[:CONTAINS]->(p:Product)
RETURN c, o, p
```

### Memory trick

**SQL = tables first, relationships later.**  
**Neo4j = relationships are part of the data model itself.**

---

# 2. Property Graph Basics

A Neo4j property graph is made of:

- **Nodes**
- **Relationships**
- **Labels**
- **Properties**

Example:

```cypher
(:Person {name: "Tom Hanks"})-[:ACTED_IN]->(:Movie {title: "Toy Story"})
```

Here:

- `Person` = label
- `Movie` = label
- `name`, `title` = properties
- `ACTED_IN` = relationship type

## Memory trick

Think:

**N-L-P-R**

- **N**odes
- **L**abels
- **P**roperties
- **R**elationships

---

# 3. Basic Cypher Pattern Syntax

## Nodes

Nodes are written with parentheses:

```cypher
(p:Person)
```

## Relationships

Relationships are written with square brackets:

```cypher
[:ACTED_IN]
```

## Direction

```cypher
(p:Person)-[:ACTED_IN]->(m:Movie)
```

means:

> Person acted in Movie.

## Undirected pattern

```cypher
(a)--(b)
```

matches a relationship in either direction.

## Memory trick

- `()` = node
- `[]` = relationship
- `->` = direction

---

# 4. MATCH

`MATCH` is similar to identifying rows with `FROM` + joins in SQL.

```cypher
MATCH (p:Person)
RETURN p
```

With a relationship:

```cypher
MATCH (p:Person)-[:ACTED_IN]->(m:Movie)
RETURN p.name, m.title
```

---

# 5. WHERE Filtering

```cypher
MATCH (m:Movie)
WHERE m.released > 2010
RETURN m
```

Inclusive range:

```cypher
MATCH (m:Movie)
WHERE m.released >= 2000
  AND m.released <= 2005
RETURN m
```

Membership:

```cypher
MATCH (p:Person)
WHERE p.born IN [1970, 1980, 1990]
RETURN p.name, p.born
```

Check that a property exists:

```cypher
WHERE p.name IS NOT NULL
```

## Memory trick

`IN` = "is one of these values"

`IS NOT NULL` = property has a value

---

# 6. RETURN

`RETURN` is like SQL `SELECT`.

```cypher
MATCH (p:Person)
RETURN p.name
```

Alias:

```cypher
RETURN p.name AS person
```

---

# 7. DISTINCT

Remove duplicates:

```cypher
MATCH (p:Person)-[:ACTED_IN]->(m:Movie)
RETURN DISTINCT p.name
```

Memory trick:

**SQL DISTINCT = Cypher DISTINCT**

---

# 8. ORDER BY and LIMIT

```cypher
MATCH (p:Person)-[:ACTED_IN]->(m:Movie)
RETURN p.name AS actor, count(m) AS movies
ORDER BY movies DESC, actor ASC
LIMIT 1
```

Useful pattern:

```text
MATCH
RETURN aggregate
ORDER BY
LIMIT
```

---

# 9. Aggregation

Common functions:

```cypher
count()
collect()
sum()
avg()
min()
max()
```

Example:

```cypher
MATCH (p:Person)-[:ACTED_IN]->(m:Movie)
RETURN p.name, count(m) AS movieCount
```

Collect into a list:

```cypher
MATCH (n:Person {name: 'Tom Cruise'})-[:ACTED_IN]->(m:Movie)
RETURN collect(m.title) AS movies
```

---

# 10. WITH

`WITH` divides a query into stages and passes selected variables forward.

Think of it like a lightweight CTE / pipeline boundary.

```cypher
MATCH (p:Person)
WITH p
WHERE p.born > 1970
RETURN p.name
```

Aggregation example:

```cypher
MATCH (m:Movie)
WITH count(m) AS totalMovies
MATCH (c:Movie)-[:IN_GENRE]->(:Genre {name: 'Comedy'})
RETURN totalMovies, count(c)
```

## Memory trick

**WITH = pass this result to the next part.**

---

# 11. CREATE vs MERGE

## CREATE

Always creates new data.

```cypher
CREATE (:Person {name: "Alice"})
```

## MERGE

Matches the pattern if it exists; otherwise creates it.

```cypher
MERGE (p:Person {name: "Alice"})
```

Relationship:

```cypher
MATCH (a:Person {name: "Alice"})
MATCH (b:Person {name: "Bob"})
MERGE (a)-[:KNOWS]->(b)
```

## Memory trick

- `CREATE` = always insert
- `MERGE` = match-or-create

Snowflake analogy: `MERGE` is conceptually similar to an upsert, although graph pattern semantics differ.

---

# 12. ON CREATE and ON MATCH

```cypher
MERGE (m:Movie {title: "Toy Story"})
ON CREATE SET m.released = 1995
ON MATCH SET m.tagline = "The adventure takes off!"
SET m.imdbRating = 8.3
```

On an empty database:

- `title` exists
- `released` exists
- `imdbRating` exists
- `tagline` does not exist because `ON MATCH` did not run

---

# 13. DELETE and DETACH DELETE

Delete a node with no relationships:

```cypher
MATCH (p:Person {name: "Tom"})
DELETE p
```

Delete a node and connected relationships:

```cypher
MATCH (p:Person {name: "Tom"})
DETACH DELETE p
```

Memory trick:

**DETACH DELETE = disconnect + delete**

---

# 14. UNION and UNION ALL

`UNION` removes duplicates.

```cypher
RETURN "A" AS value
UNION
RETURN "A" AS value
```

Result:

```text
A
```

`UNION ALL` keeps duplicates.

```cypher
RETURN "A" AS value
UNION ALL
RETURN "A" AS value
```

Result:

```text
A
A
```

Same concept as SQL.

---

# 15. Relationship Uniqueness

Within one `MATCH` clause, the same relationship cannot be reused in the same matched path/pattern.

Example:

```cypher
MATCH (p:Person)-[:ACTED_IN]->(m),
      (m)<-[:ACTED_IN]-(p2)
RETURN p2.name
```

The two relationship matches in the same `MATCH` clause must be different relationship instances.

With separate `MATCH` clauses:

```cypher
MATCH (p:Person)-[:ACTED_IN]->(m)
MATCH (m)<-[:ACTED_IN]-(p2)
RETURN p2.name
```

the matching scope differs.

## Memory trick

**Same MATCH = relationships are unique inside that match.**

---

# 16. Subqueries

## CALL subquery

Old/import style may use `WITH`, while newer Cypher supports scope variables:

```cypher
MATCH (a:Person)
WHERE a.name CONTAINS "Tom"
WITH a, a.name AS actorName
CALL (a) {
    MATCH (a)-[:ACTED_IN]->(m:Movie)
    RETURN collect(m.title) AS movies
}
RETURN actorName, movies
```

Memory trick:

**Outer variable needed inside? Pass it into CALL.**

---

# 17. EXISTS Subquery

Return only people who directed at least one movie:

```cypher
MATCH (p:Person)
WHERE EXISTS {
    MATCH (p)-[:DIRECTED]->(:Movie)
}
RETURN p.name
```

Compact pattern form accepted in modern Cypher:

```cypher
MATCH (p:Person)
WHERE EXISTS { (p)-[:DIRECTED]->(:Movie) }
RETURN p.name
```

---

# 18. COUNT Subquery

Count pattern matches inline:

```cypher
MATCH (p:Person)
RETURN p.name,
       COUNT { (p)-[:ACTED_IN]->(:Movie) } AS movies
```

Memory trick:

**COUNT { pattern } = count how many times this pattern exists.**

---

# 19. LOAD CSV

With headers:

```cypher
LOAD CSV WITH HEADERS FROM 'file:///people.csv' AS row
RETURN row.name
```

Without headers:

```cypher
LOAD CSV FROM 'file:///people.csv' AS row
RETURN row[0], row[1]
```

Without `WITH HEADERS`, each row is treated as a list.

## Data typing

CSV values are read as strings, so convert when necessary:

```cypher
toInteger(row.age)
toFloat(row.rating)
toBoolean(row.active)
```

Example:

```cypher
LOAD CSV WITH HEADERS FROM 'file:///movies.csv' AS row
CREATE (:Movie {
    title: row.title,
    released: toInteger(row.released),
    rating: toFloat(row.rating)
})
```

---

# 20. Batched LOAD CSV

```cypher
LOAD CSV WITH HEADERS FROM 'file:///people.csv' AS row
CALL (row) {
    MERGE (p:Person {id: row.id})
    SET p += row
} IN TRANSACTIONS OF 1000 ROWS
```

For 500 rows:

```cypher
LOAD CSV WITH HEADERS FROM 'file:///people.csv' AS row
CALL (row) {
    WITH row
    MERGE (p:Person {id: row.id})
} IN TRANSACTIONS OF 500 ROWS
```

Memory trick:

**CALL (...) { ... } IN TRANSACTIONS OF N ROWS**

---

# 21. Constraints

## Existence constraint on relationship property

```cypher
CREATE CONSTRAINT rated_rating
FOR ()-[r:RATED]-()
REQUIRE r.rating IS NOT NULL
```

## Node key constraint

A Node Key Constraint ensures a combination of properties:

- exists
- is unique together

Example:

```cypher
CREATE CONSTRAINT person_key
FOR (p:Person)
REQUIRE (p.firstName, p.lastName) IS NODE KEY
```

---

# 22. Indexes

Common index types discussed in Neo4j:

- RANGE
- TEXT
- VECTOR
- FULL-TEXT

## Vector index

```cypher
CREATE VECTOR INDEX moviePlots IF NOT EXISTS
FOR (m:Movie) ON m.embedding
OPTIONS {indexConfig: {
  `vector.dimensions`: 1536,
  `vector.similarity_function`: 'cosine'
}}
```

## Full-text query

```cypher
CALL db.index.fulltext.queryNodes("movieInfo", $query)
YIELD node, score
RETURN node.title, score
```

Full-text and vector indexes are specialized indexes and are not used like ordinary property indexes in standard pattern matching.

---

# 23. EXPLAIN vs PROFILE

## EXPLAIN

Shows the execution plan without executing the query.

Think:

**Estimate / plan**

## PROFILE

Runs the query and shows actual execution information.

Think:

**Actual rows / actual work**

Memory trick:

- **EXPLAIN = expected**
- **PROFILE = performed**

---

# 24. Query Cache

To improve query-plan reuse:

- use parameters
- keep query structure consistent
- keep keyword/parameter casing consistent when identical query text matters

Prefer:

```cypher
MATCH (p:Person {name: $name})
RETURN p
```

over repeatedly embedding different literals:

```cypher
MATCH (p:Person {name: "Tom"})
RETURN p
```

---

# 25. Case Sensitivity

Cypher keywords are case-insensitive:

```cypher
MATCH
match
Match
```

are treated as keywords.

However:

- variable names are case-sensitive
- labels are case-sensitive
- property keys are case-sensitive

Example:

```cypher
p.name
```

is not necessarily the same property as:

```cypher
p.Name
```

---

# 26. Escaping Identifiers

Use backticks for names containing spaces or special characters:

```cypher
CREATE USER `john smith` SET PASSWORD 'password'
```

Memory trick:

**Identifier with awkward characters? Use backticks.**

---

# 27. Drivers

A Neo4j driver connects an application to Neo4j.

Typical flow:

```text
Application
   ↓
Neo4j Driver
   ↓
Neo4j Database
```

Official Neo4j drivers use the **Bolt** protocol.

Secure connection schemes include:

```text
neo4j+s
bolt+s
```

---

# 28. Python Driver

Create a driver:

```python
from neo4j import GraphDatabase

driver = GraphDatabase.driver(
    "neo4j://localhost:7687",
    auth=("neo4j", "letmein")
)
```

## Context manager

```python
with GraphDatabase.driver(
    "neo4j://localhost:7687",
    auth=("neo4j", "letmein")
) as driver:
    pass
```

Inside a context manager, you do not need to manually call:

```python
driver.close()
```

---

# 29. execute_query()

Execute a one-off query and convert results to a Pandas DataFrame:

```python
from neo4j import GraphDatabase
from neo4j import Result

driver = GraphDatabase.driver(
    "neo4j://localhost:7687",
    auth=("neo4j", "letmein")
)

df = driver.execute_query(
    "MATCH (n) RETURN count(n) AS count",
    result_transformer_=Result.to_df
)
```

---

# 30. Driver Results

A query result is a stream of **records**.

Think:

Snowflake result set row ≈ Neo4j driver record.

---

# 31. Driver Transaction Types

Neo4j driver transaction styles include:

- transaction functions
- explicit transactions
- auto-commit transactions

Transaction functions can automatically retry supported transient failures.

Examples in Python:

```python
session.execute_read(...)
session.execute_write(...)
```

---

# 32. DateTime Import

```python
from neo4j.time import DateTime
```

---

# 33. Graph Data Modeling

## When to refactor

Refactor when:

- performance is poor
- query patterns are awkward
- the model no longer fits the business requirements

Neo4j data models are intentionally flexible.

---

# 34. Intermediate Nodes

Suppose you start with:

```text
(Customer)-[:ORDERED]->(Product)
```

You may refactor to:

```text
(Customer)-[:PLACED]->(Order)-[:CONTAINS]->(Product)
```

Now Order can connect to:

- products
- shipping company
- payment
- address
- promotions

and store:

- order date
- order status
- total amount
- tracking number

Memory trick:

**If the relationship itself becomes a business entity, turn it into a node.**

---

# 35. Modeling Categories

Instead of adding many labels:

```text
:Product_Apple
:Product_Samsung
:Product_Keyboards
```

create category nodes:

```text
(:Category {name:"Computers & Accessories"})
   ↓
(:Category {name:"Computers & Laptops"})
   ↓
(:Category {name:"Apple"})
```

and connect products:

```cypher
(:Product)-[:IN_CATEGORY]->(:Category)
```

This is more flexible for recommendation systems.

---

# 36. Labels

A node can have multiple labels:

```cypher
(p:Person:Customer)
```

Course-oriented rule of thumb from the material:

**Keep labels limited and purposeful; around 4 or fewer is a common modeling guideline.**

---

# 37. Graph vs Chart

A graph database "graph" is not a reporting chart.

It is a data structure made of:

- nodes
- relationships
- properties
- labels

---

# 38. ACID

Neo4j supports ACID transactions:

- **Atomicity**
- **Consistency**
- **Isolation**
- **Durability**

Memory trick:

**A-C-I-D = all changes are trustworthy as a transaction.**

---

# 39. Schema

Neo4j uses a schema-optional / schema-flexible model.

You do not need to define a rigid table schema first.

You can later add:

- constraints
- indexes
- labels
- relationship types

---

# 40. Performance Tips

Avoid overly broad patterns like:

```cypher
MATCH (n)--(m)
RETURN *
```

Prefer specific labels, relationship types, and directions:

```cypher
MATCH (p:Person)-[:ACTED_IN]->(m:Movie)
RETURN p, m
```

Why?

Because broad patterns can expand many relationships and generate large intermediate result sets.

## Memory trick

**Anchor early. Expand narrowly. Return only what you need.**

---

# 41. Common Snowflake-to-Neo4j Translation Patterns

## Find a person by name

Snowflake:

```sql
SELECT *
FROM person
WHERE name = 'Tom Hanks';
```

Neo4j:

```cypher
MATCH (p:Person {name: 'Tom Hanks'})
RETURN p
```

## Find actor's movies

Snowflake:

```sql
SELECT m.title
FROM person p
JOIN acted_in ai ON ...
JOIN movie m ON ...
WHERE p.name = 'Tom Hanks';
```

Neo4j:

```cypher
MATCH (:Person {name:'Tom Hanks'})-[:ACTED_IN]->(m:Movie)
RETURN m.title
```

## Count movies per actor

Snowflake:

```sql
SELECT actor_name, COUNT(*)
FROM ...
GROUP BY actor_name;
```

Neo4j:

```cypher
MATCH (a:Person)-[:ACTED_IN]->(m:Movie)
RETURN a.name, count(m)
```

---

# 42. Practical Query Patterns from Your Practice

## Most prolific actor

```cypher
MATCH (a:Person)-[:ACTED_IN]->(m:Movie)
RETURN a.name AS name, count(m) AS movieCount
ORDER BY movieCount DESC, name ASC
LIMIT 1
```

## Most prolific recent director

```cypher
MATCH (d:Person)-[:DIRECTED]->(m:Movie)
WHERE m.released >= 2010
RETURN d.name AS name, count(m) AS movieCount
ORDER BY movieCount DESC, name ASC
LIMIT 1
```

## Highest-profit movie

```cypher
MATCH (m:Movie)
WHERE m.revenue IS NOT NULL
  AND m.budget IS NOT NULL
RETURN m.title AS title
ORDER BY (m.revenue - m.budget) DESC
LIMIT 1
```

## Comedy percentage

```cypher
MATCH (m:Movie)
WITH count(m) AS totalMovies
MATCH (c:Movie)-[:IN_GENRE]->(:Genre {name: 'Comedy'})
WITH totalMovies, count(DISTINCT c) AS comedyMovies
RETURN round(100.0 * comedyMovies / totalMovies, 2) AS percentage
```

## Two degrees of separation

```cypher
MATCH (a:Person {name: 'Albert Austin'})
      -[:ACTED_IN]->(:Movie)<-[:ACTED_IN]-(p1:Person)
MATCH (p1)-[:ACTED_IN]->(:Movie)<-[:ACTED_IN]-(p2:Person)
WHERE p2 <> a
  AND NOT EXISTS {
    MATCH (a)-[:ACTED_IN]->(:Movie)<-[:ACTED_IN]-(p2)
  }
RETURN count(DISTINCT p2) AS count
```

---

# 43. Memory Tricks Cheat Sheet

| Topic | Memory trick |
|---|---|
| Node | Parentheses `()` |
| Relationship | Square brackets `[]` |
| Direction | Arrow `->` |
| MATCH | Find a graph pattern |
| WHERE | Filter |
| RETURN | SELECT |
| WITH | Pass intermediate results |
| CREATE | Always insert |
| MERGE | Match-or-create |
| DISTINCT | Remove duplicates |
| collect() | Make a list |
| count() | Count rows/patterns |
| EXISTS { } | Pattern exists? |
| COUNT { } | How many pattern matches? |
| DETACH DELETE | Remove node + relationships |
| EXPLAIN | Expected plan |
| PROFILE | Performed plan |
| LOAD CSV | Import CSV |
| IN TRANSACTIONS | Batch import |
| Bolt | Driver protocol |
| `neo4j+s` | Secure routed connection |
| `bolt+s` | Secure direct Bolt connection |
| `IS NOT NULL` | Property exists |
| Node Key | Exists + unique combination |
| Full-text | Text search |
| Vector | Embedding similarity |
| `WITH HEADERS` | Use CSV column names |
| No headers | Row becomes a list |
| Driver record | One result row |

---

# 44. Practice Questions and Answers — Multiple Choice Edition

These 90 questions are based on the Neo4j/Cypher questions you practiced. For multi-select questions, choose every correct option.

## Q1. True or False: Relationships in a graph are treated with the same importance as nodes.

- **A.** True
- **B.** False

**Answer:** **A — True**
## Q2. Related to relational databases, what does the O(n) problem mean in the course context?

- **A.** Increasing response times as database indexes/data grow larger
- **B.** The need to frequently back up data
- **C.** Difficulty implementing security protocols
- **D.** The challenge of maintaining data integrity during updates

**Answer:** **A — Increasing response times as database indexes/data grow larger**
## Q3. Why can Tom Hanks appear in the two-MATCH version but not in the single-MATCH version of a coactor query?

- **A.** Relationships are unique within the scope of a single MATCH clause only
- **B.** The import created two Tom Hanks nodes
- **C.** It would never happen
- **D.** It is a Neo4j bug

**Answer:** **A — Relationships are unique within the scope of a single MATCH clause only**
## Q4. What is the difference between EXPLAIN and PROFILE?

- **A.** EXPLAIN provides exact rows while PROFILE provides estimates
- **B.** EXPLAIN is for updates and PROFILE for reads
- **C.** EXPLAIN provides estimates while PROFILE runs the query and shows actual execution details
- **D.** There is no difference

**Answer:** **C — EXPLAIN provides estimates while PROFILE runs the query and shows actual execution details**
## Q5. Which statements about case sensitivity in Cypher are true? Select all that apply.

- **A.** Labels and property keys are case-sensitive
- **B.** All Cypher queries must be lowercase
- **C.** Cypher keywords are case-sensitive
- **D.** Variables are case-sensitive

**Answer:** **A, D** — Labels and property keys are case-sensitive; Variables are case-sensitive
## Q6. Which query counts Actor nodes?

- **A.** MATCH (a:Actor) RETURN count(a) AS actorCount
- **B.** MATCH Actor RETURN count(*)
- **C.** SELECT COUNT(*) FROM Actor
- **D.** MATCH (a) WHERE a='Actor' RETURN count(a)

**Answer:** **A — MATCH (a:Actor) RETURN count(a) AS actorCount**
## Q7. How can you delete a specific node in Neo4j using Cypher?

- **A.** Use MERGE
- **B.** There is no way
- **C.** Use MATCH to find the node and then DELETE it
- **D.** Use RETURN DELETE

**Answer:** **C — Use MATCH to find the node and then DELETE it**
## Q8. When should DETACH DELETE be used?

- **A.** When deleting a node that has relationships
- **B.** Only when deleting indexes
- **C.** Only for labels
- **D.** Never

**Answer:** **A — When deleting a node that has relationships**
## Q9. What is the difference between UNION and UNION ALL?

- **A.** UNION returns distinct results; UNION ALL keeps duplicates
- **B.** UNION ALL returns distinct results; UNION keeps duplicates
- **C.** They are identical
- **D.** UNION is only for writes

**Answer:** **A — UNION returns distinct results; UNION ALL keeps duplicates**
## Q10. How can you escape special characters such as spaces in database, user, and role names?

- **A.** Single quotes
- **B.** Backslashes
- **C.** Backticks
- **D.** Double quotes

**Answer:** **C — Backticks**
## Q11. Why is Al Pacino not returned as his own co-star in a single MATCH pattern?

- **A.** Relationship uniqueness
- **B.** The query compares names
- **C.** Al Pacino does not exist
- **D.** Cypher excludes duplicate names automatically

**Answer:** **A — Relationship uniqueness**
## Q12. What is the role of a driver in Neo4j?

- **A.** Establish a connection with the Neo4j database
- **B.** Define the graph model
- **C.** Store data inside Neo4j
- **D.** Visualize nodes

**Answer:** **A — Establish a connection with the Neo4j database**
## Q13. Complete the existence constraint: REQUIRE r.rating ...

- **A.** EXISTS
- **B.** IS UNIQUE
- **C.** IS NOT NULL
- **D.** :: NOT NULL

**Answer:** **C — IS NOT NULL**
## Q14. What makes full-text and vector indexes different from ordinary indexes in the course material?

- **A.** They constrain relationship uniqueness
- **B.** They must always be queried by procedures only
- **C.** They are not automatically used by the query planner for ordinary pattern matching
- **D.** They can only index numbers

**Answer:** **C — They are not automatically used by the query planner for ordinary pattern matching**
## Q15. Which fragment completes a batched import of 500 rows?

- **A.** IN TRANSACTIONS OF 500 ROWS
- **B.** COMMIT EVERY 500
- **C.** BATCH SIZE 500
- **D.** ROWS=500

**Answer:** **A — IN TRANSACTIONS OF 500 ROWS**
## Q16. On an empty database, which properties exist after MERGE title, ON CREATE released, ON MATCH tagline, and unconditional SET imdbRating?

- **A.** title, tagline
- **B.** title, released, imdbRating
- **C.** released, tagline
- **D.** title only

**Answer:** **B — title, released, imdbRating**
## Q17. When using the Python driver within a context manager, is driver.close() required?

- **A.** True
- **B.** False

**Answer:** **B — False**
## Q18. When is it a good idea to refactor a data model?

- **A.** When it has more than 4 relationships
- **B.** When it is not performant or does not meet business requirements
- **C.** Never
- **D.** Only after deployment

**Answer:** **B — When it is not performant or does not meet business requirements**
## Q19. True or False: A graph is simply a reporting chart.

- **A.** True
- **B.** False

**Answer:** **B — False**
## Q20. Which WHERE clause finds movies released from 2000 through 2005 inclusive?

- **A.** WHERE m.released > 2000 AND m.released < 2005
- **B.** WHERE m.released BETWEEN 2000 AND 2005
- **C.** WHERE m.released = [2000,2001,2002,2003,2004,2005]
- **D.** WHERE m.released >= 2000 AND m.released <= 2005

**Answer:** **D — WHERE m.released >= 2000 AND m.released <= 2005**
## Q21. How do you ensure the same relationship pattern is not created twice?

- **A.** UNIQUE
- **B.** DISTINCT
- **C.** MERGE
- **D.** ONLY

**Answer:** **C — MERGE**
## Q22. Which statement finds all movies released after 2010?

- **A.** MATCH (m:Movie) WHERE m.released > 2010 RETURN m
- **B.** WHERE m.released > 2010 WITH (m:Movie) RETURN m
- **C.** RETURN (m:Movie) WITH m.title WHERE m.released > 2010
- **D.** MATCH (m:Movie) WHERE released > 2010 RETURN m

**Answer:** **A — MATCH (m:Movie) WHERE m.released > 2010 RETURN m**
## Q23. Which keyword starts creation of a vector index?

- **A.** CREATE VECTOR INDEX
- **B.** CREATE INDEX
- **C.** CREATE CONSTRAINT
- **D.** CALL db.vector.createIndex

**Answer:** **A — CREATE VECTOR INDEX**
## Q24. If WITH HEADERS is omitted from LOAD CSV, what happens?

- **A.** Each row is treated as a list
- **B.** The database crashes
- **C.** An LLM generates headers
- **D.** Syntax error

**Answer:** **A — Each row is treated as a list**
## Q25. What does IN do in `WHERE p.born IN [1970,1980,1990]`?

- **A.** Finds movies released in those years
- **B.** Finds Person nodes born in one of those years
- **C.** Finds all nodes created between those years
- **D.** Invalid syntax

**Answer:** **B — Finds Person nodes born in one of those years**
## Q26. Which is the correct Person-to-Movie relationship pattern?

- **A.** (Person)-[Movie]->[Person]->(Movie)
- **B.** (p:Person)-[:ACTED_IN]->(m:Movie)
- **C.** (Person)<-[:ACTED_IN]-(Movie)
- **D.** Person ACTED_IN Movie

**Answer:** **B — (p:Person)-[:ACTED_IN]->(m:Movie)**
## Q27. True or False: Neo4j supports ACID transactions.

- **A.** True
- **B.** False

**Answer:** **A — True**
## Q28. Which statement is correct about schema in Neo4j?

- **A.** Neo4j allows an optional/flexible schema
- **B.** Neo4j does not support schema-related features
- **C.** Neo4j enforces a strict relational schema
- **D.** Schema can never change

**Answer:** **A — Neo4j allows an optional/flexible schema**
## Q29. What was the course-style recommended limit for labels per node?

- **A.** 6
- **B.** 8
- **C.** 4
- **D.** 2

**Answer:** **C — 4**
## Q30. Which procedure queries a full-text index called movieInfo?

- **A.** CALL db.index.fulltext.queryNodes
- **B.** QUERY FULLTEXT INDEX WITH
- **C.** SELECT INDEX
- **D.** MATCH fulltext

**Answer:** **A — CALL db.index.fulltext.queryNodes**
## Q31. What can help narrow a broad pattern like MATCH (n)--(m)? Select all that apply.

- **A.** Define a relationship type
- **B.** Define a label for an anchor node
- **C.** Define a direction
- **D.** Remove all labels

**Answer:** **A, B, C** — Define a relationship type; Define a label for an anchor node; Define a direction
## Q32. Which queries correctly find people who acted in Toy Story? Select all that apply.

- **A.** MATCH (p:Person)-[:ACTED_IN]->(m:Movie) WHERE m.title='Toy Story' RETURN p
- **B.** SELECT path=(p:Person)-[:ACTED_IN]->(m:Movie {title:'Toy Story'}) RETURN path
- **C.** MATCH (p:Person)-[:ACTED_IN]->(m:Movie {title:'Toy Story'}) RETURN p
- **D.** MATCH people -> acted in -> movie(toy story)

**Answer:** **A, C** — MATCH (p:Person)-[:ACTED_IN]->(m:Movie) WHERE m.title='Toy Story' RETURN p; MATCH (p:Person)-[:ACTED_IN]->(m:Movie {title:'Toy Story'}) RETURN p
## Q33. How do you pass p into a CALL subquery?

- **A.** CALL (p) {
- **B.** HAVING p {
- **C.** USING p {
- **D.** SELECT p FROM $mainQuery {

**Answer:** **A — CALL (p) {**
## Q34. Which transaction form is automatically retried on eligible transient failures by Neo4j drivers?

- **A.** Transaction functions
- **B.** Auto-commit transactions
- **C.** Explicit transactions
- **D.** None

**Answer:** **A — Transaction functions**
## Q35. What is the primary purpose of WITH?

- **A.** Filter before returning only
- **B.** Create nodes
- **C.** Divide a query into parts and pass intermediate results forward
- **D.** Select a database

**Answer:** **C — Divide a query into parts and pass intermediate results forward**
## Q36. A statement of results from a language driver comprises a stream of...

- **A.** entries
- **B.** records
- **C.** rows only
- **D.** nodes

**Answer:** **B — records**
## Q37. Which constraint ensures two or more properties exist and are unique together?

- **A.** Unique Constraint
- **B.** NodePropertyConst
- **C.** Combination Constraint
- **D.** Node Key Constraint

**Answer:** **D — Node Key Constraint**
## Q38. What protocol do official Neo4j drivers use?

- **A.** TLS
- **B.** NFJR
- **C.** RMI
- **D.** Bolt

**Answer:** **D — Bolt**
## Q39. Why avoid unnecessary labels on non-anchor nodes?

- **A.** They are not allowed
- **B.** Licensing forbids them
- **C.** They reduce WHERE clauses
- **D.** They force label checks that may be unnecessary

**Answer:** **D — They force label checks that may be unnecessary**
## Q40. Complete `WHERE ___ { (p)-[:DIRECTED]->(:Movie) }`.

- **A.** EXISTS
- **B.** CONTAINS
- **C.** HAS
- **D.** MATCH

**Answer:** **A — EXISTS**
## Q41. True or False: An index can improve data retrieval and query performance.

- **A.** True
- **B.** False

**Answer:** **A — True**
## Q42. Which method executes a one-off Cypher query in the Python driver example?

- **A.** cypher
- **B.** to_df
- **C.** execute_query
- **D.** runquery

**Answer:** **C — execute_query**
## Q43. What is wrong with a CALL subquery that uses outer variable a without importing it?

- **A.** GraphQL is required
- **B.** WITH after CALL is mandatory
- **C.** Only scalar values can enter subqueries
- **D.** You must pass/import a into the subquery

**Answer:** **D — You must pass/import a into the subquery**
## Q44. When is a bi-directional modeling approach appropriate?

- **A.** When semantics differ by direction
- **B.** Never
- **C.** Whenever a relationship could be traversed either way
- **D.** To duplicate the same meaning in both directions

**Answer:** **A — When semantics differ by direction**
## Q45. Which elements make up a property graph? Select all that apply.

- **A.** Rows
- **B.** Labels
- **C.** Foreign keys
- **D.** Collections
- **E.** Properties
- **F.** Nodes
- **G.** Relationships

**Answer:** **B, E, F, G** — Labels; Properties; Nodes; Relationships
## Q46. In `MATCH p = ()-->() RETURN p`, what do the parentheses represent?

- **A.** Nodes
- **B.** Tables
- **C.** Labels
- **D.** Relationships

**Answer:** **A — Nodes**
## Q47. True or False: A Neo4j schema/data model can be modified after data is ingested by refactoring the data.

- **A.** True
- **B.** False

**Answer:** **A — True**
## Q48. Which secure protocols are valid for the Neo4j driver? Select all that apply.

- **A.** graphdb+s
- **B.** neo4j+s
- **C.** https
- **D.** bolt+s

**Answer:** **B, D** — neo4j+s; bolt+s
## Q49. What is the best import method for Neo4j?

- **A.** LOAD CSV always
- **B.** neo4j-admin import always
- **C.** Data Importer always
- **D.** It depends on data source, size, and online/offline requirements

**Answer:** **D — It depends on data source, size, and online/offline requirements**
## Q50. How should a department/sub-category hierarchy usually be modeled?

- **A.** Array property on Product
- **B.** A new label for every category
- **C.** Hierarchy of (:Category) nodes
- **D.** One relationship type per category

**Answer:** **C — Hierarchy of (:Category) nodes**
## Q51. How do you return each person with the number of movies acted in using a COUNT subquery?

- **A.** RETURN COUNT(p)
- **B.** RETURN COUNT { (p)-[:ACTED_IN]->(:Movie) } AS movies
- **C.** RETURN collect(m)
- **D.** RETURN size(p.movies)

**Answer:** **B — RETURN COUNT { (p)-[:ACTED_IN]->(:Movie) } AS movies**
## Q52. Which statements apply to LOAD CSV? Select all that apply.

- **A.** Values are initially read as strings
- **B.** Non-string values should be explicitly cast
- **C.** Types must be declared in import.conf
- **D.** Neo4j always infers every data type automatically

**Answer:** **A, B** — Values are initially read as strings; Non-string values should be explicitly cast
## Q53. Which best describes MERGE?

- **A.** Always creates new data
- **B.** Deletes then recreates
- **C.** Only updates existing data
- **D.** Matches an existing pattern or creates it if absent

**Answer:** **D — Matches an existing pattern or creates it if absent**
## Q54. What does `MATCH (p:Person)-[r]->(m:Movie)` describe?

- **A.** Movie with outgoing relationship to Person
- **B.** Invalid MATCH
- **C.** Person with relationship in any direction
- **D.** Person with an outgoing relationship to Movie

**Answer:** **D — Person with an outgoing relationship to Movie**
## Q55. What best describes properties in Neo4j?

- **A.** Key-value pairs on nodes and relationships
- **B.** None of the above
- **C.** Only on relationships
- **D.** Only on nodes

**Answer:** **A — Key-value pairs on nodes and relationships**
## Q56. Which expression returns Tom Cruise's movie titles collected into a list?

- **A.** RETURN collect(m.title)
- **B.** RETURN m.title
- **C.** RETURN collect(n.name)
- **D.** RETURN m.title AS list

**Answer:** **A — RETURN collect(m.title)**
## Q57. Difference between MERGE and CREATE?

- **A.** MERGE matches or creates; CREATE always creates
- **B.** MERGE is only for relationships
- **C.** MERGE always creates; CREATE is conditional
- **D.** They are interchangeable

**Answer:** **A — MERGE matches or creates; CREATE always creates**
## Q58. Which command loads CSV data?

- **A.** IMPORT CSV
- **B.** LOAD CSV
- **C.** GET CSV
- **D.** READ CSV

**Answer:** **B — LOAD CSV**
## Q59. Which best describes a graph database?

- **A.** Stores data as nodes and relationships
- **B.** Stores only key-value pairs
- **C.** Stores only JSON documents
- **D.** Stores only rows and columns

**Answer:** **A — Stores data as nodes and relationships**
## Q60. How do you execute LOAD CSV in batches of 1000 rows?

- **A.** COMMIT ON 1000
- **B.** IN TRANSACTIONS OF 1000 ROWS
- **C.** OPTIONS {batchSize:1000}
- **D.** ROWS=1000

**Answer:** **B — IN TRANSACTIONS OF 1000 ROWS**
## Q61. Which statements are true? Select all that apply.

- **A.** All nodes must have the same properties
- **B.** Properties can only be stored on nodes
- **C.** Neo4j stores data as nodes and relationships
- **D.** Relationships always have a direction
- **E.** Graph databases are useful when connections matter
- **F.** A node can have multiple labels

**Answer:** **C, D, E, F** — Neo4j stores data as nodes and relationships; Relationships always have a direction; Graph databases are useful when connections matter; A node can have multiple labels
## Q62. Why can COUNT { pattern } be preferable in some counting cases?

- **A.** It can count pattern matches directly without carrying an unnecessary relationship variable
- **B.** It handles nulls magically
- **C.** It is always shorter
- **D.** It changes relationship direction

**Answer:** **A — It can count pattern matches directly without carrying an unnecessary relationship variable**
## Q63. What are the labels in `(p:Person:Customer)-[:PURCHASED]->(order:Order)`?

- **A.** Person and Purchase
- **B.** Person, Customer, Order
- **C.** Person, Purchase, Order
- **D.** Customer and Order

**Answer:** **B — Person, Customer, Order**
## Q64. Which is the correct high-level driver lifecycle?

- **A.** Connect → Execute → Verify → Parse → Close
- **B.** Verify → Connect → Execute → Parse → Close
- **C.** Connect → Verify → Execute → Parse → Close
- **D.** Execute → Connect → Verify → Parse → Close

**Answer:** **C — Connect → Verify → Execute → Parse → Close**
## Q65. What is a key benefit of extracting ORDERED into an Order node?

- **A.** It lets the order connect to multiple entities such as products and shipping companies
- **B.** It removes the need for relationships
- **C.** It links only products to customers
- **D.** It reduces node count

**Answer:** **A — It lets the order connect to multiple entities such as products and shipping companies**
## Q66. Which query pattern finds the actor with the most movies and breaks ties alphabetically?

- **A.** ORDER BY movieCount DESC, name ASC LIMIT 1
- **B.** ORDER BY name DESC LIMIT 1
- **C.** ORDER BY movieCount ASC LIMIT 1
- **D.** RETURN DISTINCT name

**Answer:** **A — ORDER BY movieCount DESC, name ASC LIMIT 1**
## Q67. How do you ensure a property has a value in modern Cypher?

- **A.** IS NULL
- **B.** NOT EXISTS
- **C.** EXISTS(property) only
- **D.** IS NOT NULL

**Answer:** **D — IS NOT NULL**
## Q68. How many records are returned by outer range(1,3) and inner range(1,3) in a correlated subquery?

- **A.** 1
- **B.** 3
- **C.** 9
- **D.** 6

**Answer:** **C — 9**
## Q69. Which condition restricts recent-director movies to 2010 or later?

- **A.** m.released > 2010
- **B.** m.released >= 2010
- **C.** m.released = 2010
- **D.** m.released <= 2010

**Answer:** **B — m.released >= 2010**
## Q70. Which index types are available in Neo4j from the options? Select all that apply.

- **A.** BLOOM
- **B.** TEXT
- **C.** RANGE
- **D.** CURRENCY
- **E.** VECTOR

**Answer:** **B, C, E** — TEXT; RANGE; VECTOR
## Q71. In 'Which customers purchased Product X?', what is the natural relationship?

- **A.** X
- **B.** Customer
- **C.** Purchased
- **D.** Product

**Answer:** **C — Purchased**
## Q72. How do you eliminate duplicate result rows?

- **A.** DISTINCT
- **B.** UNIQUE
- **C.** REMOVE DUPLICATES
- **D.** DELETE

**Answer:** **A — DISTINCT**
## Q73. Which features are Enterprise Edition features in the practice material? Select all that apply.

- **A.** Cypher query language
- **B.** Role-based access control
- **C.** Clustering architecture
- **D.** Programming APIs
- **E.** ACID transactions
- **F.** Online backup functionality

**Answer:** **B, C, F** — Role-based access control; Clustering architecture; Online backup functionality
## Q74. What formula is used for Comedy movie percentage?

- **A.** Comedy movies / total movies * 100
- **B.** Total movies / Comedy movies
- **C.** Comedy movies + total movies
- **D.** Comedy movies * total movies

**Answer:** **A — Comedy movies / total movies * 100**
## Q75. What must a two-degrees-away actor from Albert Austin satisfy?

- **A.** Acted directly with Austin
- **B.** Acted with someone who acted with Austin, but never directly with Austin
- **C.** Only directed Austin
- **D.** Has the same birth year

**Answer:** **B — Acted with someone who acted with Austin, but never directly with Austin**
## Q76. Which is the correct DateTime import?

- **A.** from neo4j.time import Calendar
- **B.** from neo4j.time import DateTime
- **C.** from neo4j.time import Duration only
- **D.** from datetime import neo4j

**Answer:** **B — from neo4j.time import DateTime**
## Q77. What helps query cache reuse? Select all that apply.

- **A.** Keep query text/casing consistent when reuse matters
- **B.** Define an ID for query reuse
- **C.** Use parameters
- **D.** Use literals

**Answer:** **A, C** — Keep query text/casing consistent when reuse matters; Use parameters
## Q78. Which query returns actors in You've Got Mail?

- **A.** MATCH (p:Person)-[:ACTED_IN]->(:Movie {title:"You've Got Mail"}) RETURN p.name
- **B.** MATCH (:Movie)-[:ACTED_IN]->(p)
- **C.** SELECT actor FROM movie
- **D.** MATCH (p:Movie)-[:ACTED_IN]->(:Person)

**Answer:** **A — MATCH (p:Person)-[:ACTED_IN]->(:Movie {title:"You've Got Mail"}) RETURN p.name**
## Q79. Which query counts movies by genre?

- **A.** MATCH (m:Movie)-[:IN_GENRE]->(g:Genre) RETURN g.name, count(m)
- **B.** MATCH (g:Genre) RETURN g
- **C.** MATCH (m) RETURN m.genre
- **D.** SELECT genre, COUNT(*)

**Answer:** **A — MATCH (m:Movie)-[:IN_GENRE]->(g:Genre) RETURN g.name, count(m)**
## Q80. Why is `WHERE m:Genre` usually wrong if m is already the Movie variable?

- **A.** It asks the Movie node to also carry Genre label rather than traverse to Genre
- **B.** WHERE cannot use labels
- **C.** Genre must be a property
- **D.** Cypher forbids multiple labels

**Answer:** **A — It asks the Movie node to also carry Genre label rather than traverse to Genre**
## Q81. For 'What parts of type X are required to make product Y?', which are natural node concepts? Select all that apply.

- **A.** Product
- **B.** X
- **C.** Type
- **D.** Y
- **E.** Part

**Answer:** **A, C, E** — Product; Type; Part
## Q82. What is a natural relationship for that product/part model?

- **A.** REQUIRES
- **B.** PRODUCT
- **C.** TYPE
- **D.** NODE

**Answer:** **A — REQUIRES**
## Q83. Which phrase is the best simple Neo4j performance reminder?

- **A.** Anchor early, expand narrowly, return only what you need
- **B.** Always match every node
- **C.** Never use labels
- **D.** Return * for all production queries

**Answer:** **A — Anchor early, expand narrowly, return only what you need**
## Q84. Which Snowflake concept is most similar to a Neo4j property?

- **A.** Table
- **B.** Column value/property field
- **C.** Warehouse
- **D.** Role

**Answer:** **B — Column value/property field**
## Q85. Which Snowflake/SQL concept is most similar to `RETURN`?

- **A.** SELECT
- **B.** JOIN
- **C.** GROUP BY
- **D.** INSERT

**Answer:** **A — SELECT**
## Q86. Which Snowflake/SQL concept is most similar to `WHERE`?

- **A.** ORDER BY
- **B.** WHERE
- **C.** COPY INTO
- **D.** CREATE TABLE

**Answer:** **B — WHERE**
## Q87. Which SQL concept is closest to `DISTINCT` in Cypher?

- **A.** DISTINCT
- **B.** HAVING
- **C.** QUALIFY only
- **D.** UNION ALL

**Answer:** **A — DISTINCT**
## Q88. Which Cypher function converts CSV text to an integer?

- **A.** toInteger()
- **B.** toNumber()
- **C.** castInt()
- **D.** integer()

**Answer:** **A — toInteger()**
## Q89. Which Cypher function converts CSV text to a floating-point number?

- **A.** toFloat()
- **B.** float()
- **C.** toDecimal()
- **D.** castFloat()

**Answer:** **A — toFloat()**
## Q90. Which query removes a node plus its relationships?

- **A.** DELETE
- **B.** DETACH DELETE
- **C.** DROP NODE
- **D.** REMOVE NODE

**Answer:** **B — DETACH DELETE**
# 45. Mini Practice Set for Self-Testing

Try answering these without looking back.

1. What does `()` mean in Cypher?
2. What does `[]` mean?
3. Difference between `CREATE` and `MERGE`?
4. What does `WITH` do?
5. What does `DISTINCT` do?
6. How do you check if a property exists?
7. How do you count pattern matches inline?
8. What does `EXISTS { ... }` do?
9. What is the driver protocol?
10. What are two secure connection schemes?
11. Difference between `EXPLAIN` and `PROFILE`?
12. What does `WITH HEADERS` change in LOAD CSV?
13. What data type is CSV input initially?
14. How do you batch CSV import?
15. What does `collect()` return?
16. What is relationship uniqueness?
17. What does a Node Key Constraint guarantee?
18. Why use an intermediate Order node?
19. What is a good category modeling approach?
20. What does `COUNT { pattern }` do?

---

# 46. 7-Day Neo4j Practice Plan for a Snowflake Developer

## Day 1 — Graph basics

Focus:

- nodes
- labels
- properties
- relationships
- MATCH
- RETURN

Practice:

```cypher
MATCH (p:Person)
RETURN p.name
LIMIT 10
```

---

## Day 2 — Filtering and aggregation

Focus:

- WHERE
- IN
- IS NOT NULL
- DISTINCT
- count()
- collect()
- ORDER BY
- LIMIT

---

## Day 3 — Relationships and graph traversal

Focus:

```cypher
(Person)-[:ACTED_IN]->(Movie)
(Person)-[:DIRECTED]->(Movie)
(Movie)-[:IN_GENRE]->(Genre)
```

Practice 1-hop and 2-hop queries.

---

## Day 4 — CREATE / MERGE / DELETE

Focus:

- CREATE
- MERGE
- ON CREATE
- ON MATCH
- SET
- DELETE
- DETACH DELETE

---

## Day 5 — LOAD CSV and constraints

Focus:

- LOAD CSV
- WITH HEADERS
- type conversions
- IN TRANSACTIONS
- indexes
- constraints

---

## Day 6 — Subqueries and performance

Focus:

- WITH
- CALL subqueries
- EXISTS
- COUNT subqueries
- EXPLAIN
- PROFILE
- query cache

---

## Day 7 — Driver practice

Focus:

- Python driver
- Bolt
- secure schemes
- execute_query
- Result records
- transaction functions
- DateTime

---

# 47. Final Quick Reference

```cypher
// Find nodes
MATCH (p:Person)
RETURN p

// Find connected nodes
MATCH (p:Person)-[:ACTED_IN]->(m:Movie)
RETURN p.name, m.title

// Filter
WHERE m.released >= 2010

// List membership
WHERE p.born IN [1970, 1980, 1990]

// Property exists
WHERE p.name IS NOT NULL

// Aggregate
RETURN p.name, count(m)

// Distinct
RETURN DISTINCT p.name

// List aggregation
RETURN collect(m.title)

// Match or create
MERGE (p:Person {name:'Alice'})

// Delete
MATCH (p:Person {name:'Alice'})
DETACH DELETE p

// Exists subquery
WHERE EXISTS { (p)-[:DIRECTED]->(:Movie) }

// Count subquery
RETURN COUNT { (p)-[:ACTED_IN]->(:Movie) }

// CSV
LOAD CSV WITH HEADERS FROM 'file:///people.csv' AS row

// Batch
CALL (row) {
    MERGE (p:Person {id: row.id})
} IN TRANSACTIONS OF 1000 ROWS

// Explain
EXPLAIN MATCH (p:Person) RETURN p

// Profile
PROFILE MATCH (p:Person) RETURN p
```

---

# 48. Final Memory Summary

If you remember only 10 things, remember these:

1. `()` = node.
2. `[]` = relationship.
3. `MATCH` = find a graph pattern.
4. `RETURN` = select output.
5. `WHERE` = filter.
6. `WITH` = pass intermediate results.
7. `MERGE` = match-or-create.
8. `DISTINCT` = remove duplicates.
9. `EXISTS {}` and `COUNT {}` are powerful pattern subqueries.
10. In Neo4j, **relationships are part of the data, not just join logic**.

That last point is the biggest shift from Snowflake/relational thinking.
