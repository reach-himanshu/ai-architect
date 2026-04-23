# Trade-Off Discussions

For each trade-off, use this structure:
> **Options** → **Key variables that determine the choice** → **My recommendation + why**

---

## ML Architecture Trade-Offs

### Fine-Tuning vs RAG vs Prompt Engineering

| Approach | Latency | Cost | Freshness | Accuracy | Effort |
|---------|--------|------|---------|---------|--------|
| Prompt engineering | Low | Low | High | Moderate | Low |
| RAG | Medium | Medium | High | High | Medium |
| Fine-tuning | Low | High | Low (stale) | Very High | High |

**When to choose:**

*   **Prompt engineering:** Simple tasks, GPT-4 class model, no private data.
*   **RAG:** Private/frequently updated knowledge, need citations.
*   **Fine-tuning:** Domain-specific style/format, very high accuracy needed, low-latency requirement.
*   **RAG + Fine-tuning:** Best of both — fine-tune on domain format, RAG for factual retrieval.

---

### Batch Inference vs Real-Time Inference

| Dimension | Batch | Real-Time |
|----------|-------|-----------|
| Latency | Hours | Milliseconds |
| Cost | Lower (spot instances) | Higher (always-on) |
| Use case | Nightly reports, email personalisation | Fraud detection, search ranking |
| Complexity | Lower | Higher (feature freshness, SLA) |

**Choose batch** when: Results can be precomputed and user doesn't need instant response.
**Choose real-time** when: Input depends on user action; < 1s SLA required.

---

### Vector DB vs Traditional Search (BM25)

| Dimension | Dense Vector (FAISS/Pinecone) | Sparse/Lexical (BM25/Elasticsearch) |
|----------|------------------------------|--------------------------------------|
| Semantic match | ✓ (paraphrase, synonyms) | ✗ (exact term) |
| Keyword match | ✗ | ✓ |
| Out-of-vocabulary | ✓ (embedding handles) | ✗ |
| Latency | ~1ms | ~1ms |
| Explainability | Low | High (TF-IDF scores) |

**Best practice:** Hybrid search (BM25 + dense) with Reciprocal Rank Fusion. Neither alone is best.

---

### Chunking Strategy for RAG

| Strategy | Size | Overlap | Pros | Cons |
|---------|------|---------|------|------|
| Fixed-size | 512 tokens | 50 tokens | Simple | Splits sentences |
| Sentence | ~3–5 sentences | 1 sentence | Natural boundaries | Variable size |
| Recursive | Up to 512 tokens | 50 tokens | Respects structure | More complex |
| Parent-child | Small child, large parent | — | Full context | Higher memory |

**Default recommendation:** Recursive character splitter (512/50) as baseline.
Upgrade to parent-child when context loss is a problem.

---

## Data Engineering Trade-Offs

### Kafka vs SQS vs Pub/Sub

| Dimension | Kafka | SQS | Pub/Sub (GCP) |
|----------|-------|-----|--------------|
| Replay | ✓ (offset-based) | ✗ | ✓ (limited) |
| Ordering | Per partition | ✗ (FIFO queue available) | ✓ (per key) |
| Consumer groups | ✓ | ✗ | ✓ |
| Operational complexity | High | Low | Medium |

**Choose Kafka** when: Event replay needed (ML reprocessing), multiple consumer groups, ordered processing.
**Choose SQS** when: Simple task queue, AWS native, low ops overhead.

---

### SQL vs NoSQL

| Use Case | Recommended | Reason |
|---------|------------|--------|
| Relational data, complex queries | PostgreSQL | ACID, JOINs, indexes |
| User sessions, caching | Redis | In-memory, sub-ms latency |
| Large-scale, simple lookups | DynamoDB | Single-digit ms, serverless |
| Time-series metrics | InfluxDB / TimescaleDB | Efficient time-range queries |
| Full-text search | Elasticsearch | Inverted index, BM25 |
| Graph relationships | Neo4j | Native graph traversal |

---

### Lambda vs Kappa Architecture

| Dimension | Lambda | Kappa |
|----------|--------|-------|
| Latency | Low (speed layer) | Low (streaming) |
| Accuracy | High (batch layer) | High (with replays) |
| Complexity | High (dual code) | Lower (single code) |
| Replay cost | Low (batch job) | Higher (Kafka retention) |

**Kappa is the modern default** when Kafka retention is feasible.
**Lambda** when historical batch accuracy is non-negotiable and teams have Spark expertise.

---

## System Design Trade-Offs

### Microservices vs Monolith for ML Platform

| | Microservices | Monolith |
|-|---------------|---------|
| Scalability | Independent scaling per service | Scale all together |
| Deploy velocity | Fast (small services) | Slower (big deploys) |
| Operational complexity | High | Low |
| Suitable for | Large teams, 50+ engineers | Small team, early stage |

**Rule of thumb:** Start with a modular monolith. Extract services when a clear scaling or team boundary emerges.

---

### Consistency vs Availability (CAP)

**Always ask: "What does the user experience in a failure?"**

*   **Prefer consistency (CP):** Financial transactions, inventory management, auth tokens.
*   **Prefer availability (AP):** Social feeds, recommendations, search results.

**Most ML systems prefer AP:** A stale recommendation is better than no recommendation.

---

### Caching Strategy

| Level | Where | TTL | Use Case |
|-------|-------|-----|---------|
| CDN | Edge | Hours | Static model responses, documentation |
| Application | Redis | Minutes | LLM responses, feature values |
| Semantic | Vector DB | Hours | Semantically similar queries |
| DB query | Query cache | Seconds | Repeated aggregations |

**LLM caching tip:** Exact match cache on SHA256(prompt). 5–30% hit rate for FAQ workloads.
Semantic cache on cosine similarity > 0.95: higher hit rate but adds ~10ms.

---

## Model Evaluation Trade-Offs

### Offline Metrics vs Online Metrics

| | Offline (AUC, F1, BLEU) | Online (CTR, Revenue, Engagement) |
|-|------------------------|-----------------------------------|
| Speed | Fast (no live traffic) | Slow (A/B test 2+ weeks) |
| Risk | None | Real user impact |
| Correlation | Moderate | Ground truth |

**Key insight:** Offline metrics can improve while online metrics degrade.
Always validate offline improvements with an A/B test before full rollout.

### Precision vs Recall Trade-Off

| Scenario | Prioritise | Why |
|---------|-----------|-----|
| Fraud detection (block transactions) | Precision | False positives are very costly (UX) |
| Cancer screening | Recall | Missing a positive is dangerous |
| Content moderation | Recall | Better to over-flag than miss harmful content |
| Recommendation spam | Precision | Users notice bad recommendations immediately |
