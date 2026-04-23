# CI/CD for ML (MLOps)

## Why ML CI/CD is Different from Software CI/CD

In traditional software, CI/CD validates code. In ML:

*   **Code + Data + Model** must all be versioned and validated.
*   A passing unit test doesn't mean the model performance is acceptable.
*   Model quality degrades over time even without code changes (data drift).
*   Experiments are exploratory; only the best get promoted to production.

---

## 1. ML Pipeline Stages

```
Code Commit / Data Change
       │
       ▼
  ① Unit Tests (code)
       │
       ▼
  ② Data Validation (schema, stats)
       │
       ▼
  ③ Model Training (experiment tracking)
       │
       ▼
  ④ Model Evaluation (metrics gate)
       │
       ▼
  ⑤ Shadow Deployment (traffic split)
       │
       ▼
  ⑥ Canary Rollout (5% → 20% → 100%)
       │
       ▼
  ⑦ Full Production + Monitoring
```

---

## 2. Model Versioning and Registry

### What to Version

| Artifact | Versioning Approach |
|---------|-------------------|
| Training code | Git commit SHA |
| Training data | Dataset version in feature store |
| Hyperparameters | Logged to MLflow / W&B |
| Model weights | Model registry (MLflow, SageMaker Registry) |
| Evaluation metrics | Stored alongside model version |
| Serving config | Helm chart / deployment manifest |

### Model Registry Lifecycle

```
Staging → [Evaluation Gate] → Production → Archived
```

*   **Staging:** New model awaiting evaluation.
*   **Production:** Currently serving traffic.
*   **Archived:** Retired; kept for rollback.

---

## 3. Evaluation Gates

### Absolute Gate

Model must achieve minimum threshold:

```python
assert precision >= 0.85, "Precision below threshold"
assert recall >= 0.80,    "Recall below threshold"
assert latency_p99_ms <= 200, "Latency SLA violated"
```

### Challenger Gate (Champion vs Challenger)

New model must beat the current production model by ≥ N%:

```python
improvement = (new_auc - prod_auc) / prod_auc
assert improvement >= 0.01, f"Model improvement {improvement:.2%} < 1% threshold"
```

### Slice Evaluation

Test across demographic/regional slices to catch unfairness:

*   Ensure metric doesn't degrade > 5% on any slice vs. baseline.

---

## 4. Shadow and Canary Deployments

### Shadow Deployment

*   New model receives a copy of production traffic.
*   Predictions are logged but not served to users.
*   Compare shadow predictions to production predictions and ground truth.
*   Zero user risk.

### Canary Deployment

```
100% → Prod Model
  5% → New Model  (monitor metrics)
       ↓
 20% → New Model  (if metrics OK)
       ↓
100% → New Model  (full rollout)
```

*   Use feature flags or load balancer weights.
*   Monitor business metrics (conversion, revenue) not just ML metrics.
*   Automated rollback if error rate > threshold.

---

## 5. Automated Retraining Triggers

| Trigger | Condition |
|---------|----------|
| Schedule | Every 7 days regardless |
| Data drift | PSI > 0.2 on key features |
| Performance drift | Accuracy drops > 5% from baseline |
| Data volume | New training data > 10% of current training set |
| Manual | On-demand by data scientist |

---

## 6. MLOps Maturity Model

| Level | Description |
|-------|-------------|
| 0 | Manual training in notebooks, manual deployment |
| 1 | ML pipeline automation; manual model deployment |
| 2 | Automated training + evaluation + deployment; CI/CD for ML |
| 3 | Full self-healing ML system with automated retraining and monitoring |

---

## Key Interview Points

*   **Champion/challenger:** Explain that new models are A/B tested against the current production model before full rollout. A performance improvement threshold (e.g., 1% AUC) guards against random noise.
*   **Shadow mode:** The safest deployment technique — zero user impact, full production traffic. Use for high-risk model changes.
*   **Automated retraining:** Triggered by drift detection, not just schedule. Explain PSI threshold and the pipeline that runs from data validation → training → evaluation → gated promotion.
*   **Model registry:** Describe staging → production → archived lifecycle with approval gates. Contrast with ad-hoc file sharing.
