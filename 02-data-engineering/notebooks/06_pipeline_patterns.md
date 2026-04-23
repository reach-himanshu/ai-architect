# Data Pipeline Patterns

## Overview

Every data architecture is some variation on a small set of canonical patterns.
Understanding these patterns — and their trade-offs — is essential for system design interviews.

---

## 1. Lambda Architecture

### Structure

```
Source Events
    │
    ├──► Batch Layer (Hadoop/Spark)
    │    • Processes all historical data
    │    • High latency (hours)
    │    • Accurate, complete
    │         │
    │         ▼ Batch Views
    │
    ├──► Speed Layer (Flink/Spark Streaming)
    │    • Processes recent data only
    │    • Low latency (seconds)
    │    • Approximate
    │         │
    │         ▼ Real-time Views
    │
    └──► Serving Layer
         • Merges batch + real-time views
         • Answers queries
```

### Trade-offs

| Advantage | Disadvantage |
|----------|-------------|
| Accurate historical queries | Dual codebase (batch + streaming) |
| Low latency for recent data | Complex merge logic |
| Fault tolerant | Two systems to operate and debug |

---

## 2. Kappa Architecture

### Structure

```
Source Events → Kafka (long retention) → Streaming Job → Serving Layer
                     │
                     └──► Reprocess historical data by replaying Kafka
```

### Key Insight

By keeping Kafka data long enough (weeks/months), you can replay it through an
updated streaming job to reprocess historical data — eliminating the need for a
separate batch layer.

### Trade-offs

| Advantage | Disadvantage |
|----------|-------------|
| Single codebase | Kafka storage cost for long retention |
| Simpler ops | Reprocessing slower than batch |
| Real-time + historical unified | Stateful jobs harder to scale |

---

## 3. Medallion / Delta Architecture

### Structure

```
Raw Sources → Bronze (raw, immutable) → Silver (cleaned, conformed) → Gold (aggregated, domain-specific)
```

| Layer | Contents | Audience |
|-------|---------|---------|
| **Bronze** | Raw data, no transformation; append-only | Data engineers |
| **Silver** | Deduplicated, validated, joined across sources | Analysts, data scientists |
| **Gold** | Aggregated business metrics, ML feature tables | Business, ML models |

### Principles

*   **Bronze is immutable:** Never modify raw data. Re-run silver/gold by reprocessing bronze.
*   **Schema-on-read (bronze) → schema-on-write (gold):** Bronze accepts any data; gold has strict schema.
*   **Incremental processing:** Process only new/changed records using Delta Lake merge or watermarks.

---

## 4. Feature Pipelines for ML

### Point-in-Time Correctness

The #1 source of train/serve skew in ML systems. When training:

*   Use the feature value as it existed **at the time of the label**, not the current value.
*   Example: For a model trained on purchases, use the user's purchase count at `purchase_time`, not today's count.

```
Timeline:
  t=0: user signs up
  t=5: purchase event (label)  ← feature values from t=5 used for training
  t=10: user buys more        ← this would be train/serve skew if used
```

### Feature Freshness SLA

| Feature Type | Acceptable Lag |
|-------------|---------------|
| Real-time features (recent clicks) | < 1 minute |
| Near-real-time (session features) | 1–15 minutes |
| Batch features (user history) | 1–24 hours |
| Slowly-changing (demographics) | Days–weeks |

### Feature Pipeline Architecture

```
Raw Events → Streaming Feature Pipeline → Online Store (Redis) → Model Serving
                                        → Offline Store (S3/Parquet) → Training
```

---

## 5. Pattern Selection Guide

| Requirement | Recommended Pattern |
|------------|-------------------|
| < 5s latency + historical accuracy | Lambda |
| Single codebase preferred | Kappa |
| Incremental layered data lake | Medallion |
| ML feature serving | Feature Pipeline |
| GDPR + row-level deletes | Delta Lake / Iceberg table |
| Event replay / reprocessing | Kappa + Kafka |

---

## 6. Anti-Patterns

| Anti-Pattern | Description | Fix |
|-------------|-------------|-----|
| Spaghetti pipelines | Too many point-to-point pipelines, no clear lineage | Medallion architecture |
| Hot path overload | All logic in streaming layer | Offload aggregations to batch |
| Training-serving skew | Different feature computation paths | Shared feature library |
| No data lineage | Can't trace where data came from | Column-level lineage tracking |
| Gold without Silver | Direct raw → gold transformations | Always have intermediate Silver |

---

## Key Interview Points

*   **Lambda vs Kappa:** Lambda gives you accurate results faster (batch layer). Kappa simplifies operations at the cost of Kafka storage. Kappa is the modern default when reprocessing latency is acceptable.
*   **Medallion Bronze immutability:** Raw data is your audit trail. Explain that you can always rebuild Silver/Gold from Bronze, but you can't recover Bronze if you overwrite it.
*   **Point-in-time joins:** Explain that a feature store uses a time-indexed join — for each label event at time T, look up the feature value as it was at time T, not as it is now.
*   **Feature freshness:** Distinguish between online features (Redis, low latency) and offline features (S3, high throughput). Explain the dual-store pattern.
