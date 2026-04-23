# Production Monitoring and Drift Detection

## Why Models Degrade in Production

A model trained today can become stale tomorrow:

*   **Data drift:** The statistical distribution of input features changes.
*   **Concept drift:** The relationship between features and labels changes.
*   **Label drift:** The distribution of the target variable changes.
*   **Infrastructure drift:** Changes in preprocessing, schema, or upstream data pipelines.

---

## 1. Types of Drift

| Type | What Changes | Example |
|------|-------------|---------|
| **Covariate drift** | P(X) changes | Users from new geography; app goes viral |
| **Concept drift** | P(Y\|X) changes | Fraud patterns change after security update |
| **Label drift** | P(Y) changes | Class imbalance shifts after product change |
| **Dataset shift** | Both P(X) and P(Y\|X) | Complete market disruption |

---

## 2. Statistical Drift Tests

### Population Stability Index (PSI)

For continuous features. Bins the distribution and compares:

```
PSI = Σ (A_i - E_i) × ln(A_i / E_i)
```

Thresholds: < 0.1 stable, 0.1–0.2 monitor, > 0.2 retrain.

### Kolmogorov-Smirnov Test

Compares the empirical CDFs of two samples. Returns a D-statistic and p-value.
Use for continuous features. P-value < 0.05 → significant drift.

### Chi-Squared Test

For categorical features. Compares observed vs expected category frequencies.

### Jensen-Shannon Divergence

Symmetric version of KL divergence. Bounded [0, 1]. Smooth version of PSI.

---

## 3. Model Performance Monitoring

### Ground Truth Availability

| Situation | Lag | Method |
|----------|-----|--------|
| Immediate label (click, fraud) | Seconds | Online evaluation |
| Delayed label (conversion, churn) | Days–weeks | Delayed evaluation |
| No label (unsupervised) | N/A | Proxy metrics (confidence scores) |

### Metrics to Monitor

| Metric | What it Catches |
|--------|---------------|
| Prediction distribution | Concept drift, calibration shift |
| Confidence / entropy | Model uncertainty increase |
| Business metric (CTR, revenue) | Downstream impact of drift |
| Feature importance shift | Which features driving predictions changed |
| Accuracy / F1 (when labels available) | Direct performance degradation |

---

## 4. Alerting Strategy

### Layered Alerts

```
Layer 1 (Immediate):  Data pipeline failure → alert within 1 min
Layer 2 (Fast):       Prediction rate anomaly → alert within 5 min
Layer 3 (Hourly):     Feature drift PSI check → alert on threshold breach
Layer 4 (Daily):      Model accuracy on delayed labels → alert if drops > 5%
```

### Alert Fatigue Prevention

*   Use exponential smoothing on metrics before alerting (ignore spikes).
*   Require drift to persist for N consecutive windows before alerting.
*   Distinguish severity: INFO (log only), WARNING (Slack), CRITICAL (PagerDuty).

---

## 5. Observability Stack for ML

```
Model Server
    │  predictions + features + metadata
    ▼
Feature/Prediction Logger (Kafka topic)
    │
    ├──► Real-time Dashboard (Grafana)
    │         • Prediction rate
    │         • Confidence distribution
    │
    └──► Drift Detector (hourly batch job)
              • PSI per feature
              • KS test on predictions
              • Label drift when labels arrive
                   │
                   ├──► Alert system (Slack, PagerDuty)
                   └──► Retraining trigger
```

---

## 6. Monitoring Checklist

```
Data Quality
 [ ] Schema validation on every inference request
 [ ] Null rate per feature within baseline ± threshold
 [ ] Feature value range checks (min/max)

Distribution Monitoring
 [ ] PSI computed hourly for top N features
 [ ] Prediction score distribution tracked daily
 [ ] KS test for continuous feature drift

Model Performance
 [ ] Accuracy / F1 computed when labels arrive
 [ ] Business metrics (CTR, conversion) tracked alongside model metrics
 [ ] Confidence calibration checked monthly

Infrastructure
 [ ] Inference latency p50/p95/p99 per endpoint
 [ ] Error rate (5xx) per endpoint
 [ ] Throughput (requests/sec)
```

---

## Key Interview Points

*   **Covariate vs concept drift:** Covariate drift = features change but relationship to label is same. Concept drift = same features now predict different outcomes. Concept drift is harder to detect without fresh labels.
*   **PSI threshold:** Explain the 0.1/0.2 rule and that you use it on the top-K most important features (from feature importance) rather than all features.
*   **Delayed labels:** For churn prediction (30-day lag), explain how you buffer predictions and join them with ground truth when labels arrive, computing rolling accuracy metrics.
*   **Proxy metrics:** When labels aren't available, use model confidence/entropy as a proxy. Sudden increase in low-confidence predictions suggests distribution shift.
