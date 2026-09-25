# Graph AI Engineer — 4-Day Interview Preparation Guide

**Target Role:** Graph AI Engineer  
**Focus:** Neo4j, Cypher, Knowledge Graphs, Graph Algorithms, Python, GraphRAG/RAG, LangChain, Amazon Bedrock, AWS Serverless, Neptune, MCP, Multi-Agent Systems  
**Candidate Positioning:** Neo4j Certified Professional with strong SQL / Snowflake background

---

# How to Use This Guide

This is a **4-day intensive interview-preparation plan** designed for a developer who already understands databases and SQL/Snowflake but needs to become interview-ready for a **Graph AI Engineer** position.

Each day contains:

1. Theory in simple language
2. SQL/Snowflake analogies
3. Memory tricks
4. Cypher syntax
5. Python examples
6. Graph AI / GraphRAG examples
7. Production interview points
8. Practice exercises
9. Interview questions and answers
10. End-of-day revision checklist

> **Recommended daily schedule**
>
> - 2 hours — concepts
> - 2 hours — Cypher/Python coding
> - 1.5 hours — GraphRAG/AWS
> - 1 hour — interview Q&A
> - 30 minutes — rapid revision

---

# 4-Day Roadmap

| Day | Main Goal | Topics |
|---|---|---|
| Day 1 | Think in graphs | Property graph, knowledge graph, modeling, Neo4j, Cypher fundamentals |
| Day 2 | Become strong in Cypher + GDS | Advanced Cypher, APOC, performance, graph algorithms, Python driver |
| Day 3 | Build Graph AI / GraphRAG thinking | Embeddings, vector search, RAG vs GraphRAG, LangChain, hybrid retrieval |
| Day 4 | Production + AWS + interview | Bedrock, Neptune, Lambda, API Gateway, security, architecture, mock interviews |

---

# DAY 1 — GRAPH FOUNDATIONS + KNOWLEDGE GRAPH + CYPHER

---

# Neo4j Graph Data Model Basics

## Nodes

Nodes are the **circles in a graph**.

They represent **objects or entities** in your data model.

Examples of entities include:

- People
- Locations
- Companies
- Products
- Accounts
- Events

In a social network, entities such as people, locations, and companies would be represented as nodes.

Each entity is stored as a separate node in the graph.

### Example

```text
(Person)
(Company)
(Location)
```

---

## Labels

Nodes are grouped or categorized using **labels**.

Labels describe **what the node is**.

Examples:

```text
Person
Company
Location
Product
Account
```

Nodes of the same type usually have the same label.

Labels help you:

- Distinguish between different types of nodes
- Filter the graph
- Organize your graph model
- Write more specific Cypher queries

### Multiple Labels

A node can have multiple labels.

For example, Michael can be both:

```text
Person
Employee
```

Conceptually:

```text
(:Person:Employee)
```

### Using Effective Node Labels

Nodes usually represent **things**, so labels should normally be:

- Singular
- Nouns

Good examples:

```text
Product
Event
Account
Customer
Employee
```

Avoid plural labels such as:

```text
Products
Events
Accounts
```

### Memory Trick

```text
Node = Thing
Label = Type of Thing
```

---

# Relationships

Relationships are the **lines connecting nodes** in a graph.

They describe **how nodes are connected to each other**.

A relationship in Neo4j connects two nodes:

```text
Start Node → Relationship → End Node
```

Example:

```text
(Michael)-[:WORKS_AT]->(Neo4j)
```

This means:

```text
Michael WORKS_AT Neo4j
```

It does **not** mean:

```text
Neo4j WORKS_AT Michael
```

## Every Relationship Has

### 1. A Type

The relationship type describes the connection.

Examples:

```text
WORKS_AT
FOUNDED_IN
KNOWS
OWNS
LIVES_IN
RATED
```

### 2. A Direction

Relationships have direction.

Example:

```text
(Person)-[:WORKS_AT]->(Company)
```

Direction matters because:

```text
Michael WORKS_AT Neo4j
```

is different from:

```text
Neo4j WORKS_AT Michael
```

## Multiple Relationships

A node can have multiple relationships with other nodes.

Example:

```text
(Person)-[:WORKS_AT]->(Company)
(Person)-[:LIVES_IN]->(Location)
(Person)-[:OWNS]->(Car)
```

## Bi-Directional Relationships

If a relationship should exist in both directions, you may need two separate relationships.

Example:

```text
(Michael)-[:LOVES]->(Sarah)
(Sarah)-[:LOVES]->(Michael)
```

You should not assume that because:

```text
Michael LOVES Sarah
```

that:

```text
Sarah LOVES Michael
```

is also true.

## Use Verbs for Relationship Types

Relationship types should normally be **verbs or verb phrases**.

Examples:

### Personal Connections

```text
(Person)-[:KNOWS]->(Person)
(Person)-[:MARRIED_TO]->(Person)
```

### Facts

```text
(Person)-[:LIVES_IN]->(Location)
(Person)-[:OWNS]->(Car)
(Person)-[:RATED]->(Movie)
```

### Hierarchies

```text
(Parent)-[:PARENT_OF]->(Child)
(Software)-[:DEPENDS_ON]->(Library)
```

### General Connections

```text
(Entity)-[:CONNECTED_TO]->(Entity)
```

### Memory Trick

```text
Node = Noun
Relationship = Verb
```

---

# Properties

Properties store additional data about **nodes and relationships**.

A property is a **key-value pair**.

Example:

```text
firstName = "Michael"
lastName = "Smith"
position = "Engineer"
```

## Node Properties

Example:

```cypher
(:Person {
    firstName: "Michael",
    lastName: "Smith",
    age: 35
})
```

Here:

```text
firstName
lastName
age
```

are property keys.

## Relationship Properties

Relationships can also have properties.

Example:

```cypher
(:Person)-[:WORKS_AT {
    since: 2022,
    position: "Engineer"
}]->(:Company)
```

Here:

```text
since
position
```

are properties of the relationship.

## Property Types

Properties can store different data types.

Examples:

```text
String
Integer
Float
Boolean
Date
DateTime
List
```

Example:

```cypher
(:Product {
    productId: "P101",
    name: "Printer",
    price: 199.99,
    available: true
})
```

## Flexible Schema

Neo4j has a flexible schema.

Nodes with the same label do not necessarily need to have exactly the same properties.

Example:

```text
Person 1
firstName
lastName
age
```

```text
Person 2
firstName
lastName
email
```

Both can still have the label:

```text
Person
```

## Unique Identifiers

Properties can also be used as unique identifiers.

Example:

```text
Customer.customerId
Product.productId
Order.orderId
```

A uniqueness constraint can be created to ensure the value is unique.

Example:

```cypher
CREATE CONSTRAINT customer_id_unique IF NOT EXISTS
FOR (c:Customer)
REQUIRE c.customerId IS UNIQUE;
```

---

# Complete Example

```text
(Customer)
    |
    | PLACED
    ▼
(Order)
    |
    | CONTAINS
    ▼
(Product)
```

With properties:

```cypher
(:Customer {
    customerId: "C101",
    name: "Alice"
})
```

```cypher
(:Order {
    orderId: "O1001",
    orderDate: date("2026-09-25")
})
```

```cypher
(:Product {
    productId: "P500",
    name: "Printer Paper",
    price: 12.99
})
```

Relationship properties:

```cypher
(:Order)-[:CONTAINS {
    quantity: 3,
    unitPrice: 12.99
}]->(:Product)
```

---

# Quick Summary

| Concept | Meaning | Example |
|---|---|---|
| Node | Entity or object | Customer |
| Label | Type/category of node | `:Customer` |
| Relationship | Connection between nodes | `PLACED` |
| Relationship Type | Meaning of the connection | `WORKS_AT` |
| Direction | Start node to end node | `Person → Company` |
| Property | Key-value data | `name: "Alice"` |
| Constraint | Rule for data integrity | Unique customerId |

()        Node

:         Label / relationship type

{}        Properties

[]        Relationship

->        Direction

$         Parameter

.         Property access

*1..3     Multi-hop traversal


---

# Memory Tricks

## Nodes

```text
Node = Thing
```

Examples:

```text
Customer
Product
Employee
```

## Labels

```text
Label = Type of Thing
```

Examples:

```text
Person
Company
Product
```

## Relationships

```text
Relationship = Verb
```

Examples:

```text
WORKS_AT
PLACED
CONTAINS
SUPPLIED_BY
```

## Properties

```text
Property = Details
```

Examples:

```text
name
price
age
customerId
```

---

# Final Memory Formula

```text
N L R P
```

- **N** = Node
- **L** = Label
- **R** = Relationship
- **P** = Property

Think:

```text
Thing
Type
Connection
Details
```


## 1. Why Graph Databases?

A relational database stores information mainly as rows and tables.

A graph database stores:

- **Nodes** — things
- **Relationships** — how things are connected
- **Properties** — attributes about nodes and relationships
- **Labels** — categories/types of nodes

Example:

```text
(Kareema:Person)-[:WORKED_AT]->(Company)
(Company)-[:USES]->(Neo4j:Technology)
```

In a graph database, the relationship is a first-class object.

## Snowflake/SQL Analogy

Relational model:

```text
CUSTOMER
ORDER
PRODUCT
ORDER_PRODUCT
```

To find products bought by friends of a customer, SQL may require many JOINs.

Graph model:

```text
(:Customer)-[:BOUGHT]->(:Product)
(:Customer)-[:FRIEND_OF]->(:Customer)
```

Traversal follows the relationships directly.

### Memory Trick

**Graph = NRP**

- **N**odes
- **R**elationships
- **P**roperties

---

# 2. Property Graph Model

Example:

```text
(:Person {
    id: 101,
    name: "Alice",
    age: 35
})
```

Relationship:

```text
(:Person)-[:WORKS_FOR {
    since: 2021
}]->(:Company)
```

### Key Terms

| Term | Meaning |
|---|---|
| Node | Entity |
| Label | Entity category/type |
| Relationship | Connection |
| Relationship type | Meaning of connection |
| Property | Attribute |
| Path | Sequence of nodes + relationships |
| Degree | Number of connected relationships |

---

# 3. Knowledge Graph

A **knowledge graph** represents real-world entities, their properties, and meaningful relationships.

Example:

```text
(Customer)-[:PURCHASED]->(Product)
(Product)-[:BELONGS_TO]->(Category)
(Customer)-[:LIVES_IN]->(City)
(Product)-[:MANUFACTURED_BY]->(Company)
```

This allows questions such as:

```text
Which customers in Atlanta bought products manufactured
by companies connected to a particular supplier?
```

## Knowledge Graph vs Ordinary Graph

A normal graph stores connected data.

A knowledge graph usually adds:

- semantic meaning
- well-defined entity types
- relationship meaning
- ontology/schema
- business vocabulary
- identifiers
- provenance
- sometimes reasoning/inference

### Memory Trick

**Knowledge Graph = Graph + Meaning**

---

# 4. Ontology

An ontology describes the vocabulary and rules of a domain.

Example healthcare ontology:

```text
Patient
Doctor
Hospital
Diagnosis
Medication
```

Relationships:

```text
Patient -[HAS_DIAGNOSIS]-> Diagnosis
Doctor -[TREATS]-> Patient
Patient -[TAKES]-> Medication
Doctor -[WORKS_AT]-> Hospital
```

Ontology tells us:

- what types of entities exist
- what relationships are valid
- what properties belong to each concept

## Interview Answer

**Q: What is an ontology?**

An ontology is a formal representation of domain concepts and the relationships between those concepts. In a knowledge graph it acts as a semantic model that defines what entities exist, how they are related, and sometimes constraints or hierarchical relationships between concepts.

---

# 5. Schema-Optional Does Not Mean Schema-Free

Neo4j is schema-optional.

You can create nodes without defining tables first.

But production applications should still define:

- labels
- relationship types
- property names
- uniqueness rules
- constraints
- indexes
- naming standards

Example:

```cypher
CREATE CONSTRAINT customer_id_unique
FOR (c:Customer)
REQUIRE c.customerId IS UNIQUE;
```

Interview phrase:

> Neo4j gives flexible schema evolution, but production graphs should still have a governed logical schema.

---

# 6. Graph Modeling — Most Important Interview Topic

Bad relational thinking:

```text
Customer table
Order table
Product table
Supplier table
```

Graph thinking:

```text
(:Customer)-[:PLACED]->(:Order)
(:Order)-[:CONTAINS]->(:Product)
(:Product)-[:SUPPLIED_BY]->(:Supplier)
```

## Modeling Rule

**Nouns usually become nodes.**

**Verbs usually become relationships.**

Example sentence:

> Customer purchased Product.

Graph:

```text
(:Customer)-[:PURCHASED]->(:Product)
```

### Memory Trick

**Noun = Node**

**Verb = Relationship**

---

# 7. When Should Something Be a Property Instead of a Node?

Use a property when the value:

- does not need independent relationships
- does not need traversal
- is usually just descriptive

Example:

```text
(:Customer {age: 35})
```

Use a node when it:

- is shared by multiple entities
- has relationships
- has its own properties
- is important for traversal

Instead of:

```text
(:Customer {city: "Atlanta"})
```

you may model:

```text
(:Customer)-[:LIVES_IN]->(:City {name:"Atlanta"})
```

if city is important to graph analysis.

---

# 8. Relationship Direction

Example:

```cypher
(:Person)-[:WORKS_FOR]->(:Company)
```

Read:

> Person WORKS_FOR Company

Query directionally:

```cypher
MATCH (p:Person)-[:WORKS_FOR]->(c:Company)
RETURN p, c;
```

Or ignore direction:

```cypher
MATCH (p:Person)-[:KNOWS]-(friend:Person)
RETURN friend;
```

---

# 9. Cypher Mental Model

SQL:

```sql
SELECT columns
FROM tables
JOIN ...
WHERE ...
GROUP BY ...
```

Cypher:

```text
MATCH pattern
WHERE filter
RETURN result
```

### Memory Trick

**M-W-R**

- MATCH
- WHERE
- RETURN


Snowflake                     Neo4j
------------------------------------------------
DATABASE          →           DATABASE

SCHEMA            →           No direct equivalent

TABLE             →           LABEL

ROW               →           NODE

COLUMN            →           PROPERTY

PRIMARY KEY       →           UNIQUENESS / KEY CONSTRAINT

FOREIGN KEY       →           RELATIONSHIP

INDEX             →           INDEX

INFORMATION_SCHEMA
                  →           SHOW commands + metadata procedures



# Neo4j CREATE Statement

`CREATE` creates the **node**.

Example:

```cypher
CREATE (p:Person {
    name: "Alice",
    age: 30
});
```

Here:

- `CREATE` → creates a new **node**
- `Person` → is the **label**
- `name` and `age` → are **properties**
- `p` → is a **query variable**

Result:

```text
(:Person {
    name: "Alice",
    age: 30
})
```

Neo4j does **not** require you to create the label separately.

When you create the node with:

```cypher
:Person
```

the `Person` label is automatically used.

---

# Is Neo4j / Cypher Case-Sensitive?

Neo4j/Cypher is **partly case-sensitive**.

## Cypher Keywords

Cypher keywords are **not case-sensitive**.

All of these work:

```cypher
MATCH
match
Match
```

Best practice is to write keywords in uppercase:

```cypher
MATCH
WHERE
RETURN
CREATE
MERGE
```

---

## Labels

Labels are **case-sensitive**.

```cypher
:Person
```

and

```cypher
:person
```

are different.

---

## Relationship Types

Relationship types are **case-sensitive**.

```cypher
:WORKS_AT
```

and

```cypher
:works_at
```

are different.

---

## Property Names

Property names are **case-sensitive**.

```cypher
p.name
```

and

```cypher
p.Name
```

are different.

---

## String Values

String comparisons are also case-sensitive.

```text
"Alice"
```

and

```text
"alice"
```

are different values.

Example:

```cypher
MATCH (p:Person)
WHERE p.name = "Alice"
RETURN p;
```

This may not match data stored as:

```text
Label: person
Property: Name
Value: alice
```

---

# Memory Trick

```text
Cypher Keywords → NOT case-sensitive

Labels          → Case-sensitive
Relationship    → Case-sensitive
Properties      → Case-sensitive
String Values   → Case-sensitive
```

---

# 10. CREATE

```cypher
CREATE (p:Person {
    name: "Alice",
    age: 30
});
```

Create relationship:

```cypher
MATCH (p:Person {name:"Alice"})
MATCH (c:Company {name:"OpenAI"})
CREATE (p)-[:WORKS_FOR]->(c);
```

---

# 11. MATCH

```cypher
MATCH (p:Person)
RETURN p;
```

Filter:

```cypher
MATCH (p:Person)
WHERE p.age > 30
RETURN p.name, p.age;
```

Compact:

```cypher
MATCH (p:Person {name:"Alice"})
RETURN p;
```

---

# 12. RETURN

```cypher
MATCH (p:Person)
RETURN p.name, p.age;
```

Alias:

```cypher
MATCH (p:Person)
RETURN p.name AS employee_name;
```

---

# 13. ORDER BY, LIMIT, SKIP

```cypher
MATCH (p:Person)
RETURN p.name, p.age
ORDER BY p.age DESC
LIMIT 10;
```

Pagination:

```cypher
MATCH (p:Person)
RETURN p
SKIP 20
LIMIT 10;
```

---

# 14. WHERE

```cypher
MATCH (p:Person)
WHERE p.age >= 30
  AND p.city = "Atlanta"
RETURN p;
```

String filtering:

```cypher
MATCH (p:Person)
WHERE p.name STARTS WITH "A"
RETURN p.name;
```

Other useful operators:

```text
CONTAINS
STARTS WITH
ENDS WITH
IN
IS NULL
IS NOT NULL
```

---

# 15. MERGE

`MERGE` means:

> Match this pattern if it exists; otherwise create it.

Example:

```cypher
MERGE (p:Person {email:"alice@example.com"})
RETURN p;
```

Add properties only during creation:

```cypher
MERGE (p:Person {email:"alice@example.com"})
ON CREATE SET
    p.createdAt = datetime(),
    p.name = "Alice"
ON MATCH SET
    p.lastSeen = datetime();
```

### SQL Analogy

MERGE is similar in spirit to:

```text
UPSERT
```

but Cypher MERGE works on graph patterns.

### Interview Trap

Do not say MERGE simply means SQL MERGE.

Cypher MERGE can match/create a whole graph pattern.

---

# 16. SET

```cypher
MATCH (p:Person {name:"Alice"})
SET p.age = 31
RETURN p;
```

Add a label:

```cypher
MATCH (p:Person {name:"Alice"})
SET p:Employee;
```

---

# 17. REMOVE

```cypher
MATCH (p:Person {name:"Alice"})
REMOVE p.age;
```

Remove label:

```cypher
MATCH (p:Person {name:"Alice"})
REMOVE p:Employee;
```

---

# 18. DELETE

Delete node with no relationships:

```cypher
MATCH (p:Person {name:"Alice"})
DELETE p;
```

Delete connected node:

```cypher
MATCH (p:Person {name:"Alice"})
DETACH DELETE p;
```

### Memory Trick

**DETACH DELETE = cut relationships + delete node**

---

# 19. OPTIONAL MATCH

SQL equivalent:

```text
LEFT OUTER JOIN
```

Cypher:

```cypher
MATCH (p:Person)
OPTIONAL MATCH (p)-[:WORKS_FOR]->(c:Company)
RETURN p.name, c.name;
```

If the company does not exist, the person row remains and company values become `null`.

---

# 20. WITH

`WITH` passes results from one query stage to another.

Think of it as:

> intermediate result / SQL CTE-like pipeline stage

Example:

```cypher
MATCH (p:Person)-[:PURCHASED]->(product:Product)
WITH p, count(product) AS productCount
WHERE productCount > 5
RETURN p.name, productCount;
```

### Memory Trick

**WITH = pipe**

```text
MATCH → WITH → MATCH → RETURN
```

---

# 21. Aggregations

```cypher
MATCH (c:Customer)-[:PURCHASED]->(p:Product)
RETURN c.name, count(p) AS purchases;
```

Functions:

```text
count()
sum()
avg()
min()
max()
collect()
```

Example:

```cypher
MATCH (c:Customer)-[:PURCHASED]->(p:Product)
RETURN c.name, collect(p.name) AS products;
```

---

# 22. UNWIND

`UNWIND` turns a list into rows.

Input:

```text
["Neo4j", "Python", "AWS"]
```

Cypher:

```cypher
UNWIND ["Neo4j", "Python", "AWS"] AS skill
RETURN skill;
```

Output:

```text
Neo4j
Python
AWS
```

Very useful for bulk insert:

```cypher
UNWIND $customers AS row
MERGE (c:Customer {customerId: row.customerId})
SET c.name = row.name;
```

### Memory Trick

**UNWIND = explode list into rows**

Snowflake analogy:

```text
FLATTEN()
```

---

# 23. Paths

```cypher
MATCH path =
    (a:Person {name:"Alice"})-[:KNOWS*1..3]->(b:Person)
RETURN path;
```

`*1..3` means one to three hops.

---

# 24. Two Degrees of Separation

Question:

> Find people exactly two relationships away from Alice but not directly connected.

```cypher
MATCH (alice:Person {name:"Alice"})-[:KNOWS]->(:Person)-[:KNOWS]->(candidate:Person)
WHERE candidate <> alice
  AND NOT (alice)-[:KNOWS]-(candidate)
RETURN DISTINCT candidate.name;
```

### Interview Logic

1. Alice → friend
2. friend → candidate
3. candidate is not Alice
4. candidate is not directly connected to Alice
5. DISTINCT removes duplicates

---

# 25. EXISTS Subquery

```cypher
MATCH (c:Customer)
WHERE EXISTS {
    MATCH (c)-[:PURCHASED]->(:Product {category:"Laptop"})
}
RETURN c.name;
```

This is useful when you care whether a pattern exists.

---

# 26. CALL Subquery

```cypher
MATCH (c:Customer)
CALL (c) {
    MATCH (c)-[:PURCHASED]->(p:Product)
    RETURN count(p) AS purchaseCount
}
RETURN c.name, purchaseCount;
```

Use subqueries for:

- isolation
- complex aggregation
- per-row logic
- readable modular Cypher

---

# 27. Constraints

Unique constraint:

```cypher
CREATE CONSTRAINT customer_id_unique IF NOT EXISTS
FOR (c:Customer)
REQUIRE c.customerId IS UNIQUE;
```

Check constraints:

```cypher
SHOW CONSTRAINTS;
```

Why constraints matter:

- data quality
- identity
- MERGE correctness
- performance benefits in some cases

---

# 28. Indexes

```cypher
CREATE INDEX customer_name_index IF NOT EXISTS
FOR (c:Customer)
ON (c.name);
```

Check:

```cypher
SHOW INDEXES;
```

Indexes help avoid scanning every node.

---

# 29. Query Plan

Use:

```cypher
EXPLAIN
MATCH (c:Customer {customerId:"C101"})
RETURN c;
```

`EXPLAIN` shows plan without executing.

Use:

```cypher
PROFILE
MATCH (c:Customer {customerId:"C101"})
RETURN c;
```

`PROFILE` executes and shows runtime statistics.

### Memory Trick

**EXPLAIN = plan only**

**PROFILE = plan + execution**

---

# 30. Day 1 Mini Project — Customer Product Knowledge Graph

Create:

```cypher
CREATE
(c1:Customer {id:"C1", name:"Alice"}),
(c2:Customer {id:"C2", name:"Bob"}),
(c3:Customer {id:"C3", name:"Carol"}),

(p1:Product {id:"P1", name:"Laptop", category:"Electronics"}),
(p2:Product {id:"P2", name:"Mouse", category:"Electronics"}),
(p3:Product {id:"P3", name:"Tennis Racket", category:"Sports"}),

(c1)-[:PURCHASED {amount:1200}]->(p1),
(c1)-[:PURCHASED {amount:50}]->(p2),
(c2)-[:PURCHASED {amount:1100}]->(p1),
(c2)-[:PURCHASED {amount:180}]->(p3),
(c3)-[:PURCHASED {amount:45}]->(p2);
```

## Practice Query 1

Find products purchased by Alice.

```cypher
MATCH (:Customer {name:"Alice"})-[:PURCHASED]->(p:Product)
RETURN p.name;
```

## Practice Query 2

Find customers who purchased Laptop.

```cypher
MATCH (c:Customer)-[:PURCHASED]->(:Product {name:"Laptop"})
RETURN c.name;
```

## Practice Query 3

Find customers who purchased the same product as Alice.

```cypher
MATCH (alice:Customer {name:"Alice"})-[:PURCHASED]->(p:Product)<-[:PURCHASED]-(other:Customer)
WHERE other <> alice
RETURN DISTINCT other.name, p.name;
```

## Practice Query 4

Most purchased product:

```cypher
MATCH (:Customer)-[:PURCHASED]->(p:Product)
RETURN p.name, count(*) AS purchases
ORDER BY purchases DESC
LIMIT 1;
```

---

# DAY 1 INTERVIEW QUESTIONS

## Q1. Why use Neo4j instead of a relational database?

Use Neo4j when relationships are central to the problem and queries involve multiple hops or highly connected data. In relational databases these queries often require many joins. Neo4j stores relationships directly and makes traversal more natural. Relational databases remain strong for tabular analytics and transaction patterns that do not depend heavily on graph traversal.

---

## Q2. What is a node?

A node represents an entity such as Customer, Product, Employee, Account, Document, or Concept.

---

## Q3. What is a relationship?

A relationship connects two nodes and has a type, direction, and optionally properties.

---

## Q4. Can relationships have properties?

Yes.

```cypher
(:Customer)-[:PURCHASED {
    orderDate: date(),
    amount: 100
}]->(:Product)
```

---

## Q5. What is a label?

A label classifies a node.

```cypher
(:Customer)
(:Product)
(:Document)
```

---

## Q6. What is a path?

A path is a sequence of connected nodes and relationships.

---

## Q7. What is a knowledge graph?

A knowledge graph represents domain entities and semantically meaningful relationships in a connected form, usually governed by a domain model or ontology.

---

## Q8. What is an ontology?

An ontology formally describes concepts, relationships, and semantic rules in a domain.

---

## Q9. What is the difference between CREATE and MERGE?

`CREATE` always creates new data. `MERGE` attempts to match a pattern first and creates it only if it does not exist.

---

## Q10. What is OPTIONAL MATCH?

It preserves rows even when the optional graph pattern does not exist, similar to SQL LEFT OUTER JOIN.

---

## Q11. What does WITH do?

It creates a pipeline boundary and passes selected variables or aggregations to the next query stage.

---

## Q12. What is UNWIND?

It converts list elements into individual rows, useful for list processing and bulk ingestion.

---

## DAY 1 REVISION CHECKLIST

You should be able to explain without notes:

- [ ] Node
- [ ] Relationship
- [ ] Label
- [ ] Property
- [ ] Property graph
- [ ] Knowledge graph
- [ ] Ontology
- [ ] CREATE
- [ ] MERGE
- [ ] MATCH
- [ ] WHERE
- [ ] OPTIONAL MATCH
- [ ] WITH
- [ ] UNWIND
- [ ] Aggregation
- [ ] Paths
- [ ] Constraints
- [ ] Indexes
- [ ] EXPLAIN vs PROFILE

---

# DAY 2 — ADVANCED CYPHER + APOC + GDS + PYTHON

---

# 31. Advanced Pattern Matching

Suppose:

```text
Customer → Order → Product → Category
```

Query:

```cypher
MATCH (c:Customer)-[:PLACED]->(o:Order)
      -[:CONTAINS]->(p:Product)
      -[:BELONGS_TO]->(cat:Category)
WHERE c.id = "C101"
RETURN p.name, cat.name;
```

Cypher is powerful because the query visually resembles the graph.

---

# 32. Variable Length Paths

```cypher
MATCH (a:Person {name:"Alice"})-[:KNOWS*1..3]->(person)
RETURN DISTINCT person.name;
```

Means:

```text
1 hop
2 hops
3 hops
```

### Production Warning

Unlimited variable-length traversals can become expensive.

Avoid casually doing:

```cypher
[:KNOWS*]
```

on a huge highly connected graph.

Use bounded traversal:

```cypher
[:KNOWS*1..4]
```

---

# 33. Shortest Path

```cypher
MATCH (a:Person {name:"Alice"}),
      (b:Person {name:"David"})
MATCH p = shortestPath((a)-[:KNOWS*]-(b))
RETURN p;
```

Use case:

- social network connection
- fraud linkage
- supply chain connection
- dependency analysis

---

# 34. Pattern Comprehension

Example idea:

```cypher
MATCH (c:Customer)
RETURN c.name,
       [(c)-[:PURCHASED]->(p:Product) | p.name] AS products;
```

This returns a list derived from matching a pattern.

---

# 35. CASE

```cypher
MATCH (c:Customer)
RETURN c.name,
       CASE
         WHEN c.score >= 800 THEN "Excellent"
         WHEN c.score >= 700 THEN "Good"
         ELSE "Review"
       END AS riskBand;
```

---

# 36. List Functions

Useful functions:

```text
size()
head()
tail()
range()
reduce()
```

Example:

```cypher
WITH [1,2,3,4] AS nums
RETURN size(nums);
```

---

# 37. APOC

APOC stands for:

> Awesome Procedures On Cypher

It extends Neo4j with many utility procedures/functions.

Common use cases:

- data transformation
- dynamic Cypher
- JSON processing
- import/export
- collections
- graph refactoring
- utility functions

Example JSON conversion:

```cypher
RETURN apoc.convert.toJson({
    name: "Alice",
    skills: ["Neo4j", "Python"]
});
```

List utility example:

```cypher
RETURN apoc.coll.toSet(["Neo4j","AWS","Neo4j"]);
```

### Interview Position

Do not say APOC replaces Cypher.

Say:

> I prefer native Cypher when possible and use APOC when it simplifies specialized transformation, integration, collection, import/export, or utility operations.

---

# 38. Query Optimization

Important rules:

1. Filter early.
2. Use indexes for lookup properties.
3. Use labels.
4. Avoid unnecessary Cartesian products.
5. Limit unbounded traversals.
6. Return only required data.
7. Parameterize queries.
8. Inspect PROFILE.
9. Batch large writes.
10. Avoid loading massive result sets into application memory.

Bad:

```cypher
MATCH (c),(p)
WHERE c.id = "C1"
RETURN c,p;
```

This may create a Cartesian product.

Better:

```cypher
MATCH (c:Customer {id:$customerId})-[:PURCHASED]->(p:Product)
RETURN p;
```

---

# 39. Parameters

Never construct Cypher like this:

```python
query = "MATCH (p:Person {name:'" + name + "'}) RETURN p"
```

Better:

```python
query = """
MATCH (p:Person {name:$name})
RETURN p
"""
```

Then:

```python
session.run(query, name=name)
```

Benefits:

- safer
- reusable query plans
- cleaner code
- protection against query injection patterns

---

# 40. Query Cache Interview Point

To improve query plan reuse:

- use parameters
- keep query structure consistent
- avoid replacing parameters with many different literals unnecessarily

Example:

```cypher
MATCH (c:Customer {customerId:$customerId})
RETURN c;
```

---

# 41. Python Neo4j Driver

Install:

```bash
pip install neo4j
```

Connection:

```python
from neo4j import GraphDatabase

URI = "neo4j+s://your-host"
AUTH = ("neo4j", "password")

driver = GraphDatabase.driver(
    URI,
    auth=AUTH
)

driver.verify_connectivity()

print("Connected")
```

Always close:

```python
driver.close()
```

Or:

```python
with GraphDatabase.driver(URI, auth=AUTH) as driver:
    driver.verify_connectivity()
```

---

# 42. Read Query in Python

```python
from neo4j import GraphDatabase

def get_customer(driver, customer_id):
    query = """
    MATCH (c:Customer {id:$customer_id})
    RETURN c.id AS id,
           c.name AS name
    """

    with driver.session() as session:
        record = session.run(
            query,
            customer_id=customer_id
        ).single()

        return dict(record) if record else None
```

---

# 43. Recommended Transaction Function Style

```python
def find_customer(tx, customer_id):
    result = tx.run(
        """
        MATCH (c:Customer {id:$customer_id})
        RETURN c.name AS name
        """,
        customer_id=customer_id
    )
    return result.single()

with driver.session() as session:
    record = session.execute_read(
        find_customer,
        "C101"
    )
```

Write:

```python
def create_customer(tx, customer_id, name):
    tx.run(
        """
        MERGE (c:Customer {id:$customer_id})
        SET c.name = $name
        """,
        customer_id=customer_id,
        name=name
    )

with driver.session() as session:
    session.execute_write(
        create_customer,
        "C101",
        "Alice"
    )
```

---

# 44. Bulk Insert from Python

```python
customers = [
    {"id":"C1", "name":"Alice"},
    {"id":"C2", "name":"Bob"},
    {"id":"C3", "name":"Carol"}
]

query = """
UNWIND $rows AS row
MERGE (c:Customer {id:row.id})
SET c.name = row.name
"""

with driver.session() as session:
    session.run(query, rows=customers)
```

### Interview Point

Prefer `UNWIND` batching over one network round trip per row.

---

# 45. Loading Relational/Snowflake Data into a Graph

Imagine Snowflake tables:

```text
CUSTOMER
ORDER
ORDER_ITEM
PRODUCT
```

Graph transformation:

```text
CUSTOMER → Customer nodes
ORDER → Order nodes
PRODUCT → Product nodes
ORDER_ITEM → relationships
```

Example Python pseudo-flow:

```python
rows = fetch_from_snowflake()

query = """
UNWIND $rows AS row

MERGE (c:Customer {id:row.customer_id})

MERGE (o:Order {id:row.order_id})

MERGE (p:Product {id:row.product_id})

MERGE (c)-[:PLACED]->(o)

MERGE (o)-[:CONTAINS]->(p)
"""
```

Excellent interview statement:

> Snowflake can remain the analytical system of record while Neo4j provides the connected semantic layer for relationship-intensive use cases.

---

# 46. Graph Data Science (GDS)

Neo4j Graph Data Science is used to analyze graph structure.

Main categories:

```text
Centrality
Community Detection
Similarity
Path Finding
Node Embeddings
Link Prediction
```

### Memory Trick

**C-C-S-P-E-L**

Say:

> Central Communities See Paths, Embeddings, Links.

---

# 47. GDS Workflow

Typical GDS flow:

```text
Database Graph
    ↓
Project graph into GDS memory
    ↓
Run algorithm
    ↓
stream / stats / mutate / write
```

### Memory Trick

**P-R-W**

- Project
- Run
- Write

---

# 48. PageRank

Purpose:

> Find important/influential nodes.

Example use cases:

- important web pages
- influential customers
- important suppliers
- high-impact entities in knowledge graph

Concept:

A node becomes important when important nodes point to it.

Example:

```cypher
CALL gds.graph.project(
    'customerGraph',
    'Customer',
    'REFERRED'
);
```

Then:

```cypher
CALL gds.pageRank.stream('customerGraph')
YIELD nodeId, score
RETURN gds.util.asNode(nodeId).name AS customer,
       score
ORDER BY score DESC;
```

---

# 49. Degree Centrality

Question:

> Which node has the most direct connections?

Example use cases:

- most connected customer
- supplier with many downstream dependencies
- frequently linked entity

Conceptually:

```text
degree = number of relationships
```

---

# 50. Betweenness Centrality

Question:

> Which node acts as a bridge between different parts of the graph?

Use cases:

- critical infrastructure node
- fraud intermediary
- supply chain bottleneck
- knowledge broker

Memory:

**Betweenness = bridge**

---

# 51. Community Detection

Goal:

> Find groups of strongly interconnected nodes.

Common algorithm:

```text
Louvain
```

Use cases:

- fraud rings
- customer communities
- product clusters
- social groups

Memory:

**Louvain = communities**

---

# 52. Weakly Connected Components (WCC)

WCC finds separate connected groups.

Use cases:

- disconnected networks
- entity resolution clusters
- fraud networks
- account/device clusters

---

# 53. Node Similarity

Goal:

> Find nodes with similar neighborhoods.

Example:

```text
Customer A bought:
Laptop, Mouse, Keyboard

Customer B bought:
Laptop, Mouse, Monitor
```

They share many products and may be similar.

Neo4j Node Similarity supports metrics such as:

- Jaccard
- Overlap
- Cosine

Jaccard:

```text
intersection / union
```

Example:

```text
A = {Laptop, Mouse, Keyboard}
B = {Laptop, Mouse, Monitor}

intersection = 2
union = 4

Jaccard = 2/4 = 0.5
```

---

# 54. Shortest Path Algorithms

Common:

```text
Dijkstra
```

Use Dijkstra when relationships have weights such as:

```text
distance
cost
time
risk
```

Example:

```text
Atlanta → City A = 100 miles
City A → City B = 80 miles
Atlanta → City B = 300 miles

Best weighted path = 180
```

---

# 55. Node Embeddings

Node embeddings turn graph nodes into numeric vectors.

Example:

```text
Customer C101
↓
[0.13, -0.82, 0.44, ...]
```

Embeddings try to preserve graph structure or meaning.

Uses:

- similarity
- ML features
- recommendation
- clustering
- link prediction

Examples in graph ecosystems:

```text
FastRP
Node2Vec
GraphSAGE
```

---

# 56. GDS Python Client

Install:

```bash
pip install graphdatascience
```

Example concept:

```python
from graphdatascience import GraphDataScience

gds = GraphDataScience(
    URI,
    auth=("neo4j", "password")
)
```

Project:

```python
G, result = gds.graph.project(
    "customerGraph",
    ["Customer", "Product"],
    ["PURCHASED"]
)
```

PageRank:

```python
result = gds.page_rank.stream(G)

print(result.head())
```

Node Similarity:

```python
similar = gds.node_similarity.stream(
    G,
    top_k=5
)

print(similar.head())
```

Important Python naming:

```text
Cypher API: pageRank
Python client: page_rank
```

---

# 57. GDS Execution Modes

Common modes:

### stream

Return algorithm results to client.

### stats

Return summary statistics.

### mutate

Write results into the in-memory projected graph.

### write

Persist results back to Neo4j database.

### estimate

Estimate memory requirements.

### Memory Trick

**S-S-M-W-E**

```text
Stream
Stats
Mutate
Write
Estimate
```

---

# DAY 2 CODING PRACTICE

## Exercise 1 — Customers sharing products

```cypher
MATCH (c1:Customer)-[:PURCHASED]->(p:Product)<-[:PURCHASED]-(c2:Customer)
WHERE c1.id < c2.id
RETURN c1.name,
       c2.name,
       collect(p.name) AS sharedProducts,
       count(p) AS sharedCount
ORDER BY sharedCount DESC;
```

Why:

```text
c1.id < c2.id
```

avoids duplicate mirrored pairs such as:

```text
Alice-Bob
Bob-Alice
```

---

## Exercise 2 — Recommendations

Recommend products bought by similar customers but not Alice.

```cypher
MATCH (alice:Customer {name:"Alice"})-[:PURCHASED]->(common:Product)
      <-[:PURCHASED]-(similar:Customer)
      -[:PURCHASED]->(recommendation:Product)

WHERE NOT (alice)-[:PURCHASED]->(recommendation)

RETURN recommendation.name,
       count(DISTINCT similar) AS score
ORDER BY score DESC;
```

---

## Exercise 3 — Find 2-hop entities

```cypher
MATCH (a:Person {name:$name})-[*2]-(x)
WHERE x <> a
RETURN DISTINCT x;
```

For production, specify relationship types whenever practical.

---

## Exercise 4 — Top 5 customers by purchase count

```cypher
MATCH (c:Customer)-[:PURCHASED]->(p:Product)
RETURN c.name,
       count(p) AS purchases
ORDER BY purchases DESC
LIMIT 5;
```

---

## Exercise 5 — Customers with no orders

```cypher
MATCH (c:Customer)
WHERE NOT EXISTS {
    MATCH (c)-[:PLACED]->(:Order)
}
RETURN c.name;
```

---

# DAY 2 INTERVIEW QUESTIONS

## Q13. Why should I use parameters?

They improve security, readability, and query-plan reuse and prevent values from being mixed directly into query strings.

---

## Q14. What is a Cartesian product?

It occurs when unrelated patterns are matched independently, generating every combination of rows. It can dramatically increase intermediate results and hurt performance.

---

## Q15. How do you optimize Cypher?

I check labels and indexes, filter as early as practical, parameterize queries, avoid accidental Cartesian products, bound variable-length paths, return only required properties, batch large writes, and inspect execution using PROFILE.

---

## Q16. EXPLAIN vs PROFILE?

EXPLAIN gives the execution plan without running the query. PROFILE runs it and provides actual execution statistics.

---

## Q17. Why UNWIND for batch ingestion?

It converts a list into rows inside Cypher, allowing many records to be processed in fewer network round trips.

---

## Q18. What is APOC?

APOC is a library of Neo4j procedures and functions that extends Cypher with utilities for transformation, integration, collections, import/export, graph refactoring, and other tasks.

---

## Q19. What is GDS?

Neo4j Graph Data Science provides algorithms and machine learning tools for analyzing graph structure, including centrality, community detection, similarity, pathfinding, embeddings, and prediction.

---

## Q20. What is PageRank?

PageRank measures node importance based on incoming relationships and the importance of the nodes creating those relationships.

---

## Q21. What is Louvain?

Louvain is a community-detection algorithm used to identify densely connected groups.

---

## Q22. What is node similarity?

Node similarity measures how similar two nodes are based on their neighborhoods or related features.

---

## Q23. When would you use Dijkstra?

For shortest-path problems where relationships have non-negative weights such as cost, distance, or time.

---

## Q24. What are node embeddings?

Numeric vector representations of nodes designed to preserve graph structure or information so they can be used for similarity, clustering, recommendation, or machine learning.

---

# DAY 2 REVISION CHECKLIST

- [ ] Complex MATCH patterns
- [ ] Variable-length paths
- [ ] Shortest path
- [ ] EXISTS subquery
- [ ] CALL subquery
- [ ] APOC purpose
- [ ] Query optimization
- [ ] Python Neo4j driver
- [ ] Transactions
- [ ] UNWIND batch loading
- [ ] GDS projections
- [ ] PageRank
- [ ] Louvain
- [ ] WCC
- [ ] Similarity
- [ ] Dijkstra
- [ ] Node embeddings

---

# DAY 3 — RAG + GRAPH RAG + EMBEDDINGS + LANGCHAIN

---

# 58. What is an LLM?

Large Language Models generate text based on patterns learned during training.

Problem:

They do not automatically know:

- your latest enterprise data
- your private documents
- proprietary relationships
- current customer state

Solution:

```text
RAG
```

---

# 59. RAG

RAG means:

> Retrieval-Augmented Generation

Pipeline:

```text
Documents
   ↓
Extract text
   ↓
Chunk
   ↓
Create embeddings
   ↓
Vector database
   ↓
User question
   ↓
Question embedding
   ↓
Similarity search
   ↓
Relevant chunks
   ↓
LLM prompt
   ↓
Answer
```

### Memory Trick

**I-C-E-R-G**

- Ingest
- Chunk
- Embed
- Retrieve
- Generate

---

# 60. What is an Embedding?

Embedding:

```text
text → numeric vector
```

Example:

```text
"Neo4j graph database"

→

[0.12, -0.84, 0.33, ...]
```

Semantically similar text has vectors located closer together.

---

# 61. Vector Search

User query:

```text
How do I detect fraud networks?
```

Vector database searches for chunks with similar meaning.

Common similarity measures:

```text
cosine similarity
dot product
Euclidean distance
```

---

# 62. Chunking

Why chunk?

LLMs and embedding systems work better when documents are divided into meaningful sections.

Bad:

```text
Entire 100-page PDF = one chunk
```

Better:

```text
Section 1
Section 2
Section 3
...
```

Consider:

- chunk size
- overlap
- document structure
- metadata
- semantic boundaries

### Interview Answer

Chunking balances context and retrieval precision. Chunks that are too large introduce unrelated information; chunks that are too small can lose meaning.

---

# 63. RAG Limitations

Traditional vector RAG is excellent for semantic similarity but can struggle with:

- multi-hop questions
- entity relationships
- global context
- exact structural questions
- provenance paths
- questions spanning multiple documents/entities

Example:

> Which supplier provides parts used by products bought by customers affected by Recall X?

This naturally requires relationship traversal.

---

# 64. GraphRAG

GraphRAG combines LLM/RAG techniques with graph structure.

Simplified:

```text
User Question
     ↓
Entity recognition
     ↓
Vector retrieval
     +
Graph traversal
     ↓
Connected context
     ↓
LLM
     ↓
Grounded answer
```

---

# 65. Vector RAG vs GraphRAG

| Vector RAG | GraphRAG |
|---|---|
| Semantic similarity | Semantic + relationship context |
| Retrieves chunks | Retrieves entities, relationships, chunks |
| Strong for textual similarity | Strong for connected/multi-hop questions |
| Often local context | Can expose broader structural context |
| Easy starting point | More modeling effort |

Interview statement:

> I do not treat GraphRAG as a replacement for vector RAG. In many enterprise designs the strongest solution is hybrid retrieval: vector search finds semantically relevant candidates, then graph traversal enriches them with connected context.

---

# 66. GraphRAG Example

Documents:

```text
Document A:
Customer Alice purchased Product P1.

Document B:
Product P1 contains Component X.

Document C:
Component X is supplied by Supplier S1.

Document D:
Supplier S1 is affected by Recall R1.
```

Vector RAG may retrieve some documents.

Graph:

```text
Alice
  |
PURCHASED
  ↓
P1
  |
CONTAINS
  ↓
X
  |
SUPPLIED_BY
  ↓
S1
  |
AFFECTED_BY
  ↓
R1
```

Now multi-hop query:

> Is Alice affected by recall R1?

Cypher:

```cypher
MATCH path =
(a:Customer {name:"Alice"})
-[:PURCHASED]->
(:Product)
-[:CONTAINS]->
(:Component)
-[:SUPPLIED_BY]->
(:Supplier)
-[:AFFECTED_BY]->
(r:Recall {id:"R1"})
RETURN path;
```

---

# 67. GraphRAG Ingestion Pipeline

Typical steps:

```text
Documents
   ↓
Parser
   ↓
Chunks
   ↓
LLM / NLP entity extraction
   ↓
Entity resolution
   ↓
Relationship extraction
   ↓
Graph upsert
   ↓
Embedding generation
   ↓
Vector index
```

Graph model:

```text
(:Document)-[:HAS_CHUNK]->(:Chunk)

(:Chunk)-[:MENTIONS]->(:Entity)

(:Entity)-[:RELATED_TO]->(:Entity)
```

---

# 68. Entity Extraction

Text:

```text
Microsoft acquired Company X in 2024.
```

Extract:

```text
Entity 1: Microsoft
Entity 2: Company X
Relationship: ACQUIRED
Year: 2024
```

Graph:

```text
(:Company {name:"Microsoft"})
-[:ACQUIRED {year:2024}]->
(:Company {name:"Company X"})
```

---

# 69. Entity Resolution

Problem:

```text
IBM
International Business Machines
I.B.M.
```

May represent same entity.

Entity resolution tries to create:

```text
one canonical node
```

Important techniques:

- canonical IDs
- normalization
- aliases
- fuzzy matching
- embeddings
- business keys
- rules
- human review

Interview phrase:

> Entity resolution is one of the most important data-quality steps in a knowledge graph because duplicate entities fragment relationships and weaken traversal results.

---

# 70. Provenance

Never lose the source.

Good model:

```text
(:Chunk)-[:MENTIONS]->(:Entity)
(:Chunk)-[:FROM_DOCUMENT]->(:Document)
```

Entity/relationship can include:

```text
source
sourceId
extractionModel
createdAt
confidence
```

Why:

- traceability
- citations
- debugging
- trust
- reprocessing

---

# 71. Hybrid Retrieval

Architecture:

```text
Question
   ↓
Embedding
   ↓
Vector Search
   ↓
Relevant Chunk / Entity IDs
   ↓
Graph Traversal
   ↓
Neighbors / Paths / Facts
   ↓
Rerank / trim context
   ↓
LLM
```

Example Cypher concept:

```cypher
MATCH (e:Entity)
WHERE e.id IN $candidateIds

MATCH (e)-[r*1..2]-(neighbor)

RETURN e, r, neighbor
LIMIT 100;
```

---

# 72. Vector Index in Neo4j — Conceptual Pattern

An entity can have an embedding property:

```text
(:Chunk {
    text: "...",
    embedding: [...]
})
```

A vector index supports nearest-neighbor retrieval.

Pipeline:

```text
Question
↓
embed(question)
↓
vector query
↓
top-k chunks
↓
graph expansion
```

---

# 73. Python GraphRAG Skeleton

```python
def graph_rag(question):
    query_vector = embed_text(question)

    candidates = vector_search(
        embedding=query_vector,
        top_k=5
    )

    entity_ids = [
        item["entity_id"]
        for item in candidates
    ]

    graph_context = expand_graph(entity_ids)

    prompt = build_prompt(
        question=question,
        context=graph_context
    )

    return call_llm(prompt)
```

---

# 74. Neo4j Graph Expansion in Python

```python
def expand_graph(driver, entity_ids):
    query = """
    MATCH (e:Entity)
    WHERE e.id IN $entity_ids

    OPTIONAL MATCH path =
        (e)-[*1..2]-(neighbor)

    RETURN DISTINCT
        e.id AS source,
        neighbor.id AS neighbor,
        labels(neighbor) AS labels
    LIMIT 100
    """

    with driver.session() as session:
        result = session.run(
            query,
            entity_ids=entity_ids
        )

        return [record.data() for record in result]
```

Production note:

Avoid unconstrained relationship types/hops on large graphs. Add graph-specific constraints.

---

# 75. Grounding

Grounding means the LLM generates its answer using retrieved trusted context rather than relying only on model memory.

Prompt example:

```text
Answer the question using only the supplied context.

If the answer is not supported by the context,
say that the information is not available.

Context:
{context}

Question:
{question}
```

---

# 76. Hallucination Reduction

Ways to reduce hallucination:

1. strong retrieval
2. citations/provenance
3. explicit grounding prompt
4. reranking
5. query decomposition
6. graph constraints
7. evaluation
8. thresholding
9. refuse unsupported claims
10. monitor retrieval quality separately from generation quality

---

# 77. Reranking

First-stage retrieval may return:

```text
top 20
```

A reranker reorders them and selects:

```text
top 5
```

Pipeline:

```text
retrieve broad
↓
rerank precisely
↓
send small high-quality context to LLM
```

---

# 78. LangChain Concepts

Core concepts commonly used:

- model
- prompt
- retriever
- vector store
- tools
- agents
- chains/runnables
- output parsing
- document loaders
- text splitters

---

# 79. Simple LangChain Mental Model

```text
Input
↓
Prompt
↓
Model
↓
Output
```

RAG:

```text
Input
↓
Retriever
↓
Context
↓
Prompt
↓
Model
```

Agent:

```text
Input
↓
LLM decides action
↓
Tool
↓
Observation
↓
LLM
↓
Answer
```

---

# 80. Graph Tool for an Agent

Expose a safe graph lookup function:

```python
def get_customer_products(customer_id: str):
    query = """
    MATCH (:Customer {id:$id})
          -[:PURCHASED]->
          (p:Product)
    RETURN p.id AS id,
           p.name AS name
    """

    with driver.session() as session:
        return [
            r.data()
            for r in session.run(
                query,
                id=customer_id
            )
        ]
```

The LLM can use this function as a tool.

---

# 81. Do Not Give LLM Unlimited Cypher Access

Risky pattern:

```text
User question
↓
LLM generates arbitrary Cypher
↓
Production graph executes it with admin access
```

Risks:

- destructive queries
- data leakage
- expensive traversals
- injection
- unauthorized data access

Safer options:

- read-only credentials
- allowlisted operations
- query validation
- restricted schemas
- timeout
- row limits
- hop limits
- parameterization
- auditing
- sandbox/test execution

---

# 82. Text-to-Cypher

User:

```text
Show products Alice purchased.
```

LLM:

```cypher
MATCH (:Customer {name:$name})-[:PURCHASED]->(p:Product)
RETURN p.name;
```

Important production design:

1. provide graph schema to LLM
2. instruct read-only queries
3. use parameters
4. validate query
5. enforce timeout
6. limit result count
7. execute with restricted user
8. log generated query

---

# 83. MCP

MCP = Model Context Protocol.

High-level idea:

> A standardized way for AI applications/models to discover and interact with external tools and contextual resources.

Interview explanation:

> Instead of writing one-off integrations for every AI client and backend tool, MCP provides a common protocol for exposing tools/resources to compatible AI applications.

Potential graph tool:

```text
get_entity(entity_id)
find_neighbors(entity_id)
find_path(source, target)
search_knowledge_graph(query)
```

---

# 84. A2A / Multi-Agent Systems

Multi-agent design:

```text
Coordinator Agent
      |
      +--- Retrieval Agent
      |
      +--- Graph Agent
      |
      +--- Validation Agent
      |
      +--- Action Agent
```

Good when tasks are clearly separable.

Do not use multiple agents just because it sounds advanced.

### Interview Answer

I use multi-agent architecture when separate responsibilities, tools, or security boundaries benefit from specialization. For simple deterministic pipelines, a single orchestrator with tools is usually easier to operate and debug.

---

# DAY 3 PRACTICE PROJECT — SUPPORT GRAPH RAG

Model:

```text
(:Customer)
(:Ticket)
(:Product)
(:ErrorCode)
(:Document)
(:Chunk)
```

Relationships:

```text
Customer -[OPENED]-> Ticket
Ticket -[ABOUT]-> Product
Ticket -[HAS_ERROR]-> ErrorCode
Document -[HAS_CHUNK]-> Chunk
Chunk -[MENTIONS]-> Product
Chunk -[EXPLAINS]-> ErrorCode
```

Question:

```text
Customer C101 has error E500.
What known solutions exist for the product involved?
```

Cypher:

```cypher
MATCH (c:Customer {id:$customerId})
      -[:OPENED]->
      (t:Ticket)
      -[:HAS_ERROR]->
      (e:ErrorCode {code:$errorCode})

MATCH (t)-[:ABOUT]->(p:Product)

MATCH (chunk:Chunk)-[:MENTIONS]->(p)
MATCH (chunk)-[:EXPLAINS]->(e)

RETURN chunk.text AS context;
```

Then pass returned chunks to LLM.

---

# DAY 3 INTERVIEW QUESTIONS

## Q25. What is RAG?

RAG retrieves relevant external information and supplies it to an LLM as context before generation, improving grounding on private or current information.

---

## Q26. Why do we use embeddings?

Embeddings represent semantic meaning numerically so similar content can be retrieved through vector similarity search.

---

## Q27. What is chunking?

Chunking divides documents into smaller meaningful sections before embedding and retrieval.

---

## Q28. What is GraphRAG?

GraphRAG enriches LLM retrieval with graph entities and relationships, enabling connected and multi-hop context that pure vector retrieval may miss.

---

## Q29. Vector RAG vs GraphRAG?

Vector RAG finds semantically similar text. GraphRAG additionally uses explicit entity relationships and paths. Hybrid systems often combine both.

---

## Q30. Give a GraphRAG use case.

Supply chain impact analysis: identify a recalled component, traverse to suppliers, products, orders, and customers, then use retrieved documents to generate an explanation with provenance.

---

## Q31. Why is entity resolution important?

Duplicate representations of the same entity fragment the graph and cause incomplete or misleading traversal results.

---

## Q32. What is provenance?

Provenance tracks where facts came from so results can be traced back to their source documents or systems.

---

## Q33. What is reranking?

Reranking is a second-stage relevance process that reorders initial retrieval results to choose the best context for generation.

---

## Q34. How do you reduce hallucinations?

Use high-quality retrieval, grounding instructions, provenance/citations, reranking, validation, thresholds, structured outputs, evaluation, and safe fallbacks when context is insufficient.

---

## Q35. What is an AI agent?

An agent uses a model to decide which actions or tools to call, observes the results, and continues until it can complete the task.

---

## Q36. What is MCP?

MCP is a protocol that standardizes how AI applications access external tools and contextual resources.

---

## Q37. What is multi-agent architecture?

It splits a complex workflow across specialized agents that collaborate or are orchestrated by a coordinator.

---

# DAY 3 REVISION CHECKLIST

- [ ] RAG flow
- [ ] embeddings
- [ ] vector search
- [ ] chunking
- [ ] metadata
- [ ] GraphRAG
- [ ] hybrid retrieval
- [ ] entity extraction
- [ ] entity resolution
- [ ] provenance
- [ ] reranking
- [ ] grounding
- [ ] hallucination reduction
- [ ] LangChain concepts
- [ ] tools/agents
- [ ] MCP
- [ ] multi-agent architecture

---

# DAY 4 — AWS + BEDROCK + NEPTUNE + SERVERLESS + SYSTEM DESIGN

---

# 85. Amazon Bedrock

Amazon Bedrock provides access to foundation models and managed generative AI capabilities on AWS.

Typical flow:

```text
Application
↓
Amazon Bedrock
↓
Foundation Model
↓
Response
```

For RAG:

```text
Data
↓
Knowledge Base / Retrieval Layer
↓
Context
↓
Foundation Model
```

---

# 86. Bedrock Knowledge Bases

Conceptually:

```text
Data source
↓
Parsing / chunking
↓
Embedding
↓
Vector store
↓
Retrieve
↓
Generate
```

Two important API styles:

```text
Retrieve
RetrieveAndGenerate
```

Use `Retrieve` when you want application control over:

- retrieved chunks
- reranking
- prompt construction
- graph expansion

Use `RetrieveAndGenerate` when a managed end-to-end experience is appropriate.

---

# 87. Bedrock + GraphRAG Architecture

```text
                 ┌─────────────┐
User ───────────>│ API Gateway │
                 └──────┬──────┘
                        │
                 ┌──────▼──────┐
                 │   Lambda    │
                 └──────┬──────┘
                        │
          ┌─────────────┼─────────────┐
          │             │             │
   ┌──────▼──────┐ ┌────▼─────┐ ┌────▼─────┐
   │Vector Search│ │ Neo4j /   │ │ Bedrock  │
   │ / OpenSearch│ │ Neptune   │ │ FM       │
   └──────┬──────┘ └────┬─────┘ └──────────┘
          │             │
          └──────┬──────┘
                 │
            Context
                 │
          ┌──────▼──────┐
          │   Bedrock   │
          └─────────────┘
```

---

# 88. Amazon Neptune

Amazon Neptune is AWS's managed graph database family.

Interview-level difference:

```text
Neo4j
- property graph platform
- Cypher ecosystem
- APOC
- GDS
- strong developer tooling

Amazon Neptune
- AWS managed graph service
- tight AWS integration
- graph workloads in AWS
```

For interviews, avoid oversimplifying as "same database."

Discuss:

- query language compatibility
- operational model
- graph analytics needs
- AWS-native architecture
- migration/testing requirements

---

# 89. Neptune Analytics + Vector Search

A powerful concept is storing graph-connected entities and vector embeddings so semantic similarity and graph structure can be combined.

Conceptual pipeline:

```text
Query
↓
query embedding
↓
nearest-neighbor vector search
↓
candidate graph nodes
↓
graph traversal / algorithms
↓
context
↓
LLM
```

---

# 90. Lambda

AWS Lambda runs code without managing servers.

Example use in Graph AI:

```text
API Gateway
↓
Lambda
↓
Neo4j / Neptune retrieval
↓
Bedrock
↓
Response
```

Python Lambda example:

```python
import json

def lambda_handler(event, context):
    question = event.get("question")

    result = {
        "question": question,
        "answer": "example"
    }

    return {
        "statusCode": 200,
        "body": json.dumps(result)
    }
```

---

# 91. API Gateway

API Gateway exposes APIs to applications.

Example:

```text
POST /ask

{
  "question": "Which suppliers are impacted?"
}
```

Flow:

```text
Client
↓
API Gateway
↓
Lambda
↓
Graph + Bedrock
↓
JSON response
```

---

# 92. DynamoDB

Useful for:

- session state
- agent state
- lightweight metadata
- request state
- conversation references
- idempotency keys

Do not use DynamoDB as a graph replacement.

---

# 93. S3

Use S3 for:

- raw documents
- ingestion files
- PDFs
- JSON exports
- batch input
- processed artifacts
- audit/archive data

Common pipeline:

```text
S3 upload
↓
event
↓
Lambda / processing job
↓
chunk
↓
embedding
↓
vector DB
↓
graph update
```

---

# 94. OpenSearch

OpenSearch can support:

- text search
- full-text search
- vector search
- hybrid retrieval

GraphRAG architecture:

```text
OpenSearch = semantic/text retrieval

Neo4j/Neptune = relationship traversal

Bedrock = generation
```

---

# 95. Bedrock Agentic Workflows

A conceptual agent workflow:

```text
Question
↓
Model reasons about task
↓
Tool selection
↓
Lambda / graph lookup / enterprise API
↓
Observation
↓
Next tool or final answer
```

For modern AWS interviews, be aware that AWS has evolved its agent platform. Describe the architecture first rather than memorizing only one product label.

---

# 96. Production Security

Must mention:

### IAM / access control

Least privilege.

### Secrets

Never hardcode:

```text
Neo4j password
AWS access keys
API keys
```

Use secure secret management.

### Encryption

- TLS in transit
- encryption at rest

### Graph authorization

Restrict access by:

- database role
- label/data scope where supported
- application-level controls

### LLM tool restrictions

Use:

- read-only graph credentials
- allowlisted tool calls
- parameterized queries
- validation
- rate limits
- timeouts
- audit logs

---

# 97. Observability

Monitor:

```text
latency
retrieval latency
graph query latency
Bedrock/model latency
token usage
errors
timeouts
retrieval quality
answer quality
cost
```

Trace:

```text
request ID
retrieved documents
graph entities
Cypher query
tool calls
model
prompt version
```

---

# 98. GraphRAG Evaluation

Evaluate retrieval separately from generation.

### Retrieval metrics

```text
Recall@K
Precision@K
MRR
hit rate
```

### Generation metrics

```text
faithfulness
answer relevance
groundedness
citation correctness
completeness
```

### Graph-specific checks

```text
correct entities?
correct relationship paths?
correct hop depth?
correct provenance?
```

---

# 99. Latency Optimization

Ways to reduce latency:

- narrow graph traversal
- index lookup keys
- limit top-k
- parallelize independent retrieval
- cache stable results
- avoid returning entire nodes
- use compact context
- tune reranking count
- batch embedding requests
- avoid repeated LLM calls
- measure each pipeline stage

---

# 100. Cost Optimization

Cost sources:

```text
LLM tokens
embedding generation
vector search
graph infrastructure
Lambda/runtime
network
storage
```

Reduce:

- unnecessary prompt context
- overly large chunks
- duplicate embeddings
- excessive top-k
- repeated model calls
- uncontrolled agent loops

---

# 101. Real-Time Architecture Question

## Interview Question

Design a production GraphRAG system that answers questions about customer-support incidents.

## Strong Answer

### Data layer

```text
S3
- product manuals
- support articles
- incident documents
```

### Ingestion

```text
S3
↓
processing pipeline
↓
document parser
↓
chunker
↓
entity/relationship extraction
```

### Storage

```text
Neo4j / Neptune
- Customer
- Product
- Incident
- ErrorCode
- Component
- relationships

Vector store
- chunk embeddings
```

### Runtime

```text
API Gateway
↓
Lambda / service
↓
query classification
↓
vector retrieval
+
graph traversal
↓
rerank
↓
prompt assembly
↓
Bedrock
↓
response with provenance
```

### Security

```text
IAM
Secrets Manager
TLS
read-only graph user
tenant filtering
```

### Observability

```text
CloudWatch
retrieval traces
query latency
model latency
token/cost metrics
```

---

# 102. Graph AI System Design Framework

When asked to design any Graph AI solution, use:

### D-M-R-G-S-O

**D — Data**

Where does data come from?

**M — Model**

What nodes, relationships, ontology?

**R — Retrieval**

Vector, graph, keyword, hybrid?

**G — Generation**

Which LLM? How grounded?

**S — Security**

Permissions, secrets, guardrails?

**O — Observability**

Latency, quality, errors, cost?

This framework keeps system-design answers structured.

---

# PYTHON PRACTICE SECTION

---

# 103. Practice Project Structure

```text
graph_ai_project/
│
├── app.py
├── config.py
├── neo4j_client.py
├── ingestion.py
├── graph_rag.py
├── bedrock_client.py
├── models.py
├── requirements.txt
└── tests/
    ├── test_graph.py
    └── test_rag.py
```

---

# 104. Neo4j Client Class

```python
from neo4j import GraphDatabase


class Neo4jClient:

    def __init__(self, uri, username, password):
        self.driver = GraphDatabase.driver(
            uri,
            auth=(username, password)
        )

    def close(self):
        self.driver.close()

    def run_read(self, query, **params):
        with self.driver.session() as session:
            result = session.run(query, **params)
            return [record.data() for record in result]

    def run_write(self, query, **params):
        with self.driver.session() as session:
            result = session.run(query, **params)
            return result.consume()
```

Better production code usually uses transaction functions and configures database, retry behavior, logging, timeouts, etc.

---

# 105. Graph Repository Pattern

```python
class CustomerGraphRepository:

    def __init__(self, client):
        self.client = client

    def find_products(self, customer_id):

        query = """
        MATCH (:Customer {id:$customer_id})
              -[:PURCHASED]->
              (p:Product)

        RETURN p.id AS id,
               p.name AS name
        """

        return self.client.run_read(
            query,
            customer_id=customer_id
        )
```

Why useful:

- keeps Cypher separate
- testable
- reusable
- easier maintenance

---

# 106. Batch Upsert

```python
def upsert_customers(client, customers):

    query = """
    UNWIND $customers AS row

    MERGE (c:Customer {id:row.id})

    SET c.name = row.name,
        c.email = row.email
    """

    client.run_write(
        query,
        customers=customers
    )
```

---

# 107. Batch Relationship Creation

```python
def upsert_purchases(client, purchases):

    query = """
    UNWIND $rows AS row

    MATCH (c:Customer {id:row.customer_id})
    MATCH (p:Product {id:row.product_id})

    MERGE (c)-[r:PURCHASED]->(p)

    SET r.amount = row.amount,
        r.orderDate = date(row.order_date)
    """

    client.run_write(
        query,
        rows=purchases
    )
```

---

# 108. Recommendation Function

```python
def recommend_products(client, customer_id):

    query = """
    MATCH (target:Customer {id:$customer_id})
          -[:PURCHASED]->
          (product:Product)
          <-[:PURCHASED]-
          (similar:Customer)
          -[:PURCHASED]->
          (recommendation:Product)

    WHERE NOT
        (target)-[:PURCHASED]->(recommendation)

    RETURN
        recommendation.id AS id,
        recommendation.name AS name,
        count(DISTINCT similar) AS score

    ORDER BY score DESC
    LIMIT 10
    """

    return client.run_read(
        query,
        customer_id=customer_id
    )
```

---

# 109. Find Connection Path

```python
def find_connection(client, source, target):

    query = """
    MATCH
        (a:Person {id:$source}),
        (b:Person {id:$target})

    MATCH p =
        shortestPath(
            (a)-[:KNOWS*]-(b)
        )

    RETURN
        [n IN nodes(p) | n.name] AS people
    """

    return client.run_read(
        query,
        source=source,
        target=target
    )
```

---

# 110. Retrieval Interface

```python
from typing import Protocol


class Retriever(Protocol):

    def retrieve(self, query: str) -> list[dict]:
        ...
```

Vector retriever:

```python
class VectorRetriever:

    def retrieve(self, query):
        vector = embed_text(query)

        return vector_search(
            vector,
            top_k=10
        )
```

Graph retriever:

```python
class GraphRetriever:

    def retrieve(self, entity_ids):
        return expand_graph(entity_ids)
```

---

# 111. Hybrid Retriever

```python
class HybridRetriever:

    def __init__(
        self,
        vector_retriever,
        graph_retriever
    ):
        self.vector = vector_retriever
        self.graph = graph_retriever

    def retrieve(self, question):

        vector_results = \
            self.vector.retrieve(question)

        entity_ids = [
            x["entity_id"]
            for x in vector_results
            if x.get("entity_id")
        ]

        graph_results = \
            self.graph.retrieve(entity_ids)

        return {
            "vector": vector_results,
            "graph": graph_results
        }
```

---

# 112. Prompt Builder

```python
def build_prompt(question, context):

    return f"""
You are an enterprise assistant.

Answer using only the supplied context.

If the answer cannot be supported by the
context, say that the information is unavailable.

Context:
{context}

Question:
{question}
"""
```

---

# 113. Bedrock Python Skeleton

Illustrative example:

```python
import boto3
import json

bedrock = boto3.client(
    "bedrock-runtime",
    region_name="us-east-1"
)

def invoke_model(model_id, payload):

    response = bedrock.invoke_model(
        modelId=model_id,
        body=json.dumps(payload)
    )

    return json.loads(
        response["body"].read()
    )
```

Exact request payload differs by model/provider/API. In an interview, say you follow the schema required by the selected Bedrock model or Converse API.

---

# 114. AWS Lambda GraphRAG Skeleton

```python
import json

def lambda_handler(event, context):

    body = json.loads(
        event.get("body", "{}")
    )

    question = body["question"]

    retrieval = hybrid_retriever.retrieve(
        question
    )

    prompt = build_prompt(
        question,
        retrieval
    )

    answer = call_bedrock(prompt)

    return {
        "statusCode": 200,
        "headers": {
            "Content-Type":
            "application/json"
        },
        "body": json.dumps({
            "answer": answer
        })
    }
```

---

# CYPHER PRACTICE — 35 HANDS-ON QUESTIONS

Assume:

```text
(:Customer {id,name,state})
(:Product {id,name,category,price})
(:Order {id,orderDate,total})
(:Supplier {id,name})
(:Category {name})
```

Relationships:

```text
(Customer)-[:PLACED]->(Order)
(Order)-[:CONTAINS]->(Product)
(Product)-[:BELONGS_TO]->(Category)
(Product)-[:SUPPLIED_BY]->(Supplier)
(Customer)-[:REFERRED]->(Customer)
```

---

## Practice 1

Find all customers.

```cypher
MATCH (c:Customer)
RETURN c;
```

---

## Practice 2

Find customer C101.

```cypher
MATCH (c:Customer {id:"C101"})
RETURN c;
```

---

## Practice 3

Use parameter.

```cypher
MATCH (c:Customer {id:$customerId})
RETURN c;
```

---

## Practice 4

Customers in Georgia.

```cypher
MATCH (c:Customer)
WHERE c.state = "GA"
RETURN c.name;
```

---

## Practice 5

Products over $500.

```cypher
MATCH (p:Product)
WHERE p.price > 500
RETURN p.name, p.price;
```

---

## Practice 6

Products sorted by price.

```cypher
MATCH (p:Product)
RETURN p.name, p.price
ORDER BY p.price DESC;
```

---

## Practice 7

Top 5 products.

```cypher
MATCH (p:Product)
RETURN p.name, p.price
ORDER BY p.price DESC
LIMIT 5;
```

---

## Practice 8

Orders placed by customer.

```cypher
MATCH (:Customer {id:$id})
      -[:PLACED]->
      (o:Order)
RETURN o;
```

---

## Practice 9

Products from customer's orders.

```cypher
MATCH (:Customer {id:$id})
      -[:PLACED]->
      (:Order)
      -[:CONTAINS]->
      (p:Product)
RETURN DISTINCT p;
```

---

## Practice 10

Product category.

```cypher
MATCH (p:Product)
      -[:BELONGS_TO]->
      (c:Category)
RETURN p.name, c.name;
```

---

## Practice 11

Product supplier.

```cypher
MATCH (p:Product)
      -[:SUPPLIED_BY]->
      (s:Supplier)
RETURN p.name, s.name;
```

---

## Practice 12

Customers with no orders.

```cypher
MATCH (c:Customer)
WHERE NOT EXISTS {
    MATCH (c)-[:PLACED]->(:Order)
}
RETURN c;
```

---

## Practice 13

Count orders by customer.

```cypher
MATCH (c:Customer)
OPTIONAL MATCH (c)-[:PLACED]->(o:Order)
RETURN c.name,
       count(o) AS orderCount;
```

---

## Practice 14

Customers with more than five orders.

```cypher
MATCH (c:Customer)-[:PLACED]->(o:Order)
WITH c, count(o) AS orderCount
WHERE orderCount > 5
RETURN c.name, orderCount;
```

---

## Practice 15

Average product price by category.

```cypher
MATCH (p:Product)
      -[:BELONGS_TO]->
      (c:Category)
RETURN c.name,
       avg(p.price) AS avgPrice;
```

---

## Practice 16

Collect products by supplier.

```cypher
MATCH (p:Product)
      -[:SUPPLIED_BY]->
      (s:Supplier)
RETURN s.name,
       collect(p.name) AS products;
```

---

## Practice 17

Customers who purchased same product.

```cypher
MATCH
(c1:Customer)-[:PLACED]->(:Order)-[:CONTAINS]->(p:Product)
<-[:CONTAINS]-(:Order)<-[:PLACED]-(c2:Customer)

WHERE c1.id < c2.id

RETURN
c1.name,
c2.name,
collect(DISTINCT p.name) AS sharedProducts;
```

---

## Practice 18

Referrals.

```cypher
MATCH (c:Customer)-[:REFERRED]->(friend:Customer)
RETURN c.name, friend.name;
```

---

## Practice 19

Second-degree referrals.

```cypher
MATCH (c:Customer {id:$id})
      -[:REFERRED]->
      (:Customer)
      -[:REFERRED]->
      (candidate:Customer)

WHERE candidate <> c
AND NOT (c)-[:REFERRED]->(candidate)

RETURN DISTINCT candidate;
```

---

## Practice 20

Variable 1–3 referral hops.

```cypher
MATCH (:Customer {id:$id})
      -[:REFERRED*1..3]->
      (c:Customer)
RETURN DISTINCT c;
```

---

## Practice 21

Create customer.

```cypher
CREATE (:Customer {
    id:"C500",
    name:"Jane"
});
```

---

## Practice 22

Upsert customer.

```cypher
MERGE (c:Customer {id:$id})
SET c.name = $name;
```

---

## Practice 23

Create relationship.

```cypher
MATCH
(c:Customer {id:$customerId}),
(p:Product {id:$productId})

MERGE (c)-[:INTERESTED_IN]->(p);
```

---

## Practice 24

Update property.

```cypher
MATCH (p:Product {id:$id})
SET p.price = $price
RETURN p;
```

---

## Practice 25

Delete relationship.

```cypher
MATCH
(c:Customer {id:$customerId})
-[r:INTERESTED_IN]->
(p:Product {id:$productId})

DELETE r;
```

---

## Practice 26

Delete customer and relationships.

```cypher
MATCH (c:Customer {id:$id})
DETACH DELETE c;
```

---

## Practice 27

UNWIND list.

```cypher
UNWIND $rows AS row
MERGE (p:Product {id:row.id})
SET p.name = row.name;
```

---

## Practice 28

Customers without supplier connection through orders.

```cypher
MATCH (c:Customer)
WHERE NOT EXISTS {
    MATCH
      (c)-[:PLACED]->
      (:Order)-[:CONTAINS]->
      (:Product)-[:SUPPLIED_BY]->
      (:Supplier)
}
RETURN c;
```

---

## Practice 29

Supplier impact.

```cypher
MATCH
(s:Supplier {id:$supplierId})
<-[:SUPPLIED_BY]-
(p:Product)
<-[:CONTAINS]-
(o:Order)
<-[:PLACED]-
(c:Customer)

RETURN DISTINCT
c.id,
c.name,
p.name;
```

---

## Practice 30

Shortest referral path.

```cypher
MATCH
(a:Customer {id:$source}),
(b:Customer {id:$target})

MATCH p =
shortestPath(
    (a)-[:REFERRED*]-(b)
)

RETURN p;
```

---

## Practice 31

Return path nodes.

```cypher
MATCH
(a:Customer {id:$source}),
(b:Customer {id:$target})

MATCH p =
shortestPath(
    (a)-[:REFERRED*]-(b)
)

RETURN
[n IN nodes(p) | n.name]
AS path;
```

---

## Practice 32

Products bought by referred customers.

```cypher
MATCH
(c:Customer {id:$id})
-[:REFERRED]->
(friend:Customer)
-[:PLACED]->
(:Order)
-[:CONTAINS]->
(p:Product)

RETURN DISTINCT p.name;
```

---

## Practice 33

Recommendation excluding existing products.

```cypher
MATCH
(target:Customer {id:$id})
-[:PLACED]->
(:Order)
-[:CONTAINS]->
(common:Product)

MATCH
(similar:Customer)
-[:PLACED]->
(:Order)
-[:CONTAINS]->
(common)

MATCH
(similar)
-[:PLACED]->
(:Order)
-[:CONTAINS]->
(candidate:Product)

WHERE similar <> target

AND NOT EXISTS {
    MATCH
    (target)
    -[:PLACED]->
    (:Order)
    -[:CONTAINS]->
    (candidate)
}

RETURN
candidate.name,
count(DISTINCT similar) AS score

ORDER BY score DESC
LIMIT 10;
```

---

## Practice 34

Supplier with most products.

```cypher
MATCH
(p:Product)-[:SUPPLIED_BY]->(s:Supplier)

RETURN
s.name,
count(p) AS productCount

ORDER BY productCount DESC
LIMIT 1;
```

---

## Practice 35

Find categories connected to a customer within the order graph.

```cypher
MATCH
(c:Customer {id:$id})
-[:PLACED]->
(:Order)
-[:CONTAINS]->
(:Product)
-[:BELONGS_TO]->
(cat:Category)

RETURN DISTINCT cat.name;
```

---

# 65 GRAPH AI INTERVIEW QUESTIONS AND ANSWERS

---

## Q38. What is a graph database?

A graph database stores data as nodes and relationships, making connected-data traversal a core operation.

---

## Q39. Why is Neo4j useful for AI?

It provides structured connected context that can support semantic search, GraphRAG, recommendation, entity resolution, fraud analysis, feature generation, and explainable multi-hop retrieval.

---

## Q40. What is a semantic graph?

A semantic graph represents entities and relationships with explicit meaning so machines and humans can interpret the structure consistently.

---

## Q41. Knowledge graph vs vector database?

A knowledge graph represents explicit entities and relationships. A vector database retrieves items by semantic similarity. They solve different problems and are frequently combined.

---

## Q42. Can Neo4j replace Snowflake?

Not generally. They optimize different workloads. Snowflake is strong for analytical/warehouse workloads. Neo4j is strong for connected-data modeling and traversal. Many enterprise architectures use both.

---

## Q43. How would you integrate Snowflake and Neo4j?

Extract relevant warehouse records, transform them into graph entities/relationships, load them in batches using stable identifiers and MERGE, and maintain incremental synchronization based on change timestamps, streams, CDC, or pipeline orchestration.

---

## Q44. Why are stable IDs critical?

MERGE and entity resolution need reliable identity. Without stable IDs, repeated ingestion can create duplicate nodes.

---

## Q45. What makes a good graph relationship?

It represents a meaningful domain connection that users or algorithms need to traverse.

---

## Q46. Should every foreign key become a relationship?

Not automatically. Model based on query patterns and domain meaning rather than mechanically converting the relational schema.

---

## Q47. What causes graph explosion?

High-degree nodes, unbounded variable-length paths, broad patterns, and unconstrained expansions can create huge intermediate path combinations.

---

## Q48. How do you control traversal?

Specify labels, relationship types, hop limits, predicates, direction, result limits, and appropriate indexes for starting nodes.

---

## Q49. Why start from selective nodes?

A selective indexed starting point reduces the number of nodes/relationships explored.

---

## Q50. What is a supernode?

A node with an extremely large number of relationships. It can cause traversal and memory challenges if queries expand it broadly.

---

## Q51. How would you handle supernodes?

Use selective relationship types/directions, intermediate modeling where appropriate, query constraints, precomputation, aggregation, or application-specific partitioning/model redesign.

---

## Q52. MATCH vs OPTIONAL MATCH?

MATCH requires the pattern. OPTIONAL MATCH keeps the row and returns null for missing optional portions.

---

## Q53. CREATE vs MERGE?

CREATE always creates. MERGE matches or creates.

---

## Q54. SET vs ON CREATE SET?

SET always updates when reached. ON CREATE SET runs only when MERGE creates the element.

---

## Q55. What does collect do?

It aggregates values into a list.

---

## Q56. What does DISTINCT do?

It removes duplicate result rows/values from a projection.

---

## Q57. What does UNWIND do?

It converts elements of a list into individual rows.

---

## Q58. How do you paginate Cypher?

Using ORDER BY with SKIP/LIMIT for simple pagination, though cursor/keyset approaches are preferable for some high-scale cases.

---

## Q59. Why is deterministic ORDER BY important for pagination?

Without deterministic ordering, results can shift between pages.

---

## Q60. How do you prevent duplicate nodes?

Use uniqueness constraints and MERGE against a stable business key.

---

## Q61. How do you prevent duplicate relationships?

MERGE relationships using the correct endpoint identity and relationship pattern; include relationship identity properties only when they are truly part of identity.

---

## Q62. What is idempotent ingestion?

Running the same ingestion more than once does not create unintended duplicates or inconsistent state.

---

## Q63. What is schema evolution in a graph?

Adding/changing labels, properties, relationships, or ontology concepts over time while keeping compatibility and migration strategy.

---

## Q64. What is GDS projection?

It creates an in-memory graph representation optimized for graph algorithms.

---

## Q65. Why not run every graph algorithm directly on stored graph structures?

Analytics algorithms often benefit from optimized in-memory representations, special configuration, and execution modes.

---

## Q66. Centrality vs community detection?

Centrality identifies important nodes. Community detection identifies densely connected groups.

---

## Q67. PageRank vs degree centrality?

Degree counts direct connections. PageRank accounts for both connections and the importance of the nodes creating those connections.

---

## Q68. Betweenness centrality?

It measures how often a node lies on shortest paths and can identify bridge/broker nodes.

---

## Q69. WCC use case?

Detect disconnected components such as separate identity clusters or isolated network groups.

---

## Q70. Louvain use case?

Detect communities such as fraud rings, customer groups, or related entity clusters.

---

## Q71. Node Similarity use case?

Recommendation and entity similarity based on shared neighbors.

---

## Q72. Jaccard similarity?

Size of intersection divided by size of union.

---

## Q73. Dijkstra?

Weighted shortest-path algorithm for non-negative edge weights.

---

## Q74. What is link prediction?

Estimating whether a relationship is likely to exist or form between two nodes.

---

## Q75. What is graph embedding?

A numerical representation of graph entities that captures structural/semantic information.

---

## Q76. Why combine graph embeddings and text embeddings?

Text embeddings capture semantic content while graph embeddings capture relational/structural context.

---

## Q77. What is hybrid search?

Combining retrieval methods such as keyword, vector, metadata, and graph traversal.

---

## Q78. Why is hybrid retrieval valuable?

Different retrieval signals cover different failure modes and can improve relevance.

---

## Q79. What is retrieval recall?

How many relevant items were successfully retrieved.

---

## Q80. What is retrieval precision?

What fraction of retrieved items are actually relevant.

---

## Q81. What is top-k?

The number of highest-ranked results returned by retrieval.

---

## Q82. What happens if top-k is too high?

More irrelevant context, higher cost, larger prompts, and potentially worse generation quality.

---

## Q83. What happens if top-k is too low?

Important supporting evidence may be missed.

---

## Q84. How do you choose top-k?

Evaluate empirically using a representative question set and retrieval metrics rather than relying on one universal number.

---

## Q85. What is metadata filtering?

Restricting retrieval by fields such as tenant, product, date, region, document type, or permission.

---

## Q86. What is tenant isolation?

Ensuring one customer's data cannot be retrieved or exposed to another tenant.

---

## Q87. Why is tenant filtering important in RAG?

Semantic similarity alone does not enforce authorization.

---

## Q88. What is prompt injection in RAG?

Untrusted content attempts to manipulate model instructions or tool behavior.

---

## Q89. How do you mitigate prompt injection?

Separate trusted instructions from retrieved content, restrict tools, validate outputs/actions, enforce authorization outside the model, and treat retrieved content as untrusted data.

---

## Q90. Why should authorization be outside the LLM?

LLMs are probabilistic; security decisions should be enforced by deterministic infrastructure and access controls.

---

## Q91. How do you secure generated Cypher?

Use read-only credentials, schema restrictions, allowlists/validation, parameters, query timeouts, row limits, logging, and controlled tool interfaces.

---

## Q92. How do you evaluate GraphRAG?

Measure retrieval correctness, entity/path correctness, provenance, faithfulness, answer relevance, latency, and cost.

---

## Q93. What is groundedness?

Whether generated claims are supported by retrieved trusted context.

---

## Q94. What is faithfulness?

Whether the answer stays consistent with the provided evidence rather than introducing unsupported claims.

---

## Q95. What is a hallucination?

A generated statement not supported by reliable context or facts.

---

## Q96. Can GraphRAG eliminate hallucinations?

No. It can reduce them by improving retrieval and grounding, but generation still requires evaluation and guardrails.

---

## Q97. What is query decomposition?

Breaking a complex question into smaller subquestions for retrieval/reasoning.

---

## Q98. When is multi-hop retrieval needed?

When answering requires following multiple relationships such as customer → product → component → supplier → recall.

---

## Q99. What is entity-centric retrieval?

Retrieve entities first, then expand their graph neighborhoods.

---

## Q100. What is chunk-centric retrieval?

Retrieve text chunks first, then derive connected entities/documents from those chunks.

---

## Q101. Which is better: entity-first or chunk-first GraphRAG?

It depends on question type and data quality. Hybrid routing can choose or combine both.

---

## Q102. What is a retriever?

A component that returns relevant information for a query.

---

# BEHAVIORAL + PROJECT QUESTIONS

---

## Q103. Tell me about yourself for this role.

### Sample answer

I am an experienced data engineer with a strong SQL and Snowflake background, and I am also a Neo4j Certified Professional. My graph focus includes property-graph modeling, Cypher query development, connected-data analysis, and Python integration. I have been strengthening my Graph AI skills around knowledge graphs, GraphRAG, embeddings, LangChain, and AWS Bedrock.

What interests me in this Graph AI Engineer role is the combination of data modeling and AI. I can use my data-engineering experience for ingestion, data quality, performance, and production pipelines, while using Neo4j and GraphRAG for relationship-aware retrieval and grounded LLM applications.

---

## Q104. You have strong Snowflake experience. Why Graph AI?

### Sample answer

Snowflake helped me build strong skills in data modeling, SQL, pipelines, and large-scale data processing. Graph AI extends that experience into use cases where relationships are central. Instead of repeatedly joining tables to understand connected entities, I can represent and traverse those relationships directly in a graph. Combining that with embeddings and LLMs makes it possible to build GraphRAG applications that use both semantic similarity and explicit business relationships.

---

## Q105. You are Neo4j certified. What practical value does certification give you?

### Sample answer

Certification validates my fundamentals in graph concepts and Cypher, but for a production role I focus on applying those concepts: choosing stable entity keys, designing relationship-oriented models, writing parameterized queries, using constraints and indexes, profiling query plans, integrating Neo4j from Python, and understanding where GDS and GraphRAG fit.

---

## Q106. Describe a GraphRAG project.

Use this structure:

```text
Problem
Data
Graph model
Ingestion
Retrieval
LLM
Security
Evaluation
Result
```

Example:

> I designed a support knowledge graph containing Customer, Product, Ticket, ErrorCode, Document, and Chunk entities. Documents were chunked and embedded for semantic retrieval. Retrieved chunks mapped to graph entities, and Cypher expanded related product/error-code context. The combined context was passed to an LLM. Provenance links allowed the answer to cite its supporting documents.

---

## Q107. What was the hardest part?

Strong answer:

> Entity resolution and retrieval control are usually harder than simply calling the LLM. If entities are duplicated or graph expansions are too broad, the LLM receives incomplete or noisy context. I address this using canonical IDs, constraints, ingestion validation, hop limits, relationship filters, and retrieval evaluation.

---

## Q108. How would you explain graph technology to a business person?

> A traditional database is like spreadsheets connected using IDs. A graph database is more like a map: the objects are locations and the relationships are roads. If the business question is about how things are connected, the map representation makes those questions easier to express and analyze.

---

# MEMORY TRICKS MASTER SHEET

## Graph

```text
NRP
Node
Relationship
Property
```

## Basic Cypher

```text
MWR
MATCH
WHERE
RETURN
```

## Data loading

```text
U-M-S
UNWIND
MERGE
SET
```

## Query tuning

```text
I-F-B-P
Index
Filter early
Bound traversal
PROFILE
```

## RAG

```text
ICERG
Ingest
Chunk
Embed
Retrieve
Generate
```

## GraphRAG

```text
V-G-L
Vector
Graph
LLM
```

## Graph AI system design

```text
DMRGSO
Data
Model
Retrieval
Generation
Security
Observability
```

## GDS

```text
CCSPEL

Centrality
Community
Similarity
Path
Embedding
Link prediction
```

## GDS execution

```text
SSMWE

Stream
Stats
Mutate
Write
Estimate
```

## Production GraphRAG

```text
IRGSE

Identity
Retrieval
Grounding
Security
Evaluation
```

---

# SQL / SNOWFLAKE TO CYPHER CHEAT SHEET

| SQL / Snowflake | Cypher |
|---|---|
| Table | Node label |
| Row | Node |
| Foreign key | Relationship |
| Column | Property |
| JOIN | Pattern traversal |
| SELECT | RETURN |
| FROM/JOIN | MATCH |
| WHERE | WHERE |
| GROUP BY | aggregation with WITH/RETURN |
| ARRAY FLATTEN-like thinking | UNWIND |
| LEFT JOIN | OPTIONAL MATCH |
| UPSERT-style pattern | MERGE |
| CTE/pipeline | WITH / subquery |
| query plan | EXPLAIN / PROFILE |

Important:

These are mental analogies, not exact one-to-one equivalents.

---

# QUICK CYPHER CHEAT SHEET

## Read

```cypher
MATCH (n:Label)
RETURN n;
```

## Filter

```cypher
MATCH (n:Label)
WHERE n.id = $id
RETURN n;
```

## Relationship

```cypher
MATCH (a)-[:REL]->(b)
RETURN a,b;
```

## Create

```cypher
CREATE (:Person {name:$name});
```

## Upsert

```cypher
MERGE (p:Person {id:$id})
SET p.name = $name;
```

## Optional

```cypher
OPTIONAL MATCH ...
```

## Aggregate

```cypher
MATCH (c)-[:BOUGHT]->(p)
RETURN c.id, count(p);
```

## Pipeline

```cypher
MATCH ...
WITH ...
WHERE ...
RETURN ...
```

## List to rows

```cypher
UNWIND $rows AS row
...
```

## Variable traversal

```cypher
MATCH (a)-[:REL*1..3]->(b)
RETURN b;
```

## Existence

```cypher
WHERE EXISTS {
    MATCH ...
}
```

## Query plan

```cypher
EXPLAIN ...
PROFILE ...
```

---

# FINAL 4-DAY REVISION PLAN

---

## DAY 1 MORNING

Study:

```text
Graph concepts
Property graph
Knowledge graph
Ontology
Graph modeling
```

Practice:

```text
CREATE
MATCH
WHERE
RETURN
MERGE
SET
DELETE
```

## DAY 1 AFTERNOON

Study:

```text
WITH
UNWIND
OPTIONAL MATCH
Aggregations
Paths
Constraints
Indexes
```

Practice at least:

```text
15 Cypher queries
```

Before sleeping, explain aloud:

```text
What is Knowledge Graph?
Why Neo4j?
Node vs property?
MERGE vs CREATE?
OPTIONAL MATCH?
```

---

## DAY 2 MORNING

Study:

```text
Advanced Cypher
Subqueries
Shortest paths
Performance
APOC
```

## DAY 2 AFTERNOON

Study:

```text
GDS
PageRank
Degree
Betweenness
Louvain
WCC
Node Similarity
Dijkstra
Embeddings
```

Practice:

```text
Python Neo4j driver
Batch ingestion
Transactions
```

Minimum target:

```text
20 Cypher problems
5 Python functions
```

---

## DAY 3 MORNING

Study:

```text
RAG
Embeddings
Chunking
Vector search
Metadata
Reranking
```

## DAY 3 AFTERNOON

Study:

```text
GraphRAG
Entity extraction
Entity resolution
Hybrid retrieval
Grounding
Evaluation
```

Practice describing this architecture from memory:

```text
Question
↓
Vector retrieval
↓
Graph expansion
↓
Reranking
↓
Prompt
↓
LLM
↓
Grounded response
```

---

## DAY 4 MORNING

Study:

```text
Bedrock
Knowledge Bases
Neptune
Lambda
API Gateway
DynamoDB
S3
OpenSearch
```

## DAY 4 AFTERNOON

Practice:

```text
System design
Security
Observability
Performance
Cost
Agent design
MCP
Multi-agent
```

Then conduct a mock interview:

```text
20 Cypher questions
10 Graph concepts
10 RAG/GraphRAG
10 AWS
5 system design
5 behavioral
```

---

# LAST 2 HOURS BEFORE INTERVIEW

Do NOT start learning new large topics.

Review only:

1. Graph modeling
2. MATCH / MERGE / WITH / UNWIND
3. 2-hop and recommendation queries
4. EXPLAIN / PROFILE
5. GDS categories
6. PageRank / Louvain / similarity / shortest path
7. Python Neo4j driver
8. RAG flow
9. GraphRAG flow
10. hybrid retrieval
11. Bedrock + Lambda architecture
12. security
13. one GraphRAG project explanation
14. 60-second introduction

---

# RAPID-FIRE MOCK INTERVIEW

Try answering each in 30–60 seconds without reading the answers.

1. What is a graph database?
2. Why Neo4j?
3. Graph vs relational?
4. What is a knowledge graph?
5. What is an ontology?
6. Node vs property?
7. Relationship direction?
8. CREATE vs MERGE?
9. MATCH vs OPTIONAL MATCH?
10. WITH?
11. UNWIND?
12. EXISTS?
13. CALL subquery?
14. Constraints?
15. Indexes?
16. EXPLAIN vs PROFILE?
17. Why parameters?
18. What causes Cartesian product?
19. How do you optimize Cypher?
20. What is a supernode?
21. What is APOC?
22. What is GDS?
23. What is graph projection?
24. PageRank?
25. Louvain?
26. WCC?
27. Node Similarity?
28. Jaccard?
29. Dijkstra?
30. Node embeddings?
31. RAG?
32. Embedding?
33. Chunking?
34. Vector database?
35. Top-k?
36. Reranking?
37. GraphRAG?
38. Why GraphRAG?
39. Hybrid retrieval?
40. Entity extraction?
41. Entity resolution?
42. Provenance?
43. Grounding?
44. Hallucination?
45. Evaluation?
46. LangChain?
47. Agent?
48. Tool calling?
49. MCP?
50. Multi-agent?
51. Bedrock?
52. Bedrock Knowledge Base?
53. Neptune?
54. Lambda?
55. API Gateway?
56. DynamoDB?
57. S3?
58. OpenSearch?
59. GraphRAG security?
60. How would you design a production Graph AI application?

---

# 10 MOST IMPORTANT QUESTIONS FOR THIS JOB DESCRIPTION

If time becomes very short, master these first.

---

## 1. Explain Knowledge Graph architecture.

Answer:

```text
Source systems
↓
ingestion
↓
entity extraction
↓
entity resolution
↓
graph model
↓
Neo4j/Neptune
↓
Cypher/traversal
↓
applications / GraphRAG
```

---

## 2. Explain GraphRAG.

Answer:

```text
Vector retrieval finds semantically relevant information.
Graph traversal adds explicit connected context.
The combined context grounds the LLM.
```

---

## 3. How do you optimize a Cypher query?

Answer keywords:

```text
indexes
selective starting node
filter early
relationship types
hop limits
parameters
PROFILE
avoid Cartesian products
return only needed data
```

---

## 4. Explain MERGE.

```text
match-or-create graph pattern
stable keys
constraints
ON CREATE
ON MATCH
```

---

## 5. Explain GDS algorithms.

```text
PageRank → importance
Louvain → communities
WCC → connected groups
Node Similarity → similar neighborhoods
Dijkstra → shortest weighted path
Embeddings → numeric graph representation
```

---

## 6. Explain Neo4j Python integration.

```text
GraphDatabase.driver
session
execute_read / execute_write
parameterized Cypher
UNWIND batches
connection reuse
error handling
```

---

## 7. Explain a hybrid GraphRAG pipeline.

```text
Question
↓
embedding
↓
vector search
↓
candidate entities/chunks
↓
Cypher graph expansion
↓
rerank
↓
prompt
↓
Bedrock LLM
```

---

## 8. How would you deploy it in AWS?

```text
API Gateway
↓
Lambda/service
↓
Neo4j/Neptune + vector store
↓
Bedrock
↓
response

S3 for documents
DynamoDB for state/metadata
CloudWatch for monitoring
IAM + secret management for security
```

---

## 9. How do you secure Graph AI?

```text
least privilege
read-only graph user
IAM
secrets
TLS
tenant filtering
query validation
tool allowlists
timeouts
audit logs
```

---

## 10. How do you evaluate GraphRAG?

```text
retrieval recall/precision
entity accuracy
path correctness
groundedness
faithfulness
latency
cost
citation/provenance correctness
```

---

# CURRENT PLATFORM NOTES FOR INTERVIEW PREP

Technology changes quickly, so avoid memorizing older product behavior as permanent.

As of the preparation date:

- Neo4j's Graph Data Science Python client provides Python interfaces for graph projections, algorithms, and ML pipelines.
- GDS algorithm families include centrality, community detection, similarity, path finding, node embeddings, and link prediction-related capabilities.
- Neo4j Node Similarity supports neighborhood similarity approaches including Jaccard, Overlap, and Cosine.
- Amazon Bedrock Knowledge Bases supports RAG workflows and APIs such as Retrieve and RetrieveAndGenerate.
- Neptune Analytics supports vector similarity search so graph relationships and embedding-based similarity can be used together.
- AWS's agent platform has evolved; Bedrock Agents Classic is no longer the only agent architecture to know. For interviews, discuss the underlying tool/action orchestration design and be aware of newer AgentCore capabilities.

---

# OFFICIAL DOCUMENTATION REFERENCES

Use these for last-minute verification:

- Neo4j Cypher Manual: https://neo4j.com/docs/cypher-manual/current/
- Neo4j Cypher Cheat Sheet: https://neo4j.com/docs/cypher-manual/current/cheat-sheet/
- Neo4j Graph Data Science: https://neo4j.com/docs/graph-data-science/current/
- Neo4j GDS Python Client: https://neo4j.com/docs/graph-data-science-client/current/
- Amazon Bedrock Knowledge Bases: https://docs.aws.amazon.com/bedrock/latest/userguide/knowledge-base.html
- Amazon Neptune Analytics: https://docs.aws.amazon.com/neptune-analytics/latest/userguide/
- Amazon Bedrock AgentCore: https://docs.aws.amazon.com/bedrock-agentcore/

---

# FINAL INTERVIEW POSITIONING

Your best positioning is:

> **Data Engineer + Neo4j Certified Professional + Graph AI Engineer**

Do not present yourself only as someone who memorized Cypher.

Connect your prior data background to the Graph AI role:

```text
Snowflake
    ↓
data engineering discipline
    ↓
clean governed enterprise data
    ↓
Knowledge Graph / Neo4j
    ↓
relationship intelligence
    ↓
GraphRAG
    ↓
LLM / Bedrock applications
```

A strong interview statement:

> My data-engineering background helps me think beyond the demo. I focus not only on generating an LLM response, but on building reliable ingestion, canonical entity identity, graph modeling, query performance, retrieval quality, security, observability, and data lineage. Neo4j adds the connected-data layer, and GraphRAG allows the LLM to use those relationships as grounded context.

---

# FINAL MEMORY MAP

```text
                     GRAPH AI ENGINEER
                           |
        ┌──────────────────┼───────────────────┐
        |                  |                   |
      GRAPH               AI                 AWS
        |                  |                   |
   Neo4j/Cypher          RAG                Bedrock
   Knowledge Graph       GraphRAG           Neptune
   APOC                  Embeddings          Lambda
   GDS                   LangChain           API Gateway
   Python                Agents              S3
                         MCP                 DynamoDB
                         Multi-Agent         OpenSearch
```

If you can confidently explain this map and write the Cypher/Python exercises in this guide without looking at the answers, you will be well prepared for a Graph AI Engineer interview.

---

# EXPANDED HANDS-ON PYTHON PRACTICE — CYPHER + APOC + GDS

This section extends the 4-day plan with **more practical Python coding** so you can practice Graph AI exactly from an interview perspective.

The recommended progression is:

```text
Python → Neo4j Driver → Cypher → APOC → GDS → GraphRAG
```

A good rule for interview preparation is:

> First write the Cypher manually. Then call the same Cypher from Python. Finally explain how you would productionize it.

---

# PART A — PYTHON + CYPHER PRACTICE

## 115. Modern Neo4j Python Driver Setup

Install:

```bash
pip install neo4j
```

Basic connection:

```python
from neo4j import GraphDatabase

URI = "neo4j+s://<host>"
AUTH = ("neo4j", "<password>")
DATABASE = "neo4j"

with GraphDatabase.driver(URI, auth=AUTH) as driver:
    driver.verify_connectivity()
    print("Neo4j connection successful")
```

### Interview point

Keep one long-lived `Driver` per application instead of creating a new driver for every request.

---

## 116. Run a Simple Read Query from Python

Cypher:

```cypher
MATCH (c:Customer)
RETURN c.id AS id,
       c.name AS name
ORDER BY c.name
LIMIT 10;
```

Python:

```python
from neo4j import GraphDatabase

URI = "neo4j+s://<host>"
AUTH = ("neo4j", "<password>")

with GraphDatabase.driver(URI, auth=AUTH) as driver:

    records, summary, keys = driver.execute_query(
        """
        MATCH (c:Customer)
        RETURN c.id AS id,
               c.name AS name
        ORDER BY c.name
        LIMIT 10
        """,
        database_="neo4j"
    )

    for record in records:
        print(record["id"], record["name"])
```

### Practice

Change the query to return only customers from Georgia.

Answer:

```python
records, summary, keys = driver.execute_query(
    """
    MATCH (c:Customer)
    WHERE c.state = $state
    RETURN c.id AS id,
           c.name AS name
    ORDER BY c.name
    """,
    state="GA",
    database_="neo4j"
)
```

---

## 117. Parameterized Query Practice

Never do this:

```python
customer_id = "C101"

query = f"""
MATCH (c:Customer {{id:'{customer_id}'}})
RETURN c
"""
```

Prefer:

```python
query = """
MATCH (c:Customer {id:$customer_id})
RETURN c.id AS id,
       c.name AS name
"""

records, summary, keys = driver.execute_query(
    query,
    customer_id="C101",
    database_="neo4j"
)
```

### Memory trick

```text
Python values → $parameters
```

---

## 118. Create a Node from Python

```python
query = """
CREATE (c:Customer {
    id:$id,
    name:$name,
    state:$state
})
RETURN c.id AS id,
       c.name AS name
"""

records, summary, keys = driver.execute_query(
    query,
    id="C900",
    name="John",
    state="GA",
    database_="neo4j"
)

print(records[0].data())
```

### Interview follow-up

Why would you usually prefer `MERGE` over `CREATE` in ingestion pipelines?

Because retries can otherwise create duplicate entities.

---

## 119. Upsert Using MERGE from Python

```python
query = """
MERGE (c:Customer {id:$id})
ON CREATE SET
    c.name = $name,
    c.state = $state,
    c.createdAt = datetime()
ON MATCH SET
    c.name = $name,
    c.state = $state,
    c.updatedAt = datetime()
RETURN c
"""

records, summary, keys = driver.execute_query(
    query,
    id="C101",
    name="Alice",
    state="GA",
    database_="neo4j"
)
```

### Interview point

Use a uniqueness constraint on the MERGE key:

```cypher
CREATE CONSTRAINT customer_id_unique IF NOT EXISTS
FOR (c:Customer)
REQUIRE c.id IS UNIQUE;
```

---

## 120. Create Relationships from Python

```python
query = """
MATCH (c:Customer {id:$customer_id})
MATCH (p:Product {id:$product_id})

MERGE (c)-[r:PURCHASED]->(p)
SET r.amount = $amount,
    r.purchaseDate = date($purchase_date)

RETURN c.name AS customer,
       p.name AS product,
       r.amount AS amount
"""

records, summary, keys = driver.execute_query(
    query,
    customer_id="C101",
    product_id="P200",
    amount=599.99,
    purchase_date="2026-09-20",
    database_="neo4j"
)
```

---

## 121. Read One Record Safely

```python
query = """
MATCH (c:Customer {id:$id})
RETURN c.id AS id,
       c.name AS name,
       c.state AS state
"""

records, summary, keys = driver.execute_query(
    query,
    id="C101",
    database_="neo4j"
)

customer = records[0].data() if records else None

print(customer)
```

---

## 122. Convert Neo4j Results into List of Dictionaries

```python
records, summary, keys = driver.execute_query(
    """
    MATCH (p:Product)
    RETURN p.id AS id,
           p.name AS name,
           p.price AS price
    LIMIT 20
    """,
    database_="neo4j"
)

products = [record.data() for record in records]

for product in products:
    print(product)
```

---

## 123. Batch Insert with UNWIND

Python:

```python
customers = [
    {"id": "C1", "name": "Alice", "state": "GA"},
    {"id": "C2", "name": "Bob", "state": "NY"},
    {"id": "C3", "name": "Carol", "state": "TX"}
]

query = """
UNWIND $customers AS row

MERGE (c:Customer {id:row.id})
SET c.name = row.name,
    c.state = row.state
"""

records, summary, keys = driver.execute_query(
    query,
    customers=customers,
    database_="neo4j"
)

print(summary.counters)
```

### Snowflake analogy

Think:

```text
Python list
   ↓
UNWIND
   ↓
rows inside Cypher
```

Similar mental idea to expanding arrays/variants with `FLATTEN`.

---

## 124. Batch Insert Products and Relationships

```python
purchases = [
    {
        "customer_id": "C1",
        "product_id": "P1",
        "product_name": "Laptop"
    },
    {
        "customer_id": "C1",
        "product_id": "P2",
        "product_name": "Mouse"
    },
    {
        "customer_id": "C2",
        "product_id": "P1",
        "product_name": "Laptop"
    }
]

query = """
UNWIND $rows AS row

MATCH (c:Customer {id:row.customer_id})

MERGE (p:Product {id:row.product_id})
SET p.name = row.product_name

MERGE (c)-[:PURCHASED]->(p)
"""

driver.execute_query(
    query,
    rows=purchases,
    database_="neo4j"
)
```

---

## 125. Python Function — Find Customer Purchases

```python
def get_customer_products(driver, customer_id):

    query = """
    MATCH (:Customer {id:$customer_id})
          -[r:PURCHASED]->
          (p:Product)

    RETURN
        p.id AS product_id,
        p.name AS product_name,
        r.amount AS amount

    ORDER BY r.amount DESC
    """

    records, _, _ = driver.execute_query(
        query,
        customer_id=customer_id,
        database_="neo4j"
    )

    return [record.data() for record in records]
```

Usage:

```python
products = get_customer_products(
    driver,
    "C101"
)

for product in products:
    print(product)
```

---

## 126. Python Function — Find Two-Degree Connections

```python
def two_degree_connections(driver, person_name):

    query = """
    MATCH
      (a:Person {name:$name})
      -[:KNOWS]->
      (:Person)
      -[:KNOWS]->
      (candidate:Person)

    WHERE candidate <> a
      AND NOT (a)-[:KNOWS]-(candidate)

    RETURN DISTINCT candidate.name AS name
    ORDER BY name
    """

    records, _, _ = driver.execute_query(
        query,
        name=person_name,
        database_="neo4j"
    )

    return [r["name"] for r in records]
```

Interview question:

> Why is DISTINCT needed?

Because the same second-degree person may be reachable through multiple mutual connections.

---

## 127. Python Recommendation Query

```python
def recommend_products(driver, customer_id, limit=10):

    query = """
    MATCH
      (target:Customer {id:$customer_id})
      -[:PURCHASED]->
      (common:Product)
      <-[:PURCHASED]-
      (similar:Customer)
      -[:PURCHASED]->
      (recommendation:Product)

    WHERE recommendation <> common
      AND NOT (target)-[:PURCHASED]->(recommendation)

    RETURN
      recommendation.id AS product_id,
      recommendation.name AS product_name,
      count(DISTINCT similar) AS score

    ORDER BY score DESC
    LIMIT $limit
    """

    records, _, _ = driver.execute_query(
        query,
        customer_id=customer_id,
        limit=limit,
        database_="neo4j"
    )

    return [r.data() for r in records]
```

### Interview explanation

This is collaborative-filtering style logic expressed as graph traversal:

```text
Target customer
→ shared product
→ similar customer
→ candidate product
```

---

## 128. Find Shortest Path from Python

```python
def shortest_connection(driver, source, target):

    query = """
    MATCH
      (a:Person {name:$source}),
      (b:Person {name:$target})

    MATCH p = shortestPath(
      (a)-[:KNOWS*]-(b)
    )

    RETURN
      [node IN nodes(p) | node.name]
      AS path
    """

    records, _, _ = driver.execute_query(
        query,
        source=source,
        target=target,
        database_="neo4j"
    )

    return records[0]["path"] if records else None
```

---

## 129. Managed Transaction Example

Use managed transactions when several operations logically belong together.

```python
def transfer_relationship(tx, old_owner, new_owner, asset_id):

    tx.run(
        """
        MATCH (old:Person {id:$old_owner})
              -[r:OWNS]->
              (asset:Asset {id:$asset_id})
        DELETE r
        """,
        old_owner=old_owner,
        asset_id=asset_id
    )

    tx.run(
        """
        MATCH (new:Person {id:$new_owner}),
              (asset:Asset {id:$asset_id})
        MERGE (new)-[:OWNS]->(asset)
        """,
        new_owner=new_owner,
        asset_id=asset_id
    )


with driver.session(database="neo4j") as session:
    session.execute_write(
        transfer_relationship,
        "P1",
        "P2",
        "A100"
    )
```

### Interview point

The managed transaction retries transient failures according to driver behavior and keeps logically related operations together.

---

## 130. Query Summary and Counters

```python
records, summary, keys = driver.execute_query(
    """
    MERGE (p:Person {id:$id})
    SET p.name = $name
    """,
    id="P100",
    name="Alice",
    database_="neo4j"
)

print("Nodes created:", summary.counters.nodes_created)
print("Properties set:", summary.counters.properties_set)
print("Query time:", summary.result_available_after)
```

Useful for:

- logging
- unit tests
- performance testing
- verifying ingestion

---

## 131. Python Exception Handling

```python
from neo4j.exceptions import Neo4jError

try:
    records, summary, keys = driver.execute_query(
        """
        MATCH (c:Customer {id:$id})
        RETURN c
        """,
        id="C101",
        database_="neo4j"
    )

except Neo4jError as exc:
    print("Neo4j error:", exc)
```

Production application:

```text
catch
log
classify retryable vs non-retryable
return safe error
```

---

## 132. Python + Cypher Pagination

```python
def get_products_page(driver, skip, limit):

    query = """
    MATCH (p:Product)
    RETURN p.id AS id,
           p.name AS name
    ORDER BY p.id
    SKIP $skip
    LIMIT $limit
    """

    records, _, _ = driver.execute_query(
        query,
        skip=skip,
        limit=limit,
        database_="neo4j"
    )

    return [r.data() for r in records]
```

Interview caveat:

For very large datasets, keyset/cursor-style pagination can be more scalable than large `SKIP` values.

---

## 133. Dynamic Filtering Without Building Unsafe Query Strings

```python
def find_products(driver, category=None, min_price=None):

    query = """
    MATCH (p:Product)

    WHERE
      ($category IS NULL OR p.category = $category)
      AND
      ($min_price IS NULL OR p.price >= $min_price)

    RETURN p.id AS id,
           p.name AS name,
           p.category AS category,
           p.price AS price

    ORDER BY p.name
    """

    records, _, _ = driver.execute_query(
        query,
        category=category,
        min_price=min_price,
        database_="neo4j"
    )

    return [r.data() for r in records]
```

---

## 134. Python Graph Data Validation

Check orphan products:

```python
def find_products_without_supplier(driver):

    query = """
    MATCH (p:Product)
    WHERE NOT EXISTS {
        MATCH (p)-[:SUPPLIED_BY]->(:Supplier)
    }
    RETURN p.id AS id,
           p.name AS name
    """

    records, _, _ = driver.execute_query(
        query,
        database_="neo4j"
    )

    return [r.data() for r in records]
```

This is useful in graph ingestion validation.

---

## 135. Data Quality Check — Duplicate Business Keys

```python
def duplicate_customer_ids(driver):

    query = """
    MATCH (c:Customer)
    WITH c.id AS id,
         count(*) AS count
    WHERE count > 1
    RETURN id, count
    ORDER BY count DESC
    """

    records, _, _ = driver.execute_query(
        query,
        database_="neo4j"
    )

    return [r.data() for r in records]
```

Better prevention:

```cypher
CREATE CONSTRAINT customer_id_unique IF NOT EXISTS
FOR (c:Customer)
REQUIRE c.id IS UNIQUE;
```

---

# PART B — PYTHON + APOC PRACTICE

## 136. What APOC Adds

Use APOC when native Cypher becomes unnecessarily complex for operations such as:

```text
collection utilities
JSON conversion
schema helpers
graph refactoring
path expansion
dynamic operations
import/export utilities
```

Interview rule:

> Native Cypher first; APOC when it adds a clear benefit.

---

## 137. Call APOC from Python — Deduplicate a List

Cypher:

```cypher
RETURN apoc.coll.toSet($skills) AS uniqueSkills;
```

Python:

```python
skills = [
    "Neo4j",
    "Python",
    "AWS",
    "Neo4j",
    "Python"
]

records, _, _ = driver.execute_query(
    """
    RETURN apoc.coll.toSet($skills)
           AS uniqueSkills
    """,
    skills=skills,
    database_="neo4j"
)

print(records[0]["uniqueSkills"])
```

Expected concept:

```text
Neo4j
Python
AWS
```

---

## 138. APOC JSON Serialization from Python

```python
payload = {
    "customer": "C101",
    "products": ["Laptop", "Mouse"]
}

records, _, _ = driver.execute_query(
    """
    RETURN apoc.convert.toJson($payload)
           AS json
    """,
    payload=payload,
    database_="neo4j"
)

print(records[0]["json"])
```

Useful when:

- API output
- logging
- integration payload preparation

---

## 139. Parse JSON with APOC

```python
json_text = '''
{
  "customerId":"C101",
  "name":"Alice",
  "skills":["Neo4j","Python"]
}
'''

records, _, _ = driver.execute_query(
    """
    WITH apoc.convert.fromJsonMap($json)
         AS payload

    RETURN
      payload.customerId AS customerId,
      payload.name AS name,
      payload.skills AS skills
    """,
    json=json_text,
    database_="neo4j"
)

print(records[0].data())
```

---

## 140. APOC Text Utility Example

```python
records, _, _ = driver.execute_query(
    """
    RETURN apoc.text.capitalize($value)
           AS formatted
    """,
    value="neo4j",
    database_="neo4j"
)
```

Interview point:

Text cleanup is often better done before loading, but APOC can help when graph-side transformation is convenient.

---

## 141. APOC Collection Intersection

Question:

> How many skills do two engineers share?

```python
skills_a = ["Neo4j", "Python", "AWS"]
skills_b = ["Python", "AWS", "LangChain"]

records, _, _ = driver.execute_query(
    """
    RETURN apoc.coll.intersection(
        $skills_a,
        $skills_b
    ) AS common
    """,
    skills_a=skills_a,
    skills_b=skills_b,
    database_="neo4j"
)

print(records[0]["common"])
```

---

## 142. APOC Collection Difference

```python
records, _, _ = driver.execute_query(
    """
    RETURN apoc.coll.subtract(
        $required,
        $candidate
    ) AS missingSkills
    """,
    required=["Neo4j", "Python", "AWS", "Bedrock"],
    candidate=["Neo4j", "Python", "AWS"],
    database_="neo4j"
)

print(records[0]["missingSkills"])
```

Possible interview application:

```text
Job-required skills
minus
candidate skills
=
missing skills
```

---

## 143. APOC Virtual Node Example

Virtual nodes can be useful for visualization or computed graph views without persisting a real node.

```python
records, _, _ = driver.execute_query(
    """
    RETURN apoc.create.vNode(
      ['InterviewSummary'],
      {
        candidate:$candidate,
        score:$score
      }
    ) AS virtualNode
    """,
    candidate="Alice",
    score=92,
    database_="neo4j"
)
```

Interview point:

A virtual node is returned for the query result and is not stored as a normal persisted node.

---

## 144. APOC Path Expansion Concept

A useful advanced interview topic is controlled path expansion.

Conceptual Cypher:

```cypher
MATCH (start:Customer {id:$customerId})
CALL apoc.path.expandConfig(
    start,
    {
      relationshipFilter:'PURCHASED>|SUPPLIED_BY>',
      minLevel:1,
      maxLevel:3,
      uniqueness:'NODE_GLOBAL'
    }
)
YIELD path
RETURN path;
```

Python:

```python
def impacted_network(driver, customer_id):

    query = """
    MATCH (start:Customer {id:$customer_id})

    CALL apoc.path.expandConfig(
      start,
      {
        relationshipFilter:'PURCHASED>|SUPPLIED_BY>',
        minLevel:1,
        maxLevel:3,
        uniqueness:'NODE_GLOBAL'
      }
    )
    YIELD path

    RETURN path
    LIMIT 100
    """

    records, _, _ = driver.execute_query(
        query,
        customer_id=customer_id,
        database_="neo4j"
    )

    return records
```

### Why this matters

It lets you control traversal behavior more explicitly than a broad unrestricted path expression.

---

## 145. APOC Procedure Discovery

During practice, you can inspect available procedures/functions because APOC availability differs by environment.

```cypher
SHOW PROCEDURES
YIELD name
WHERE name STARTS WITH 'apoc.'
RETURN name
ORDER BY name;
```

Python:

```python
records, _, _ = driver.execute_query(
    """
    SHOW PROCEDURES
    YIELD name
    WHERE name STARTS WITH 'apoc.'
    RETURN name
    ORDER BY name
    """,
    database_="neo4j"
)

for r in records:
    print(r["name"])
```

---

# PART C — PYTHON + GRAPH DATA SCIENCE (GDS)

## 146. Install the GDS Python Client

```bash
pip install graphdatascience
```

Import:

```python
from graphdatascience import GraphDataScience
```

Connect:

```python
from graphdatascience import GraphDataScience

GDS_URI = "neo4j+s://<host>"
AUTH = ("neo4j", "<password>")

gds = GraphDataScience(
    GDS_URI,
    auth=AUTH,
    database="neo4j"
)

print(gds.version())
```

### Interview point

The GDS Python client is a higher-level Python interface over the GDS procedures and uses familiar Python objects and DataFrames.

---

## 147. Create Sample GDS Graph Data

```python
cypher = """
MERGE (a:Person {name:'Alice'})
MERGE (b:Person {name:'Bob'})
MERGE (c:Person {name:'Carol'})
MERGE (d:Person {name:'David'})
MERGE (e:Person {name:'Emma'})

MERGE (a)-[:KNOWS]->(b)
MERGE (a)-[:KNOWS]->(c)
MERGE (b)-[:KNOWS]->(d)
MERGE (c)-[:KNOWS]->(d)
MERGE (d)-[:KNOWS]->(e)
"""

gds.run_cypher(cypher)
```

---

## 148. Project an In-Memory Graph

```python
G, result = gds.graph.project(
    "socialGraph",
    "Person",
    "KNOWS"
)

print(result)
```

Important concept:

```text
Neo4j database graph
        ↓
GDS projection
        ↓
in-memory analytical graph
```

---

## 149. Check Graph Projection

```python
print(G.name())
print(G.node_count())
print(G.relationship_count())
```

Interview question:

> Why does GDS project data?

Because graph algorithms need optimized in-memory structures tailored for analytical processing.

---

## 150. PageRank with Python

```python
result = gds.page_rank.stream(G)

print(result.head())
```

Common returned fields include algorithm-specific identifiers and scores.

To return useful business properties, enrich results using graph/node lookups or run equivalent Cypher-based result handling as appropriate to your environment.

### Interview explanation

PageRank identifies influential nodes by considering both incoming connections and the importance of connected nodes.

---

## 151. PageRank Configuration

```python
result = gds.page_rank.stream(
    G,
    maxIterations=20,
    dampingFactor=0.85
)

print(result.sort_values(
    "score",
    ascending=False
).head(10))
```

Important parameters:

```text
maxIterations
 dampingFactor
 tolerance
```

Do not memorize every parameter. Understand what affects convergence and score behavior.

---

## 152. Write PageRank Back to Neo4j

```python
stats = gds.page_rank.write(
    G,
    writeProperty="pageRankScore"
)

print(stats)
```

Now query:

```python
result = gds.run_cypher(
    """
    MATCH (p:Person)
    RETURN p.name AS name,
           p.pageRankScore AS score
    ORDER BY score DESC
    """
)

print(result)
```

Interview distinction:

```text
stream → return result
write → persist property to Neo4j
```

---

## 153. Degree Centrality in Python

```python
result = gds.degree.stream(G)

print(
    result.sort_values(
        "score",
        ascending=False
    ).head(10)
)
```

Use case:

```text
most directly connected entity
```

---

## 154. Betweenness Centrality

```python
result = gds.betweenness.stream(G)

print(
    result.sort_values(
        "score",
        ascending=False
    ).head(10)
)
```

Memory:

```text
Betweenness = bridge
```

Interview use cases:

- supply chain bottleneck
- fraud intermediary
- network broker
- critical dependency

---

## 155. Louvain Community Detection

```python
result = gds.louvain.stream(G)

print(result.head())
```

Sort/group by community:

```python
print(
    result.sort_values(
        "communityId"
    ).head(20)
)
```

Use cases:

```text
fraud rings
customer communities
social communities
entity clusters
```

---

## 156. Write Community ID to Nodes

```python
stats = gds.louvain.write(
    G,
    writeProperty="communityId"
)

print(stats)
```

Then:

```python
communities = gds.run_cypher(
    """
    MATCH (p:Person)
    RETURN p.communityId AS community,
           collect(p.name) AS members
    ORDER BY community
    """
)

print(communities)
```

---

## 157. Weakly Connected Components

```python
result = gds.wcc.stream(G)

print(result.head())
```

Interpretation:

Nodes with the same component ID are connected when relationship direction is ignored according to the algorithm's semantics/configuration.

---

## 158. Node Similarity Dataset

Create customers/products:

```python
gds.run_cypher(
    """
    MERGE (a:Customer {id:'C1', name:'Alice'})
    MERGE (b:Customer {id:'C2', name:'Bob'})
    MERGE (c:Customer {id:'C3', name:'Carol'})

    MERGE (p1:Product {id:'P1', name:'Laptop'})
    MERGE (p2:Product {id:'P2', name:'Mouse'})
    MERGE (p3:Product {id:'P3', name:'Keyboard'})
    MERGE (p4:Product {id:'P4', name:'Monitor'})

    MERGE (a)-[:PURCHASED]->(p1)
    MERGE (a)-[:PURCHASED]->(p2)
    MERGE (a)-[:PURCHASED]->(p3)

    MERGE (b)-[:PURCHASED]->(p1)
    MERGE (b)-[:PURCHASED]->(p2)
    MERGE (b)-[:PURCHASED]->(p4)

    MERGE (c)-[:PURCHASED]->(p4)
    """
)
```

---

## 159. Project Customer-Product Graph

```python
customer_graph, result = gds.graph.project(
    "customerProductGraph",
    ["Customer", "Product"],
    "PURCHASED"
)

print(result)
```

---

## 160. Run Node Similarity

```python
similarity = gds.node_similarity.stream(
    customer_graph,
    topK=5
)

print(similarity.head(20))
```

Concept:

Customers are similar if they connect to many of the same products.

---

## 161. Filter Similarity Threshold

```python
similarity = gds.node_similarity.stream(
    customer_graph,
    topK=10,
    similarityCutoff=0.3
)

print(similarity)
```

Interview answer:

`similarityCutoff` can reduce noisy weak matches and reduce returned results.

---

## 162. Build Recommendations from Similarity Results

Conceptual approach:

```text
1. compute similar customers
2. identify products of similar customers
3. remove already purchased products
4. rank candidates
```

You can combine GDS similarity with Cypher recommendation logic.

Python example:

```python
def recommend_from_similar_customer(
    driver,
    customer_id,
    similar_customer_ids
):

    query = """
    MATCH (candidateCustomer:Customer)
    WHERE candidateCustomer.id IN $similar_ids

    MATCH (candidateCustomer)-[:PURCHASED]->(p:Product)

    MATCH (target:Customer {id:$customer_id})

    WHERE NOT (target)-[:PURCHASED]->(p)

    RETURN
      p.id AS product_id,
      p.name AS product_name,
      count(*) AS score

    ORDER BY score DESC
    LIMIT 10
    """

    records, _, _ = driver.execute_query(
        query,
        customer_id=customer_id,
        similar_ids=similar_customer_ids,
        database_="neo4j"
    )

    return [r.data() for r in records]
```

---

## 163. Dijkstra Weighted Shortest Path — Conceptual GDS Practice

Imagine:

```text
City A --100--> City B
City B --80--> City C
City A --300--> City C
```

Relationships:

```cypher
(:City)-[:ROAD {distance:100}]->(:City)
```

Projection must include the relationship weight property.

Example concept:

```python
road_graph, result = gds.graph.project(
    "roadGraph",
    "City",
    {
        "ROAD": {
            "properties": "distance"
        }
    }
)
```

Then weighted shortest-path algorithms can use `distance` as relationship weight.

### Interview answer

Dijkstra is appropriate for shortest paths when weights are non-negative.

---

## 164. FastRP Node Embeddings

```python
embedding_result = gds.fast_rp.stream(
    G,
    embeddingDimension=64
)

print(embedding_result.head())
```

Concept:

```text
Node
↓
64-dimensional vector
↓
similarity / ML feature
```

Interview answer:

FastRP produces graph-structure-aware node embeddings efficiently and can be useful as features for downstream similarity or machine learning.

---

## 165. Write FastRP Embeddings

```python
stats = gds.fast_rp.write(
    G,
    embeddingDimension=64,
    writeProperty="graphEmbedding"
)

print(stats)
```

Then nodes contain a numerical vector property suitable for downstream use according to your architecture.

---

## 166. Memory Estimation

Before running expensive algorithms, estimate memory when supported by the relevant algorithm/projection workflow.

Interview point:

> In production I do not run large GDS jobs blindly. I estimate memory, validate projected graph size, and test algorithm parameters before full execution.

Useful checks:

```python
print(G.node_count())
print(G.relationship_count())
```

---

## 167. Drop GDS Projection

Always clean up temporary graphs when appropriate.

```python
gds.graph.drop(G)
```

Or:

```python
gds.graph.drop(customer_graph)
```

Memory trick:

```text
Project
Run
Use result
Drop
```

---

## 168. List Existing GDS Graphs

```python
catalog = gds.graph.list()

print(catalog)
```

Useful during debugging if a graph name already exists.

---

## 169. Avoid Duplicate Projection Names

```python
exists = gds.graph.exists("socialGraph")

print(exists)
```

Conceptual defensive logic:

```python
if gds.graph.exists("socialGraph")["exists"]:
    existing = gds.graph.get("socialGraph")
    gds.graph.drop(existing)
```

Then recreate projection.

Exact object/return handling can vary by client version, so verify it against your installed version during practice.

---

# PART D — COMBINED PYTHON + CYPHER + APOC + GDS MINI PROJECTS

## 170. Mini Project 1 — Fraud Network

### Graph Model

```text
(:Customer)-[:OWNS]->(:Account)
(:Account)-[:TRANSFERRED_TO {amount,date}]->(:Account)
(:Customer)-[:USES]->(:Device)
(:Customer)-[:HAS_ADDRESS]->(:Address)
```

### Questions

1. Which customers share devices?
2. Which customers share addresses?
3. Which accounts participate in transfer cycles?
4. Which customers form connected fraud communities?
5. Which account is a bridge between communities?

### Shared device Cypher

```cypher
MATCH
(c1:Customer)-[:USES]->(d:Device)<-[:USES]-(c2:Customer)
WHERE c1.id < c2.id
RETURN
c1.id,
c2.id,
d.id;
```

Python:

```python
def customers_sharing_devices(driver):

    query = """
    MATCH
      (c1:Customer)-[:USES]->(d:Device)<-[:USES]-(c2:Customer)
    WHERE c1.id < c2.id
    RETURN
      c1.id AS customer1,
      c2.id AS customer2,
      d.id AS device
    """

    records, _, _ = driver.execute_query(
        query,
        database_="neo4j"
    )

    return [r.data() for r in records]
```

### GDS extension

Use:

```text
WCC → identify connected suspicious groups
Louvain → find dense communities
Betweenness → identify bridge accounts
PageRank → identify structurally influential accounts
```

---

## 171. Mini Project 2 — Product Recommendation

### Graph

```text
Customer → PURCHASED → Product
Product → BELONGS_TO → Category
```

### Phase 1

Cypher co-purchase recommendations.

### Phase 2

GDS Node Similarity.

### Phase 3

Python API exposing:

```text
GET /customers/{id}/recommendations
```

Core Python function:

```python
def recommendations(driver, customer_id):

    query = """
    MATCH
      (target:Customer {id:$id})
      -[:PURCHASED]->
      (shared:Product)
      <-[:PURCHASED]-
      (neighbor:Customer)
      -[:PURCHASED]->
      (candidate:Product)

    WHERE NOT (target)-[:PURCHASED]->(candidate)

    RETURN
      candidate.id AS product_id,
      candidate.name AS product,
      count(DISTINCT neighbor) AS score

    ORDER BY score DESC
    LIMIT 10
    """

    records, _, _ = driver.execute_query(
        query,
        id=customer_id,
        database_="neo4j"
    )

    return [r.data() for r in records]
```

---

## 172. Mini Project 3 — Supply Chain Impact Analysis

Graph:

```text
Supplier
↓ SUPPLIES
Component
↓ USED_IN
Product
↓ INCLUDED_IN
Order
↓ PLACED_BY
Customer
```

Question:

> Supplier S1 has a quality issue. Which customers are affected?

Cypher:

```cypher
MATCH
(s:Supplier {id:$supplier_id})
-[:SUPPLIES]->
(component:Component)
-[:USED_IN]->
(product:Product)
<-[:CONTAINS]-
(order:Order)
<-[:PLACED]-
(customer:Customer)

RETURN DISTINCT
customer.id,
customer.name,
product.name,
component.name;
```

Python:

```python
def supplier_impact(driver, supplier_id):

    query = """
    MATCH
      (s:Supplier {id:$supplier_id})
      -[:SUPPLIES]->
      (component:Component)
      -[:USED_IN]->
      (product:Product)
      <-[:CONTAINS]-
      (order:Order)
      <-[:PLACED]-
      (customer:Customer)

    RETURN DISTINCT
      customer.id AS customer_id,
      customer.name AS customer_name,
      product.name AS product,
      component.name AS component
    """

    records, _, _ = driver.execute_query(
        query,
        supplier_id=supplier_id,
        database_="neo4j"
    )

    return [r.data() for r in records]
```

This is an excellent GraphRAG interview scenario because a text explanation can be generated after graph traversal determines impacted entities.

---

# PART E — PYTHON CODING EXERCISES WITHOUT ANSWERS FIRST

Try these yourself before checking the solutions.

## Exercise 1

Write Python that returns all products purchased by customer `C101`.

## Exercise 2

Write Python that returns customers with more than five orders.

## Exercise 3

Write Python that batch-loads 100 customers using `UNWIND`.

## Exercise 4

Write Python that finds customers exactly two referral hops away.

## Exercise 5

Write Python that finds the shortest path between two people.

## Exercise 6

Write Python that returns customers without an order.

## Exercise 7

Write Python that returns the top five suppliers by product count.

## Exercise 8

Use APOC to remove duplicates from a list.

## Exercise 9

Use APOC to calculate intersection of two skill lists.

## Exercise 10

Use APOC to convert a Python dictionary passed to Cypher into JSON.

## Exercise 11

Create a GDS graph projection from `Person` and `KNOWS`.

## Exercise 12

Run PageRank from Python.

## Exercise 13

Run Louvain from Python.

## Exercise 14

Run Node Similarity on Customer/Product graph.

## Exercise 15

Write FastRP embeddings to a node property.

---

# PART F — SOLUTIONS TO PYTHON PRACTICE EXERCISES

## Solution 1

```python
def products_for_customer(driver, customer_id):
    records, _, _ = driver.execute_query(
        """
        MATCH (:Customer {id:$id})-[:PURCHASED]->(p:Product)
        RETURN p.id AS id,
               p.name AS name
        ORDER BY p.name
        """,
        id=customer_id,
        database_="neo4j"
    )

    return [r.data() for r in records]
```

---

## Solution 2

```python
def customers_with_many_orders(driver, minimum=5):
    records, _, _ = driver.execute_query(
        """
        MATCH (c:Customer)-[:PLACED]->(o:Order)
        WITH c, count(o) AS orderCount
        WHERE orderCount > $minimum
        RETURN c.id AS id,
               c.name AS name,
               orderCount
        ORDER BY orderCount DESC
        """,
        minimum=minimum,
        database_="neo4j"
    )

    return [r.data() for r in records]
```

---

## Solution 3

```python
def batch_load_customers(driver, customers):
    driver.execute_query(
        """
        UNWIND $rows AS row
        MERGE (c:Customer {id:row.id})
        SET c.name = row.name,
            c.state = row.state
        """,
        rows=customers,
        database_="neo4j"
    )
```

---

## Solution 4

```python
def two_hop_referrals(driver, customer_id):
    records, _, _ = driver.execute_query(
        """
        MATCH
          (c:Customer {id:$id})
          -[:REFERRED]->
          (:Customer)
          -[:REFERRED]->
          (target:Customer)

        WHERE target <> c
          AND NOT (c)-[:REFERRED]->(target)

        RETURN DISTINCT
          target.id AS id,
          target.name AS name
        """,
        id=customer_id,
        database_="neo4j"
    )

    return [r.data() for r in records]
```

---

## Solution 5

```python
def shortest_path(driver, source, target):
    records, _, _ = driver.execute_query(
        """
        MATCH
          (a:Person {id:$source}),
          (b:Person {id:$target})

        MATCH path = shortestPath(
            (a)-[:KNOWS*]-(b)
        )

        RETURN
          [n IN nodes(path) | n.name]
          AS people
        """,
        source=source,
        target=target,
        database_="neo4j"
    )

    return records[0]["people"] if records else None
```

---

## Solution 6

```python
def customers_without_orders(driver):
    records, _, _ = driver.execute_query(
        """
        MATCH (c:Customer)
        WHERE NOT EXISTS {
            MATCH (c)-[:PLACED]->(:Order)
        }
        RETURN c.id AS id,
               c.name AS name
        """,
        database_="neo4j"
    )

    return [r.data() for r in records]
```

---

## Solution 7

```python
def top_suppliers(driver):
    records, _, _ = driver.execute_query(
        """
        MATCH (p:Product)-[:SUPPLIED_BY]->(s:Supplier)
        RETURN
          s.id AS id,
          s.name AS name,
          count(p) AS products
        ORDER BY products DESC
        LIMIT 5
        """,
        database_="neo4j"
    )

    return [r.data() for r in records]
```

---

## Solution 8

```python
def unique_values(driver, values):
    records, _, _ = driver.execute_query(
        """
        RETURN apoc.coll.toSet($values)
               AS uniqueValues
        """,
        values=values,
        database_="neo4j"
    )

    return records[0]["uniqueValues"]
```

---

## Solution 9

```python
def common_skills(driver, skills1, skills2):
    records, _, _ = driver.execute_query(
        """
        RETURN apoc.coll.intersection(
          $skills1,
          $skills2
        ) AS common
        """,
        skills1=skills1,
        skills2=skills2,
        database_="neo4j"
    )

    return records[0]["common"]
```

---

## Solution 10

```python
def map_to_json(driver, payload):
    records, _, _ = driver.execute_query(
        """
        RETURN apoc.convert.toJson($payload)
               AS json
        """,
        payload=payload,
        database_="neo4j"
    )

    return records[0]["json"]
```

---

## Solution 11

```python
from graphdatascience import GraphDataScience

G, result = gds.graph.project(
    "peopleGraph",
    "Person",
    "KNOWS"
)
```

---

## Solution 12

```python
page_rank = gds.page_rank.stream(G)

print(
    page_rank.sort_values(
        "score",
        ascending=False
    ).head(10)
)
```

---

## Solution 13

```python
communities = gds.louvain.stream(G)

print(communities.head())
```

---

## Solution 14

```python
similarity = gds.node_similarity.stream(
    customer_graph,
    topK=10,
    similarityCutoff=0.2
)

print(similarity.head())
```

---

## Solution 15

```python
stats = gds.fast_rp.write(
    G,
    embeddingDimension=64,
    writeProperty="fastRPEmbedding"
)

print(stats)
```

---

# PART G — INTERVIEW CODING DRILLS

## Drill 1

**Interviewer:** Write a Cypher query and Python code to find the top three products purchased by the largest number of unique customers.

Cypher:

```cypher
MATCH (c:Customer)-[:PURCHASED]->(p:Product)
RETURN
p.name AS product,
count(DISTINCT c) AS customerCount
ORDER BY customerCount DESC
LIMIT 3;
```

Python:

```python
def top_products(driver):
    records, _, _ = driver.execute_query(
        """
        MATCH (c:Customer)-[:PURCHASED]->(p:Product)
        RETURN
          p.name AS product,
          count(DISTINCT c) AS customerCount
        ORDER BY customerCount DESC
        LIMIT 3
        """,
        database_="neo4j"
    )

    return [r.data() for r in records]
```

---

## Drill 2

**Interviewer:** Find customers sharing at least two products.

```python
def similar_customers(driver):
    records, _, _ = driver.execute_query(
        """
        MATCH
          (a:Customer)-[:PURCHASED]->(p:Product)<-[:PURCHASED]-(b:Customer)

        WHERE a.id < b.id

        WITH
          a,
          b,
          collect(DISTINCT p.name) AS sharedProducts

        WHERE size(sharedProducts) >= 2

        RETURN
          a.name AS customer1,
          b.name AS customer2,
          sharedProducts,
          size(sharedProducts) AS sharedCount

        ORDER BY sharedCount DESC
        """,
        database_="neo4j"
    )

    return [r.data() for r in records]
```

---

## Drill 3

**Interviewer:** Find suppliers affecting the largest number of customers.

```python
def supplier_customer_impact(driver):
    records, _, _ = driver.execute_query(
        """
        MATCH
          (s:Supplier)<-[:SUPPLIED_BY]-(p:Product)<-[:CONTAINS]-(:Order)<-[:PLACED]-(c:Customer)

        RETURN
          s.id AS supplier,
          s.name AS supplierName,
          count(DISTINCT c) AS affectedCustomers

        ORDER BY affectedCustomers DESC
        """,
        database_="neo4j"
    )

    return [r.data() for r in records]
```

---

## Drill 4

**Interviewer:** How would you identify influential accounts in a transaction network?

Answer structure:

```text
1. Model Account-[:TRANSFERRED_TO]->Account
2. Project graph to GDS
3. Start with degree/PageRank depending business meaning
4. Compare with transaction amount / frequency
5. Investigate communities and bridge nodes
6. Persist or stream score
7. Validate against known fraud labels/business outcomes
```

Python skeleton:

```python
transaction_graph, _ = gds.graph.project(
    "transactionGraph",
    "Account",
    "TRANSFERRED_TO"
)

rank = gds.page_rank.stream(transaction_graph)

print(rank.sort_values(
    "score",
    ascending=False
).head(20))
```

---

# PART H — DEBUGGING PRACTICE

## Bug 1 — Unsafe Query Construction

Bad:

```python
query = f"MATCH (c:Customer {{id:'{customer_id}'}}) RETURN c"
```

Fix:

```python
query = "MATCH (c:Customer {id:$id}) RETURN c"

records, _, _ = driver.execute_query(
    query,
    id=customer_id,
    database_="neo4j"
)
```

---

## Bug 2 — One Query Per Row

Bad:

```python
for customer in customers:
    driver.execute_query(
        "CREATE (:Customer {id:$id})",
        id=customer["id"],
        database_="neo4j"
    )
```

Better:

```python
driver.execute_query(
    """
    UNWIND $rows AS row
    MERGE (:Customer {id:row.id})
    """,
    rows=customers,
    database_="neo4j"
)
```

Reason:

```text
fewer round trips
better throughput
idempotency with MERGE
```

---

## Bug 3 — Unbounded Traversal

Risky:

```cypher
MATCH (a)-[*]-(b)
RETURN a,b;
```

Better:

```cypher
MATCH (a:Customer {id:$id})
      -[:REFERRED*1..3]->
      (b:Customer)
RETURN DISTINCT b;
```

---

## Bug 4 — Missing Uniqueness Constraint

Problem:

```cypher
MERGE (c:Customer {id:$id})
```

but no unique constraint.

Improve:

```cypher
CREATE CONSTRAINT customer_id_unique IF NOT EXISTS
FOR (c:Customer)
REQUIRE c.id IS UNIQUE;
```

---

## Bug 5 — Returning Entire Graph Objects to API

Less ideal:

```cypher
RETURN c,p,r;
```

Prefer API-specific projection:

```cypher
RETURN
c.id AS customerId,
p.id AS productId,
p.name AS productName,
r.amount AS amount;
```

Reason:

- smaller payload
- stable contract
- less accidental data exposure

---

# PART I — 4-DAY PYTHON PRACTICE ASSIGNMENT

## Day 1 Python

Write these without copying:

```text
1. connect to Neo4j
2. verify connectivity
3. query one customer
4. create a customer
5. MERGE a customer
6. create PURCHASED relationship
7. OPTIONAL MATCH customer orders
8. batch load customers using UNWIND
```

Target:

```text
8 working Python programs
```

---

## Day 2 Python

Write:

```text
1. two-degree query
2. shortest path
3. recommendation query
4. shared-product query
5. APOC list dedupe
6. APOC intersection
7. APOC JSON conversion
8. one managed transaction
```

Target:

```text
8 more working examples
```

---

## Day 3 Python + GDS

Practice:

```text
1. connect GraphDataScience client
2. project graph
3. PageRank stream
4. PageRank write
5. Degree centrality
6. Betweenness
7. Louvain
8. WCC
9. Node Similarity
10. FastRP
11. drop graph
```

Target:

```text
11 GDS exercises
```

---

## Day 4 Integration

Create one Python script that does:

```text
Question
↓
Cypher retrieval
↓
optional APOC transformation
↓
GDS score / community context
↓
format context
↓
LLM-ready dictionary
```

Example output:

```python
{
    "question": "Which suppliers are high impact?",
    "graph_results": [...],
    "centrality_results": [...],
    "communities": [...],
    "sources": [...]
}
```

That is the type of integration thinking expected from a Graph AI Engineer rather than only a Cypher developer.

---

# PYTHON INTERVIEW MEMORY SHEET

## Neo4j Driver

```text
GraphDatabase.driver
verify_connectivity
execute_query
session.execute_read
session.execute_write
close
```

Memory:

```text
Connect → Query → Consume → Close
```

## Python + Cypher

```text
Python dict/list
↓
parameters
↓
Cypher
↓
records
↓
record.data()
```

## Batch Loading

```text
Python list
↓
UNWIND
↓
MERGE
↓
SET
```

Memory:

```text
U-M-S
UNWIND → MERGE → SET
```

## APOC

```text
APOC = Cypher utility toolbox
```

Remember:

```text
Collections
JSON
Paths
Refactoring
Utilities
```

## GDS

```text
Connect
Project
Run
Stream/Write
Drop
```

Memory:

```text
C-P-R-S-D
```

## GDS algorithms

```text
PageRank      → influence
Degree        → direct connectivity
Betweenness   → bridge
Louvain       → community
WCC           → connected component
NodeSimilarity→ similar neighborhoods
Dijkstra      → weighted shortest path
FastRP        → embeddings
```

---

# FINAL PYTHON CHECKLIST BEFORE THE INTERVIEW

You should be able to code from memory:

- [ ] Neo4j Python connection
- [ ] `driver.execute_query()`
- [ ] parameterized Cypher
- [ ] `MATCH`
- [ ] `MERGE`
- [ ] relationship creation
- [ ] `OPTIONAL MATCH`
- [ ] `WITH`
- [ ] `UNWIND`
- [ ] two-hop traversal
- [ ] shortest path
- [ ] recommendation query
- [ ] result → Python dictionaries
- [ ] transaction functions
- [ ] exception handling
- [ ] APOC collection function
- [ ] APOC JSON conversion
- [ ] APOC path expansion concept
- [ ] `GraphDataScience` connection
- [ ] graph projection
- [ ] PageRank
- [ ] Degree centrality
- [ ] Betweenness
- [ ] Louvain
- [ ] WCC
- [ ] Node Similarity
- [ ] FastRP
- [ ] write algorithm result
- [ ] drop projected graph

If you can write at least **70–80% of these without looking at the guide**, you will be in a strong position for the coding portion of a Graph AI Engineer interview.
