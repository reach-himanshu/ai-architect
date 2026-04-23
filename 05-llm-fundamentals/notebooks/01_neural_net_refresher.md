# LLM 01: Neural Net Refresher for LLMs

Before transformers, we need a crisp mental model of what a neural network even is. This session is deliberately short — skip it if you already know forward/backward pass.

## 1. A Neural Network Is Just a Function

A neural net maps input → output. It's composed of layers; each layer is a matrix multiply plus a non-linearity.

```python
import numpy as np

def forward(x, W1, b1, W2, b2):
    h = np.tanh(x @ W1 + b1)   # hidden layer
    y = h @ W2 + b2            # output layer
    return y
```

Given enough data and the right loss, we can learn `W1, b1, W2, b2` to approximate almost any function. That's the Universal Approximation Theorem in one sentence.

## 2. Training: Forward, Loss, Backward, Update

```
for each batch:
    pred = model(x)              # forward
    loss = loss_fn(pred, y)      # how wrong
    grads = d(loss)/d(weights)   # backward
    weights -= lr * grads        # update
```

Every framework (PyTorch, JAX, TF) automates the gradient step via **automatic differentiation**.

## 3. Why Depth Matters

Each layer learns features built on top of the previous layer. Vision: edges → shapes → objects. Language: characters → tokens → phrases → meaning. Depth gives compositional representations.

## 4. The Transformer Twist

Before 2017, the best language models were **RNNs / LSTMs**: they processed tokens sequentially, carrying hidden state forward. This was hard to parallelize and struggled with long contexts.

The Transformer replaced recurrence with **self-attention**: every token can look at every other token directly, in parallel. This made training on massive corpora feasible and is the reason LLMs got big.

We'll unpack self-attention in session 02.

## 5. Why Scale Matters

Kaplan et al. (2020) and Chinchilla (2022) showed: for a fixed compute budget, there's an optimal tradeoff between model size and data size. Bigger models trained on more data keep getting better on language benchmarks — the "scaling laws."

This is why LLMs are such a capital-heavy field: capability is largely bought with compute.

## 6. Pretrained vs Instruction-Tuned vs RLHF-Tuned

Modern chat models go through three stages:

1. **Pretraining** — next-token prediction over trillions of tokens. Produces a base model that completes text (GPT-3 base, Llama-3 base).
2. **Supervised fine-tuning (SFT)** — fine-tune on curated `(instruction, response)` pairs. Makes the model follow instructions.
3. **Preference optimization (RLHF / DPO)** — further align to human preferences over candidate responses.

When you call Claude or GPT-4 via API, you're calling a model that went through all three stages.

## Prereqs / Connections

- `02-dsa/01_big_o` — self-attention is O(n²); matters for context length
- `01-python/16_async_await` — LLM APIs are async-first

## Further Reading

- 3Blue1Brown, "But what is a Neural Network?" (YouTube)
- Karpathy, "Neural Networks: Zero to Hero" (YouTube)
- Jay Alammar, "The Illustrated Transformer"
- Kaplan et al. 2020, "Scaling Laws for Neural Language Models"
