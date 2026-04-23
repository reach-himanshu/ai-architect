# Key Concepts Cheatsheet

> Quick-reference tables for the 5 minutes before your interview.

---

## LLMs & RAG

| Concept | 1-Line Answer |
|---------|-------------|
| Tokenization | Split text into sub-word units; 1 token ≈ 4 chars / 0.75 words |
| Context window | Max tokens model can process at once; scales attention quadratically |
| Temperature | Controls randomness; 0 = greedy, 1 = more creative |
| Embedding | Dense vector representation of semantic meaning |
| Cosine similarity | Dot product of unit vectors; range [-1, 1]; 1 = identical |
| Chunking | Split docs into overlapping segments for retrieval |
| BM25 | Sparse lexical retrieval; TF-IDF variant; great for keyword match |
| HNSW | Graph-based ANN index; O(log n) search; tunable recall/latency |
| HyDE | Generate hypothetical doc → embed it → retrieve similar real docs |
| RRF | Reciprocal Rank Fusion; merge ranked lists: score = Σ 1/(k+rank_i) |
| RAG hallucination | LLM makes up info not in retrieved chunks; fix: faithfulness check |
| PSI | Population Stability Index; drift > 0.2 → retrain |
| Faithfulness | RAGAS metric: answer claims supported by retrieved context |
| ReAct | Reasoning + Acting: LLM generates thought → action → observation loop |
| LoRA | Low-Rank Adaptation: freeze base model, train small rank-r matrices |

---

## ML Fundamentals

| Concept | 1-Line Answer |
|---------|-------------|
| Bias-variance tradeoff | High bias = underfitting; high variance = overfitting |
| Regularisation | L1 (Lasso, sparsity), L2 (Ridge, weight decay) |
| AUC-ROC | Area under ROC curve; 0.5 = random, 1.0 = perfect; threshold-agnostic |
| F1 score | Harmonic mean of precision and recall; use when classes are imbalanced |
| Cross-validation | K-fold: train K-1 folds, validate on 1; repeat K times |
| Precision | TP / (TP + FP) — of predicted positives, how many are correct? |
| Recall | TP / (TP + FN) — of all positives, how many did we find? |
| Gradient descent | Iteratively step in direction of steepest loss reduction |
| Batch norm | Normalise layer inputs to zero mean, unit variance; accelerates training |
| Dropout | Randomly zero activations during training; reduces overfitting |
| Attention | Weighted sum of values; weight = softmax(QKᵀ/√d_k) |
| Transformer | Stack of self-attention + FFN blocks; scales with data + compute |

---

## System Design

| Concept | 1-Line Answer |
|---------|-------------|
| CAP theorem | Can have at most 2 of: Consistency, Availability, Partition Tolerance |
| ACID | Atomicity, Consistency, Isolation, Durability (relational DB guarantees) |
| BASE | Basically Available, Soft state, Eventually consistent (NoSQL) |
| Consistent hashing | Map nodes + keys to ring; node removal affects only adjacent keys |
| Vector clock | Tracks causality in distributed systems; detects concurrent events |
| Saga pattern | Distributed transaction via compensating actions per step |
| Token bucket | Rate limiter: tokens added at rate R, bucket capacity B; smooth bursts |
| Backpressure | Slow downstream signals slow upstream to avoid overload |
| Circuit breaker | Stop calling failing service after N errors; retry after timeout |
| Sidecar | Container in same pod providing cross-cutting concerns (logging, proxy) |
| CQRS | Command Query Responsibility Segregation: separate read/write models |
| Event sourcing | Store events not state; reconstruct state by replaying events |
| Two-phase commit | Distributed transaction protocol; coordinator → prepare → commit/abort |
| Gossip protocol | Nodes share state with random peers; eventually all nodes converge |

---

## Data Engineering

| Concept | 1-Line Answer |
|---------|-------------|
| Watermark | Timestamp below which system assumes all events have arrived |
| Exactly-once | Idempotent sink + transactional Kafka producer; prevents duplicates |
| Parquet | Columnar format; stores min/max stats per row group for predicate pushdown |
| Delta Lake | Adds ACID + time travel to Parquet via `_delta_log` transaction log |
| PSI (data) | Measures distribution shift; > 0.2 = significant; trigger retraining |
| Schema evolution | Backward compat: new schema reads old data; forward: old reads new |
| Medallion | Bronze (raw) → Silver (cleaned) → Gold (aggregated) |
| Point-in-time | Join features as they existed at label time; prevents train/serve skew |
| Write-Audit-Publish | Write to staging → validate → atomic swap to production |
| Kappa vs Lambda | Kappa: streaming only + replay; Lambda: batch + streaming merged |
| Feature store | Dual store: Redis (online, < 10ms) + S3 Parquet (offline, training) |

---

## Cloud & Production

| Concept | 1-Line Answer |
|---------|-------------|
| VRAM estimation | Params × bytes (FP16=2, INT8=1) × 1.2 overhead |
| Spot instance | 60–90% cheaper; can be preempted; use with checkpointing |
| Canary deploy | 5% → 20% → 100% traffic; rollback on metric degradation |
| Shadow deploy | Duplicate traffic to new model; log predictions; zero user risk |
| Blue/Green | Two full environments; instant switch; expensive but safe |
| HPA | K8s autoscaler; scale on CPU or custom metrics (RPS, queue depth) |
| KEDA | K8s Event Driven Autoscaling; scale on Kafka lag, SQS depth |
| Requests vs limits | Requests = K8s scheduling guarantee; limits = hard cap |
| Readiness probe | K8s removes pod from Service endpoints if not ready |
| Liveness probe | K8s restarts pod if not alive |
| PVC | PersistentVolumeClaim; attach durable storage to pods |
| FinOps | Cost-per-unit tracking + reserved instances + tagging enforcement |
| Prompt caching | Cache identical prompt prefixes; 90% cost reduction on static context |

---

## Numbers to Remember

| Metric | Value |
|--------|-------|
| Tokens in GPT-4o context | 128k |
| 1 token ≈ | 4 chars / 0.75 words |
| text-embedding-3-small dims | 1536 |
| Parquet row group default | 128 MB |
| PSI drift threshold | > 0.2 → retrain |
| HNSW search complexity | O(log n) |
| Spot instance savings | 60–90% |
| 1-year reserved savings | ~40% |
| GPU VRAM: FP16 7B model | ~14 GB |
| GPU VRAM: INT4 7B model | ~4 GB |
| Canary start percentage | 5% |
| Rate limit algorithm | Token bucket |
| Circuit breaker threshold | 5 errors in 10s (typical) |
| K8s rolling update default | maxSurge=1, maxUnavailable=0 |

---

## Interview Anti-Patterns to Avoid

*   ✗ Starting to design without asking requirements and constraints.
*   ✗ Choosing a technology without explaining why (say "I'd use X because...").
*   ✗ Giving a solution without mentioning trade-offs.
*   ✗ Saying "we" in behavioural stories — always "I".
*   ✗ Giving generic answers ("I believe in communication") — always specific.
*   ✗ Not quantifying results in STAR stories.
*   ✗ Designing for 1000× the stated scale — solve the stated problem first.
*   ✗ Ignoring failure modes — always discuss what happens when things go wrong.
