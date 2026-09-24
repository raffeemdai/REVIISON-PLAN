# Neo4j & Graph Databases: Complete Study Notes

> **Sources:** class transcript (Graph DB session) + `graphdbneo4j.ipynb` + `graphrag.ipynb`
> **Legend:**
> 📝 = taught in the class transcript
> 📓 = taken from your uploaded notebooks
> ➕ = extra standard Cypher/Neo4j material added so the notes are complete (not shown in class)
> ⚠️ = common mistake / gotcha

---

## Table of Contents

1. [Theory concepts of Neo4j & graph databases](#1-theory-concepts)
2. [Basic concepts (glossary and Cypher syntax anatomy)](#2-basic-concepts)
3. [Setup: Aura instance, driver, `run_query` helper](#3-setup)
4. [CRUD operations in Cypher (with examples)](#4-crud-operations-using-cypher)
   - 4.1 Create nodes · 4.2 Bulk create · 4.3 Create relationships · 4.4 Read (MATCH / WHERE / RETURN / ORDER BY / LIMIT / GROUP BY / aggregates) · 4.5 Update · 4.6 Delete · 4.7 MERGE vs CREATE · 4.8 UNWIND · 4.9 Constraints & indexes
5. [Cell-by-cell walkthrough of `graphdbneo4j.ipynb`](#5-walkthrough-of-graphdbneo4jipynb)
6. [Graph RAG (transcript + `graphrag.ipynb`)](#6-graph-rag)
7. [SQL vs Cypher comparison](#7-sql-vs-cypher)
8. [Memory tricks](#8-memory-tricks)
9. [Errors and gotchas](#9-errors-and-gotchas)
10. [One-page cheat sheet](#10-one-page-cheat-sheet)
11. [Interview Q&A](#11-interview-qa)
12. [Practice exercises](#12-practice-exercises)
13. [Everything else said in class (timeline & admin notes)](#13-other-points-from-the-class)

---

# 1. Theory Concepts

## 1.1 What is a graph? 📝

In the simplest terms (instructor's definition):

> **Graph = Nodes + Edges.**
> - **Node** = an entity that **stores information**.
> - **Edge** = a **connection/relationship** between nodes (the instructor calls it "the association").

**Class example, LinkedIn:**

```
 (Sudh) ──── connected ────► (Anand) ──── connected ────► (Hitesh)
```

- Each person is a **node** holding their own data (email, photo, resume, experience, certificates).
- Sudh ↔ Anand is a **1st-degree connection** (direct).
- Sudh ↔ Hitesh is a **2nd-degree connection** (reached *through* Anand).
- The database stores **the data and the relationship**, so anyone can later ask *"how is Anand related to Sudh / Hitesh?"* and the system answers **without defining the relationship externally at query time**. It reads it from the stored relationship.

## 1.2 What is a graph database? 📝

A database that stores data as **nodes and relationships** instead of tables/rows or documents. **Neo4j** is the graph database used in class. The instructor stressed it is *not the only provider* (others exist), but it is the one you'll use in the upcoming project.

## 1.3 Where graph fits among the databases you've already studied 📝

| Database type | How it stores data | Class reference |
|---|---|---|
| Relational (SQL/RDBMS) | Tables → rows & columns | "column/attribute, row by row" |
| Document NoSQL (MongoDB) | JSON / dictionary / key-value | "in form of JSON, dictionary, key-value pair" |
| Vector DB (Qdrant/FAISS/Chroma…) | Numeric vectors (embeddings) | "numeric format" |
| **Graph DB (Neo4j)** | **Nodes + relationships (with properties)** | **"store the info AND define the relation"** |

## 1.4 Why graph databases? 📝

1. **Relationships are first-class citizens**, stored explicitly, not re-derived by JOINs each time.
2. **Recommendation systems.** Facebook, Instagram, LinkedIn: modern recommenders store *every relationship plus a weightage/degree* of that relationship.
3. **Fast search on connected data.** Once you land on a node, you follow its links; you don't scan everything (see 1.6).
4. **Flexibility.** You define the relationship names and how many you want ("you have leverage to create as many as you want").
5. **Graph RAG.** Combining a graph DB with a vector DB gives better retrieval for LLM applications (Section 6).
6. Instructor's claim: *"graph makes AI agents ~80% more truthful."* (This is the instructor's statement in class; treat it as a motivating claim, not a benchmark I've verified.)
7. **Interview relevance.** Graph RAG questions will be asked, and real projects (the upcoming class project) will use it.

## 1.5 The Property Graph Model ➕

Neo4j uses the **labeled property graph** model:

- **Nodes** have **labels** (type/category, e.g. `:student`) and **properties** (key-value, e.g. `name: "Rahul"`).
- **Relationships** have a **type** (e.g. `ENROLLED_IN`), a **direction** (start → end), and can also have **properties** (e.g. `grade: "A"`).
- **Schema-flexible:** nodes with the same label don't have to have the same properties (📓 in the notebook, `SUDH` and `shadiya` were created *without* `student_id`, while later students have it).

## 1.7 Why is graph search fast? 📝 + ➕

**Class explanation (RDBMS vs graph search):**
- RDBMS performs a **row-wise search**: scans rows (or uses an index) and JOINs tables.
- In a graph, once you reach a node you **already know who it's connected to**, so you scan only the data **around that node**, not all other nodes → **very low search time complexity**, even for a very complex graph.

**The technical name for this (added):** *index-free adjacency*. Each node physically holds direct pointers to its neighbouring relationships, so traversing one hop costs roughly the same no matter how big the total graph is. In SQL, every hop is a JOIN whose cost grows with table sizes.

> ⚖️ **Honest balance:** graphs shine at *connected/relationship-heavy* questions (friends-of-friends, supply chains, recommendations, fraud rings). For big flat aggregations/reporting on tabular data, SQL is often the better tool.

## 1.8 Functional difference vs RDBMS 📝

| | RDBMS | Graph DB |
|---|---|---|
| Relationships | Built from **primary key / foreign key** constraints across tables → a *limited*, predefined set | You create **your own custom relationships**, as many as you want, any name |
| Search | Row-wise search + JOINs | Traverse from a node to its neighbours |
| Structure | Fixed schema (columns) | Flexible: properties per node |

## 1.9 Typical use cases (class + ➕)

Social networks · recommendation engines · **RAG / Graph RAG** · knowledge graphs · supply-chain dependency tracking (📓 the `graphrag.ipynb` example) · fraud detection · org charts · learning platforms (student ↔ course ↔ skill ↔ mentor, the 📓 notebook example).

---

# 2. Basic Concepts

## 2.1 Glossary

| Term | Meaning | Example from notebook |
|---|---|---|
| **Node** | An entity holding data | A student, a course |
| **Label** | The *type* of a node (like a table name in SQL) | `:student`, `:course`, `:skill`, `:mentor`, `:Document`, `:Chunk` |
| **Property** | A key-value pair inside a node or relationship | `name: "Rahul"`, `age: 24` |
| **Relationship / Edge** | A **directed**, **typed** connection between 2 nodes | `(Rahul)-[:ENROLLED_IN]->(Agentic AI)` |
| **Relationship type** | The name of the connection, **you choose it** ("my own custom keyword") | `ENROLLED_IN`, `HAS_SKILL`, `HAS_CHUNK` |
| **Alias / variable** | A temporary name for a matched element inside one query | `s`, `c`, `sk`, `r` |
| **Parameter** | A placeholder `$name` filled at run time from Python | `$name`, `$age`, `$city` |
| **Cypher** | Neo4j's query language | everything you wrote in the notebook |
| **Driver** | The communication bridge between your Python code and Neo4j | `GraphDatabase.driver(...)` |
| **Aura** | Neo4j's managed cloud service (free instance used in class) | `neo4j+s://xxxx.databases.neo4j.io` |
| **Traversal** | Walking from node to node along relationships | Rahul → skill |
| **Degree** | Number of hops between nodes (1st-degree = direct) | Sudh→Anand→Hitesh |

> **Class analogy:** *"Think of the **node label as a table name**, and each **node as one record (row)** in that table. The extra thing a graph adds is the **relationship between nodes**."*

## 2.2 Anatomy of Cypher patterns (ASCII art!)

Cypher lets you **draw** the pattern:

```cypher
(s:student {name:"Rahul"}) -[r:ENROLLED_IN]-> (c:course)
 │  │       │               │  │              │  │
 │  │       └ property map  │  │              │  └ label
 │  └ label                │  │              └ alias for the course node
 └ alias (variable)        │  └ relationship type
                           └ alias for the relationship
```

| Symbol | Meaning |
|---|---|
| `( )` | a **node** (round like a circle) |
| `[ ]` | a **relationship** (square brackets) |
| `-->` or `->` | direction of the relationship |
| `:Label` | node label / relationship type after the colon |
| `{k: v}` | properties |
| `$param` | parameter placeholder |
| `s.name` | read property `name` of node `s` |

**Direction forms:**

```cypher
(a)-[:KNOWS]->(b)    // a → b
(a)<-[:KNOWS]-(b)    // b → a
(a)-[:KNOWS]-(b)     // either direction (only when READING/matching, not when creating)
```

## 2.3 Case-sensitivity rules ⚠️

- **Keywords** (`MATCH`, `RETURN`, `create`) → **not** case-sensitive.
- **Labels, relationship types, property names, and property VALUES** → **case-sensitive**.
  - 📝 In class, searching name `"rahul"` returned **blank**; it had to be `"Rahul"`.
  - 📝 Course IDs are stored as `C001` (capital C): using `c001` silently matched nothing.
- **Convention:** Labels in `PascalCase` (`Student`), relationship types in `UPPER_SNAKE_CASE` (`ENROLLED_IN`), properties in `camelCase`/`snake_case`. (The class used lowercase labels like `student`; it works, but remember to be consistent since `student` ≠ `Student`.)

## 2.4 The 3 layers of the class demo 📝

1. **Nodes only** first (students, courses, skills, mentor), *"a node, not a relation"*.
2. **Then relationships** using `MATCH … MERGE`.
3. **Then querying** with `MATCH … WHERE … RETURN`.

Nodes without relationships are "technically independent nodes." You can still search them, but *"a proper graph"* needs associations.

---

# 3. Setup

## 3.1 Create a free Aura instance 📝

1. Go to the Neo4j Aura sign-up link (given in class). Choose **one-tap login with Google**.
2. Onboarding shows a sample graph (`Sudh` –WORKS_AT→ `Euron`), just a demo.
3. Click **Create a free instance**.
4. You get a **username & password**. **Download the credentials file immediately**. (Instructor: you can regenerate later if lost.)
5. Wait for the instance to become **Running** (in class this took several minutes).
6. Open **Query** section (left panel). Initially the database is empty. As you add nodes, the graph shows up on the left; you can view results as **graph**, **table**, or **raw**.
7. ⚠️ Make sure you **select/connect to your current running instance** in the console, otherwise nodes won't be displayed (this happened in class).

## 3.2 Install & connect from Python 📓

```python
%pip install neo4j          # 📓 notebook typed `pip install neo4j` (use %pip inside Jupyter)

import os
from neo4j import GraphDatabase

NEO4J_URI      = os.getenv("NEO4J_URI")        # e.g. neo4j+s://<id>.databases.neo4j.io
NEO4J_USERNAME = os.getenv("NEO4J_USERNAME")
NEO4J_PASSWORD = os.getenv("NEO4J_PASSWORD")

driver = GraphDatabase.driver(NEO4J_URI, auth=(NEO4J_USERNAME, NEO4J_PASSWORD))
driver.verify_connectivity()     # raises an error if connection/credentials are wrong
```

> 🔐 **Security:** Your uploaded notebooks contain the real URI, username and password in plain text. **Do not commit them to GitHub or share the notebooks.** Rotate/regenerate the password from the Aura console (class said regenerating is possible) and load secrets from environment variables or a `.env` file, as above.

**What is the driver?** 📝 *"Driver is basically the communication between my system and Neo4j."*

## 3.3 The reusable `run_query` helper 📓📝

```python
def run_query(query, parameters=None):
    records, summary, keys = driver.execute_query(query, parameters_=parameters or {})
    return [record.data() for record in records]
```

**What `execute_query` returns** 📝: three things, `records` (actual data), `summary` (info about execution), `keys` (column names).
**Why a helper?** 📝 *"To remove the redundancy of writing code"*; write once, call again and again.
**`record.data()`** converts each record into a plain Python dict, and the list comprehension gives you a list of dicts.

> ⚠️ **The bug hit live in class** 📝: `Neo.ClientError … Missing expected parameter name, age and city`. Cause: the parameters were not passed under the right keyword. The fix: pass them via **`parameters_=`** (with the trailing underscore) as a dict. That's why the helper uses `parameters_=parameters or {}`.

---

# 4. CRUD Operations using Cypher

**CRUD ↔ Cypher keywords:**

| CRUD | Cypher | Notes |
|---|---|---|
| **C**reate | `CREATE`, `MERGE` | `MERGE` = create *only if it doesn't exist* |
| **R**ead | `MATCH … WHERE … RETURN` | the core read pattern |
| **U**pdate | `SET`, `REMOVE` | after a `MATCH` |
| **D**elete | `DELETE`, `DETACH DELETE` | `DETACH` also removes attached relationships |

All examples below use the 📓 dataset:

- `:student {student_id, name, age, city}`
- `:course {course_id, name}`
- `:skill {name}`
- `:mentor {mentor_id, name, company}`
- Relationships: `(:student)-[:ENROLLED_IN]->(:course)`, `(:student)-[:HAS_SKILL]->(:skill)`

## 4.1 CREATE: nodes

### Single node with parameters 📓📝

```cypher
CREATE (s:student {name:$name, age:$age, city:$city})
RETURN s
```

Python:

```python
query = """create(s:student{name:$name,age:$age ,city:$city}) RETURN s"""
result = run_query(query, parameters={"name": "SUDH", "age": 21, "city": "New York"})
```

**Line-by-line (class explanation):**
- `CREATE` → make something new.
- `(s:student {...})` → make a **node**, alias it `s`, give it label `student`, and store `name`, `age`, `city` as properties.
- `$name` → a **parameter**, the value is supplied from Python (safer and reusable than string concatenation, and prevents Cypher injection).
- `RETURN s` → send the created node back to Python.

Then 📓 a second student was added by re-using the same query:

```python
run_query(query, parameters={"name": "shadiya", "age": 24, "city": "BLR"})
# → [{'s': {'city': 'BLR', 'name': 'shadiya', 'age': 24}}]
```

In class the instructor also created `Pavan` (24, Bangalore), `Hitesh`, `Sadia`, etc., **the same way**.

### Create without parameters (literal values) ➕

```cypher
CREATE (:student {student_id:"S005", name:"Kiran", age:22, city:"Pune"})
```

### Create several node types 📓

```python
# courses
query = """create(c:course{course_id:$course_id,name:$name}) RETURN c"""

# skills
query = """create(s:skill{name:$name}) RETURN s"""

# mentor
query = """create(m:mentor{mentor_id:$mentor_id,name:$name,company:$company}) RETURN m"""
```

> 📝 After running these, the console showed **different kinds of nodes**: student, course, skill, mentor (*"no relation created so far, just nodes"*).

⚠️ **Re-running `CREATE` makes duplicates.** 📝 A student in class ran the same query 3 times and got **3 nodes**. Fixes: use `MERGE` (4.7), add a **uniqueness constraint** (4.9), or delete the duplicates.

## 4.2 Bulk create (many records)

### Method 1: Python `for` loop 📓📝 (used in class)

```python
students = [
    {"student_id": "S001", "name": "Rahul", "age": 24, "city": "Bengaluru"},
    {"student_id": "S002", "name": "Priya", "age": 23, "city": "Delhi"},
    {"student_id": "S003", "name": "Aman",  "age": 27, "city": "Mumbai"},
    {"student_id": "S004", "name": "Sneha", "age": 25, "city": "Bengaluru"},
]

query = """create(s:student{student_id:$student_id,name:$name,age:$age ,city:$city}) RETURN s"""
for student in students:
    run_query(query, parameters=student)   # dict keys == parameter names
```

*Trick:* because each dict's keys match the `$parameters`, you can pass the dict straight in.

The same loop pattern created the 3 courses (`C001 Agentic AI`, `C002 Data Science`, `C003 Backend Engineering`) and 6 skills (Python, Machine Learning, RAG, Neo4j, FastAPI, LangGraph).

### Method 2: `UNWIND` (mentioned in class as the other way 📝, syntax ➕)

One round-trip to the DB instead of N:

```cypher
UNWIND $rows AS row
CREATE (s:student {student_id: row.student_id, name: row.name, age: row.age, city: row.city})
```

```python
run_query(query, parameters={"rows": students})
```

> 📝 Instructor: *"There is something called UNWIND… but even the for-loop works, that's completely fine."*
> ➕ For large data prefer `UNWIND` (much faster) and `MERGE` instead of `CREATE`.

## 4.3 CREATE: relationships

**Rule (📝):** *"To create a relationship, write `MATCH` and then `MERGE`."*
`MATCH` finds the two existing nodes → `MERGE` draws the edge between them.

### Basic relationship 📓📝

```cypher
MATCH (s:student {student_id:$student_id})
MATCH (c:course  {course_id:$course_id})
MERGE (s)-[:ENROLLED_IN]->(c)
RETURN s, c
```

```python
run_query(query, parameters={"student_id": "S001", "course_id": "C001"})
# Rahul ENROLLED_IN Agentic AI
```

**Explanation:**
1. Find student `S001` (Rahul) → alias `s`.
2. Find course `C001` (Agentic AI) → alias `c`.
3. `MERGE (s)-[:ENROLLED_IN]->(c)` → create the arrow **if it isn't already there**.
4. `ENROLLED_IN` is **your own keyword**: *"I can write my name, your name, anything I want to represent a relationship."*
5. In the console you can now see the **arrow** Rahul → Agentic AI.

### Relationship **with properties** 📓📝

```cypher
MATCH (s:student {student_id:$student_id})
MATCH (c:course  {course_id:$course_id})
MERGE (s)-[r:ENROLLED_IN]->(c)
SET r.enrolled_on = date($enrolled_on),
    r.status      = $status,
    r.grade       = $grade,
    r.remarks     = $remarks
RETURN s, c
```

```python
run_query(query, parameters={
    "student_id": "S001", "course_id": "C001",
    "enrolled_on": "2024-06-01", "status": "active",
    "grade": "A", "remarks": "Excellent performance"
})
```

Key learnings:
- Give the relationship an **alias** (`r`) so you can `SET` properties on it.
- 📝 Running this again did **not** create a second edge; it **updated the same relationship** (because of `MERGE`).
- `date("2024-06-01")` converts a string into a Neo4j **date** type.
- 📝 Instructor: you can also add **conditions** so the relationship is created only if they hold.

### Bulk relationships (enrollments) 📓📝

```python
enrollments = [
    {"student": "S001", "course": "C001"},
    {"student": "S002", "course": "C002"},
    {"student": "S003", "course": "C003"},
    {"student": "S004", "course": "C003"},
    {"student": "S005", "course": "C002"},   # ⚠️ S005 doesn't exist in the DB!
]

query = """MATCH (s:student {student_id:$student_id})
MATCH (c:course {course_id:$course_id})
MERGE (s)-[r:ENROLLED_IN]->(c)
RETURN s, c
"""
for enrollment in enrollments:
    run_query(query, parameters={"student_id": enrollment["student"],
                                 "course_id":  enrollment["course"]})
```

> ⚠️ **Silent failure lesson:** if either `MATCH` finds nothing (e.g. `S005` doesn't exist, or you typed `c001` instead of `C001`), the query returns **zero rows and creates nothing**, with **no error**. 📝 In class, the first attempt "did not create" relationships; the cause was **case mismatch** (`s001` vs `S001`; `c001` vs `C001`).
> Result in class: Aman & Sneha → Backend; Priya → Data Science; Rahul → Agentic AI.

### Student ↔ Skill relationships 📓📝

```python
student_skills = [
    {"student": "S001", "skill": "Python"},
    {"student": "S001", "skill": "RAG"},
    {"student": "S001", "skill": "LangGraph"},
    {"student": "S002", "skill": "Python"},
    {"student": "S002", "skill": "Neo4j"},
    {"student": "S003", "skill": "Python"},
    {"student": "S003", "skill": "Machine Learning"},
    {"student": "S004", "skill": "FastAPI"},
    {"student": "S004", "skill": "Python"},
]

query = """MATCH (s:student {student_id:$student_id})
MATCH (sk:skill {name:$skill})
MERGE (s)-[r:HAS_SKILL]->(sk)
RETURN s, sk
"""
for student_skill in student_skills:
    run_query(query, parameters={"student_id": student_skill["student"],
                                 "skill": student_skill["skill"]})
```

> 📝 **Debug story from class:** the first version tried to match the skill using a property that doesn't exist (`label`/`skill`). The `skill` node only has **`name`**, so the match returned nothing. Fix: `MATCH (sk:skill {name:$skill})`.
> 📝 The class dataset also had a **skill level** (`advance`, `intermediate`). That belongs **on the relationship**: `MERGE (s)-[r:HAS_SKILL]->(sk) SET r.level = $level`. (Your final notebook version leaves level out.)

**Resulting graph (📝 as described in class):** Python: Aman, Sneha, Rahul, Priya · Priya also Neo4j · Aman also ML · Rahul: LangGraph, RAG, Python · Sneha: Python, FastAPI.

📝 *"Similarly we can create a relationship for mentors too if required."* ➕ Example:

```cypher
MATCH (m:mentor {mentor_id:"M001"}), (c:course {course_id:"C001"})
MERGE (m)-[:TEACHES]->(c)
```

## 4.4 READ: `MATCH … WHERE … RETURN`

**Mental model:** `MATCH` = *"go to this kind of node/pattern"*, `WHERE` = *filter*, `RETURN` = *what to send back*.

### 4.4.1 Return everything of a label ➕

```cypher
MATCH (s:student)
RETURN s
```

### 4.4.2 Query through a relationship, "What skills does Rahul have?" 📓📝

```cypher
MATCH (s:student {name:$name})-[:HAS_SKILL]->(sk:skill)
RETURN s, sk
```

```python
run_query(query, parameters={"name": "Rahul"})
# → 3 rows: Rahul → Python, RAG, LangGraph
```

Class notes: the first attempt with `"rahul"` returned **blank** → had to use **`"Rahul"`** (case-sensitive). Returning only the skill alias (`RETURN sk`) gives just the skills.

### 4.4.3 `WHERE`: filtering 📓📝

```cypher
MATCH (s:student)
WHERE s.age <= $age AND s.age >= $min_age
RETURN s.name, s.age
```

```python
run_query(query, parameters={"age": 25, "min_age": 18})
# 📓 output: SUDH 21, pavan 24, Hitesh 24, shadiya 24, Rahul 24, Priya 23, Sneha 25
# (Aman, age 27, correctly excluded)
```

- `s.age` = property access (`alias.property`).
- Returning `s.name, s.age` returns **partial columns**; returning `s` returns the **whole node**. 📝 *"If I don't want the complete record, I can return just a partial record."*
- 📝 Class exercise idea: *filter students with age ≤ 23 **or** city = Delhi, or both.*

```cypher
MATCH (s:student)
WHERE s.age <= 23 OR s.city = "Delhi"
RETURN s.name, s.age, s.city
```

**`WHERE` operators cheat-list ➕**

| Need | Cypher |
|---|---|
| equals / not equals | `=`  `<>` |
| comparison | `<  <=  >  >=` |
| logic | `AND  OR  NOT  XOR` |
| in a list | `s.city IN ["Delhi","Mumbai"]` |
| text | `s.name STARTS WITH "S"` · `ENDS WITH` · `CONTAINS "an"` |
| regex | `s.name =~ "(?i)r.*"` |
| null checks | `s.student_id IS NULL` / `IS NOT NULL` |
| case-insensitive | `toLower(s.name) = "rahul"` |
| relationship exists | `WHERE (s)-[:ENROLLED_IN]->(:course)` |
| relationship absent | `WHERE NOT (s)-[:ENROLLED_IN]->()` |
| between | `s.age >= 18 AND s.age <= 25` |

Shortcut: inline property match `MATCH (s:student {city:"Delhi"})` is the same as `WHERE s.city = "Delhi"`.

### 4.4.4 `RETURN` options ➕

```cypher
MATCH (s:student)
RETURN s.name AS student_name, s.city AS home_city      // AS = alias / rename column
```

```cypher
MATCH (s:student) RETURN DISTINCT s.city                 // unique values
```

### 4.4.5 `ORDER BY`, `LIMIT`, `SKIP` ➕

📝 *"You can write aggregation, ORDER BY, everything we do in general."*

```cypher
MATCH (s:student)
RETURN s.name, s.age
ORDER BY s.age DESC          // ASC is default
LIMIT 3                      // top 3 only
```

```cypher
MATCH (s:student) RETURN s.name ORDER BY s.name SKIP 5 LIMIT 5   // pagination (page 2)
```

### 4.4.6 Aggregations: `count`, `sum`, `avg`, `min`, `max`, `collect` ➕

```cypher
MATCH (s:student)
RETURN count(s)      AS total_students,
       avg(s.age)    AS avg_age,
       min(s.age)    AS youngest,
       max(s.age)    AS oldest
```

### 4.4.7 **GROUP BY** in Cypher ➕ (there is no `GROUP BY` keyword!)

> **Rule:** In Cypher, **every non-aggregated expression in `RETURN` (or `WITH`) automatically becomes the grouping key.** The aggregate function goes alongside it.

**Students per city** (SQL: `SELECT city, COUNT(*) FROM student GROUP BY city`)

```cypher
MATCH (s:student)
RETURN s.city AS city, count(*) AS total
ORDER BY total DESC
```

**Students per course** (through a relationship)

```cypher
MATCH (s:student)-[:ENROLLED_IN]->(c:course)
RETURN c.name AS course, count(s) AS enrolled_students
ORDER BY enrolled_students DESC
```

**Skills per student, collected as a list**

```cypher
MATCH (s:student)-[:HAS_SKILL]->(sk:skill)
RETURN s.name AS student, collect(sk.name) AS skills, count(sk) AS skill_count
```

**Average age per city**

```cypher
MATCH (s:student)
RETURN s.city, avg(s.age) AS avg_age
```

### 4.4.8 `HAVING` equivalent → `WITH … WHERE` ➕

SQL's `HAVING` filters *after* grouping. In Cypher, use `WITH` to pass the grouped result on, then `WHERE`:

```cypher
// Skills known by at least 2 students   (SQL: … GROUP BY skill HAVING COUNT(*) >= 2)
MATCH (s:student)-[:HAS_SKILL]->(sk:skill)
WITH sk.name AS skill, count(s) AS n
WHERE n >= 2
RETURN skill, n
ORDER BY n DESC
```

`WITH` = *"pipe the results forward and continue the query"*.

### 4.4.9 Multi-hop traversal (the graph superpower, no JOINs!) ➕

**Who else is in the same course as Aman?**

```cypher
MATCH (a:student {name:"Aman"})-[:ENROLLED_IN]->(c:course)<-[:ENROLLED_IN]-(peer:student)
WHERE peer <> a
RETURN peer.name AS classmate, c.name AS course
```

**Recommendation-style: students who share skills with Priya, ranked**

```cypher
MATCH (p:student {name:"Priya"})-[:HAS_SKILL]->(sk:skill)<-[:HAS_SKILL]-(other:student)
WHERE other <> p
RETURN other.name AS similar_student, collect(sk.name) AS shared_skills, count(sk) AS score
ORDER BY score DESC
```

*(This is the same principle as the class's LinkedIn / recommendation example: connections-of-connections.)*

**Variable-length paths (1st, 2nd, 3rd degree, like Sudh → Anand → Hitesh)**

```cypher
MATCH (me:Person {name:"Sudh"})-[:CONNECTED_TO*1..2]-(other:Person)
WHERE other <> me
RETURN DISTINCT other.name
```

`*1..2` means "between 1 and 2 hops".

### 4.4.10 `OPTIONAL MATCH` (LEFT JOIN) ➕

```cypher
MATCH (s:student)
OPTIONAL MATCH (s)-[:ENROLLED_IN]->(c:course)
RETURN s.name, c.name          // students without a course show c.name = null
```

### 4.4.11 Case expressions & null handling ➕

```cypher
MATCH (s:student)
RETURN s.name,
       CASE WHEN s.age < 25 THEN "junior" ELSE "senior" END AS level,
       coalesce(s.student_id, "N/A") AS sid
```

## 4.5 UPDATE: `SET` and `REMOVE` ➕ (📝 class: *"create, update, insert, delete, everything can be done"*)

```cypher
// update one property
MATCH (s:student {student_id:"S002"})
SET s.city = "Hyderabad"
RETURN s
```

```cypher
// increment
MATCH (s:student {student_id:"S002"})
SET s.age = s.age + 1
```

```cypher
// set several at once  (+= merges, keeps other props)
MATCH (s:student {student_id:"S002"})
SET s += {city:"Pune", active:true}
```

```cypher
// update a RELATIONSHIP property (📓 pattern used r.grade / r.status)
MATCH (:student {student_id:"S001"})-[r:ENROLLED_IN]->(:course {course_id:"C001"})
SET r.grade = "A+", r.status = "completed"
```

```cypher
// remove a property
MATCH (s:student {student_id:"S002"})
REMOVE s.city            // same effect as SET s.city = null
```

```cypher
// add / remove a label
MATCH (s:student {student_id:"S001"}) SET s:alumni
MATCH (s:student {student_id:"S001"}) REMOVE s:alumni
```

> 📝 Instructor also said you can **change a relationship**, which in practice means (a) update its properties with `SET`, or (b) delete it and `MERGE` a new one. (Neo4j can't change a relationship's *type* in place.)

## 4.6 DELETE 📓📝

### Delete a node (and everything attached) 📓📝

```cypher
MATCH (s:student {name:$name})
DETACH DELETE s
```

```python
run_query(query, parameters={"name": "Rahul"})   # 📓 returned []
```

Result in class: **Rahul disappeared** from the student nodes (and his relationships with him).

**`DELETE` vs `DETACH DELETE`:**

| Command | Behaviour |
|---|---|
| `DELETE s` | Deletes the node **only if it has no relationships**, otherwise error |
| `DETACH DELETE s` | Deletes the node **and all its relationships** ✅ (what the class used) |

> 📝 Instructor: "detach delete means permanently it is going to delete the entire thing."

⚠️ Deleting by `name` deletes **every** node with that name. Prefer a unique ID: `MATCH (s:student {student_id:"S001"}) DETACH DELETE s`.

### Delete only a relationship ➕ (📝 *"I can delete our relationship as well"*)

```cypher
MATCH (:student {student_id:"S001"})-[r:ENROLLED_IN]->(:course {course_id:"C001"})
DELETE r
```

### Delete all nodes of a label ➕

```cypher
MATCH (s:student) DETACH DELETE s
```

### Wipe the whole database ➕ ⚠️ (dangerous!)

```cypher
MATCH (n) DETACH DELETE n
```

## 4.7 `MERGE` vs `CREATE` 📝➕

| | `CREATE` | `MERGE` |
|---|---|---|
| If pattern exists | **Creates a duplicate** | **Reuses it** (no duplicate) |
| If it doesn't exist | Creates it | Creates it |
| Use for | Guaranteed-new data | Idempotent loads, relationships |

> 📝 In class, `MERGE` for relationships meant re-running the query did **not** create a second `ENROLLED_IN` edge, it *"created the same relation… not a different relation."*

**`ON CREATE` / `ON MATCH`** ➕

```cypher
MERGE (s:student {student_id:"S010"})
  ON CREATE SET s.name = "Neha", s.created_at = datetime()
  ON MATCH  SET s.last_seen = datetime()
RETURN s
```

⚠️ Merge on the **unique key only** (e.g. `student_id`) and set other properties with `SET`. If you merge on the whole property map and any value differs, you'll get a new duplicate node.

## 4.8 `UNWIND` (list → rows) ➕

```cypher
UNWIND ["Python","SQL","Docker"] AS skill_name
MERGE (:skill {name: skill_name})
```

Batch relationships in one call:

```cypher
UNWIND $pairs AS p
MATCH (s:student {student_id:p.student})
MATCH (sk:skill  {name:p.skill})
MERGE (s)-[:HAS_SKILL]->(sk)
```

```python
run_query(query, parameters={"pairs": student_skills})
```

## 4.9 Constraints & indexes ➕

```cypher
// Prevent duplicate students (fixes the "ran it 3 times" problem)
CREATE CONSTRAINT student_id_unique IF NOT EXISTS
FOR (s:student) REQUIRE s.student_id IS UNIQUE

// Speed up name lookups
CREATE INDEX student_name_idx IF NOT EXISTS
FOR (s:student) ON (s.name)

SHOW CONSTRAINTS
SHOW INDEXES
```

## 4.10 Inspecting the graph ➕

```cypher
CALL db.labels()                 // list node labels
CALL db.relationshipTypes()      // list relationship types
CALL db.schema.visualization()   // draw the schema
MATCH (n) RETURN labels(n) AS label, count(*) AS total     // nodes per label
MATCH ()-[r]->() RETURN type(r) AS rel, count(*) AS total  // relationships per type
```

---

# 5. Walkthrough of `graphdbneo4j.ipynb`

| # | What the cell does | Concept |
|---|---|---|
| 1 | `pip install neo4j` (installed neo4j 6.3.1) | Install driver (use `%pip` in Jupyter) |
| 2 | `from neo4j import GraphDatabase` | Import |
| 3 | Sets `NEO4J_URI / USERNAME / PASSWORD` | Connection details, 🔐 move to env vars |
| 4 | `driver = GraphDatabase.driver(...)` | Create driver |
| 5 | `driver.verify_connectivity()` | Test connection |
| 6 | `run_query()` helper | Reusable wrapper around `execute_query` |
| 7 | `create(s:student{…})` with SUDH | First node |
| 8 | Same query for `shadiya` | Reuse query with new parameters |
| 9 | `students` list (S001–S004) + loop | Bulk create nodes (`for` loop) |
| 10 | `courses` list (C001–C003) + `create(c:course…)` | Second node type |
| 11 | `skills` list (6 skills) + `create(s:skill…)` | Third node type |
| 12 | `mentor` dict + `create(m:mentor…)` | Fourth node type |
| 13 | `MATCH … MATCH … MERGE (s)-[:ENROLLED_IN]->(c)` for S001/C001 | First relationship |
| 14 | Same, plus `SET r.enrolled_on, status, grade, remarks` | **Relationship properties** (`date()` function) |
| 15 | `enrollments` list + loop `MERGE` | Bulk relationships (⚠️ `S005` doesn't exist → silently skipped) |
| 16 | `student_skills` list + `MERGE (s)-[:HAS_SKILL]->(sk)` | Many-to-many relationships |
| 17 | `MATCH (s:student {name:$name})-[:HAS_SKILL]->(sk:skill) RETURN s, sk` | **Traversal query** (Rahul's skills) |
| 18 | `WHERE s.age <= $age AND s.age >= $min_age` | **Filtering** |
| 19 | `MATCH (s:student {name:$name}) DETACH DELETE s` | **Delete** Rahul |

**Resulting graph:**

```
(Rahul S001)──ENROLLED_IN──►(Agentic AI C001)     {enrolled_on, status, grade, remarks}
(Priya S002)──ENROLLED_IN──►(Data Science C002)
(Aman  S003)──ENROLLED_IN──►(Backend Eng. C003)
(Sneha S004)──ENROLLED_IN──►(Backend Eng. C003)

(Rahul)──HAS_SKILL──►(Python) (RAG) (LangGraph)
(Priya)──HAS_SKILL──►(Python) (Neo4j)
(Aman) ──HAS_SKILL──►(Python) (Machine Learning)
(Sneha)──HAS_SKILL──►(FastAPI) (Python)

(Mentor M001 Sudhanshu @ Euron)   ← node created, no relationships yet
```

**Notebook observations worth remembering:**
1. The first 2 students (`SUDH`, `shadiya`) were created **without `student_id`**, so they can never be matched by ID → shows schema flexibility *and* why you should keep a consistent unique key.
2. Cell 15's `S005` does not exist → that enrollment did nothing, silently.
3. After Rahul's deletion, his `ENROLLED_IN` and `HAS_SKILL` edges vanished too (because of `DETACH`).
4. The age filter output listed 7 students (SUDH, pavan, Hitesh, shadiya, Rahul, Priya, Sneha), since the DB also contained nodes created live in class.

---

# 6. Graph RAG

## 6.1 The idea 📝

> **Graph RAG = Vector database + Graph database working together.**
> - **Vector DB** → holds chunks, their metadata and **numeric embeddings** (for semantic similarity).
> - **Graph DB** → holds the **relationships** (document → chunk, chunk → entity, entity → entity …).
> - Combined, you get *"the power of both inside our entire system"* → **better retrieval**.

## 6.2 Recap: how a normal (vector) RAG stores data 📝

```
Document ──► split into CHUNKS ──► EMBEDDING model (LLM) ──► VECTORS ──► vector DB
```
(Vector DBs named in class: Qdrant, FAISS, Weaviate, Chroma; the transcript spelling is garbled: "coordinate", "FAIS", "Bay-V at chroma".)

Graph RAG adds: **store the relationships in Neo4j**, and optionally the chunk embedding as a property too (📝 *"chunk embedding, chunk vector also I can store, that's also possible"*).

## 6.3 Step-by-step from `graphrag.ipynb` 📓📝

**Step 1: the source document** (a supply-chain scenario)

```python
document = """
Apex Motion's Pune plant manufactures BMS Controller X1.
BMS Controller X1 requires MCU-900.
MCU-900 is supplied by Shenzhen Microelectronics.
Shenzhen Microelectronics depends on Taiwan Silicon Corp for semiconductor wafers.
Taiwan Silicon Corp operates a facility in Hsinchu.
The Hsinchu facility was affected by an earthquake.
Current estimated production delay is 14 days.
"""
```

**Step 2: same imports, credentials, `driver`, and `run_query`** (reused from the previous notebook).

**Step 3: build a dictionary for the document** (📝 *"all these names are fabricated by me"*, i.e. you choose the property names)

```python
document_data = {
    "document": "DOC001",
    "title":  "Apex Motion's Pune plant manufactures BMS Controller X1.",
    "source": "Apex Motion's Pune plant",
    "text":   document
}
```

**Step 4: store the whole document as ONE node**

```python
query = """create (d:Document {document:$document, title:$title, source:$source, text:$text})"""
run_query(query, parameters=document_data)     # returns [] because there is no RETURN
```

> Returning `[]` (blank) is expected: the query has no `RETURN` clause. 📝 The instructor noticed the blank result, then verified in the console that the node exists.

**Step 5: break the document into chunks**

```python
chunks = [
    {"chunk_id": "C001", "text": "Apex Motion's Pune plant manufactures BMS Controller X1."},
    {"chunk_id": "C002", "text": "BMS Controller X1 requires MCU-900."},
    {"chunk_id": "C003", "text": "MCU-900 is supplied by Shenzhen Microelectronics."},
    {"chunk_id": "C004", "text": "Shenzhen Microelectronics depends on Taiwan Silicon Corp for semiconductor wafers."},
    {"chunk_id": "C005", "text": "Taiwan Silicon Corp operates a facility in Hsinchu."},
    {"chunk_id": "C006", "text": "The Hsinchu facility was affected by an earthquake. Current estimated production delay is 14 days."},
]
```

(📝 "One line = one chunk" for this demo. The last two sentences were merged into C006.)

**Step 6: store each chunk as a node**

```python
query = """create (c:Chunk {chunk_id:$chunk_id, text:$text})"""
for chunk in chunks:
    run_query(query, parameters=chunk)
```

**Step 7: link the document to its chunks**

```python
query = """MATCH (d:Document {document:$document})
MATCH (c:Chunk {chunk_id:$chunk_id})
MERGE (d)-[:HAS_CHUNK]->(c)
"""
for chunk in chunks:
    run_query(query, parameters={"document": "DOC001", "chunk_id": chunk["chunk_id"]})
```

**Resulting graph:**

```
                    ┌─HAS_CHUNK─► (Chunk C001)
                    ├─HAS_CHUNK─► (Chunk C002)
(Document DOC001) ──┼─HAS_CHUNK─► (Chunk C003)
                    ├─HAS_CHUNK─► (Chunk C004)
                    ├─HAS_CHUNK─► (Chunk C005)
                    └─HAS_CHUNK─► (Chunk C006)
```

> 📝 The instructor's point: *"All the chunks belong to Document 001, so I can establish a relationship between the document and all its chunks."*

⚠️ **ID clash in the same DB:** `Chunk.chunk_id = "C001"` and `course.course_id = "C001"` look alike. It's harmless because the **labels differ** (`:Chunk` vs `:course`), which is exactly why you must **always include the label** in `MATCH`.

## 6.4 Real-world scaling of the idea 📝

- A PDF has **pages** → make `Page` nodes (PAGE 001, 002, 003…), each with **chunks**, plus tables & images.
- A department has many files/dossiers → `Department → File → Page → Chunk`.
- *"Anyone can come at any point and retrieve the relationship between a chunk and the document."*
- 📝 **You will not create these manually.** Parsing a PDF gives page numbers/metadata; **Python code creates all relationships in bulk** (classes, objects, functions). *"Baby steps now, but in projects nothing is done manually."*

## 6.5 Chunking strategies mentioned 📝

*(The transcript writes "checking", which is the auto-transcription of **chunking**.)*

- **Fixed-size chunking**
- **Sentence-based**
- **Paragraph-based**
- **Document-based**
- **Semantic chunking**
- **Window-based (sliding window) chunking**: e.g. 10–20 characters per chunk with an **overlap** between consecutive chunks, so chunks are created **dynamically**. Overlap keeps context from being cut at boundaries.
- The instructor teaches ~8–10 strategies in the dedicated RAG classes (later), along with how RAG works mathematically in the backend.

## 6.6 ➕ Taking the notebook one step further: the entity graph (where the real GraphRAG value is)

The notebook stops at Document → Chunk. The 7 sentences are really a **dependency chain**; extracting *entities* and *relationships* from each chunk turns it into a knowledge graph:

```cypher
// entities
MERGE (a:Company  {name:"Apex Motion Pune Plant"})
MERGE (p:Product  {name:"BMS Controller X1"})
MERGE (m:Part     {name:"MCU-900"})
MERGE (s1:Company {name:"Shenzhen Microelectronics"})
MERGE (s2:Company {name:"Taiwan Silicon Corp"})
MERGE (f:Facility {name:"Hsinchu Facility"})
MERGE (e:Event    {name:"Earthquake", delay_days:14})

// relationships
MERGE (a)-[:MANUFACTURES]->(p)
MERGE (p)-[:REQUIRES]->(m)
MERGE (m)-[:SUPPLIED_BY]->(s1)
MERGE (s1)-[:DEPENDS_ON]->(s2)
MERGE (s2)-[:OPERATES]->(f)
MERGE (e)-[:AFFECTED]->(f)

// link each entity back to the chunk it came from (provenance)
MATCH (c:Chunk {chunk_id:"C002"}), (p:Product {name:"BMS Controller X1"})
MERGE (c)-[:MENTIONS]->(p)
```

Now a question no single chunk can answer, *"Why might BMS Controller X1 be delayed?"*, becomes a **multi-hop traversal**:

```cypher
MATCH path = (p:Product {name:"BMS Controller X1"})-[*1..6]->(f:Facility)<-[:AFFECTED]-(e:Event)
RETURN [n IN nodes(path) | n.name] AS chain, e.delay_days
```

Vector search alone would retrieve isolated sentences by similarity; the **graph** connects *X1 → MCU-900 → Shenzhen → Taiwan Silicon → Hsinchu → Earthquake → 14 days*. That is the "power of both" (graph for relationships, vector for similarity).

---

# 7. SQL vs Cypher

## 7.1 Concept mapping

| SQL / RDBMS | Cypher / Neo4j | Class link |
|---|---|---|
| Database | Database | |
| **Table** | **Node label** (`:student`) | 📝 *"node name itself is a table"* |
| **Row / record** | **Node** | 📝 *"node as a record in a table"* |
| **Column** | **Property** | |
| Primary key | Unique property + uniqueness constraint | |
| **Foreign key + JOIN** | **Relationship** (stored, traversed) | 📝 no JOIN needed |
| Join table (many-to-many) | Just a relationship (can hold properties) | e.g. `ENROLLED_IN` with `grade` |
| Fixed schema | Flexible schema | |
| Row-wise scan + JOIN | Traverse neighbours from a start node | 📝 lower search complexity |
| `SELECT` | `RETURN` | |
| `FROM` | `MATCH` | |
| `WHERE` | `WHERE` | same 😀 |
| `GROUP BY` | *implicit* (non-aggregated items in `RETURN`) | |
| `HAVING` | `WITH … WHERE` | |
| `ORDER BY / LIMIT / OFFSET` | `ORDER BY / LIMIT / SKIP` | |
| `DISTINCT` | `DISTINCT` | |
| `LEFT JOIN` | `OPTIONAL MATCH` | |
| `INSERT INTO` | `CREATE` / `MERGE` | |
| `UPDATE … SET` | `MATCH … SET` | |
| `DELETE FROM` | `MATCH … DELETE` / `DETACH DELETE` | |
| `INSERT … ON DUPLICATE KEY` / upsert | `MERGE` | |
| `UNION` | `UNION` | |
| `COUNT/SUM/AVG/MIN/MAX` | same names (+ `collect()`) | |

## 7.2 Side-by-side query examples (same dataset)

**a) Insert a student**

```sql
INSERT INTO student (student_id, name, age, city) VALUES ('S005','Kiran',22,'Pune');
```
```cypher
CREATE (:student {student_id:'S005', name:'Kiran', age:22, city:'Pune'})
```

**b) Select all**

```sql
SELECT * FROM student;
```
```cypher
MATCH (s:student) RETURN s
```

**c) Filter**

```sql
SELECT name, age FROM student WHERE age <= 25 AND age >= 18;
```
```cypher
MATCH (s:student) WHERE s.age <= 25 AND s.age >= 18 RETURN s.name, s.age
```

**d) Group by**

```sql
SELECT city, COUNT(*) AS total FROM student GROUP BY city ORDER BY total DESC;
```
```cypher
MATCH (s:student) RETURN s.city AS city, count(*) AS total ORDER BY total DESC
```

**e) Update**

```sql
UPDATE student SET city='Pune' WHERE student_id='S002';
```
```cypher
MATCH (s:student {student_id:'S002'}) SET s.city = 'Pune'
```

**f) Delete**

```sql
DELETE FROM student WHERE name='Rahul';        -- FK rows must be handled first!
```
```cypher
MATCH (s:student {name:'Rahul'}) DETACH DELETE s   -- relationships go with it
```

**g) The JOIN vs pattern: "Which courses is Rahul enrolled in?"**

```sql
SELECT c.name
FROM student s
JOIN enrollment e ON e.student_id = s.student_id
JOIN course c     ON c.course_id  = e.course_id
WHERE s.name = 'Rahul';
```
```cypher
MATCH (s:student {name:'Rahul'})-[:ENROLLED_IN]->(c:course)
RETURN c.name
```
→ 3 tables + 2 JOINs in SQL, **1 drawn pattern** in Cypher.

**h) Students per skill (many-to-many)**

```sql
SELECT sk.name, COUNT(*) FROM student_skill ss JOIN skill sk ON ss.skill_id = sk.id GROUP BY sk.name;
```
```cypher
MATCH (:student)-[:HAS_SKILL]->(sk:skill) RETURN sk.name, count(*) 
```

**i) Friends of friends (painful in SQL, easy in Cypher)**

```sql
-- needs self-joins on the connection table, one per hop
```
```cypher
MATCH (me:Person {name:'Sudh'})-[:CONNECTED_TO*2]-(fof) RETURN DISTINCT fof.name
```

## 7.3 When to use which

| Prefer **SQL** | Prefer **Graph (Neo4j)** |
|---|---|
| Fixed, tabular data | Highly connected data |
| Heavy reporting / bulk aggregations | Deep relationship queries (multi-hop) |
| Strong transactions across simple entities | Recommendations, fraud, knowledge graphs, Graph RAG |
| Few relationships | Relationships are the *point* of the data |

---

# 8. Memory Tricks

## 🧠 Trick 1: "Nodes are NOUNS, Edges are VERBS"

- **Nouns** (things): Student, Course, Skill, Mentor → **nodes**
- **Verbs** (actions): ENROLLED_IN, HAS_SKILL, HAS_CHUNK, TEACHES → **relationships**
- Read a pattern as a sentence: `(Rahul)-[:ENROLLED_IN]->(Agentic AI)` = *"Rahul enrolled in Agentic AI."*

## 🧠 Trick 2: The shapes of the brackets

> **( ) round = node** (a circle/ball) · **[ ] square = relationship** (a rod/link) · **→ arrow = direction**
> *"Balls and sticks"*, like a molecule model: `(ball)-[stick]->(ball)`

## 🧠 Trick 3: Cypher reads like a sentence: **M-W-R**

**M**atch → **W**here → **R**eturn ("**M**y **W**ife **R**ocks" 😄)
= SQL's **F**rom → **W**here → **S**elect, but *reversed in order*: Cypher says **where to look first, what to show last**.

## 🧠 Trick 4: The write verbs: **C-M-S-D**

**C**reate · **M**erge · **S**et · **D**elete (`DETACH DELETE`)
Mnemonic: **"Cats Make Soft Dens."**
- Create = new · Merge = new-if-missing · Set = update · Delete = remove

## 🧠 Trick 5: The relationship recipe = **"MATCH twice, MERGE once"**

```cypher
MATCH (a)   // find first node
MATCH (b)   // find second node
MERGE (a)-[:REL]->(b)   // draw the line
```
*(If a MATCH finds nothing → nothing happens. Check spelling and case!)*

## 🧠 Trick 6: SQL → Cypher translator

| Say this in SQL | …think this in Cypher |
|---|---|
| **Table** | **Label** |
| **Row** | **Node** |
| **Column** | **Property** |
| **JOIN** | **Arrow** (`-[]->`) |
| **GROUP BY** | "Just add `count()` next to the column" |
| **HAVING** | "**WITH** + WHERE" (*W* for **W**ith → **W**here) |
| **LEFT JOIN** | **OPTIONAL MATCH** |

> One-liner: **"Tables become Labels, Joins become Arrows."**

## 🧠 Trick 7: CREATE vs MERGE

- **CREATE = "Copy machine"** 🖨️, every run makes another copy (duplicates!).
- **MERGE = "Find-or-Fix"** 🔎, looks first, creates only if missing.
- Rule: **Data loads & relationships → MERGE.**

## 🧠 Trick 8: DELETE vs DETACH DELETE

- **DELETE** = "remove the *person* but he's still holding hands" → error.
- **DETACH DELETE** = "let go of everyone's hands **first**, then remove." ✅

## 🧠 Trick 9: Degrees of connection (LinkedIn)

**1st = direct friend, 2nd = friend-of-friend.** In Cypher: `*1` = 1st, `*2` = 2nd, `*1..3` = up to 3rd.

## 🧠 Trick 10: Graph RAG = **Vector for "similar", Graph for "connected"**

> *"Vector finds what **looks like** it. Graph finds what's **linked to** it."*
> Chunks + embeddings → Vector DB. Document→Chunk→Entity links → Neo4j.

## 🧠 Trick 11: Debug checklist for "returned nothing" → **C-S-L-D**

**C**ase (`Rahul` vs `rahul`, `C001` vs `c001`) · **S**pelling of label/property · **L**abel exists? · **D**oes the node actually exist (like `S005`)?

---

# 9. Errors and Gotchas

| Symptom | Cause | Fix |
|---|---|---|
| `Missing expected parameter name, age, city` 📝 | Parameters dict not passed to the driver properly | Use `driver.execute_query(query, parameters_=params)` |
| Query runs but **returns `[]`** 📝 | (a) Case mismatch, (b) node/property doesn't exist, (c) no `RETURN` clause | Check exact values (`Rahul`), property names (`name` not `label`), add `RETURN` |
| Relationships not created in bulk 📝 | `MATCH` found nothing (wrong-case IDs, non-existent `S005`) | Verify IDs; use `RETURN count(*)` to confirm |
| **Duplicate nodes** 📝 | Re-ran `CREATE` (3 times → 3 nodes) | Use `MERGE` + uniqueness constraint; delete duplicates |
| Graph not showing in console 📝 | Not connected to the current instance in the UI | Select your running instance |
| Node not deleted / error on `DELETE` | Node still has relationships | Use `DETACH DELETE` |
| `MERGE` still creates duplicates | Merged on a full property map with differing values | Merge on the unique key, `SET` the rest |
| Notebook `SyntaxError` on `pip install neo4j` | `pip` is a shell command | Use `%pip install neo4j` or `!pip install neo4j` |
| Deleted wrong people | Matching on non-unique property such as `name` | Match by unique ID |
| Secrets leaked | Credentials hard-coded in notebook | Environment variables / `.env`, add to `.gitignore`, rotate password |
| Wrong-direction pattern returns nothing | `(c)-[:ENROLLED_IN]->(s)` instead of `(s)-[:ENROLLED_IN]->(c)` | Match the stored direction (or drop the arrowhead when reading) |

**Verify your work quickly:**

```cypher
MATCH (n) RETURN labels(n), count(*)            // node counts per label
MATCH ()-[r]->() RETURN type(r), count(*)       // relationship counts per type
```

---

# 10. One-Page Cheat Sheet

```cypher
// ── CREATE ───────────────────────────────────────────────
CREATE (s:student {name:$name, age:$age})           // node
MERGE  (s:student {student_id:$id})                 // node, no duplicates
MATCH (a:student {student_id:$s}), (b:course {course_id:$c})
MERGE (a)-[r:ENROLLED_IN]->(b) SET r.grade = "A"    // relationship (+ property)

// ── READ ─────────────────────────────────────────────────
MATCH (s:student) RETURN s                          // all
MATCH (s:student) WHERE s.age >= 18 AND s.city = "Delhi"
RETURN s.name AS n, s.age ORDER BY s.age DESC LIMIT 5
MATCH (s:student)-[:HAS_SKILL]->(sk:skill)          // traverse
RETURN s.name, collect(sk.name)
MATCH (s:student) RETURN s.city, count(*) AS total  // group by
MATCH (s)-[:HAS_SKILL]->(sk) WITH sk.name AS k, count(s) AS n WHERE n>=2 RETURN k,n   // having
MATCH (s:student) OPTIONAL MATCH (s)-[:ENROLLED_IN]->(c) RETURN s.name, c.name        // left join
MATCH (a)-[:KNOWS*1..3]-(b) RETURN DISTINCT b       // multi-hop

// ── UPDATE ───────────────────────────────────────────────
MATCH (s:student {student_id:$id}) SET s.city = "Pune", s.age = s.age + 1
MATCH (s:student {student_id:$id}) SET s += {city:"Pune", active:true}
MATCH (s:student {student_id:$id}) REMOVE s.city

// ── DELETE ───────────────────────────────────────────────
MATCH (s:student {student_id:$id}) DETACH DELETE s  // node + its edges
MATCH (a)-[r:ENROLLED_IN]->(b) DELETE r             // edge only
MATCH (n) DETACH DELETE n                           // ⚠️ everything

// ── BULK ─────────────────────────────────────────────────
UNWIND $rows AS row MERGE (s:student {student_id: row.student_id}) SET s += row

// ── ADMIN ────────────────────────────────────────────────
CREATE CONSTRAINT c1 IF NOT EXISTS FOR (s:student) REQUIRE s.student_id IS UNIQUE
CREATE INDEX i1 IF NOT EXISTS FOR (s:student) ON (s.name)
```

**Python skeleton**

```python
from neo4j import GraphDatabase
driver = GraphDatabase.driver(URI, auth=(USER, PWD)); driver.verify_connectivity()

def run_query(query, parameters=None):
    records, summary, keys = driver.execute_query(query, parameters_=parameters or {})
    return [r.data() for r in records]
```

---

# 11. Interview Q&A

**Q1. What is a graph database?**
A database that stores data as **nodes** (entities with properties) and **relationships** (typed, directed connections, which may also have properties), instead of tables or documents.

**Q2. Difference between node and edge?**
Node = entity that stores information. Edge = the relationship that connects two nodes; you name it yourself.

**Q3. What is Cypher?**
Neo4j's declarative query language; it uses ASCII-art patterns like `(a)-[:REL]->(b)`.

**Q4. How is a graph DB different from an RDBMS?** *(class answer 📝)*
RDBMS: multiple tables with primary/foreign keys → a limited set of predefined relationships, and row-wise search plus JOINs. Graph: custom relationships in any number, and search starts at a node and looks only at its connections → very low search complexity even for complex graphs.

**Q5. Why is traversal fast?**
Index-free adjacency: each node directly references its relationships, so cost depends on the neighbourhood you touch, not on total data size.

**Q6. `CREATE` vs `MERGE`?**
`CREATE` always creates (duplicates possible). `MERGE` = match-or-create (idempotent).

**Q7. `DELETE` vs `DETACH DELETE`?**
`DELETE` fails if the node still has relationships; `DETACH DELETE` removes the relationships too.

**Q8. How do you create a relationship between existing nodes?**
`MATCH` both nodes, then `MERGE (a)-[:TYPE]->(b)`.

**Q9. How do you do GROUP BY in Cypher?**
There's no keyword; non-aggregated columns in `RETURN`/`WITH` are the grouping keys: `RETURN s.city, count(*)`.

**Q10. What is Graph RAG?**
RAG that uses a **graph DB (relationships) together with a vector DB (embeddings)** for retrieval, giving richer, multi-hop, more accurate context to the LLM.

**Q11. Why store chunks in a graph if the vector DB already has them?**
The vector DB answers "what is *similar*?"; the graph answers "what is *connected*?" (document → page → chunk → entities). Combining them supports multi-hop reasoning and traceability.

**Q12. What are parameters (`$name`) and why use them?**
Placeholders supplied at execution time; they make queries reusable, are cleaner than string formatting, allow query-plan caching, and protect against Cypher injection.

**Q13. Are labels/property values case-sensitive?**
Yes (`Rahul` ≠ `rahul`; `student` ≠ `Student`). Keywords are not.

**Q14. Is Neo4j the only graph DB?**
No (the instructor noted other providers exist); Neo4j is the one used in the course.

**Q15. What is a chunking strategy? Name some.**
How you split a document before embedding: fixed-size, sentence, paragraph, document-level, semantic, sliding-window with overlap.

---

# 12. Practice Exercises

*(Use the notebook dataset. Answers are collapsed below.)*

1. Create a node `:student {student_id:"S006", name:"Zoya", age:22, city:"Delhi"}` using a parameterized query.
2. Enroll Zoya in `C002` with `status="active"`.
3. Give Zoya the skills `Python` and `Neo4j`.
4. List all students older than 23 living in Bengaluru.
5. Count students per city.
6. List each course with the number of enrolled students.
7. Which students share a course with Priya?
8. Which skills are known by more than one student?
9. Find students who have **no** skills.
10. Change Sneha's city to `Chennai`.
11. Remove Zoya's `HAS_SKILL` link to `Neo4j` only.
12. Delete Zoya completely.
13. Make `Sudhanshu (M001)` teach all three courses (`TEACHES`).
14. *(Graph RAG)* Add a `:Page` layer: `Document -[:HAS_PAGE]-> Page -[:HAS_CHUNK]-> Chunk` for the supply-chain document.
15. *(Graph RAG)* Add a `NEXT` relationship between consecutive chunks (`C001→C002→…`) to preserve reading order.

<details>
<summary>Answers</summary>

```cypher
// 1
CREATE (s:student {student_id:$student_id, name:$name, age:$age, city:$city})
// 2
MATCH (s:student {student_id:"S006"}), (c:course {course_id:"C002"})
MERGE (s)-[r:ENROLLED_IN]->(c) SET r.status = "active"
// 3
MATCH (s:student {student_id:"S006"})
UNWIND ["Python","Neo4j"] AS k
MATCH (sk:skill {name:k}) MERGE (s)-[:HAS_SKILL]->(sk)
// 4
MATCH (s:student) WHERE s.age > 23 AND s.city = "Bengaluru" RETURN s.name, s.age
// 5
MATCH (s:student) RETURN s.city, count(*) AS total
// 6
MATCH (c:course) OPTIONAL MATCH (s:student)-[:ENROLLED_IN]->(c) RETURN c.name, count(s) AS students
// 7
MATCH (:student {name:"Priya"})-[:ENROLLED_IN]->(c)<-[:ENROLLED_IN]-(o:student)
RETURN o.name, c.name
// 8
MATCH (s:student)-[:HAS_SKILL]->(sk:skill)
WITH sk.name AS skill, count(s) AS n WHERE n > 1 RETURN skill, n
// 9
MATCH (s:student) WHERE NOT (s)-[:HAS_SKILL]->() RETURN s.name
// 10
MATCH (s:student {name:"Sneha"}) SET s.city = "Chennai"
// 11
MATCH (:student {student_id:"S006"})-[r:HAS_SKILL]->(:skill {name:"Neo4j"}) DELETE r
// 12
MATCH (s:student {student_id:"S006"}) DETACH DELETE s
// 13
MATCH (m:mentor {mentor_id:"M001"}), (c:course)
MERGE (m)-[:TEACHES]->(c)
// 14
MATCH (d:Document {document:"DOC001"}), (c:Chunk)
MERGE (p:Page {page_id:"P001"})
MERGE (d)-[:HAS_PAGE]->(p)
MERGE (p)-[:HAS_CHUNK]->(c)
// 15
MATCH (a:Chunk {chunk_id:"C001"}), (b:Chunk {chunk_id:"C002"}) MERGE (a)-[:NEXT]->(b)
// (repeat, or build with UNWIND over consecutive pairs)
```

</details>

---

# 13. Other Points from the Class

## 13.1 Session flow (timeline) 📝

| Time | Topic |
|---|---|
| 00:05 | Intro: graph-based DB, why it matters for the upcoming real-time project |
| 00:06–00:12 | What is a graph, nodes/edges, LinkedIn example, comparison to tabular/Mongo/vector; recommendation systems |
| 00:12–00:23 | Neo4j Aura sign-up; create free instance; download credentials |
| 00:23–00:31 | Query console; write `run_query`; first `CREATE`; fix parameter error |
| 00:32–00:45 | Verify nodes in console; create Pavan/Hitesh/Sadia; duplicates when re-running; bulk insert with loop; courses, skills, mentor nodes |
| 00:46–01:03 | Relationships: `MATCH` + `MERGE`; `SET` relationship properties; bulk enrollments; debug case-sensitivity; student–skill relationships |
| 01:04–01:08 | Query: Rahul's skills; the query language is called **Cypher** |
| 01:08–01:14 | `WHERE` filters; node ≈ record, label ≈ table; `DETACH DELETE` |
| 01:15–01:31 | Graph RAG demo: Document node, Chunk nodes, `HAS_CHUNK` relationship, PDFs/pages idea |
| 01:32–01:34 | Q&A: graph vs RDBMS (functional + search complexity) |
| 01:34–01:38 | Q&A: dynamic (sliding window) chunking, other chunking strategies; manual relationships? (No, code/scripts do it in bulk) |
| 01:39–01:42 | Wrap-up and the project plan |

## 13.2 Instructor's closing summary 📝

> *"Graph is a very simple concept, it's all about **nodes and edges**. **Node** where we hold the data; **edges** where we create the relation **of my own, with my own choice**. And then every kind of operation you'll be able to do: **create, update, insert, delete**, everything."*

Advice: those who didn't follow → **re-watch the lecture and execute the notebooks yourself.** *"Just try to practice this entire thing on your own."*

## 13.3 What's coming next (course roadmap) 📝

- **Tomorrow:** design the project: use case, business benefit, team size, timeline, client requirements, tech stack (mock "forward-deployed engineer" style). Start building **locally**; it will include a **graph (Graph RAG)**, **databases**, **APIs**, a **front end**, and a little **vibe coding**. The project should be resume-worthy.
- Project is only *started* tomorrow (~2 classes to finish locally).
- **Then the Ops part:** Docker → Kubernetes → Terraform → Git/GitHub (not yet covered "in a proper manner") → CI/CD (multiple pipelines), multi-cloud (Azure, AWS, GCP).
- Then integrate ops with the project, then **system design** (security, scalability), then **agentic system design / AI architecture**.
- **Full RAG classes** (chunking strategies ×8–10, RAG math in the backend) come **in a couple of weeks**, after the above.
- Rule for projects: **nothing manual**, everything scripted in Python (classes, objects, functions).

## 13.4 Transcript-spelling decoder 🔤

The transcript is auto-generated, so some words are garbled:

| Transcript says | Actually means |
|---|---|
| "RG", "RIG", "Reg" | **RAG** |
| "checking" / "checking strategy" | **chunking** / chunking strategy |
| "Sudh / Dhanchu / Anshu / SEUDH" | the instructor **Sudhanshu** |
| "Hites" | **Hitesh** |
| "coordinate", "FAIS", "Bay-V at chroma" | probably **Qdrant, FAISS, Weaviate/Chroma** |
| "hedge skill(s)" | **HAS_SKILL** |
| "land graph" | **LangGraph** |
| "enrolled in" | `ENROLLED_IN` |
| "Euron" | the company/platform (mentor's company in the data) |
| "Neo4j"/"new 4J" | **Neo4j** |

---

### ✅ Final revision checklist

- [ ] I can explain node, edge, label, property, relationship type, direction
- [ ] I can create nodes (single, loop, `UNWIND`) with parameters
- [ ] I can create relationships with `MATCH … MATCH … MERGE`, with properties via `SET`
- [ ] I can read with `MATCH / WHERE / RETURN`, `ORDER BY`, `LIMIT`, aggregates, "group by", `WITH … WHERE`
- [ ] I can update (`SET`, `REMOVE`) and delete (`DELETE`, `DETACH DELETE`)
- [ ] I know `CREATE` vs `MERGE`, and why case-sensitivity bites
- [ ] I can map every SQL concept to Cypher
- [ ] I can explain Graph RAG: vector DB (similarity) + graph DB (relationships)
- [ ] I moved my Neo4j credentials out of the notebooks 🔐
