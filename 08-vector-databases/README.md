# Vector Databases Module

The storage and search substrate for RAG and semantic memory. This module covers index theory, vendor tradeoffs, and operational concerns.

## 🚀 Roadmap

### 1. Vector DB Landscape & Selection
- Self-hosted vs managed
- Purpose-built vs hybrid (pgvector)
- Selection matrix

### 2. Embedding Dimensionality & Quality
- Dimensions vs recall vs cost
- Model selection impacts on storage
- Normalization

### 3. Similarity Metrics
- Cosine, dot product, L2
- When each applies
- Prereq: `02-dsa/10_searching`

### 4. ANN Index: HNSW
- Graph construction
- `M`, `ef_construction`, `ef_search` parameters
- Prereq: `02-dsa/09_graphs`

### 5. ANN Index: IVF, PQ, ScaNN
- Inverted file (IVF) basics
- Product quantization
- ScaNN highlights

### 6. Chroma & FAISS
- Local development workflows
- Persistence and memory-mapped files

### 7. Pinecone
- Managed service basics
- Pod types, capacity planning
- Pricing model

### 8. Weaviate & Qdrant
- Schema-aware vector DBs
- Filtering capabilities
- Self-host options

### 9. pgvector on Postgres
- Extension setup
- HNSW vs IVFFlat in pgvector
- Combining with SQL filters

### 10. Metadata Filtering & Hybrid Queries
- Pre-filter vs post-filter
- Hybrid (vector + keyword) queries
- BM25 integration

### 11. Sharding, Replication, Scaling
- When to shard
- Replica strategies
- Consistency tradeoffs

### 12. Benchmarking
- Recall vs latency vs cost
- ANN-Benchmarks methodology
- Building your own benchmark harness
