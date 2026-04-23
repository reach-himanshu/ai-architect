# AI System Design Cases

Use the **RADIO framework** for each:

> **R**equirements → **A**PI → **D**ata → **I**mplementation → **O**ptimisation

---

## Case 1: Design a RAG-based Enterprise Knowledge Search

### Requirements

*   **Functional:** Users query a natural language question; system returns accurate answers with citations from internal documents (PDFs, Confluence, SharePoint).
*   **Non-functional:** p95 latency < 3s; 10,000 employees; document corpus 500k docs; 100k queries/day.

### API

```
POST /search
  { "query": string, "user_id": string, "top_k": int=5 }
→ { "answer": string, "sources": [{doc_id, title, excerpt, score}], "latency_ms": int }
```

### Data

*   **Ingestion:** Crawl documents → chunk (512 tokens, 50-token overlap) → embed (text-embedding-3-small) → store in vector DB (Pinecone / Weaviate).
*   **Metadata:** Store doc title, URL, last_modified, permissions in a relational DB.
*   **Filtering:** Pre-filter by user's permission groups before vector search.

### Implementation

```
Query → Permission Filter → Hybrid Search (BM25 + dense) → RRF re-rank
     → Prompt Construction (query + top-5 chunks)
     → LLM (GPT-4o or Claude Sonnet) → Response + Citations
     → Cache (exact query hash) → Return
```

### Optimisation

*   **Prompt caching:** System prompt + frequent context → cached prefix (90% cost reduction).
*   **Semantic cache:** Embed query, nearest-neighbour in Redis → cache hit if similarity > 0.92.
*   **Re-ranking:** Cross-encoder re-ranker on top-20 → serve top-5 (improved precision).
*   **Freshness:** Delta ingestion (only changed documents) every 15 minutes.

---

## Case 2: Design a Real-Time Fraud Detection System

### Requirements

*   Detect fraudulent transactions in < 100ms.
*   Handle 50,000 transactions/second peak.
*   False positive rate < 0.1% (user experience critical).

### Implementation

```
Transaction Event → Kafka → Feature Server → ML Model → Decision
                         ↓
                 Real-time Feature Store (Redis)
                 - user_tx_count_1h
                 - merchant_risk_score
                 - velocity_ratio
                         ↓
                 Gradient Boosting Model (XGBoost)
                 Latency: ~10ms
```

### Key Design Decisions

*   **Model choice:** GBT (not LLM) — 10ms latency requirement, interpretability needed.
*   **Feature freshness:** Online features from Redis (< 1s lag); offline features from DynamoDB (daily).
*   **Thresholding:** Two thresholds — BLOCK (high confidence), REVIEW (manual review queue), PASS.
*   **Fallback:** If ML service is down → rule-based fallback (velocity checks, blacklist).

---

## Case 3: Design a Recommendation System for an E-Commerce Platform

### Requirements

*   Homepage recommendations, "customers also bought", similar items.
*   100M users, 10M products, 1B interactions/day.
*   p95 latency < 200ms.

### Architecture

```
Offline:
  User interactions → Spark → Matrix Factorisation → User/Item embeddings
  → Batch precompute top-100 candidates per user → DynamoDB

Online:
  Request → User embedding lookup → ANN search (FAISS) → Re-ranking
  → Business rules (exclude out-of-stock, diversity) → Response
```

### Two-Tower Model

*   Query tower: user embedding (age, location, history).
*   Item tower: item embedding (category, price, description).
*   Trained to maximise dot product for user-item interactions.

---

## Case 4: Design an LLM-Based Customer Support Agent

### Requirements

*   Handle 80% of tier-1 support tickets automatically.
*   Escalate to human when confidence is low.
*   Integrate with CRM, order management, knowledge base.

### Architecture

```
Customer Message → Intent Classifier → Router
    ├─ FAQ (confidence > 0.9) → RAG over knowledge base → Response
    ├─ Order status           → Tool call → Order API → Response
    ├─ Refund request         → Tool call → CRM → Human approval flow
    └─ Complex/uncertain      → Escalate to human agent
```

### Safety Design

*   **Guardrails:** PII redaction before LLM; output filtering for harmful content.
*   **Confidence gating:** If classifier confidence < 0.7, route to human.
*   **Audit log:** Every LLM call logged with prompt, response, user_id, session_id.

---

## Case 5: Design a Model Training Platform

### Requirements

*   Data scientists submit training jobs; platform manages compute, tracking, and deployment.
*   Support PyTorch, XGBoost, Scikit-learn.
*   100 researchers, 500 jobs/day.

### Components

| Component | Tool | Purpose |
|----------|------|---------|
| Job scheduler | Kubernetes Jobs | Allocate GPU/CPU pods |
| Experiment tracking | MLflow | Log params, metrics, artifacts |
| Model registry | MLflow Registry | Version + promote models |
| Feature store | Feast / Tecton | Consistent feature access |
| Artifact store | S3 + DVC | Large file versioning |
| Monitoring | Evidently + Grafana | Drift + performance |

---

## Case 6: Design a Vector Database

### Requirements

*   Store 1B 1536-dimensional embeddings.
*   Sub-100ms approximate nearest neighbour (ANN) search at p95.
*   Support metadata filtering (e.g., `category = "legal"`).

### Design

*   **Index:** HNSW (Hierarchical Navigable Small World) — O(log n) search, tunable recall.
*   **Metadata index:** Inverted index for categorical filters; pre-filter reduces search space.
*   **Sharding:** Shard by tenant or embedding cluster; distributed query fan-out.
*   **Replication:** 3× replicas for HA; reads served from any replica.
*   **Persistence:** WAL + periodic snapshots to S3.

---

## Case 7: Design a Feature Store

### Requirements

*   Serve features with < 10ms latency for online inference.
*   Provide consistent features for training (point-in-time correct).
*   Support 500 features, 50M entities.

### Dual-Store Pattern

```
Offline Store (S3 + Parquet)     Online Store (Redis / DynamoDB)
  - Historical feature values      - Latest feature values only
  - Used for training              - Used for inference
  - Point-in-time correct          - Low latency (< 5ms)
  - Batch read                     - Key-value lookup

Feature Pipeline → writes to BOTH stores simultaneously
```

---

## Case 8: Design an MLOps Platform for Model Lifecycle

### Lifecycle

```
[Experiment] → [Staging] → [Shadow] → [Canary 5%] → [Canary 50%] → [Production]
                                                                         │
                                                               [Monitor + Drift Alert]
                                                                         │
                                                               [Auto-retrain Trigger]
```

### Key Components

*   **CI/CD:** GitHub Actions → training pipeline → evaluation gate → model registry.
*   **Deployment:** Kubernetes + Helm charts for each model endpoint.
*   **Monitoring:** Evidently for drift; Grafana for latency; custom dashboards for business metrics.
*   **Rollback:** Keep last 3 production model versions in registry; one-click rollback.
