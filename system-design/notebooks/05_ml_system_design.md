# System Design 05: ML System Design

## 1. The ML System Map

```
Data Sources → Feature Pipeline → Feature Store
                                       ↓
           Experiment Tracking ← Model Training
                                       ↓
                              Model Registry
                                       ↓
              Shadow Mode → Canary → Production Serving
                                       ↓
                     Monitoring → Alerting → Retraining
```

## 2. Feature Stores

A feature store centralizes feature computation and ensures training-serving consistency.

| Without Feature Store | With Feature Store |
| :--- | :--- |
| Features recomputed in every pipeline | Compute once, serve everywhere |
| Training-serving skew is common | Same code at train and serve time |
| Hard to share across teams | Feature discovery, reuse |
| No point-in-time correctness | Built-in time-travel for training |

Popular options: **Feast** (open source), **Tecton** (managed), **Vertex Feature Store**.

---

## 3. Model Registry

Tracks for every model version:
*   Artifact location (weights, code, config)
*   Metrics (accuracy, latency, F1)
*   Lineage (data version, training run, git commit)
*   Stage: `staging → production → archived`

Popular options: **MLflow**, **Weights & Biases**, **SageMaker Model Registry**.

---

## 4. Model Serving Patterns

| Pattern | Latency | Throughput | Use Case |
| :--- | :--- | :--- | :--- |
| **Online (real-time)** | ms | Low–Medium | User-facing predictions |
| **Batch (offline)** | Hours | Very High | Pre-compute recommendations |
| **Streaming** | Seconds | Medium | Near-real-time fraud |
| **Edge** | ms | Low | Mobile, IoT |

---

## 5. Safe Deployment Strategies

**Shadow Mode**: New model runs alongside production, output discarded. Zero user impact. Catch bugs early.

**Canary**: Route 5% → 25% → 100% of traffic progressively. Rollback automatically if metrics drop.

**Champion-Challenger**: A/B test between current champion and new challenger with statistical significance.

**Blue-Green**: Two identical environments. Instant traffic cutover. Easy rollback.

---

## 6. Drift Detection

| Type | Description | Detection Method |
| :--- | :--- | :--- |
| **Data drift** | Input distribution changes | PSI, KS test, Jensen-Shannon divergence |
| **Concept drift** | X→Y relationship changes | Monitor output distribution, accuracy |
| **Model drift** | Accuracy degrades over time | Sliding window performance metrics |
| **Upstream drift** | Pipeline feeding model changes | Schema validation, data contracts |

**Trigger retraining when**: PSI > 0.2 on a key feature, or prediction accuracy drops > 5%.
