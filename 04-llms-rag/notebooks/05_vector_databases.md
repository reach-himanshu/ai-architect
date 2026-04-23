# LLMs 05: Vector Databases

Vector databases are the backbone of RAG systems. They store embeddings and enable fast approximate nearest-neighbor (ANN) search at scale.

## 1. Why Not Just Use a Regular Database?

Regular databases (PostgreSQL, MongoDB) can store vectors, but:
*   SQL `WHERE` clauses don't compute cosine similarity efficiently.
*   Brute-force search over millions of vectors is O(n) — too slow.
*   No built-in ANN indexes (HNSW, IVF, LSH).

Vector databases solve this with purpose-built indexes.

---

## 2. Key Vector Database Options

| Database | Type | Best For | Notes |
| :--- | :--- | :--- | :--- |
| **Pinecone** | Managed SaaS | Production, ease of use | Fully managed, expensive at scale |
| **Weaviate** | Open source / Cloud | Multi-tenancy, hybrid search | GraphQL API, rich filtering |
| **Qdrant** | Open source / Cloud | High performance, filtering | Rust-based, excellent ANN performance |
| **Chroma** | Open source | Local dev, prototyping | Simple API, Python-native |
| **pgvector** | PostgreSQL ext. | Existing Postgres users | No new infra, limited ANN scale |
| **FAISS** | Library (Meta) | Research, custom pipelines | No server, just a library |
| **Milvus** | Open source | Enterprise scale | Complex ops, very scalable |

---

## 3. Indexing Algorithms

| Algorithm | Full Name | Trade-off |
| :--- | :--- | :--- |
| **Flat** | Brute force | Exact results, O(n) — only for small datasets |
| **HNSW** | Hierarchical Navigable Small World | Fast ANN, high memory usage |
| **IVF** | Inverted File Index | Lower memory, slightly less accurate |
| **PQ** | Product Quantization | Compressed storage, faster search, some accuracy loss |
| **IVFPQ** | IVF + PQ combined | Best balance for large scale |

**HNSW** is the most widely used in production (used by Pinecone, Qdrant, Weaviate).

---

## 4. Metadata Filtering

A critical production feature: filter by metadata before or after ANN search.

```python
# Retrieve documents about "RAG" from a specific date range
results = collection.query(
    query_embeddings=[query_vector],
    where={
        "topic": "rag",
        "date": {"$gte": "2024-01-01"},
        "language": "en",
    },
    n_results=5,
)
```

**Pre-filtering** (filter first, then ANN): Fast but may miss relevant results if filter is too narrow.
**Post-filtering** (ANN first, then filter): More accurate but wastes computation on filtered-out results.

---

## 5. Collection Design Patterns

**Single collection**: All documents in one collection, use metadata filtering.
- Simple, flexible.
- Risk of "cross-contamination" between unrelated document types.

**Multiple collections**: Separate collection per document type / tenant.
- Clean isolation, easier authorization.
- Harder to do cross-collection search.

**Multi-tenancy with namespaces**: One collection, tenant ID in metadata.
- Used by Pinecone (namespaces) and Qdrant (payload filtering).
- Best for SaaS products with many customers.

---

## 6. Operational Considerations

*   **Index rebuild**: HNSW indexes are mutable but rebuilding improves performance. Plan for periodic rebuilds.
*   **Replication**: For production, always run at least 2 replicas.
*   **Backups**: Vector stores need backup strategies — not all support this natively.
*   **Versioning**: When embedding models change, you need to re-embed and reload.
*   **Monitoring**: Track p99 query latency, index size growth, and recall@k.

---

## 7. Hybrid Search

Combine dense (vector) search with sparse (keyword BM25) search for better results.

```
Final Score = α × Dense_Score + (1-α) × Sparse_Score
```

This handles:
*   Exact keyword matches (product codes, names) — sparse excels.
*   Semantic similarity — dense excels.
*   Mixed queries — hybrid wins.

Supported natively by: Weaviate, Qdrant, Elasticsearch with vector plugin.
