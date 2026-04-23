# Cloud 03: Managed ML Services

## 1. The Three Major Platforms

| Feature | AWS SageMaker | GCP Vertex AI | Azure ML |
| :--- | :--- | :--- | :--- |
| **Training jobs** | SageMaker Training | Custom Training | Azure ML Jobs |
| **Pipelines** | SageMaker Pipelines | Vertex Pipelines (Kubeflow) | Azure ML Pipelines |
| **AutoML** | Autopilot | AutoML | Automated ML |
| **Model Registry** | SageMaker Model Registry | Vertex Model Registry | Azure ML Model Registry |
| **Feature Store** | SageMaker Feature Store | Vertex Feature Store | Azure ML Feature Store |
| **Endpoints** | Real-time + async + batch | Prediction endpoints | Managed online endpoints |
| **Experiments** | SageMaker Experiments | Vertex Experiments | Azure ML Experiments |
| **Notebooks** | SageMaker Studio | Vertex Workbench | Azure ML Notebooks |

---

## 2. AWS SageMaker

**Training Jobs**: Managed containers with your training code + data from S3.
```python
estimator = PyTorch(
    entry_point="train.py",
    instance_type="ml.p3.2xlarge",
    instance_count=1,
)
estimator.fit({"train": s3_train_uri})
```

**Endpoints**: Deploy as real-time (low latency), asynchronous (large payloads), or batch (high throughput).

**SageMaker Pipelines**: Orchestrate multi-step ML workflows (preprocess → train → evaluate → register → deploy).

**SageMaker Feature Store**: Online (low-latency serving) + offline (historical, S3-backed for training).

---

## 3. GCP Vertex AI

**Model Garden**: Hundreds of models from Google (Gemini, Gemma) and open community (Llama, Mistral).

**Vertex AI Pipelines**: Based on Kubeflow Pipelines — more portable than SageMaker Pipelines.

**Vertex Feature Store**: Online + offline store, BigQuery-integrated for training data.

**Vertex AI Search**: Managed RAG with Vertex matching engine and Gemini — good for document search.

---

## 4. Azure ML

**Automated ML**: Automatically tries dozens of models and hyperparameters — fast baseline.

**MLflow Integration**: First-class MLflow support — easy to bring existing MLflow workloads.

**Managed Online Endpoints**: Blue-green deployment, traffic splitting built-in.

**Responsible AI Dashboard**: Fairness, explainability, error analysis baked in.

---

## 5. When to Use Managed vs. Custom MLOps

| Scenario | Use Managed | Use Custom (Kubeflow/Metaflow) |
| :--- | :--- | :--- |
| Small ML team | Yes — reduces ops burden | No — too much to manage |
| Heavy multi-cloud | Avoid lock-in | Yes — portable |
| Advanced customization | Limited | Yes — full control |
| Compliance/audit trail | Managed has built-in | Custom needs setup |
| Cost at scale | Managed can be expensive | Custom cheaper at high volume |
