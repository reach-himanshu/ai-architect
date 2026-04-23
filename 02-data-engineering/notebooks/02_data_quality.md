# Data Quality for AI/ML

## Why Data Quality is Critical

"Garbage in, garbage out" is the oldest ML proverb. A model is only as good as its
training data. Data quality issues compound — a 5% error rate in training data
can translate to a 20%+ degradation in model performance.

---

## 1. Data Quality Dimensions

| Dimension | Definition | Measurement |
|----------|-----------|-------------|
| **Completeness** | Required fields are populated | Null rate per column |
| **Accuracy** | Values are correct | Validation against reference |
| **Consistency** | No contradictions across systems | Cross-system row counts, FK integrity |
| **Timeliness** | Data is fresh enough | `now() - max(event_time)` |
| **Uniqueness** | No unexpected duplicates | Duplicate rate on primary key |
| **Validity** | Values conform to rules | Range checks, enum checks, regex |

---

## 2. Schema Validation

### Contract Testing

Define expectations on a dataset before processing:

```python
# Great Expectations style (conceptual)
expect_column_to_exist("user_id")
expect_column_values_to_not_be_null("user_id", mostly=1.0)
expect_column_values_to_be_between("age", min_value=0, max_value=120)
expect_column_values_to_match_regex("email", r"^[^@]+@[^@]+\.[^@]+$")
expect_column_values_to_be_in_set("status", ["active", "inactive", "pending"])
```

### Schema Registry Validation

*   Compare incoming schema against the registered schema version.
*   Reject data that adds required fields (backward incompatible).
*   Warn on unexpected columns (schema drift).

---

## 3. Statistical Profiling

### Column-Level Metrics

| Metric | When to Alert |
|--------|-------------|
| Null rate | > threshold (e.g., > 5%) |
| Mean / stddev | Shift > 3σ from historical baseline |
| Min / max | Outside expected range |
| Cardinality | Unexpected spike or drop |
| Distribution | KL-divergence > threshold vs reference |

### Dataset-Level Metrics

*   **Row count:** ±20% from 7-day moving average → alert.
*   **Byte size:** Sudden 10× increase → possible data explosion.
*   **Freshness lag:** `now() - max(event_time)` > SLA → alert.

---

## 4. Anomaly Detection Strategies

### Z-Score (Simple)

Flag a metric value `x` if `|x - μ| / σ > k` where `k` is typically 3.

### IQR Method (Robust to Outliers)

Flag if `x < Q1 - 1.5·IQR` or `x > Q3 + 1.5·IQR`.

### Population Stability Index (PSI)

Measure distribution drift between training baseline and production:

```
PSI = Σ (actual% - expected%) × ln(actual% / expected%)
```

| PSI | Interpretation |
|-----|---------------|
| < 0.1 | No significant change |
| 0.1–0.2 | Moderate change — monitor |
| > 0.2 | Significant shift — investigate |

---

## 5. Handling Bad Data

### Strategies

| Strategy | When to Use | Trade-off |
|---------|-------------|----------|
| **Fail fast** | Hard schema violations | Prevents propagation; blocks pipeline |
| **Quarantine** | Soft violations (nullable) | Pipeline continues; manual review |
| **Impute** | Missing numeric values | Introduces bias if rate > 5% |
| **Drop rows** | Non-recoverable corruption | Loses data; OK if < 1% |
| **Alert + continue** | Late data within SLA | Downstream may use stale data |

### Dead Letter Queue (DLQ) Pattern

*   Route bad records to a DLQ topic/table.
*   Retain original payload + validation errors + timestamp.
*   Enable replay after upstream fix.

---

## 6. Data Quality in ML Pipelines

### Training Data Validation

Before training starts:

1.  Validate schema matches expected feature schema.
2.  Check label distribution — class imbalance > 100:1 needs handling.
3.  Run statistical profile; compare to previous training run.
4.  Check for train/test leakage (future data in training window).

### Serving Data Validation

At inference time:

1.  Validate input schema against training schema.
2.  Alert on feature drift (PSI > 0.2).
3.  Flag out-of-range features as "uncertain" predictions.

---

## Key Interview Points

*   **PSI for drift detection:** Explain the formula and thresholds. Note it requires binning continuous features.
*   **DLQ pattern:** Never silently drop bad data. Always route to a DLQ with error context for auditability.
*   **Train/serve skew:** Feature computation differs between training (batch) and serving (online). Use a shared feature computation library to avoid skew.
*   **Great Expectations:** Mention as the industry standard for declarative data quality contracts.
