# Data Formats for AI/ML

## Row vs Column Storage

| Aspect | Row Store (CSV, JSON) | Column Store (Parquet, ORC) |
|--------|----------------------|----------------------------|
| Write pattern | Append one row | Write column chunks |
| Read pattern | Full row fetch | Scan selected columns only |
| Compression | Low (mixed types) | High (same types together) |
| Analytics queries | Slow | Fast (predicate pushdown) |
| ML training | Sequential OK | Better for feature selection |
| Streaming | Natural | Unnatural (batch-oriented) |

---

## 1. Parquet Internals

### Physical Layout

```
File
├── Row Group 0 (128MB default)
│   ├── Column Chunk: user_id  [Dictionary encoded + Snappy compressed]
│   ├── Column Chunk: age      [Delta encoded + Snappy]
│   └── Column Chunk: revenue  [Plain + Snappy]
├── Row Group 1
│   └── ...
└── Footer (column stats, schema, row group offsets)
```

### Key Optimisations

**Predicate Pushdown:** Footer stores min/max per column per row group.
Query `WHERE age > 60` skips entire row groups where `max(age) ≤ 60`.

**Dictionary Encoding:** For low-cardinality columns (e.g., `status ∈ {active, inactive}`),
store a dictionary and replace values with integer indices. Massive compression.

**Bloom Filter:** Optional per-column bloom filter to skip row groups for equality predicates.

### Parquet vs ORC vs Avro

| Format | Best For | Encoding | Splittable |
|--------|---------|---------|-----------|
| Parquet | Analytics, ML | Column | ✓ |
| ORC | Hive/Hudi | Column | ✓ |
| Avro | Kafka events, row-level | Row | ✓ (with sync markers) |
| JSON | APIs, debugging | Row | ✗ (usually) |
| CSV | Simple interchange | Row | ✓ (by line) |

---

## 2. Delta Lake / Apache Iceberg

### The Problem They Solve

Object storage (S3/GCS) has no ACID transactions. Without it:

*   Reading while writing → partial data visible.
*   Failed jobs leave orphan files.
*   No schema evolution guarantees.
*   No rollback.

### Delta Lake Architecture

```
s3://bucket/table/
├── _delta_log/
│   ├── 00000000000000000000.json   ← transaction 0: CREATE TABLE
│   ├── 00000000000000000001.json   ← transaction 1: INSERT
│   ├── 00000000000000000002.json   ← transaction 2: UPDATE (copy-on-write)
│   └── 00000000000000000010.checkpoint.parquet
└── part-00001.parquet
    part-00002.parquet
    ...
```

### Key Delta Lake Features

| Feature | How |
|---------|-----|
| ACID | Optimistic concurrency on `_delta_log`; last writer wins with conflict check |
| Time Travel | Read `VERSION AS OF 5` or `TIMESTAMP AS OF '2025-01-01'` |
| Schema Evolution | Merge schema on write (opt-in) |
| Vacuum | Remove files older than N hours not referenced by any version |
| Z-Order | Multi-column data skipping — co-locate related data |

### Iceberg vs Delta Lake vs Hudi

| Feature | Delta Lake | Apache Iceberg | Apache Hudi |
|---------|-----------|---------------|------------|
| Origin | Databricks | Netflix | Uber |
| Time Travel | ✓ | ✓ | ✓ |
| ACID | ✓ | ✓ | ✓ |
| Row-level deletes | ✓ (GDPR) | ✓ | ✓ (native) |
| Ecosystem | Spark-native | Multi-engine | Spark-native |
| Catalog | Delta catalog | Any catalog (Hive, Glue) | Hive metastore |

---

## 3. Apache Arrow

### In-Memory Columnar Format

Arrow provides a language-agnostic, zero-copy columnar memory layout:

*   Same binary format in Python, Java, C++, Rust.
*   Pandas, Spark, DuckDB all natively read/write Arrow.
*   Zero-copy: pass a pointer, not a copy.

### Why Arrow Matters for ML

*   Feature stores return Arrow tables → zero-copy into PyTorch/NumPy.
*   Parquet → Arrow is optimised (Arrow is Parquet's in-memory sibling).
*   IPC (inter-process communication) between model servers and preprocessors.

---

## 4. Format Selection Guide

| Scenario | Format |
|---------|--------|
| Kafka events | Avro (with schema registry) |
| Batch feature store | Parquet on S3/GCS |
| GDPR-compliant table (row deletes) | Delta Lake or Iceberg |
| Data science exploration | Parquet or CSV |
| API response | JSON |
| High-throughput ML training | Parquet + Arrow |
| Streaming aggregations → OLAP | Iceberg (Flink sink) |

---

## Key Interview Points

*   **Predicate pushdown:** Explain min/max statistics in Parquet footer allow skipping row groups without reading data — critical for query performance on large datasets.
*   **Delta Lake vs plain Parquet:** Delta adds `_delta_log` for ACID; plain Parquet has no atomicity guarantees.
*   **GDPR row deletes:** Delta Lake and Iceberg support `DELETE FROM table WHERE user_id = 'xxx'` without rewriting the entire table — achieved via copy-on-write or merge-on-read.
*   **Arrow zero-copy:** In a production feature serving system, returning Arrow IPC buffers avoids serialisation overhead between the feature store and the model server.
