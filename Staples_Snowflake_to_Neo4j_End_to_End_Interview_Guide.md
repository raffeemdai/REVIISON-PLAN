# Staples Snowflake → Neo4j Data Loading
## End-to-End Interview Preparation Guide

**Use case:** Load Staples sales and product data from Snowflake into Neo4j using Python and Cypher.

**Resume point:**

> Developed Python-based data pipelines to extract Staples sales and product data from Snowflake and load it into Neo4j using parameterized Cypher queries for graph-based analysis.

---

# 1. Project in One Sentence

We extracted customer, sales, order, product, category, store, and supplier data from Snowflake using Python, transformed the relational data into graph entities and relationships, and loaded it into Neo4j using parameterized Cypher queries.

---

# 2. Why Neo4j?

Snowflake is strong for data warehousing, reporting, aggregations, and SQL analytics.

Neo4j is useful when business questions depend heavily on relationships, for example:

- Which customers bought the same products?
- Which products are frequently bought together?
- Which suppliers are connected to a product category?
- Which customers are impacted by a supplier/product issue?
- Which products could be recommended based on purchase relationships?

Instead of repeatedly joining several relational tables, Neo4j stores the connections directly as graph relationships.

---

# 3. End-to-End Architecture

```text
                     STAPLES DATA FLOW

              ┌─────────────────────────┐
              │       SNOWFLAKE         │
              │                         │
              │ CUSTOMER                │
              │ SALES_ORDER             │
              │ ORDER_ITEM              │
              │ PRODUCT                 │
              │ CATEGORY                │
              │ STORE                   │
              │ SUPPLIER                │
              └────────────┬────────────┘
                           │
                           │ SQL extraction
                           ▼
                    ┌──────────────┐
                    │    PYTHON    │
                    │              │
                    │ Extract      │
                    │ Transform    │
                    │ Validate     │
                    │ Batch        │
                    └──────┬───────┘
                           │
                           │ Neo4j Python Driver
                           │ Parameterized Cypher
                           ▼
                    ┌──────────────┐
                    │    NEO4J     │
                    │              │
                    │ Nodes        │
                    │ Relations    │
                    │ Constraints  │
                    │ Indexes      │
                    └──────┬───────┘
                           │
                           ▼
              ┌─────────────────────────┐
              │ GRAPH-BASED ANALYTICS   │
              │                         │
              │ Product affinity        │
              │ Customer behavior       │
              │ Supplier relationships  │
              │ Recommendations         │
              │ Multi-hop analysis      │
              └─────────────────────────┘
```

---

# 4. Interview Explanation

> Snowflake was the source warehouse containing Staples sales and product data. We used Python to extract full and incremental records from Snowflake, validate them, and convert the relational data into graph-ready structures. We then used the Neo4j Python driver with parameterized Cypher, mainly UNWIND, MERGE, MATCH, and SET, to load nodes and relationships in batches. We created uniqueness constraints on business keys to prevent duplicates and used reconciliation checks after loading. Once the data was available in Neo4j, we used Cypher for customer-product relationships, product affinity, supplier impact, and other multi-hop analysis.

---

# 5. Snowflake Source Model

```text
CUSTOMER
--------
CUSTOMER_ID
CUSTOMER_NAME
STATE
EMAIL

SALES_ORDER
-----------
ORDER_ID
CUSTOMER_ID
STORE_ID
ORDER_DATE
ORDER_TOTAL

ORDER_ITEM
----------
ORDER_ID
PRODUCT_ID
QUANTITY
UNIT_PRICE

PRODUCT
-------
PRODUCT_ID
PRODUCT_NAME
CATEGORY_ID
SUPPLIER_ID
PRICE

CATEGORY
--------
CATEGORY_ID
CATEGORY_NAME

STORE
-----
STORE_ID
STORE_NAME
STATE

SUPPLIER
--------
SUPPLIER_ID
SUPPLIER_NAME
```

---

# 6. Snowflake Relational Query

```sql
SELECT
    c.customer_id,
    c.customer_name,
    o.order_id,
    o.order_date,
    p.product_id,
    p.product_name,
    cat.category_name,
    s.supplier_name,
    st.store_name
FROM customer c
JOIN sales_order o
    ON c.customer_id = o.customer_id
JOIN order_item oi
    ON o.order_id = oi.order_id
JOIN product p
    ON oi.product_id = p.product_id
JOIN category cat
    ON p.category_id = cat.category_id
JOIN supplier s
    ON p.supplier_id = s.supplier_id
JOIN store st
    ON o.store_id = st.store_id;
```

This illustrates the relational joins that become explicit relationships in Neo4j.

---

# 7. Neo4j Graph Data Model

```text
(Customer)
     |
     | PLACED
     ▼
  (Order)
    /   \
   /     \
CONTAINS  PURCHASED_AT
 /           \
▼             ▼
(Product)    (Store)
  |
  ├──────── BELONGS_TO ───────► (Category)
  |
  └──────── SUPPLIED_BY ───────► (Supplier)
```

Nodes:

```text
Customer
Order
Product
Category
Supplier
Store
```

Relationships:

```text
Customer -[:PLACED]-> Order
Order -[:CONTAINS]-> Product
Order -[:PURCHASED_AT]-> Store
Product -[:BELONGS_TO]-> Category
Product -[:SUPPLIED_BY]-> Supplier
```

The `CONTAINS` relationship can store transaction-specific values:

```cypher
(:Order)-[:CONTAINS {
    quantity: 3,
    unitPrice: 12.99
}]->(:Product)
```

### Memory Trick

**Nouns → Nodes**

**Verbs → Relationships**

---

# 8. Complete Data Flow

```text
1. Connect to Snowflake
        ↓
2. Run SQL extraction
        ↓
3. Convert rows to Python dictionaries
        ↓
4. Validate mandatory IDs
        ↓
5. Break records into batches
        ↓
6. Connect to Neo4j
        ↓
7. Create constraints/indexes
        ↓
8. MERGE nodes
        ↓
9. MATCH node endpoints
        ↓
10. MERGE relationships
        ↓
11. Reconcile Snowflake vs Neo4j
        ↓
12. Execute graph business queries
```

---

# 9. Required Python Packages

```bash
pip install snowflake-connector-python
pip install neo4j
```

```python
import snowflake.connector
from neo4j import GraphDatabase
```

---

# 10. Snowflake Connection

```python
import snowflake.connector

def get_snowflake_connection():
    return snowflake.connector.connect(
        user="USERNAME",
        password="PASSWORD",
        account="ACCOUNT",
        warehouse="COMPUTE_WH",
        database="STAPLES_DB",
        schema="SALES"
    )
```

For production, credentials should come from environment variables or an enterprise secrets manager, not source code.

---

# 11. Neo4j Connection

```python
from neo4j import GraphDatabase

NEO4J_URI = "neo4j+s://your-host"
NEO4J_USERNAME = "neo4j"
NEO4J_PASSWORD = "password"

driver = GraphDatabase.driver(
    NEO4J_URI,
    auth=(NEO4J_USERNAME, NEO4J_PASSWORD)
)

driver.verify_connectivity()
```

Close:

```python
driver.close()
```

---

# 12. Extract Customers from Snowflake

```python
def extract_customers(conn):
    sql = '''
    SELECT
        CUSTOMER_ID,
        CUSTOMER_NAME,
        STATE,
        EMAIL
    FROM CUSTOMER
    WHERE CUSTOMER_ID IS NOT NULL
    '''

    cursor = conn.cursor()

    try:
        cursor.execute(sql)
        rows = cursor.fetchall()

        return [
            {
                "customer_id": row[0],
                "customer_name": row[1],
                "state": row[2],
                "email": row[3]
            }
            for row in rows
        ]
    finally:
        cursor.close()
```

---

# 13. Extract Products

```python
def extract_products(conn):
    sql = '''
    SELECT
        PRODUCT_ID,
        PRODUCT_NAME,
        CATEGORY_ID,
        SUPPLIER_ID,
        PRICE
    FROM PRODUCT
    WHERE PRODUCT_ID IS NOT NULL
    '''

    cursor = conn.cursor()

    try:
        cursor.execute(sql)
        rows = cursor.fetchall()

        return [
            {
                "product_id": row[0],
                "product_name": row[1],
                "category_id": row[2],
                "supplier_id": row[3],
                "price": float(row[4]) if row[4] is not None else None
            }
            for row in rows
        ]
    finally:
        cursor.close()
```

---

# 14. Extract Orders

```python
def extract_orders(conn):
    sql = '''
    SELECT
        ORDER_ID,
        CUSTOMER_ID,
        STORE_ID,
        ORDER_DATE,
        ORDER_TOTAL
    FROM SALES_ORDER
    WHERE ORDER_ID IS NOT NULL
    '''

    cursor = conn.cursor()

    try:
        cursor.execute(sql)
        rows = cursor.fetchall()

        return [
            {
                "order_id": row[0],
                "customer_id": row[1],
                "store_id": row[2],
                "order_date": str(row[3]),
                "order_total": float(row[4]) if row[4] is not None else None
            }
            for row in rows
        ]
    finally:
        cursor.close()
```

---

# 15. Extract Order Items

```python
def extract_order_items(conn):
    sql = '''
    SELECT
        ORDER_ID,
        PRODUCT_ID,
        QUANTITY,
        UNIT_PRICE
    FROM ORDER_ITEM
    WHERE ORDER_ID IS NOT NULL
      AND PRODUCT_ID IS NOT NULL
    '''

    cursor = conn.cursor()

    try:
        cursor.execute(sql)
        rows = cursor.fetchall()

        return [
            {
                "order_id": row[0],
                "product_id": row[1],
                "quantity": int(row[2]),
                "unit_price": float(row[3])
            }
            for row in rows
        ]
    finally:
        cursor.close()
```

---

# 16. Validate Data in Python

```python
def validate_customer(row):
    if not row.get("customer_id"):
        return False

    if not row.get("customer_name"):
        return False

    return True
```

```python
valid_customers = [
    row
    for row in customers
    if validate_customer(row)
]
```

---

# 17. Create Neo4j Constraints

```cypher
CREATE CONSTRAINT customer_id_unique IF NOT EXISTS
FOR (c:Customer)
REQUIRE c.customerId IS UNIQUE;
```

```cypher
CREATE CONSTRAINT order_id_unique IF NOT EXISTS
FOR (o:Order)
REQUIRE o.orderId IS UNIQUE;
```

```cypher
CREATE CONSTRAINT product_id_unique IF NOT EXISTS
FOR (p:Product)
REQUIRE p.productId IS UNIQUE;
```

```cypher
CREATE CONSTRAINT category_id_unique IF NOT EXISTS
FOR (c:Category)
REQUIRE c.categoryId IS UNIQUE;
```

```cypher
CREATE CONSTRAINT supplier_id_unique IF NOT EXISTS
FOR (s:Supplier)
REQUIRE s.supplierId IS UNIQUE;
```

```cypher
CREATE CONSTRAINT store_id_unique IF NOT EXISTS
FOR (s:Store)
REQUIRE s.storeId IS UNIQUE;
```

### Interview Point

> We created uniqueness constraints on business identifiers before loading because MERGE depends on reliable entity identity and we wanted to prevent duplicate graph nodes.

---

# 18. Why MERGE Instead of CREATE?

`CREATE` always creates.

```cypher
CREATE (:Customer {customerId:$id});
```

If the same pipeline runs again, another node can be created.

`MERGE` performs match-or-create behavior.

```cypher
MERGE (c:Customer {customerId:$id});
```

### Memory Trick

```text
MERGE = MATCH OR CREATE
```

---

# 19. Load Customers One at a Time

```python
def load_customer(tx, customer):
    query = '''
    MERGE (c:Customer {
        customerId:$customer_id
    })

    SET
        c.name = $customer_name,
        c.state = $state,
        c.email = $email
    '''

    tx.run(
        query,
        customer_id=customer["customer_id"],
        customer_name=customer["customer_name"],
        state=customer["state"],
        email=customer["email"]
    )
```

Useful for learning, but not ideal for large loads.

---

# 20. Production-Friendly Batch Load with UNWIND

```cypher
UNWIND $rows AS row

MERGE (c:Customer {
    customerId:row.customer_id
})

SET
    c.name = row.customer_name,
    c.state = row.state,
    c.email = row.email;
```

Python:

```python
def load_customers(tx, customers):
    query = '''
    UNWIND $rows AS row

    MERGE (c:Customer {
        customerId:row.customer_id
    })

    SET
        c.name = row.customer_name,
        c.state = row.state,
        c.email = row.email
    '''

    tx.run(query, rows=customers).consume()
```

### Memory Trick

```text
U-M-S

UNWIND
MERGE
SET
```

---

# 21. Python Batch Helper

```python
def batches(data, batch_size=1000):
    for i in range(0, len(data), batch_size):
        yield data[i:i + batch_size]
```

Usage:

```python
for batch in batches(customers, 1000):
    with driver.session() as session:
        session.execute_write(
            load_customers,
            batch
        )
```

---

# 22. Load Product Nodes

```python
def load_products(tx, products):
    query = '''
    UNWIND $rows AS row

    MERGE (p:Product {
        productId:row.product_id
    })

    SET
        p.name = row.product_name,
        p.price = row.price
    '''

    tx.run(query, rows=products).consume()
```

---

# 23. Load Order Nodes

```python
def load_orders(tx, orders):
    query = '''
    UNWIND $rows AS row

    MERGE (o:Order {
        orderId:row.order_id
    })

    SET
        o.orderDate = date(row.order_date),
        o.total = row.order_total
    '''

    tx.run(query, rows=orders).consume()
```

---

# 24. Customer → Order Relationship

```python
def load_customer_orders(tx, orders):
    query = '''
    UNWIND $rows AS row

    MATCH (c:Customer {
        customerId:row.customer_id
    })

    MATCH (o:Order {
        orderId:row.order_id
    })

    MERGE (c)-[:PLACED]->(o)
    '''

    tx.run(query, rows=orders).consume()
```

---

# 25. Order → Product Relationship

```python
def load_order_items(tx, order_items):
    query = '''
    UNWIND $rows AS row

    MATCH (o:Order {
        orderId:row.order_id
    })

    MATCH (p:Product {
        productId:row.product_id
    })

    MERGE (o)-[r:CONTAINS]->(p)

    SET
        r.quantity = row.quantity,
        r.unitPrice = row.unit_price
    '''

    tx.run(query, rows=order_items).consume()
```

---

# 26. Product → Category

```python
def load_product_categories(tx, products):
    query = '''
    UNWIND $rows AS row

    MATCH (p:Product {
        productId:row.product_id
    })

    MATCH (c:Category {
        categoryId:row.category_id
    })

    MERGE (p)-[:BELONGS_TO]->(c)
    '''

    tx.run(query, rows=products).consume()
```

---

# 27. Product → Supplier

```python
def load_product_suppliers(tx, products):
    query = '''
    UNWIND $rows AS row

    MATCH (p:Product {
        productId:row.product_id
    })

    MATCH (s:Supplier {
        supplierId:row.supplier_id
    })

    MERGE (p)-[:SUPPLIED_BY]->(s)
    '''

    tx.run(query, rows=products).consume()
```

---

# 28. Order → Store

```python
def load_order_stores(tx, orders):
    query = '''
    UNWIND $rows AS row

    MATCH (o:Order {
        orderId:row.order_id
    })

    MATCH (s:Store {
        storeId:row.store_id
    })

    MERGE (o)-[:PURCHASED_AT]->(s)
    '''

    tx.run(query, rows=orders).consume()
```

---

# 29. Loading Order

Load nodes first, then relationships.

```text
Customer nodes
Product nodes
Order nodes
Category nodes
Supplier nodes
Store nodes
      ↓
Customer → Order
Order → Product
Product → Category
Product → Supplier
Order → Store
```

The relationship load uses `MATCH`, so endpoint nodes must already exist.

---

# 30. End-to-End Python Skeleton

```python
from neo4j import GraphDatabase

def main():

    sf_conn = get_snowflake_connection()

    customers = extract_customers(sf_conn)
    products = extract_products(sf_conn)
    orders = extract_orders(sf_conn)
    order_items = extract_order_items(sf_conn)

    driver = GraphDatabase.driver(
        NEO4J_URI,
        auth=(NEO4J_USERNAME, NEO4J_PASSWORD)
    )

    try:

        for batch in batches(customers, 1000):
            with driver.session() as session:
                session.execute_write(
                    load_customers,
                    batch
                )

        for batch in batches(products, 1000):
            with driver.session() as session:
                session.execute_write(
                    load_products,
                    batch
                )

        for batch in batches(orders, 1000):
            with driver.session() as session:
                session.execute_write(
                    load_orders,
                    batch
                )

        for batch in batches(orders, 1000):
            with driver.session() as session:
                session.execute_write(
                    load_customer_orders,
                    batch
                )

        for batch in batches(order_items, 1000):
            with driver.session() as session:
                session.execute_write(
                    load_order_items,
                    batch
                )

    finally:
        driver.close()
        sf_conn.close()


if __name__ == "__main__":
    main()
```

---

# 31. Incremental Loading

A real pipeline normally does not reload all history each day.

```text
Last successful timestamp
        ↓
Snowflake:
LAST_UPDATED_TS > watermark
        ↓
Python extracts changed records
        ↓
Cypher MERGE
        ↓
Neo4j create/update
        ↓
Validate
        ↓
Update watermark
```

Snowflake example:

```sql
SELECT
    PRODUCT_ID,
    PRODUCT_NAME,
    CATEGORY_ID,
    SUPPLIER_ID,
    PRICE,
    LAST_UPDATED_TS
FROM PRODUCT
WHERE LAST_UPDATED_TS > %s;
```

---

# 32. MERGE for Incremental Updates

```cypher
UNWIND $rows AS row

MERGE (p:Product {
    productId:row.product_id
})

ON CREATE SET
    p.createdAt = datetime()

ON MATCH SET
    p.updatedAt = datetime()

SET
    p.name = row.product_name,
    p.price = row.price;
```

---

# 33. Error Handling

```python
def load_batch(driver, load_function, batch):

    try:
        with driver.session() as session:
            session.execute_write(
                load_function,
                batch
            )

        print(f"Loaded {len(batch)} rows")

    except Exception as exc:
        print("Batch failed:", exc)
        raise
```

Production considerations:

```text
Batch ID
Source record IDs
Retry transient errors
Failure logging
Alerting
Do not advance watermark after failure
```

---

# 34. Logging

```python
import logging

logging.basicConfig(level=logging.INFO)

logger = logging.getLogger(
    "snowflake_to_neo4j"
)

logger.info("Starting product load")
```

Error:

```python
logger.exception(
    "Product batch failed"
)
```

---

# 35. Reconciliation

Snowflake:

```sql
SELECT COUNT(*)
FROM CUSTOMER;
```

Neo4j:

```cypher
MATCH (c:Customer)
RETURN count(c) AS customerCount;
```

Python:

```python
def get_neo4j_customer_count(driver):

    query = '''
    MATCH (c:Customer)
    RETURN count(c) AS count
    '''

    with driver.session() as session:
        return session.run(
            query
        ).single()["count"]
```

Comparison:

```python
snowflake_count = len(customers)
neo4j_count = get_neo4j_customer_count(driver)

if snowflake_count == neo4j_count:
    print("Validation passed")
else:
    print("Validation failed")
```

Also validate:

```text
Missing IDs
Duplicate IDs
Relationship counts
Nulls
Sample records
Property values
Failed batches
```

---

# 36. Data Quality Queries

Products without category:

```cypher
MATCH (p:Product)

WHERE NOT EXISTS {
    MATCH
    (p)-[:BELONGS_TO]->(:Category)
}

RETURN
    p.productId,
    p.name;
```

Products without supplier:

```cypher
MATCH (p:Product)

WHERE NOT EXISTS {
    MATCH
    (p)-[:SUPPLIED_BY]->(:Supplier)
}

RETURN
    p.productId,
    p.name;
```

Orders without customer:

```cypher
MATCH (o:Order)

WHERE NOT EXISTS {
    MATCH
    (:Customer)-[:PLACED]->(o)
}

RETURN o.orderId;
```

---

# 37. Business Query — Customer Purchase History

```cypher
MATCH
(c:Customer {
    customerId:$customerId
})
-[:PLACED]->
(o:Order)
-[:CONTAINS]->
(p:Product)

RETURN
    o.orderId,
    o.orderDate,
    p.productId,
    p.name

ORDER BY o.orderDate DESC;
```

---

# 38. Business Query — Top Products

```cypher
MATCH
(:Order)-[r:CONTAINS]->(p:Product)

RETURN
    p.name,
    sum(r.quantity) AS unitsSold

ORDER BY unitsSold DESC

LIMIT 10;
```

---

# 39. Business Query — Products Bought Together

```cypher
MATCH
(o:Order)-[:CONTAINS]->(p1:Product)

MATCH
(o)-[:CONTAINS]->(p2:Product)

WHERE
p1.productId < p2.productId

RETURN
    p1.name AS product1,
    p2.name AS product2,
    count(o) AS ordersTogether

ORDER BY ordersTogether DESC

LIMIT 10;
```

Interview point:

> This is a simple product-affinity query. Products that are connected to the same Order nodes can be ranked by the number of common orders.

---

# 40. Business Query — Similar Customers

```cypher
MATCH
(c1:Customer)-[:PLACED]->
(:Order)-[:CONTAINS]->
(p:Product)
<-[:CONTAINS]-(:Order)
<-[:PLACED]-(c2:Customer)

WHERE
c1.customerId < c2.customerId

RETURN
    c1.name,
    c2.name,
    count(DISTINCT p) AS sharedProducts

ORDER BY sharedProducts DESC;
```

---

# 41. Business Query — Supplier Impact

```cypher
MATCH
(s:Supplier {
    supplierId:$supplierId
})
<-[:SUPPLIED_BY]-
(p:Product)
<-[:CONTAINS]-
(o:Order)
<-[:PLACED]-
(c:Customer)

RETURN DISTINCT
    c.customerId,
    c.name,
    p.productId,
    p.name;
```

Graph path:

```text
Customer
   |
PLACED
   ▼
Order
   |
CONTAINS
   ▼
Product
   |
SUPPLIED_BY
   ▼
Supplier
```

This is a good multi-hop traversal example.

---

# 42. Product Recommendation Query

```cypher
MATCH
(target:Customer {
    customerId:$customerId
})
-[:PLACED]->
(:Order)-[:CONTAINS]->
(common:Product)

MATCH
(similar:Customer)
-[:PLACED]->
(:Order)-[:CONTAINS]->
(common)

MATCH
(similar)
-[:PLACED]->
(:Order)-[:CONTAINS]->
(recommended:Product)

WHERE
similar <> target

AND NOT EXISTS {
    MATCH
    (target)-[:PLACED]->
    (:Order)-[:CONTAINS]->
    (recommended)
}

RETURN
    recommended.productId,
    recommended.name,
    count(DISTINCT similar) AS score

ORDER BY score DESC

LIMIT 10;
```

---

# 43. Query Optimization

Use:

```cypher
EXPLAIN
MATCH
(c:Customer {
    customerId:$customerId
})
RETURN c;
```

`EXPLAIN` shows the plan without executing.

Use:

```cypher
PROFILE
MATCH
(c:Customer {
    customerId:$customerId
})
RETURN c;
```

`PROFILE` executes the query and shows actual runtime information.

### Performance Rules

```text
Use constraints/indexes
Start from selective IDs
Use explicit labels
Use explicit relationship types
Use parameters
Use UNWIND for batches
Avoid Cartesian products
Avoid unnecessary unbounded traversal
PROFILE slow queries
Return only required properties
```

---

# 44. Bad Cypher

```cypher
MATCH (c:Customer), (p:Product)

WHERE c.customerId = $id

RETURN c, p;
```

This creates unrelated combinations.

---

# 45. Better Cypher

```cypher
MATCH
(c:Customer {
    customerId:$id
})
-[:PLACED]->
(:Order)
-[:CONTAINS]->
(p:Product)

RETURN p;
```

---

# 46. Parameterized Queries

Avoid:

```python
query = (
    "MATCH (c:Customer {customerId:'"
    + customer_id +
    "'}) RETURN c"
)
```

Use:

```python
query = '''
MATCH (c:Customer {
    customerId:$customer_id
})
RETURN c
'''

result = session.run(
    query,
    customer_id=customer_id
)
```

Benefits:

```text
Security
Cleaner code
Query plan reuse
Maintainability
```

---

# 47. Python Query Example

```python
def get_customer_products(
    driver,
    customer_id
):

    query = '''
    MATCH
    (:Customer {
        customerId:$customer_id
    })
    -[:PLACED]->
    (:Order)
    -[:CONTAINS]->
    (p:Product)

    RETURN DISTINCT
        p.productId AS id,
        p.name AS name
    '''

    with driver.session() as session:

        result = session.run(
            query,
            customer_id=customer_id
        )

        return [
            record.data()
            for record in result
        ]
```

---

# 48. Python Top-Products Query

```python
def get_top_products(
    driver,
    limit=10
):

    query = '''
    MATCH
    (:Order)-[r:CONTAINS]->(p:Product)

    RETURN
        p.productId AS id,
        p.name AS name,
        sum(r.quantity) AS quantity

    ORDER BY quantity DESC

    LIMIT $limit
    '''

    with driver.session() as session:

        result = session.run(
            query,
            limit=limit
        )

        return [
            record.data()
            for record in result
        ]
```

---

# 49. Why Python?

Interview answer:

> Python acted as the integration layer. It handled Snowflake extraction, source-to-graph transformation, validation, batching, logging, error handling, incremental-load logic, and execution of parameterized Cypher through the Neo4j driver.

---

# 50. Why Snowflake?

> Snowflake remained the analytical source and warehouse for Staples sales and product data. Neo4j complemented it for connected-data use cases rather than replacing the warehouse.

---

# 51. Why Neo4j?

> The use cases required traversing relationships across customers, orders, products, categories, suppliers, and stores. Neo4j made these relationships explicit and easier to query using Cypher.

---

# 52. How Did You Model the Graph?

> I identified business entities as nodes and business interactions as relationships. Customer, Order, Product, Category, Supplier, and Store were nodes. PLACED, CONTAINS, BELONGS_TO, SUPPLIED_BY, and PURCHASED_AT were relationships. Transaction-specific fields such as quantity and unit price were stored on the Order-to-Product relationship.

---

# 53. How Did You Avoid Duplicates?

> We used stable Snowflake business keys such as CUSTOMER_ID, PRODUCT_ID, and ORDER_ID. Neo4j uniqueness constraints enforced identity, and Cypher MERGE made repeated loads idempotent.

---

# 54. Why UNWIND?

> UNWIND allowed us to send a list of Python dictionaries to Neo4j and process the list as rows inside one Cypher statement. This was more efficient than issuing one network request per record.

---

# 55. How Did You Handle Large Volumes?

> Records were processed in configurable batches instead of one record at a time or one huge transaction. We used UNWIND plus parameterized MERGE statements and tuned the batch size based on throughput and transaction behavior.

---

# 56. How Did You Handle Incremental Loads?

> We maintained a last-successful-load watermark. Snowflake extraction selected records with a modified timestamp after the watermark. MERGE created new nodes or updated existing ones, and the watermark was advanced only after the batch passed validation.

---

# 57. How Did You Validate the Pipeline?

> We compared Snowflake and Neo4j entity counts, checked missing business keys, verified relationship counts, validated sample records, and ran graph-specific data-quality checks such as products without categories or suppliers.

---

# 58. How Did You Handle Failures?

> Loads were processed in batches. When a batch failed, we logged the batch and source IDs, retried transient errors, and did not advance the incremental watermark until the issue was resolved.

---

# 59. Main Cypher Commands Used

```text
UNWIND
MERGE
MATCH
SET
WHERE
WITH
RETURN
OPTIONAL MATCH
EXISTS
EXPLAIN
PROFILE
```

For ingestion:

```text
UNWIND + MERGE + MATCH + SET
```

For analysis:

```text
MATCH + WHERE + WITH + aggregation + RETURN
```

---

# 60. CREATE vs MERGE

```text
CREATE
Always creates a new graph element.

MERGE
Attempts to match the pattern.
Creates it only if it does not exist.
```

For repeatable ingestion, MERGE is generally the important pattern.

---

# 61. MATCH vs OPTIONAL MATCH

```text
MATCH
Requires the pattern to exist.

OPTIONAL MATCH
Keeps the result even when the optional pattern does not exist.
```

Useful SQL analogy:

```text
MATCH ≈ INNER JOIN
OPTIONAL MATCH ≈ LEFT JOIN
```

It is only a mental analogy, not an exact one-to-one mapping.

---

# 62. WITH

```cypher
MATCH
(c:Customer)-[:PLACED]->(o:Order)

WITH
    c,
    count(o) AS orderCount

WHERE orderCount > 10

RETURN
    c.name,
    orderCount;
```

> WITH passes intermediate variables and aggregations to the next stage of the Cypher pipeline.

---

# 63. 60-Second Interview Story

> In the Staples project, sales and product data was stored in Snowflake across tables such as Customer, Sales Order, Order Item, Product, Category, Supplier, and Store. For relationship-heavy analysis, we created a Neo4j graph model. Python extracted and validated records from Snowflake and processed them in batches. Using the Neo4j Python driver, we ran parameterized Cypher with UNWIND and MERGE to create Customer, Order, Product, Category, Supplier, and Store nodes and relationships such as PLACED, CONTAINS, BELONGS_TO, and SUPPLIED_BY. We created uniqueness constraints to prevent duplicates, supported incremental loading using modified timestamps, and performed source-to-target reconciliation. The resulting graph supported customer purchase analysis, product affinity, recommendations, and supplier impact queries.

---

# 64. Two-Minute Interview Story

> The source for this use case was Snowflake, where Staples sales and product data existed in a relational model. The main tables included Customer, Sales Order, Order Item, Product, Category, Supplier, and Store.

> Some business questions required following multiple relationships, such as finding which customers bought products supplied by a particular supplier or which products were commonly purchased together. We therefore represented this subset of the data in Neo4j.

> The graph model used Customer, Order, Product, Category, Supplier, and Store nodes. The main relationships were Customer PLACED Order, Order CONTAINS Product, Product BELONGS_TO Category, Product SUPPLIED_BY Supplier, and Order PURCHASED_AT Store.

> Python acted as the integration layer. It connected to Snowflake, executed SQL, transformed source rows into Python dictionaries, validated mandatory business keys, and broke the data into batches.

> We used the Neo4j Python driver to execute parameterized Cypher. UNWIND converted each Python batch into Cypher rows, MERGE created or matched nodes, MATCH located existing endpoints, and MERGE created the relationships. We used uniqueness constraints to prevent duplicate entity IDs.

> Incremental loads were based on last-modified timestamps, and MERGE allowed us to create new records or update existing graph entities. After loading, we reconciled counts and business keys between Snowflake and Neo4j and ran data-quality checks before advancing the load watermark.

> Once loaded, Cypher was used for multi-hop queries, product affinity, customer purchase behavior, supplier impact analysis, and recommendation-oriented use cases.

---

# 65. Whiteboard Architecture

If asked to draw it:

```text
Snowflake
    |
    | SQL
    ▼
Python
    |
    | Extract
    | Validate
    | Transform
    | Batch
    ▼
Neo4j Driver
    |
    | UNWIND
    | MERGE
    | MATCH
    ▼
Neo4j
    |
    ├── Customer
    ├── Order
    ├── Product
    ├── Category
    ├── Supplier
    └── Store
```

Then draw:

```text
Customer
   |
 PLACED
   ▼
 Order
   |
 CONTAINS
   ▼
 Product
  /     \
 ▼       ▼
Category Supplier
```

---

# 66. Whiteboard Loading Flow

```text
Snowflake query
      ↓
Python rows
      ↓
Dictionary list
      ↓
Batch
      ↓
UNWIND $rows
      ↓
MERGE nodes
      ↓
MATCH endpoints
      ↓
MERGE relationships
      ↓
Reconcile
```

---

# 67. Troubleshooting Example

Scenario:

```text
Snowflake products = 50,000
Neo4j products = 49,800
```

Check:

1. Null `PRODUCT_ID`
2. Duplicate source IDs
3. Transformation filters
4. Failed batches
5. Neo4j constraint errors
6. Source and target ID differences
7. Load logs
8. Retry/reload only missing records

Interview answer:

> I would not blindly rerun the complete pipeline. I would first compare source and target business keys and determine whether the difference came from null IDs, duplicate source rows, validation filtering, or failed Neo4j transactions.

---

# 68. Common Interview Mistakes

Do not say:

```text
Neo4j replaced Snowflake.
```

Say:

```text
Neo4j complemented Snowflake for relationship-heavy use cases.
```

Do not say:

```text
We loaded every record individually.
```

Say:

```text
We used UNWIND-based batch loading.
```

Do not say:

```text
Neo4j has no schema.
```

Say:

```text
Neo4j is schema-flexible, but production graphs still need a governed data model, constraints, indexes, and naming standards.
```

---

# 69. Most Important Cypher to Memorize

Node batch:

```cypher
UNWIND $rows AS row

MERGE (p:Product {
    productId:row.product_id
})

SET
    p.name = row.product_name,
    p.price = row.price;
```

Relationship batch:

```cypher
UNWIND $rows AS row

MATCH (o:Order {
    orderId:row.order_id
})

MATCH (p:Product {
    productId:row.product_id
})

MERGE (o)-[r:CONTAINS]->(p)

SET
    r.quantity = row.quantity,
    r.unitPrice = row.unit_price;
```

Business query:

```cypher
MATCH
(c:Customer {
    customerId:$id
})
-[:PLACED]->
(:Order)
-[:CONTAINS]->
(p:Product)

RETURN p.name;
```

---

# 70. Most Important Python to Memorize

```python
from neo4j import GraphDatabase

driver = GraphDatabase.driver(
    URI,
    auth=(USERNAME, PASSWORD)
)

with driver.session() as session:
    result = session.run(
        query,
        customer_id="C101"
    )
```

Batch write:

```python
with driver.session() as session:
    session.execute_write(
        load_products,
        batch
    )
```

---

# 71. Memory Formula

```text
S P C V
```

**S — Snowflake**

Source data.

**P — Python**

Extract, transform, validate, batch.

**C — Cypher**

UNWIND, MERGE, MATCH, SET.

**V — Validate**

Counts, IDs, relationships, errors.

---

# 72. Full Memory Flow

```text
SNOWFLAKE
    ↓
PYTHON
    ↓
VALIDATE
    ↓
BATCH
    ↓
UNWIND
    ↓
MERGE NODES
    ↓
MATCH NODES
    ↓
MERGE RELATIONSHIPS
    ↓
RECONCILE
    ↓
GRAPH QUERIES
```

---

# 73. Ten Interview Questions to Practice

1. Why did you move some Snowflake data into Neo4j?
2. What nodes and relationships did you create?
3. How did Python connect Snowflake to Neo4j?
4. Why did you use UNWIND?
5. Why did you use MERGE?
6. How did you avoid duplicate nodes?
7. How did you handle incremental loads?
8. How did you validate source and target?
9. How did you tune Cypher?
10. Give one multi-hop business query.

---

# 74. Quick Answers

### Why Neo4j?

Relationship-heavy analysis across customers, orders, products, categories, stores, and suppliers.

### Why Python?

Extraction, transformation, validation, batching, error handling, and Neo4j integration.

### Why UNWIND?

Efficient batch processing.

### Why MERGE?

Repeatable/idempotent create-update loading.

### Why constraints?

Reliable entity identity and duplicate prevention.

### How incremental?

Last-modified timestamp + MERGE + watermark.

### How validate?

Counts, business keys, relationship checks, nulls, sample records, and failed batches.

---

# 75. CV Point

> Developed Python-based pipelines to extract Staples sales and product data from Snowflake and load it into Neo4j using parameterized Cypher queries for graph-based relationship analysis.

More technical version:

> Developed Python-based Snowflake-to-Neo4j pipelines using batch processing, UNWIND, MERGE, and parameterized Cypher to load and maintain sales and product graph data.

---

# 76. Final Interview Closing Statement

> My role connected traditional data engineering with graph engineering. Snowflake remained the warehouse, Python handled extraction and pipeline processing, and Neo4j provided the connected-data layer. Cypher was used both for efficient ingestion and relationship-based analysis.

---

# 77. One-Page Final Revision

```text
SOURCE
------
Snowflake

Customer
Order
Order Item
Product
Category
Supplier
Store


PYTHON
------
Snowflake connector
Neo4j driver

Extract
Validate
Transform
Batch
Log
Reconcile


NEO4J
-----
Customer
Order
Product
Category
Supplier
Store


RELATIONSHIPS
-------------
Customer -PLACED-> Order

Order -CONTAINS-> Product

Product -BELONGS_TO-> Category

Product -SUPPLIED_BY-> Supplier

Order -PURCHASED_AT-> Store


CYPHER
------
UNWIND
MERGE
MATCH
SET
WITH
WHERE
RETURN
PROFILE


PERFORMANCE
-----------
Constraints
Indexes
Batching
Parameters
PROFILE


VALIDATION
----------
Counts
IDs
Relationships
Nulls
Errors


BUSINESS
--------
Product affinity
Customer behavior
Recommendations
Supplier impact
Multi-hop analysis
```

---

# 78. Final Practice Assignment

Without looking at the answers, practice writing:

### Python

1. Snowflake connection
2. Product extraction
3. Neo4j connection
4. Product batch loader
5. 1,000-row batching
6. Customer-product query

### Cypher

1. Customer MERGE
2. Product MERGE
3. Order MERGE
4. Customer → Order
5. Order → Product
6. Product → Supplier
7. Product → Category
8. Customer purchase history
9. Top products
10. Supplier impact

### Verbal

Explain in under two minutes:

```text
Snowflake
↓
Python
↓
Cypher
↓
Neo4j
↓
Validation
↓
Graph analytics
```

If you can explain that flow naturally and write the main `UNWIND + MERGE` patterns without looking at notes, you will be ready to discuss this project in a Neo4j / Graph AI interview.
