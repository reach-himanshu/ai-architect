# Fine-Tuning Module

When and how to fine-tune — from SFT to preference optimization — and how to serve the results.

## 🚀 Roadmap

### 1. When to Fine-Tune vs Prompt vs RAG
- Decision framework
- Cost and latency impacts
- Common anti-patterns

### 2. Dataset Curation & Formatting
- Instruction, chat, preference formats
- Quality vs quantity
- Prereq: `07-data-engineering-for-ai/`

### 3. Supervised Fine-Tuning (SFT) Basics
- Full-parameter SFT
- Learning rate, epochs, batch size
- Overfitting signals

### 4. PEFT Overview
- Parameter-efficient tuning landscape
- Adapters, prefix tuning, prompt tuning

### 5. LoRA Deep Dive
- Low-rank decomposition intuition
- Rank, alpha, target modules
- Merging adapters

### 6. QLoRA & Quantization Basics
- 4-bit NF4
- Double quantization
- VRAM budgeting

### 7. Training Infrastructure
- HF Trainer, `accelerate`
- DeepSpeed ZeRO stages
- FSDP

### 8. Evaluation During/After Training
- Train/val loss interpretation
- Held-out eval sets
- Downstream task eval
- Prereq: `11-evaluation/`

### 9. RLHF Overview
- Reward model training
- PPO basics
- Why teams move off PPO

### 10. DPO, ORPO, KTO
- Direct preference optimization
- When each applies
- Simpler pipelines

### 11. Distillation & Model Compression
- Teacher-student setups
- Distilling Claude/GPT to open weights
- Compression vs capability tradeoff

### 12. Serving Fine-Tuned Models
- vLLM, TGI, Ollama
- Adapter hot-swapping
- Bridges to `14-llmops-deployment/`

### 13. Domain Adaptation Case Study
- Worked example (code / legal / medical)
- Baseline → SFT → DPO comparison
- Eval-driven iteration
