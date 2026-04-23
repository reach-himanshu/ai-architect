# 02 — Data Engineering for AI/ML

> **Goal:** Understand how data flows from raw sources to model-ready features — the
> plumbing that every AI architect must be able to design and defend in a system-design interview.

---

## Roadmap

```
02-data-engineering/
├── README.md
└── notebooks/
    ├── 01_batch_vs_streaming.md + .ipynb      ← ETL, batch windows, streaming semantics
    ├── 02_data_quality.md + .ipynb            ← Validation, profiling, anomaly detection
    ├── 03_schema_evolution.md + .ipynb        ← Avro/Protobuf, backward/forward compat
    ├── 04_orchestration.md + .ipynb           ← DAGs, Airflow patterns, retries, SLAs
    ├── 05_data_formats.md + .ipynb            ← Parquet, Delta Lake, Iceberg, Arrow
    └── 06_pipeline_patterns.md + .ipynb       ← Lambda, Kappa, medallion, feature pipelines
```

---

## Section 1 — Batch vs Streaming (Notebook 01)

*   ETL vs ELT; micro-batch as a bridge pattern
*   Watermarks and late-arriving events
*   Kafka as the universal event bus
*   Flink / Spark Structured Streaming concepts

## Section 2 — Data Quality (Notebook 02)

*   Schema validation with Great Expectations-style contracts
*   Statistical profiling (null rates, cardinality, distribution shifts)
*   Data quality dimensions: completeness, accuracy, consistency, timeliness
*   Quarantine vs fail-fast strategies

## Section 3 — Schema Evolution (Notebook 03)

*   Backward / forward / full compatibility
*   Avro schema registry pattern
*   Safe vs unsafe changes (adding nullable field vs renaming required field)
*   Protobuf field numbering rules

## Section 4 — Orchestration (Notebook 04)

*   DAG design principles: idempotency, atomicity, observability
*   Airflow patterns: sensors, XCom, branching, dynamic tasks
*   Retry strategies with exponential back-off
*   SLA monitoring and alerting

## Section 5 — Data Formats (Notebook 05)

*   Row vs column store; when each shines
*   Parquet internals: row groups, pages, predicate pushdown, dictionary encoding
*   Delta Lake / Iceberg: ACID transactions on object storage, time travel
*   Apache Arrow: zero-copy in-memory columnar format

## Section 6 — Pipeline Patterns (Notebook 06)

*   Lambda architecture (batch + speed layer)
*   Kappa architecture (streaming only, reprocessing via replaying Kafka)
*   Medallion architecture (Bronze → Silver → Gold)
*   Feature pipelines: point-in-time correctness, feature freshness SLAs

---

## Key Interview Concepts

| Topic | Key Answer |
|-------|-----------|
| Exactly-once semantics | Idempotent sinks + transactional Kafka producers |
| Late data handling | Watermark + allowed lateness + side output |
| Schema registry | Central schema versioning with compatibility checks |
| Idempotent ETL | Upsert by surrogate key + write-audit-publish pattern |
| Feature freshness | Max-age SLA + staleness alerts in feature store |
