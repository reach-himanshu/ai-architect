# System Design 04: Database Selection

## 1. The Database Landscape

| Type | Examples | Use Case |
| :--- | :--- | :--- |
| **Relational (OLTP)** | PostgreSQL, MySQL | Transactions, relationships, ACID |
| **Relational (OLAP)** | BigQuery, Snowflake, Redshift | Large analytical scans |
| **Document** | MongoDB, CouchDB | Flexible schema, nested documents |
| **Key-Value** | Redis, DynamoDB | Sessions, caching, simple lookups |
| **Wide-Column** | Cassandra, HBase | Time-series, massive write throughput |
| **Graph** | Neo4j, Neptune | Relationship traversal |
| **Vector** | Pinecone, Qdrant, pgvector | Semantic search, RAG |
| **Search** | Elasticsearch | Full-text search, log analysis |
| **Time-Series** | InfluxDB, TimescaleDB | Metrics, monitoring, IoT |

---

## 2. Choosing Based on Access Patterns

The access pattern is the primary driver of database choice:

| Access Pattern | Best Choice |
| :--- | :--- |
| Point lookups by ID | Redis, DynamoDB |
| Complex relational queries, transactions | PostgreSQL |
| Flexible/evolving schema | MongoDB |
| Full-text search | Elasticsearch |
| Graph traversals | Neo4j |
| Vector similarity search | Qdrant, Pinecone, pgvector |
| Columnar analytics | BigQuery, Snowflake |
| Write-heavy, massive scale | Cassandra |
| Time-ordered metrics | InfluxDB, TimescaleDB |

---

## 3. Indexing Essentials

| Index Type | Use |
| :--- | :--- |
| **B-tree** | Equality and range queries (PostgreSQL default) |
| **Hash** | Equality-only, faster than B-tree for exact match |
| **Composite** | Multi-column `WHERE` and `ORDER BY` |
| **Partial** | Index subset of rows (e.g., `WHERE active = true`) |
| **Full-text (GIN)** | Text search in PostgreSQL |
| **HNSW / IVF** | Vector similarity in vector databases |

**Rule**: Index columns in `WHERE`, `JOIN ON`, and `ORDER BY`. Over-indexing slows writes.

---

## 4. Read Replicas

*   Direct all writes to primary, all reads to one or more replicas.
*   Replication lag: typically 10ms–seconds.
*   Never read your own writes from a replica — route writes to primary, then wait or read from primary.
*   Use replicas for: analytics queries, reporting, read-heavy workloads.

---

## 5. Sharding Strategies

| Strategy | Pros | Cons | Best For |
| :--- | :--- | :--- | :--- |
| **Range** | Range queries efficient | Hot spots on new data | Time-series |
| **Hash** | Even distribution | No range queries | User data |
| **Directory** | Flexible routing | Lookup overhead | Complex rules |
| **Geo** | Data locality | Uneven if geo-skewed | Global apps |

---

## 6. Connection Pooling

Every DB has a max connection limit. Without pooling, each request opens a new connection:

*   **Without pooling**: 1000 requests = 1000 connections (PostgreSQL max ≈ 200–500).
*   **With PgBouncer/pgpool**: 1000 requests → pool of 20 connections.

Connection pool sizing rule: `pool_size ≈ num_cores × 2`
