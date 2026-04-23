# Cloud 04: Serverless for AI Inference

## 1. Serverless vs. Always-On

| | Serverless (Lambda / Cloud Run) | Always-On (ECS / K8s) |
| :--- | :--- | :--- |
| **Billing** | Per-request (ms precision) | Per-hour (idle waste) |
| **Cold start** | 100ms–5s | None |
| **Max memory** | Lambda: 10GB / Cloud Run: 32GB | Limited by instance |
| **GPU support** | Limited (Lambda: none, Cloud Run: L4) | Full GPU catalog |
| **Scaling** | Instant to thousands | Minutes (autoscale) |
| **Best for** | Bursty, unpredictable, lightweight | Steady, latency-sensitive, heavy models |

---

## 2. Cold Start Problem

A cold start occurs when a new container instance is spun up to handle a request:

```
Request arrives → No warm instance → Start container → Load model → Handle request
                                      └── adds 1–10 seconds ──┘
```

**Cold start mitigation strategies**:

1.  **Provisioned concurrency** (Lambda) / **min-instances** (Cloud Run): Keep N instances warm. Costs $$.
2.  **Lazy loading**: Load model on first request, not at container start. Reduces startup time slightly.
3.  **Smaller models**: Lighter models (quantized, distilled) load faster.
4.  **Optimized containers**: Pre-load model weights in container image layer.
5.  **Keep-alive pings**: Periodic synthetic requests to prevent scale-to-zero.

---

## 3. Cost Model: Serverless vs. Always-On

**Serverless cost** = requests × duration × memory price

**Always-on cost** = instances × hours × instance price (fixed, regardless of traffic)

**Break-even formula**:
```
Serverless = Always-On when:
RPS × avg_duration_s × memory_price_s ≈ N_instances × hourly_cost / 3600
```

For low, bursty traffic: serverless is cheaper.
For sustained high traffic: always-on (reserved instances) is cheaper.

---

## 4. Container-Based Inference (Between Serverless and VMs)

**AWS Fargate / GCP Cloud Run / Azure Container Instances**:
*   Container-based — no server management.
*   Supports GPU (Cloud Run supports L4 GPU).
*   Slower autoscaling than Lambda but faster than EC2 ASG.
*   Good middle ground for ML inference.

**AWS ECS / EKS on EC2**:
*   Full GPU support.
*   More complex but maximum flexibility.
*   Use when Lambda/Cloud Run don't support your model size.

---

## 5. Patterns for Async LLM Inference

For long-running LLM calls (30–120s), use async patterns:

```
Client → POST /generate → Job ID returned immediately (202 Accepted)
                        → Background job enqueued (SQS / Pub/Sub)
                        → Worker processes, stores result in S3/DB
Client → GET /results/{job_id} → Poll or use webhook
```

This avoids Lambda timeouts (15 min max) and HTTP client timeouts.
