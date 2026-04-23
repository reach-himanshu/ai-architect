# Cloud 02: GPU & Compute for AI

## 1. GPU Instance Types

| GPU | VRAM | Performance | Cloud Instance | Use Case |
| :--- | :--- | :--- | :--- | :--- |
| **NVIDIA H100** | 80GB | Fastest | AWS p5, GCP a3 | Large model training, inference |
| **NVIDIA A100** | 40/80GB | Excellent | AWS p4, GCP a2 | Training, large inference |
| **NVIDIA A10G** | 24GB | Good | AWS g5 | Inference, fine-tuning small models |
| **NVIDIA L40S** | 48GB | Good inference | GCP g2 | Inference, multimodal |
| **NVIDIA T4** | 16GB | Budget | AWS g4dn, GCP n1+T4 | Small model inference |
| **NVIDIA V100** | 16/32GB | Older | AWS p3 | Legacy workloads |
| **Google TPU v5** | 96GB HBM | Very fast for TF | GCP | Google-stack training |

---

## 2. Model VRAM Requirements

| Model Size | Minimum VRAM (FP16) | Minimum VRAM (INT4) |
| :--- | :--- | :--- |
| 7B parameters | 14GB | 4GB |
| 13B parameters | 26GB | 7GB |
| 30B parameters | 60GB | 16GB |
| 70B parameters | 140GB (multi-GPU) | 35GB |
| 405B parameters | Multi-node required | 200GB (multi-GPU) |

**Rule**: Model parameters × 2 bytes/param ≈ minimum VRAM for FP16.

---

## 3. Spot vs. On-Demand vs. Reserved

| Type | Cost vs. On-Demand | Risk | Best For |
| :--- | :--- | :--- | :--- |
| **On-Demand** | 1× (baseline) | None | Short jobs, unpredictable |
| **Spot/Preemptible** | 0.3–0.7× | Interruption (2-min notice) | Batch training, fault-tolerant |
| **Reserved (1yr)** | 0.6× | Capital commitment | Inference endpoints |
| **Reserved (3yr)** | 0.4× | Long commitment | Stable production |

**For training**: Use spot with checkpointing. Resume from last checkpoint on interruption.

---

## 4. Inference Optimization Techniques

| Technique | Memory Saving | Speed Gain | Quality Impact |
| :--- | :--- | :--- | :--- |
| **FP16** (vs FP32) | 2× | 2× | Negligible |
| **INT8 quantization** | 4× | 1.5–2× | Slight |
| **INT4/GPTQ** | 8× | 2–3× | Moderate |
| **Continuous batching** | None | 2–10× | None |
| **Flash Attention** | None | 2–4× | None |
| **KV Cache** | None | 5–20× for long context | None |
| **Speculative decoding** | None | 2–3× | None |

**vLLM** combines continuous batching + PagedAttention (KV cache management) for production serving.

---

## 5. Multi-GPU Strategies

| Strategy | Use Case | When |
| :--- | :--- | :--- |
| **Tensor parallelism** | Model too large for single GPU | Model > single GPU VRAM |
| **Pipeline parallelism** | Very large models | Multi-node |
| **Data parallelism** | Speed up training | Model fits in one GPU |
| **Sequence parallelism** | Very long sequences | Transformer training |
