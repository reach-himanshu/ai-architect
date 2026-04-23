# Containers and Kubernetes for ML

## Why Containers Matter for ML

Containers solve the "works on my machine" problem:

*   Package model, dependencies, and serving code in one immutable image.
*   Reproducible across dev, staging, and production environments.
*   Enable horizontal scaling, rolling deployments, and rollback.

---

## 1. Docker for ML Workloads

### Multi-Stage Build

```dockerfile
# Stage 1: Build dependencies
FROM python:3.11-slim AS builder
WORKDIR /build
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt --target /packages

# Stage 2: Runtime (smaller image)
FROM python:3.11-slim AS runtime
COPY --from=builder /packages /usr/local/lib/python3.11/site-packages
WORKDIR /app
COPY src/ ./src/
COPY model/ ./model/
EXPOSE 8080
CMD ["python", "src/serve.py"]
```

**Benefits:** Final image contains no build tools → smaller, fewer vulnerabilities.

### ML-Specific Dockerfile Tips

*   Use `COPY requirements.txt` before `COPY src/` — layer caching means dependencies aren't reinstalled on every code change.
*   Base image: `python:3.11-slim` (not `python:3.11` — full image is 1GB+).
*   GPU: Use official `nvidia/cuda:12.x-runtime` base images, not `devel` (runtime is 3× smaller).
*   Pin all dependency versions in `requirements.txt`.

---

## 2. Kubernetes for ML Serving

### Core Resources

| Resource | Purpose |
|---------|---------|
| `Pod` | One or more containers sharing network/storage |
| `Deployment` | Manages pod replicas; handles rolling updates |
| `Service` | Stable network endpoint for pods |
| `HorizontalPodAutoscaler` | Auto-scales based on CPU/custom metrics |
| `ConfigMap` | Configuration data (model paths, thresholds) |
| `Secret` | Sensitive data (API keys, credentials) |
| `PersistentVolumeClaim` | Attach persistent storage (model weights) |

### Model Serving Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: churn-model-serving
spec:
  replicas: 3
  selector:
    matchLabels: {app: churn-model}
  template:
    spec:
      containers:
      - name: model-server
        image: registry.io/churn-model:v2.1
        resources:
          requests: {cpu: "500m", memory: "1Gi"}
          limits:   {cpu: "2000m", memory: "4Gi"}
        readinessProbe:
          httpGet: {path: /health, port: 8080}
          initialDelaySeconds: 30
        env:
        - name: MODEL_VERSION
          value: "v2.1"
```

### Rolling Update Strategy

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 1        # 1 extra pod during update
    maxUnavailable: 0  # never reduce capacity below desired
```

---

## 3. Horizontal Pod Autoscaling

### CPU-Based (Simple)

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
spec:
  minReplicas: 2
  maxReplicas: 20
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

### Custom Metrics (Requests per Second, Queue Depth)

*   Use KEDA (Kubernetes Event-Driven Autoscaling) for queue-based scaling.
*   Scale on Kafka consumer lag, SQS queue depth, or Redis list length.
*   Better for ML inference: scale on inference requests/second, not CPU.

---

## 4. GPU Workloads on Kubernetes

*   Use `nvidia.com/gpu: 1` resource request to schedule GPU pods.
*   `nodeSelector` or `nodeAffinity` to target GPU node pools.
*   GPU pods are expensive — use `ttlSecondsAfterFinished` for training jobs.

```yaml
resources:
  limits:
    nvidia.com/gpu: 1
nodeSelector:
  cloud.google.com/gke-accelerator: nvidia-tesla-v100
```

### GPU Sharing

*   **MIG (Multi-Instance GPU):** A100 supports 7× 1/7-GPU slices.
*   **Time-slicing:** Multiple pods share GPU time (higher utilisation, lower isolation).
*   Use for small models / batch inference where full GPU is wasteful.

---

## 5. Helm for ML Platform

Helm packages Kubernetes manifests into reusable charts:

```bash
# Deploy model serving endpoint
helm install churn-model ./charts/model-serving \
  --set model.version=v2.1 \
  --set model.replicas=3 \
  --set model.image=registry.io/churn-model:v2.1
```

```
charts/model-serving/
├── Chart.yaml
├── values.yaml          ← default configuration
└── templates/
    ├── deployment.yaml
    ├── service.yaml
    ├── hpa.yaml
    └── configmap.yaml
```

---

## 6. Model Serving Patterns on K8s

| Pattern | Description | Tool |
|---------|-------------|------|
| Sidecar | Inference server as sidecar to preprocessor | Triton + custom preprocessor |
| Multi-model serving | One server, many models | Triton, Torchserve |
| Canary via Istio | Traffic split at service mesh level | Istio VirtualService |
| Blue/Green | Two full deployments, instant switch | K8s Service selector swap |

---

## Key Interview Points

*   **Resource requests vs limits:** Requests = guaranteed allocation (for scheduling). Limits = hard cap (container killed if exceeded). Always set both — without requests, pods get evicted under node pressure.
*   **Liveness vs readiness probes:** Liveness restarts unhealthy pods. Readiness removes pods from Service endpoints if unhealthy (won't receive traffic). Always implement both for ML servers.
*   **Rolling update + minReplicas:** Setting `maxUnavailable: 0` ensures capacity is never reduced during rollout — critical for SLA compliance.
*   **KEDA for ML:** Explain queue-based autoscaling: KEDA watches Kafka consumer lag and scales inference pods to drain the queue. Better than CPU-based for bursty inference workloads.
