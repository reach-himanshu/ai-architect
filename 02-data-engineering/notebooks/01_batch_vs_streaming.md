# Batch vs Streaming Data Processing

## The Core Distinction

| Dimension | Batch | Streaming |
|----------|-------|-----------|
| Trigger | Scheduled (hourly, daily) | Continuous / event-driven |
| Latency | Minutes to hours | Milliseconds to seconds |
| Throughput | Very high | Moderate |
| Complexity | Lower | Higher (state, ordering) |
| Cost model | Burst compute | Always-on (or micro-burst) |
| Use cases | Nightly reports, model training | Fraud detection, recommendations |

---

## 1. ETL vs ELT

**ETL (Extract → Transform → Load)**

*   Classic data warehouse pattern.
*   Transform happens in a staging layer before loading to destination.
*   Transformation logic owned by data engineers.

**ELT (Extract → Load → Transform)**

*   Load raw data first, transform later in-place (BigQuery, Snowflake, Databricks).
*   Enables ad-hoc queries on raw data.
*   dbt is the dominant transformation tool.

**Micro-batch (Bridge Pattern)**

*   Run a streaming job but process in small time windows (e.g., every 30s).
*   Spark Structured Streaming default mode.
*   Lower operational complexity than true streaming; latency ~30s.

---

## 2. Streaming Semantics

### Delivery Guarantees

| Guarantee | Description | How |
|----------|-------------|-----|
| At-most-once | May lose messages | Don't retry; fire and forget |
| At-least-once | May duplicate | Retry + idempotent sink |
| Exactly-once | No loss, no duplicate | Idempotent + transactional (Kafka 0.11+) |

### Watermarks and Late Data

*   **Event time:** When the event actually occurred (encoded in payload).
*   **Processing time:** When the system processes it.
*   **Watermark:** A timestamp `W` such that the system assumes all events with `event_time < W` have arrived.
*   **Allowed lateness:** Accept events up to `L` seconds after the watermark; update window result.
*   **Side output:** Route events later than allowed lateness to a dead-letter topic for reprocessing.

### Window Types

| Window | Description | Example |
|--------|-------------|---------|
| Tumbling | Fixed, non-overlapping | Hourly revenue |
| Sliding | Fixed size, moves every step | 1-hr window every 15 min |
| Session | Dynamic, gap-based | User session (30 min inactivity) |
| Global | One window, emit on trigger | Aggregate until count=100 |

---

## 3. Kafka as the Universal Event Bus

*   **Topics:** Named, partitioned, ordered log.
*   **Partitions:** Unit of parallelism; consumer group members each own partitions.
*   **Offset:** Position of a consumer in a partition; committed to `__consumer_offsets`.
*   **Retention:** Data kept for configurable duration (default 7 days) — enables replaying.
*   **Consumer group:** Multiple consumers reading a topic in parallel, each partition owned by one consumer.

### Why Kafka for ML?

*   Feature pipelines publish features as events.
*   Model predictions published back to Kafka for downstream consumers.
*   Replay capability: retrain models by replaying historical events.
*   Acts as the "source of truth" in a Kappa architecture.

---

## 4. Batch Pipeline Design

### Idempotent Batch Jobs

*   Write to a dated partition: `s3://bucket/date=2025-01-15/`.
*   Re-running the same date overwrites the partition → idempotent.
*   Use `INSERT OVERWRITE` in SQL engines; avoid `APPEND` without deduplication.

### Write-Audit-Publish Pattern

```
1. Write → temp/staging location
2. Audit → validate row count, schema, null rates
3. Publish → swap pointer to prod location (atomic rename / table swap)
```

Guarantees consumers never see partial data.

---

## 5. Choosing Batch vs Streaming

**Use batch when:**

*   Business needs reports on a schedule (daily, hourly).
*   Transformations require full dataset (global sorts, exact deduplication).
*   Cost is a constraint (burst compute much cheaper than always-on).

**Use streaming when:**

*   Latency SLA < 5 minutes.
*   Fraud detection, anomaly alerts, live dashboards.
*   Event-driven architecture where downstream consumers are also streaming.

**Use micro-batch when:**

*   5–60 second latency is acceptable.
*   Team has Spark expertise but not Flink.
*   Stateful operations (windowed aggregations) are needed.

---

## Key Interview Points

*   **Exactly-once**: Combine idempotent writes (upsert by unique key) with Kafka transactions. Without both, you get at-least-once.
*   **Watermark choice**: Too tight → drop valid late events. Too loose → high memory usage. Tune based on SLA and observed data lateness distribution.
*   **Backpressure**: When a sink is slow, a streaming system must slow the source or buffer. Kafka back-pressure is implicit — consumers just fall behind on offsets. Flink has credit-based backpressure.
