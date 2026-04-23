# LLM 02: Transformer Architecture

The Transformer (Vaswani et al., 2017, "Attention Is All You Need") is the substrate of every modern LLM. If you internalize one architecture in your career, make it this one.

## 1. The Problem Transformers Solved

Before 2017, the best sequence models were **RNNs** and **LSTMs**. They processed tokens one at a time, carrying hidden state forward. Two problems:

1. **Sequential bottleneck** — can't parallelize across the time dimension, so training on large corpora was slow.
2. **Long-range dependencies** — information from early tokens decayed as it passed through many hidden-state updates.

The Transformer replaces recurrence with **attention**: every token sees every other token in one matmul. Training parallelizes across the sequence. Long-range dependencies become a direct dot-product, not a game of telephone.

## 2. Self-Attention, in Math

Given input tokens `x_1, ..., x_n`, each token is first embedded into a `d_model`-dimensional vector. Three learned projections produce Query, Key, Value:

```
Q = X @ W_Q     # (n, d_k)
K = X @ W_K     # (n, d_k)
V = X @ W_V     # (n, d_v)
```

Scaled dot-product attention:

```
attn(Q, K, V) = softmax(Q @ K.T / sqrt(d_k)) @ V
```

Intuition:
- `Q @ K.T` — dot-product similarity of every Query against every Key; shape `(n, n)`.
- `/ sqrt(d_k)` — keeps the softmax from saturating as `d_k` grows.
- `softmax(...)` — each row becomes a probability distribution over "which tokens to attend to."
- `... @ V` — weighted average of Value vectors.

The output is a new sequence of `n` vectors, each a content-aware mix of the whole input.

### Causal masking (decoder-only)

For autoregressive language modeling, token `i` cannot attend to tokens `> i`. We mask the upper triangle of the attention scores with `-inf` before the softmax. This is the single line that separates GPT from BERT, conceptually.

## 3. Multi-Head Attention

A single attention head can only track one kind of relationship at a time. **Multi-head attention** runs `h` attention computations in parallel (each with its own `W_Q, W_K, W_V` of smaller dimension), concatenates the outputs, then projects back:

```
head_i = attn(X W_Q_i, X W_K_i, X W_V_i)
MHA(X) = Concat(head_1, ..., head_h) W_O
```

Typical values: `d_model=4096`, `h=32`, so each head works in a 128-dim subspace. Some heads end up specializing (positional, syntactic, semantic) — see Voita et al. 2019, "Analyzing Multi-Head Self-Attention."

### Grouped Query Attention (GQA)

Modern models (Llama-2 70B, Mistral, Llama-3) share keys/values across groups of query heads:

| Variant | Q heads | KV heads | When to use |
| --- | --- | --- | --- |
| MHA | `h` | `h` | Highest quality, biggest KV cache |
| MQA | `h` | `1` | Smallest KV cache, quality loss |
| GQA | `h` | `h/g` | Sweet spot for inference |

GQA was the standard in 2024–2026 because KV-cache size dominates inference memory for long context.

## 4. The Full Transformer Block

One "layer" of a decoder-only model is:

```
x1 = x  + MHA(LayerNorm(x))           # residual + attention
x2 = x1 + FFN(LayerNorm(x1))          # residual + feed-forward
```

- **Residual connections** (`x + ...`) let gradients flow directly through dozens of layers.
- **LayerNorm** stabilizes activations. Note: pre-norm (inside the residual) is standard in modern LLMs; the original paper used post-norm.
- **FFN** is `Linear(d_model, 4*d_model) → GELU/SwiGLU → Linear(4*d_model, d_model)`. It's where most parameters live.

Stack N such blocks (N = 32 for Llama-3 8B, 80 for Llama-3 70B, more for frontier models). Add an input embedding, positional encoding, and an output projection to vocab size, and you have GPT.

## 5. Positional Encoding

Attention is **permutation-invariant** — shuffle the tokens and the output is the same shuffle. To make order matter, we add position information to the embeddings.

| Scheme | How it works | Used by |
| --- | --- | --- |
| Sinusoidal (original) | `sin/cos` of position at different frequencies | Vanilla Transformer |
| Learned absolute | A learned vector per position | GPT-2, BERT |
| RoPE (Rotary) | Rotate Q, K by angle proportional to position | Llama, GPT-J, Mistral |
| ALiBi | Bias attention scores by relative distance | BLOOM, MPT |
| NoPE | No position at all | Research, limited use |

**RoPE won.** It extends to longer contexts than training, composes cleanly with attention, and supports tricks like `RoPE scaling` / YaRN for long-context fine-tuning.

## 6. Encoder vs Decoder vs Encoder-Decoder

| Family | Example | Attention | Typical use |
| --- | --- | --- | --- |
| Encoder-only | BERT, RoBERTa | Bidirectional | Classification, embeddings |
| Decoder-only | GPT, Llama, Claude, Gemini | Causal (masked) | Generation, chat, reasoning |
| Encoder-decoder | T5, BART, original Transformer | Encoder bidirectional + decoder causal w/ cross-attn | Translation, summarization |

Decoder-only dominates in 2026 because:
1. Same architecture handles every task via next-token prediction.
2. KV caching makes generation efficient.
3. It scales further than encoder-decoder did empirically.

## 7. Mixture of Experts (MoE)

An MoE layer replaces the single FFN with `E` FFNs ("experts") and a router that picks `k` experts per token.

```
h = x
for each token t in seq:
    experts_for_t = router(t)       # top-k of E
    h[t] = sum(gate_i * FFN_i(t) for i in experts_for_t)
```

Only `k` of `E` experts run per token, so total parameters go up while FLOPs stay roughly flat. This is how Mixtral 8x7B (47B total params, 13B active) and GPT-4 (rumored MoE) get capacity without proportional inference cost.

Tradeoffs:
- **Pros:** more knowledge per inference dollar; specialization.
- **Cons:** routing is brittle, requires careful load-balancing; memory footprint of all experts on all GPUs; distillation/fine-tuning harder.

## 8. What Scales, What Doesn't

As of 2026, the knobs that matter:

| Knob | Effect |
| --- | --- |
| Parameters | Capacity; cost dominates everything else |
| Training tokens | Data diversity; Chinchilla-optimal ratio is ~20 tokens/param |
| Context length | Architectural: RoPE scaling; memory: KV cache grows linearly |
| Depth vs width | Width is cheaper (matmul efficient); depth helps reasoning |
| Active params (MoE) | Inference cost tracks active, not total |

## 9. Architect Takeaways

- For **building apps**, treat transformer internals as a fixed primitive. Pick a model by capability + cost, not by head count.
- For **selecting open-weight models**, check: context length, KV cache size (GQA?), whether RoPE scaling was applied.
- For **debugging latency**, know that inference is dominated by KV cache reads after prefill. Long context → more memory bandwidth.
- For **cost modeling**, understand that attention is O(n²) in context length; most "long context is expensive" headlines come from this.

## Prereqs / Connections

- `01_neural_net_refresher` — forward/backward pass basics
- `02-dsa/09_graphs` — attention is a soft graph over tokens
- `02-dsa/15_heaps` — top-k sampling in decoding uses heaps

## Further Reading

- Vaswani et al. 2017, "Attention Is All You Need" (the original paper)
- Jay Alammar, "The Illustrated Transformer" (visual walkthrough)
- Lilian Weng, "The Transformer Family" (evolution survey)
- Karpathy, "Let's build GPT: from scratch" (YouTube)
- Su et al. 2021, "RoFormer: Enhanced Transformer with Rotary Position Embedding"
