# Cloud Cost Optimization for AI/ML

## Why Cost Optimization Matters

AI/ML workloads are notoriously expensive. A single LLM fine-tuning run can cost
thousands of dollars; always-on inference endpoints can run $10k+/month. FinOps
for AI is a first-class engineering discipline — not an afterthought.

---

## 1. Compute Cost Levers

### Reserved / Committed-Use Discounts

| Commitment | AWS Saving | GCP Saving | Azure Saving |
|-----------|-----------|-----------|-------------|
| 1-year RI/CUD | ~40% | ~37% | ~36% |
| 3-year RI/CUD | ~60% | ~55% | ~52% |
| Spot/Preemptible | ~70-90% | ~60-91% | ~60-80% |

**Rule of thumb:** Use reserved capacity for baseline load, spot for burst/training.

### Spot / Preemptible Strategy

*   Use spot for stateless, fault-tolerant workloads (batch inference, training).
*   Checkpoint every N steps so preemption loses at most N steps of work.
*   Use Spot Instance Advisor (AWS) or spot history to pick stable instance types.
*   Fall back to on-demand if spot unavailable — avoid job stalls.

### Rightsizing

*   Profile GPU utilisation with `nvidia-smi` or cloud-native metrics.
*   Target ≥ 70% GPU memory utilisation; if < 40%, downsize instance.
*   Use CPU instances for light preprocessing; don't pay for GPU idle time.

---

## 2. Inference Cost Optimisation

### Model Quantisation

| Precision | Memory | Throughput | Quality Loss |
|----------|--------|------------|-------------|
| FP32 | Baseline | 1× | None |
| FP16 / BF16 | 50% | 1.5–2× | Negligible |
| INT8 | 25% | 2–3× | Slight |
| INT4 (GPTQ/AWQ) | 12.5% | 3–4× | Moderate |

### Response Caching

*   **Exact cache:** hash(prompt) → cached response. Hit rate 5–30% for FAQ-style apps.
*   **Semantic cache:** embed prompt, nearest-neighbour lookup below threshold. Higher hit rate but adds latency.
*   **Prompt caching (provider-level):** Anthropic, OpenAI cache prefix tokens automatically.

### Batching

*   Batch small requests together to amortise GPU kernel launch overhead.
*   Dynamic batching: wait up to T ms or B requests, whichever comes first.
*   Continuous batching (vLLM): adds new requests mid-flight without waiting for all to finish.

### Prompt Optimisation

*   Measure token count of system prompts; trim by 20% = 20% cost cut.
*   Move static context to a cached prefix.
*   Use smaller models for routing/classification, larger only for generation.

---

## 3. Storage Cost Optimisation

### Data Lifecycle Tiering

```
Hot (S3 Standard / GCS Standard)    — Active training data
↓ 30 days
Warm (S3 Intelligent-Tiering)       — Recent experiment artifacts
↓ 90 days
Cold (S3 Glacier Instant Retrieval) — Checkpoints for compliance
↓ 1 year
Archive (S3 Deep Archive)           — Long-term model lineage
```

### Compression & Format

*   Parquet + Snappy: ~70% smaller than CSV, 10× faster queries.
*   Use Delta Lake / Iceberg for versioned datasets — avoid full re-uploads.
*   Deduplicate embeddings: store once, reference by ID.

---

## 4. FinOps Practices

### Tagging Strategy

Every cloud resource should carry:

| Tag | Example | Purpose |
|-----|---------|---------|
| `project` | `rag-search` | Chargebacks per product |
| `environment` | `prod` / `dev` | Identify waste in dev |
| `team` | `ml-platform` | Team-level budgets |
| `cost-center` | `CC-4412` | Finance allocation |
| `owner` | `alice@corp.com` | Accountability |

### Budget Alerts

*   Set budgets at 50%, 80%, 100% of monthly target → Slack / PagerDuty alerts.
*   Use anomaly detection (AWS Cost Anomaly Detection, GCP Billing Budgets).
*   Review weekly: kill zombies (stopped instances still paying for attached volumes).

### Unit Economics

Track cost-per-outcome, not just total spend:

*   **Cost per 1k API calls** (inference service)
*   **Cost per training token** (fine-tuning)
*   **Cost per GB processed** (data pipeline)
*   **Cost per active user** (product-level FinOps)

---

## 5. Multi-Region & Egress Costs

*   Data egress is often hidden and expensive (~$0.08–0.09/GB on AWS).
*   Keep training data in same region as compute.
*   Use CDN (CloudFront, Cloudflare) to cache model responses at edge.
*   Prefer private endpoints (VPC endpoints) — zero egress cost.

---

## 6. Cost Optimisation Checklist

```
Compute
 [ ] Reserved instances for production inference
 [ ] Spot instances for batch training + checkpointing
 [ ] GPU utilisation > 70%; rightsized instance type

Inference
 [ ] Response caching (exact + semantic)
 [ ] Dynamic / continuous batching enabled
 [ ] Quantised model (INT8 or INT4) evaluated
 [ ] Prompt length minimised

Storage
 [ ] Lifecycle rules configured on all S3/GCS buckets
 [ ] Parquet + compression for all datasets
 [ ] Orphaned snapshots / volumes cleaned up

FinOps
 [ ] All resources tagged with project, team, env
 [ ] Budget alerts at 50%/80%/100%
 [ ] Unit cost metrics tracked in dashboard
 [ ] Weekly cost review meeting
```

---

## Key Interview Points

*   **Spot checkpointing:** Explain that you checkpoint every N steps and use an SQS queue to re-enqueue interrupted jobs — zero data loss.
*   **Prompt caching:** Describe moving static context to a cached prefix, reducing input token cost by up to 90% for repeated system prompts.
*   **Quantisation tradeoffs:** INT8 is almost always safe; INT4 needs quality eval on your specific task before production.
*   **Tagging enforcement:** Use AWS Service Control Policies or GCP Organization Policies to enforce mandatory tags at account/org level — prevents untagged drift.
