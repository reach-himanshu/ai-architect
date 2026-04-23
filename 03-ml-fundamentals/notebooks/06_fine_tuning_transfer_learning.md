# ML 06: Fine-Tuning & Transfer Learning

When to fine-tune, how it works, and the modern efficient techniques (LoRA, PEFT) that make it practical.

## 1. The Spectrum of Adaptation

```
Prompt Engineering
    → Few-shot prompting
        → RAG (retrieval at inference time)
            → Adapter fine-tuning
                → LoRA / QLoRA
                    → Full fine-tuning
                        → Pre-training from scratch
```

Each step increases control and cost. Start from the left and only go right when necessary.

---

## 2. When to Fine-Tune vs. RAG

| Scenario | RAG | Fine-Tune |
| :--- | :--- | :--- |
| Proprietary knowledge updates frequently | ✅ Better | ❌ Costly to retrain |
| Need to cite sources | ✅ Natural | ❌ Harder |
| Need specific output style/format | ❌ Less reliable | ✅ Better |
| Specific domain vocabulary | Partial | ✅ Better |
| Latency is critical | ❌ Adds retrieval step | ✅ Faster (no retrieval) |
| Data is < 1000 examples | ❌ Not enough for fine-tune | ❌ Fine-tune needs more |
| Complex multi-step reasoning | ❌ Context limited | ✅ Can be trained in |

**Rule of thumb**: Try RAG + prompt engineering first. Fine-tune only if quality is insufficient and you have 1000+ high-quality examples.

---

## 3. Full Fine-Tuning

Train all model weights on your dataset:

```python
from transformers import AutoModelForCausalLM, Trainer, TrainingArguments

model = AutoModelForCausalLM.from_pretrained("meta-llama/Llama-3.1-8B")
# All 8 billion parameters will be updated

trainer = Trainer(
    model=model,
    args=TrainingArguments(output_dir="./output", num_train_epochs=3),
    train_dataset=my_dataset,
)
trainer.train()
```

**Cost**: Requires GPU with large VRAM (80GB for 7B model). Storage of full model weights.

---

## 4. LoRA (Low-Rank Adaptation)

LoRA freezes original weights and adds small trainable matrices:

```
W_updated = W_original + ΔW
Where ΔW = A × B  (rank decomposition)

A: (d × r), B: (r × d)  where r << d (rank << full dimension)
```

Instead of updating all d×d = 4096×4096 = 16.7M parameters per layer,
LoRA updates r×(d+d) = 8×8192 = 65K parameters — a 256x reduction.

```python
from peft import LoraConfig, get_peft_model

lora_config = LoraConfig(
    r=8,                      # Rank — controls capacity
    lora_alpha=32,            # Scaling factor
    target_modules=["q_proj", "v_proj"],  # Which layers to adapt
    lora_dropout=0.1,
)

model = get_peft_model(base_model, lora_config)
model.print_trainable_parameters()
# trainable params: 4,194,304 || all params: 6,742,609,920 || trainable%: 0.0622
```

---

## 5. QLoRA (Quantized LoRA)

QLoRA combines LoRA with 4-bit quantization:
*   Quantize base model to 4-bit (4x memory reduction).
*   Apply LoRA adapters in 16-bit.
*   Fine-tune a 70B model on a single 48GB GPU.

```python
from transformers import BitsAndBytesConfig

bnb_config = BitsAndBytesConfig(
    load_in_4bit=True,
    bnb_4bit_quant_type="nf4",
    bnb_4bit_compute_dtype=torch.float16,
)
model = AutoModelForCausalLM.from_pretrained(model_id, quantization_config=bnb_config)
```

---

## 6. Fine-Tuning Data Requirements

| Task | Minimum Examples | Recommended |
| :--- | :--- | :--- |
| Classification | 100–500 | 5,000+ |
| Instruction following | 500–1,000 | 10,000+ |
| Style adaptation | 200–500 | 2,000+ |
| Domain adaptation | 1,000+ | 50,000+ |
| RLHF | 10,000+ preference pairs | 100,000+ |

**Data quality > data quantity**: 1,000 high-quality examples beat 100,000 noisy ones.

---

## 7. RLHF Intuition

Reinforcement Learning from Human Feedback (RLHF) is how ChatGPT/Claude are trained:

```
1. SFT (Supervised Fine-Tuning): Fine-tune on demonstration data.
2. Reward Model: Train a model to score responses (human preferences).
3. RL (PPO): Use reward signal to optimize the LLM policy.
```

For practitioners: RLHF is expensive and complex. DPO (Direct Preference Optimization) is a simpler alternative that doesn't require a separate reward model.
