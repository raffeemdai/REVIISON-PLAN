# APOC Complete Notes — Theory, Syntax, Practice & Interview Questions

APOC = **A**wesome **P**rocedures **O**n **C**ypher. It's the most widely used Neo4j plugin: a library of hundreds of procedures and functions that fill in the gaps native Cypher doesn't cover — JSON/XML handling, bulk import/export, dynamic/parameterized Cypher, graph refactoring, batching, triggers, and more.

All practice queries below reuse the **same dataset** as `Day1_Cypher_Practice_Queries.md` (Alice/Bob/Carol/Dave people, OpenAI/Neo4j companies, and the Customer→PURCHASED→Product mini graph), so you can run them in the same sandbox.

---

# 0. Theory — What APOC Is and When to Use It

## What APOC Is

APOC is **not** part of core Neo4j — it's a separate plugin (a `.jar` file) that adds hundreds of extra procedures (`CALL apoc.xxx`) and functions (`apoc.xxx()`) on top of Cypher.

- **Procedures** (`CALL apoc.something(...)`) — can do multi-row, side-effecting work (writes, batching, I/O).
- **Functions** (`RETURN apoc.something(...)`) — return a single value, usable inline inside `RETURN`, `WHERE`, `WITH`.

## Why It Exists

Native Cypher is intentionally minimal. APOC fills real gaps:

| Native Cypher gap | APOC fills it with |
|---|---|
| No dynamic/parameterized labels or relationship types | `apoc.merge.node`, `apoc.create.relationship` |
| No built-in JSON parsing/serialization | `apoc.convert.*` |
| No CSV/JSON/JDBC bulk import | `apoc.load.*` |
| No safe large-transaction batching | `apoc.periodic.iterate` |
| No built-in database triggers | `apoc.trigger.*` |
| No easy schema/graph introspection | `apoc.meta.*` |
| No node/relationship "refactoring" helpers | `apoc.refactor.*` |

## Installing / Enabling APOC

- **Neo4j Desktop / Aura:** APOC Core is usually pre-installed or a one-click plugin install.
- **Self-managed (Docker, server):** drop the `apoc-<version>-core.jar` into the `plugins/` folder and restart.
- Some procedures (file import/export, JSON APIs) require explicit config in `neo4j.conf`:

```text
dbms.security.procedures.unrestricted=apoc.*
apoc.export.file.enabled=true
apoc.import.file.enabled=true
```

## Interview Position (how to talk about APOC)

Don't say "APOC replaces Cypher." Say:

> I prefer native Cypher when possible, and reach for APOC when it simplifies specialized transformation, integration, collection handling, import/export, batching, or utility operations that Cypher doesn't cover natively.

### Memory Trick

```text
APOC = Cypher's utility toolbox
Native Cypher first. APOC when Cypher runs out of tools.
```

---

# APOC Object / Topic Map

APOC's procedures and functions are organized into namespaces (think of these as APOC's "packages"). These are the ones worth knowing cold for interviews:

| # | Namespace | Purpose |
|---|---|---|
| 1 | `apoc.coll.*` | List/collection utilities |
| 2 | `apoc.text.*` | String utilities |
| 3 | `apoc.convert.*` | JSON / type conversion |
| 4 | `apoc.map.*` | Map (dictionary) utilities |
| 5 | `apoc.date.*` / `apoc.temporal.*` | Date & time utilities |
| 6 | `apoc.create.*` | Dynamic / virtual node & relationship creation |
| 7 | `apoc.merge.*` | Dynamic MERGE (runtime label/type) |
| 8 | `apoc.refactor.*` | Restructure existing graph data |
| 9 | `apoc.path.*` | Path expansion / traversal helpers |
| 10 | `apoc.load.*` | Import from JSON, CSV, JDBC, XML |
| 11 | `apoc.export.*` | Export to JSON, CSV, Cypher, GraphML |
| 12 | `apoc.periodic.*` | Batching, scheduling, background jobs |
| 13 | `apoc.trigger.*` | Run Cypher automatically on data changes |
| 14 | `apoc.meta.*` | Schema / graph introspection |
| 15 | `apoc.schema.*` | Constraint/index inspection |
| 16 | `apoc.algo.*` | Legacy graph algorithms (mostly superseded by GDS) |
| 17 | `apoc.util.*` / `apoc.log.*` | Misc utilities, validation, logging |
| 18 | `apoc.help` / `apoc.version` | Discovery — find what's installed |

### Memory Trick

```text
C-T-C-M-D  |  C-M-R-P  |  L-E-P-T  |  M-S-A-U-H
Coll, Text, Convert, Map, Date   → basic data utilities
Create, Merge, Refactor, Path    → graph-shaping utilities
Load, Export, Periodic, Trigger  → integration & automation utilities
Meta, Schema, Algo, Util, Help   → introspection & misc
```

---

# 1. `apoc.coll.*` — Collection (List) Utilities

## Theory

Cypher has some list functions built in (`size()`, `head()`, `range()`), but APOC adds set operations, sorting, partitioning, and more — the kind of things you'd reach for `set()`/`sorted()` for in Python.

## Syntax & Examples

```cypher
// Deduplicate a list (like Python's set())
RETURN apoc.coll.toSet(["Neo4j", "AWS", "Neo4j"]) AS uniqueSkills;
// -> ["Neo4j", "AWS"]
```

```cypher
// Intersection of two lists
RETURN apoc.coll.intersection(
    ["Neo4j", "Python", "AWS"],
    ["Python", "Docker", "AWS"]
) AS common;
// -> ["Python", "AWS"]
```

```cypher
// Difference (subtract)
RETURN apoc.coll.subtract(
    ["Neo4j", "Python", "AWS"],
    ["Python"]
) AS remaining;
// -> ["Neo4j", "AWS"]
```

```cypher
// Sort a list of maps by a key
MATCH (c:Customer)-[r:PURCHASED]->(p:Product)
WITH collect({name: p.name, amount: r.amount}) AS purchases
RETURN apoc.coll.sortMaps(purchases, "amount") AS sortedByAmount;
```

### Memory Trick

```text
apoc.coll = "collections" = set math on lists
toSet, intersection, subtract, sort — like Python set()/sorted()
```

---

# 2. `apoc.text.*` — String Utilities

## Theory

Fills gaps in Cypher's limited string functions — capitalization, fuzzy matching, phonetic comparison, string distance, formatting.

## Syntax & Examples

```cypher
RETURN apoc.text.capitalize("alice") AS name;
// -> "Alice"
```

```cypher
// Fuzzy/typo-tolerant matching — useful for entity resolution
RETURN apoc.text.levenshteinDistance("Neo4j", "Neo4J") AS distance;
// -> 1
```

```cypher
// Clean/join text
RETURN apoc.text.join(["Neo4j", "Python", "AWS"], ", ") AS skillsLine;
// -> "Neo4j, Python, AWS"
```

### Memory Trick

```text
apoc.text = string toolbox Cypher forgot: capitalize, distance, join, format
```

---

# 3. `apoc.convert.*` — JSON & Type Conversion

## Theory

Neo4j's property values are limited to primitives and lists of primitives — you can't store a nested map as a property. APOC's `convert` namespace serializes/deserializes JSON so you can pass structured data in and out of Cypher.

## Syntax & Examples

```cypher
// Map -> JSON string
RETURN apoc.convert.toJson({
    name: "Alice",
    skills: ["Neo4j", "Python"]
}) AS json;
```

```cypher
// JSON string -> Map
WITH '{"name":"Alice","age":30}' AS jsonString
RETURN apoc.convert.fromJsonMap(jsonString) AS parsed;
```

```cypher
// JSON string -> List
WITH '["Neo4j","Python","AWS"]' AS jsonString
RETURN apoc.convert.fromJsonList(jsonString) AS parsed;
```

### Memory Trick

```text
apoc.convert = the JSON <-> Cypher bridge
toJson = serialize, fromJsonMap/fromJsonList = deserialize
```

---

# 4. `apoc.map.*` — Map (Dictionary) Utilities

## Theory

Helpers for merging, filtering, and reshaping map/dictionary-like data — handy when you've just pulled a JSON payload apart with `apoc.convert`.

## Syntax & Examples

```cypher
// Merge two maps (second one wins on key conflicts)
RETURN apoc.map.merge(
    {name: "Alice", age: 30},
    {age: 31, city: "Atlanta"}
) AS merged;
// -> {name:"Alice", age:31, city:"Atlanta"}
```

```cypher
// Remove keys from a map
RETURN apoc.map.removeKeys(
    {name:"Alice", age:31, city:"Atlanta"},
    ["city"]
) AS trimmed;
// -> {name:"Alice", age:31}
```

### Memory Trick

```text
apoc.map = merge/remove/filter for dictionaries, like Python dict.update()
```

---

# 5. `apoc.date.*` / `apoc.temporal.*` — Date & Time Utilities

## Theory

Neo4j has native `date()`, `datetime()`, `duration()` functions, but APOC adds conversions to/from epoch millis/seconds and custom formatting — common when ingesting timestamps from external systems.

## Syntax & Examples

```cypher
// Convert an epoch timestamp (ms) to a formatted date string
RETURN apoc.date.format(1735689600000, "ms", "yyyy-MM-dd") AS formatted;
```

```cypher
// Parse a formatted string into epoch millis
RETURN apoc.date.parse("2026-09-25", "ms", "yyyy-MM-dd") AS epochMillis;
```

### Memory Trick

```text
apoc.date = epoch <-> human-readable date, when native date()/datetime() isn't enough
```

---

# 6. `apoc.create.*` — Dynamic & Virtual Node/Relationship Creation

## Theory

Native Cypher requires labels and relationship types to be **literal** at query-write time — you can't do `CREATE (n:$label)`. APOC lets you pass label/type names as **runtime strings**, which matters for generic ingestion code. It also supports **virtual nodes/relationships** — in-memory-only objects for visualization, never persisted to the graph.

## Syntax & Examples

```cypher
// Dynamic label at runtime
CALL apoc.create.node(["Person", "Employee"], {name: "Grace"})
YIELD node
RETURN node;
```

```cypher
// Dynamic relationship type at runtime
MATCH (a:Person {name:"Alice"}), (b:Company {name:"OpenAI"})
CALL apoc.create.relationship(a, "CONTRACTED_BY", {since: 2024}, b)
YIELD rel
RETURN rel;
```

```cypher
// Virtual node — exists only in the query result, not saved to the graph
RETURN apoc.create.vNode(
    ["Summary"],
    {label: "Total Customers: 3"}
) AS previewNode;
```

### Memory Trick

```text
apoc.create = CREATE with variables instead of literals
vNode/vRelationship = "preview only" nodes — never written to disk
```

---

# 7. `apoc.merge.*` — Dynamic MERGE

## Theory

Same idea as `apoc.create`, but for `MERGE` semantics (find-or-create) — needed when your ingestion pipeline doesn't know the label name until runtime.

## Syntax & Examples

```cypher
CALL apoc.merge.node(
    ["Customer"],
    {customerId: "C6"},
    {name: "Fiona"}
) YIELD node
RETURN node;
```

### Memory Trick

```text
apoc.merge = MERGE with a variable label, same find-or-create idea
```

---

# 8. `apoc.refactor.*` — Restructuring Existing Data

## Theory

Cypher can't easily rename a relationship type or merge two duplicate nodes into one — these are common cleanup tasks after a messy import, and `apoc.refactor` handles them directly.

## Syntax & Examples

```cypher
// Rename a relationship type (Cypher has no native way to do this)
MATCH (a:Person)-[r:KNOWS]->(b:Person)
CALL apoc.refactor.setType(r, "CONNECTED_TO")
YIELD input, output
RETURN output;
```

```cypher
// Merge two duplicate nodes into one
MATCH (a:Person {name:"Alice"}), (dup:Person {name:"Alice_duplicate"})
CALL apoc.refactor.mergeNodes([a, dup], {properties: "combine"})
YIELD node
RETURN node;
```

```cypher
// Convert a node's category into a label ("category-to-label" pattern)
MATCH (p:Product)
CALL apoc.create.addLabels(p, [p.category])
YIELD node
RETURN node;
```

### Memory Trick

```text
apoc.refactor = graph "fix-it" toolkit: rename types, dedupe nodes, promote properties to labels
```

---

# 9. `apoc.path.*` — Path Expansion

## Theory

Native variable-length patterns (`*1..3`) are great for simple cases, but real traversals often need constraints Cypher's pattern syntax can't express cleanly — filter by relationship type, direction, node label, min/max depth, all in one config map.

## Syntax & Examples

```cypher
MATCH (start:Person {name:"Alice"})
CALL apoc.path.expandConfig(start, {
    relationshipFilter: "KNOWS>",
    minLevel: 1,
    maxLevel: 2,
    uniqueness: "NODE_GLOBAL"
})
YIELD path
RETURN path;
```

### Memory Trick

```text
apoc.path.expandConfig = variable-length traversal with a full config map
(relationship filter + direction + label filter + depth, all in one call)
```

---

# 10. `apoc.load.*` — Import from External Sources

## Theory

Bulk-loading real-world data usually means pulling from a file or another system. `apoc.load` handles JSON, CSV, XML, and JDBC sources directly inside Cypher — the standard way to bring external data into a graph without writing separate ETL code.

## Syntax & Examples

```cypher
// Load JSON from a URL or file (requires apoc.import.file.enabled or a public URL)
CALL apoc.load.json("https://example.com/customers.json")
YIELD value
MERGE (c:Customer {customerId: value.customerId})
SET c.name = value.name;
```

```cypher
// Load CSV
CALL apoc.load.csv("file:///customers.csv")
YIELD map
MERGE (c:Customer {customerId: map.customerId})
SET c.name = map.name;
```

### Memory Trick

```text
apoc.load = "bring data in" — JSON/CSV/XML/JDBC → Cypher rows
```

---

# 11. `apoc.export.*` — Export to External Formats

## Theory

The reverse of `apoc.load` — dump query results or the whole graph out as JSON, CSV, Cypher statements, or GraphML (for tools like Gephi).

## Syntax & Examples

```cypher
// Export a query's results to CSV on disk
CALL apoc.export.csv.query(
    "MATCH (c:Customer)-[:PURCHASED]->(p:Product) RETURN c.name, p.name",
    "purchases.csv",
    {}
);
```

```cypher
// Export the whole graph as a Cypher script (great for migrating between environments)
CALL apoc.export.cypher.all("full_graph_backup.cypher", {});
```

### Memory Trick

```text
apoc.export = "send data out" — the mirror image of apoc.load
```

---

# 12. `apoc.periodic.*` — Batching & Background Jobs

## Theory

A single giant `MATCH ... SET` over millions of rows can blow out memory and lock the whole transaction. `apoc.periodic.iterate` runs the write in small batches (e.g. 10,000 rows at a time), each in its own transaction — the standard production pattern for large migrations.

## Syntax & Examples

```cypher
CALL apoc.periodic.iterate(
  "MATCH (c:Customer) RETURN c",
  "SET c.migrated = true",
  {batchSize: 1000, parallel: false}
)
YIELD batches, total, errorMessages
RETURN batches, total, errorMessages;
```

```cypher
// Schedule a job to run every N seconds
CALL apoc.periodic.repeat(
  "cleanup-job",
  "MATCH (n:TempNode) WHERE n.createdAt < datetime() - duration('P1D') DETACH DELETE n",
  3600
);
```

### Memory Trick

```text
apoc.periodic.iterate = "big job? break it into small batched transactions"
```

---

# 13. `apoc.trigger.*` — Database Triggers

## Theory

Neo4j core has no built-in trigger system. `apoc.trigger` lets you register Cypher that runs automatically whenever nodes/relationships of a certain shape are created, updated, or deleted — useful for maintaining derived properties or audit logs.

## Syntax & Examples

```cypher
CALL apoc.trigger.add(
  "set-updated-timestamp",
  "UNWIND $createdNodes AS n SET n.createdAt = datetime()",
  {phase: "before"}
);
```

```cypher
// List installed triggers
CALL apoc.trigger.list();
```

### Memory Trick

```text
apoc.trigger = "run this Cypher automatically whenever data changes"
```

---

# 14. `apoc.meta.*` — Graph & Schema Introspection

## Theory

Answers "what's actually in this graph?" — label counts, relationship type counts, property keys — useful for exploring an unfamiliar database or generating documentation.

## Syntax & Examples

```cypher
// High-level graph statistics
CALL apoc.meta.stats()
YIELD labelCount, relTypeCount, nodeCount, relCount
RETURN labelCount, relTypeCount, nodeCount, relCount;
```

```cypher
// Full schema graph (labels, relationship types, and how they connect)
CALL apoc.meta.schema()
YIELD value
RETURN value;
```

### Memory Trick

```text
apoc.meta = "describe this database to me" — like SQL's INFORMATION_SCHEMA
```

---

# 15. `apoc.schema.*` — Constraint & Index Helpers

## Theory

Complements native `SHOW CONSTRAINTS` / `SHOW INDEXES` with more programmatic helpers (e.g. checking schema state before running a migration script).

## Syntax & Examples

```cypher
CALL apoc.schema.nodes()
YIELD label, properties, type
RETURN label, properties, type;
```

### Memory Trick

```text
apoc.schema = programmatic version of SHOW CONSTRAINTS / SHOW INDEXES
```

---

# 16. `apoc.algo.*` — Legacy Graph Algorithms

## Theory

Before the **Graph Data Science (GDS)** library existed, APOC shipped its own graph algorithms (PageRank, shortest path variants, etc.). GDS is now the standard, production-grade choice — `apoc.algo` mostly survives for backward compatibility.

## Syntax & Example

```cypher
MATCH (a:Person {name:"Alice"}), (d:Person {name:"Dave"})
CALL apoc.algo.dijkstra(a, d, "KNOWS", "weight")
YIELD path, weight
RETURN path, weight;
```

### Interview Trap

Don't say "APOC and GDS are the same thing." Say:

> APOC has some legacy algorithm procedures, but GDS is the dedicated, actively developed graph algorithms library — that's what I'd reach for in production for PageRank, community detection, centrality, or embeddings.

### Memory Trick

```text
apoc.algo = the "before GDS existed" algorithms — GDS is the modern choice now
```

---

# 17. `apoc.util.*` / `apoc.log.*` — Misc Utilities

## Theory

A grab bag: validation helpers, sleep/wait for testing, and structured logging from inside a Cypher query.

## Syntax & Examples

```cypher
// Fail the query with a custom message if a condition isn't met
MATCH (c:Customer {customerId:"C1"})
CALL apoc.util.validate(c.name IS NULL, "Customer name cannot be null", [])
RETURN c;
```

```cypher
// Write to the Neo4j log from within a query
CALL apoc.log.info("Migration batch completed successfully");
```

### Memory Trick

```text
apoc.util/apoc.log = validation + logging you'd otherwise have to do in application code
```

---

# 18. `apoc.help` / `apoc.version` — Discovery

## Theory

Since APOC availability and exact procedure names vary by version/environment, these let you check what's actually installed before relying on a specific procedure.

## Syntax & Examples

```cypher
// Search for anything related to "json"
CALL apoc.help("json");
```

```cypher
// List every installed apoc.* procedure/function
SHOW PROCEDURES YIELD name
WHERE name STARTS WITH 'apoc.'
RETURN name
ORDER BY name;
```

```cypher
RETURN apoc.version();
```

### Memory Trick

```text
apoc.help = "what APOC procedures actually exist in this database right now?"
```

---

# Practice Questions

Each one names the APOC namespace it targets — try to write the query yourself before checking the solution.

### PQ1. Deduplicate a list (`apoc.coll`)

**Question:** Given the list `["Alice", "Bob", "Alice", "Carol", "Bob"]`, return the unique names.

**Solution:**
```cypher
RETURN apoc.coll.toSet(["Alice", "Bob", "Alice", "Carol", "Bob"]) AS uniqueNames;
```

### PQ2. Find shared skills between two candidates (`apoc.coll`)

**Question:** Candidate A knows `["Cypher", "Python", "AWS"]`, Candidate B knows `["Python", "Docker", "AWS"]`. Return the skills they have in common.

**Solution:**
```cypher
RETURN apoc.coll.intersection(
  ["Cypher", "Python", "AWS"],
  ["Python", "Docker", "AWS"]
) AS sharedSkills;
```

### PQ3. Serialize a customer's purchase history to JSON (`apoc.convert`)

**Question:** For Alice, return a JSON string containing her name and a list of product names she purchased.

**Solution:**
```cypher
MATCH (c:Customer {name:"Alice"})-[:PURCHASED]->(p:Product)
WITH c.name AS name, collect(p.name) AS products
RETURN apoc.convert.toJson({name: name, products: products}) AS json;
```

### PQ4. Parse a JSON payload back into properties (`apoc.convert`)

**Question:** Given the JSON string `'{"customerId":"C7","name":"Grace"}'`, create a matching `Customer` node.

**Solution:**
```cypher
WITH apoc.convert.fromJsonMap('{"customerId":"C7","name":"Grace"}') AS data
MERGE (c:Customer {customerId: data.customerId})
SET c.name = data.name
RETURN c;
```

### PQ5. Create a node with a label decided at query time (`apoc.create`)

**Question:** Write a query that creates a node whose label comes from a parameter `$label` (e.g. `"VIP"`), something plain `CREATE (:$label)` cannot do.

**Solution:**
```cypher
CALL apoc.create.node([$label], {name: "Priority Customer"})
YIELD node
RETURN node;
```

### PQ6. Rename a relationship type across the whole graph (`apoc.refactor`)

**Question:** Rename every `KNOWS` relationship to `CONNECTED_TO`.

**Solution:**
```cypher
MATCH (:Person)-[r:KNOWS]->(:Person)
CALL apoc.refactor.setType(r, "CONNECTED_TO")
YIELD output
RETURN count(output) AS relationshipsRenamed;
```

### PQ7. Promote a property to a label (`apoc.refactor` / `apoc.create`)

**Question:** Every `Product` has a `category` property. Add each product's category as an extra label on that node (e.g. a Laptop gets `:Product:Electronics`).

**Solution:**
```cypher
MATCH (p:Product)
CALL apoc.create.addLabels(p, [p.category])
YIELD node
RETURN node;
```

### PQ8. Traverse up to 2 hops of a specific relationship type (`apoc.path`)

**Question:** Starting from Alice, find everyone reachable within 2 hops of outgoing `KNOWS` relationships.

**Solution:**
```cypher
MATCH (start:Person {name:"Alice"})
CALL apoc.path.expandConfig(start, {
    relationshipFilter: "KNOWS>",
    minLevel: 1,
    maxLevel: 2
})
YIELD path
RETURN path;
```

### PQ9. Batch-update every Customer without one giant transaction (`apoc.periodic`)

**Question:** Add a `loyaltyTier: "standard"` property to every `Customer` node, processed in batches of 500.

**Solution:**
```cypher
CALL apoc.periodic.iterate(
  "MATCH (c:Customer) RETURN c",
  "SET c.loyaltyTier = 'standard'",
  {batchSize: 500}
)
YIELD batches, total
RETURN batches, total;
```

### PQ10. Get a quick statistical overview of the whole graph (`apoc.meta`)

**Question:** Without writing your own aggregation query, find out how many labels, relationship types, nodes, and relationships exist in the database.

**Solution:**
```cypher
CALL apoc.meta.stats()
YIELD labelCount, relTypeCount, nodeCount, relCount
RETURN labelCount, relTypeCount, nodeCount, relCount;
```

### PQ11. Check what JSON-related APOC procedures are available (`apoc.help`)

**Question:** You're not sure of the exact JSON procedure names in this environment — find them.

**Solution:**
```cypher
CALL apoc.help("json");
```

---

# Interview Questions

## Q1. What does APOC stand for, and what is it?

APOC stands for **Awesome Procedures On Cypher**. It's a plugin library for Neo4j that adds hundreds of extra procedures and functions covering things native Cypher doesn't handle well — JSON/XML processing, bulk import/export, dynamic labels/types, graph refactoring, batching large writes, database triggers, and schema introspection.

---

## Q2. Is APOC part of core Neo4j?

No. It's a separate plugin that must be installed (a `.jar` in the `plugins/` folder for self-managed instances, or pre-installed/one-click in Desktop and Aura). Some procedures also need explicit config flags (e.g. `apoc.import.file.enabled=true`) before they'll run.

---

## Q3. When would you use APOC instead of native Cypher?

When native Cypher can't express what you need directly — dynamic labels/relationship types decided at runtime, JSON parsing/serialization, importing from CSV/JSON/JDBC, batching a write across millions of rows safely, renaming relationship types, merging duplicate nodes, or running Cypher automatically via triggers. The right framing: prefer native Cypher first, reach for APOC when it provides a clear capability Cypher lacks.

---

## Q4. What's the difference between an APOC procedure and an APOC function?

A **procedure** (called with `CALL apoc.xxx(...)  YIELD ...`) can return multiple rows and perform side effects like writes or I/O. A **function** (called inline, e.g. `RETURN apoc.xxx(...)`) returns a single value and can be used directly inside `RETURN`, `WHERE`, or `WITH` clauses like a built-in function.

---

## Q5. How would you safely update millions of nodes without one giant transaction?

Use `apoc.periodic.iterate`, passing a `MATCH`-only query to fetch rows and a separate write statement to run per batch:

```cypher
CALL apoc.periodic.iterate(
  "MATCH (c:Customer) RETURN c",
  "SET c.migrated = true",
  {batchSize: 1000}
)
YIELD batches, total, errorMessages
RETURN batches, total, errorMessages;
```

This runs the writes in small, separate transactions instead of one massive one, avoiding memory blowups and long lock contention.

---

## Q6. How do you rename a relationship type in Neo4j? Can Cypher do this natively?

No — Cypher has no native `ALTER` for relationship types; a relationship type is fixed at creation. You have to use `apoc.refactor.setType(rel, "NEW_TYPE")`, which internally creates a new relationship of the new type and removes the old one.

---

## Q7. How would you merge two duplicate nodes that represent the same real-world entity?

`apoc.refactor.mergeNodes([nodeA, nodeB], {properties: "combine"})` — it merges both nodes' relationships and properties into one surviving node, with configurable conflict-resolution for properties (`"combine"`, `"overwrite"`, `"discard"`).

---

## Q8. What's the difference between `apoc.algo` and GDS (Graph Data Science)?

`apoc.algo` is APOC's older, legacy set of graph algorithms (shortest path, some centrality). GDS is Neo4j's dedicated, actively maintained graph algorithms library with far more algorithms (PageRank, Louvain community detection, node embeddings like FastRP/Node2Vec, similarity, etc.) and better performance at scale. In an interview, the correct position is: GDS is the modern, production choice; `apoc.algo` mostly exists for backward compatibility.

---

## Q9. How do you inspect what's actually in a graph you didn't build?

`CALL apoc.meta.stats()` for high-level counts (labels, relationship types, nodes, relationships), or `CALL apoc.meta.schema()` for the full label/relationship-type structure — similar in spirit to querying `INFORMATION_SCHEMA` in a relational database.

---

## Q10. How does APOC help with JSON, and why does Neo4j need help with JSON at all?

Neo4j property values can only be primitives or lists of primitives — you cannot store a nested object as a single property natively. `apoc.convert.toJson()` serializes a map/object into a JSON string (which *can* be stored as a plain string property or sent to an external API), and `apoc.convert.fromJsonMap()` / `fromJsonList()` parse a JSON string back into a usable Cypher map or list.

---

## Q11. What are virtual nodes/relationships in APOC, and why use them?

`apoc.create.vNode()` / `apoc.create.vRelationship()` create nodes/relationships that exist **only in the query result** — they are never persisted to the graph. They're useful for building custom visualizations or preview/summary nodes (e.g. an aggregate "Total: $1,250" node) without polluting the real data model.

---

## Q12. How would you import a large CSV file of customers into Neo4j?

`apoc.load.csv("file:///customers.csv") YIELD map` streams each row as a map, which you then pipe into a `MERGE` (using a business key like `customerId`) to upsert without creating duplicates — usually wrapped in `apoc.periodic.iterate` if the file is large, to batch the writes.

---

## Q13. Does Neo4j support database triggers natively? How do you get similar behavior?

No native trigger system exists in core Neo4j. `apoc.trigger.add(name, statement, config)` registers Cypher that runs automatically before/after node or relationship changes — for example, auto-stamping a `createdAt` timestamp on every new node.

---

## Q14. What's a real risk of using `apoc.load.json` or `apoc.load.csv` against an external URL in production?

It introduces an external dependency and potential security/availability risk — the query depends on a third-party endpoint being reachable and returning well-formed data at query time. In production pipelines, it's usually safer to first land the file in trusted, controlled storage (or pass validated data in as parameters) rather than trusting `apoc.load` to hit an arbitrary external URL directly inside a live Cypher query.

---

## Q15. Give an example of a bad use of APOC — something you'd push back on in a code review.

Using `apoc.create.node()` with a dynamic/user-supplied label string with no allow-list or validation — that lets arbitrary label names flow straight from user input into the graph schema, which is both a data-quality risk and, in a web-facing system, an injection-style risk. The fix: validate/allow-list the possible label values before passing them to APOC.

---

# Quick Reference Cheat Sheet

| Need to... | Use |
|---|---|
| Deduplicate / intersect / diff lists | `apoc.coll.*` |
| Clean up or compare strings | `apoc.text.*` |
| Convert between JSON and Cypher maps/lists | `apoc.convert.*` |
| Merge/filter maps | `apoc.map.*` |
| Convert epoch <-> formatted date | `apoc.date.*` |
| Create a node/relationship with a runtime label/type | `apoc.create.*` |
| Find-or-create with a runtime label | `apoc.merge.*` |
| Rename a relationship type / merge duplicate nodes | `apoc.refactor.*` |
| Traverse with fine-grained filters/depth | `apoc.path.expandConfig` |
| Import CSV/JSON/JDBC/XML | `apoc.load.*` |
| Export to CSV/JSON/Cypher/GraphML | `apoc.export.*` |
| Safely batch a huge write | `apoc.periodic.iterate` |
| Run Cypher automatically on data changes | `apoc.trigger.*` |
| Inspect graph/schema shape | `apoc.meta.*`, `apoc.schema.*` |
| Legacy graph algorithms (prefer GDS instead) | `apoc.algo.*` |
| Validate / log inside a query | `apoc.util.*`, `apoc.log.*` |
| Discover what's installed | `apoc.help`, `apoc.version` |

### Final Memory Formula

```text
APOC = Awesome Procedures On Cypher
"If Cypher can't do it natively, APOC probably can."
Native Cypher first. APOC when it adds real capability.
```
