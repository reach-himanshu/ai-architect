# Cloud 05: Storage for ML

## 1. Object Storage (S3 / GCS / Azure Blob)

The backbone of ML data storage. Unlimited scale, cheap, durable (11 nines).

```
Raw data → s3://bucket/raw/
Processed → s3://bucket/processed/YYYY-MM-DD/
Features  → s3://bucket/features/v2/
Models    → s3://bucket/models/{model_name}/{version}/
Artifacts → s3://bucket/artifacts/{run_id}/
```

**Key patterns**:
*   Partition by date/category for efficient queries.
*   Use lifecycle policies to move old data to cheaper tiers (Glacier, Coldline).
*   Enable versioning for audit trails.

---

## 2. Data Formats for ML

| Format | Type | Compression | Schema | Best For |
| :--- | :--- | :--- | :--- | :--- |
| **CSV** | Row | None/gzip | None | Small data, interop |
| **JSON / JSONL** | Row | None/gzip | None | Logs, flexibility |
| **Parquet** | Columnar | snappy/zstd | Yes | Analytics, training |
| **Avro** | Row | Various | Yes (evolution) | Kafka, streaming |
| **TFRecord** | Binary | gzip | Proto | TensorFlow training |
| **Arrow/Feather** | Columnar | LZ4 | Yes | In-memory ML |
| **HDF5** | Hierarchical | Yes | Yes | Scientific data |

**For ML training**: **Parquet** is the default. Columnar = only read needed columns; push-down filters skip rows early.

---

## 3. Data Lake Architecture

```
Bronze (Raw):   Exact copy of source, no transforms. Immutable.
Silver (Clean): Deduplicated, validated, typed. Quality checks passed.
Gold (Curated): Aggregated, feature-engineered, model-ready.
```

**Delta Lake / Apache Iceberg**: Add ACID transactions, schema evolution, and time-travel to Parquet on S3.

---

## 4. Feature Stores: Online vs. Offline

| | Online Store | Offline Store |
| :--- | :--- | :--- |
| **Backend** | Redis, DynamoDB, Cassandra | S3 + Parquet, BigQuery |
| **Latency** | < 10ms | Minutes |
| **Use case** | Real-time serving | Training dataset generation |
| **Point-in-time** | Current value only | Historical with time-travel |

**Critical**: Training-serving skew happens when online and offline stores differ. Feature stores prevent this.

---

## 5. Dataset Versioning

Use **DVC** (Data Version Control) or **Delta Lake** to version datasets:

```bash
dvc add data/training.parquet     # Track file
git add data/training.parquet.dvc # Commit pointer to git
dvc push                          # Upload to S3
```

*   Reproducibility: Any model can be traced back to the exact dataset.
*   Rollback: Revert to previous dataset if new one causes regression.
*   Lineage: Know which models were trained on which data.
