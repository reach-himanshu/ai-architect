# LLM 06: Decoding Strategies

The model outputs a probability distribution over the vocabulary at every step. **Decoding** is the process of turning that distribution into text. Different strategies change the quality, diversity, determinism, and cost of your application.

## 1. The Decoding Step

Every generation happens one token at a time:

```
for step in range(max_new_tokens):
    logits = model(ids)          # (vocab,)
    next_id = sample(logits)     # <-- the strategy
    ids.append(next_id)
    if next_id == EOS:
        break
```

The `sample(logits)` function is where everything interesting happens.

## 2. Greedy Decoding

```python
next_id = logits.argmax()
```

Pick the single most likely next token. Deterministic, fast, and often terrible for open-ended generation: loops and bland text. Still the right choice for:
- Short factual answers where there is one right word
- Code / structured output
- Anywhere you need reproducibility

## 3. Beam Search

Maintain the top-`k` partial sequences; extend all of them; keep top-`k` again; at the end pick the best.

```
Beam size 3, step 2:
  [The cat sat ...]  score -2.1
  [A cat sat ...]    score -2.3
  [The dog ran ...]  score -2.8
```

**Pros:** higher-likelihood completions than greedy; good for translation/summarization.
**Cons:** repetitive, under-diverse for chat; incompatible with real-time streaming; rarely used for modern chat models.

## 4. Sampling with Temperature

```
P_τ(x) = softmax(logits / τ)
next_id = sample(P_τ)
```

| τ | Effect |
| --- | --- |
| 0 | Equivalent to greedy |
| <1 | Peaks sharpened — more deterministic, more likely tokens |
| 1 | Model's native distribution |
| >1 | Flattened — more diverse, more creative, more nonsense risk |

Temperature alone is coarse. Paired with top-k or top-p it's the workhorse of chat generation.

## 5. Top-k Sampling

Keep only the `k` most likely tokens, renormalize, sample.

```python
topk = logits.topk(k)
probs = softmax(topk.values / τ)
```

- `k=1` = greedy.
- `k=40` is classic.
- `k=100+` is essentially full sampling.

**Pros:** simple, kills long-tail nonsense.
**Cons:** `k` is fixed — good for some contexts, too restrictive for others where the model is genuinely uncertain.

## 6. Top-p (Nucleus) Sampling

Keep the smallest set of tokens whose cumulative probability exceeds `p`.

```
sort logits descending
cum_prob = cumulative sum of softmax
keep tokens until cum_prob ≥ p
```

- `p=0.9` is typical.
- Adapts the candidate set to the model's confidence.

**This is the default for chat**: GPT, Claude, Llama serving stacks all default to nucleus sampling with `p=0.9–1.0` plus `τ=0.7–1.0`.

## 7. Combining: the Standard Stack

Production chat generation typically runs:

```
1. Apply repetition / frequency / presence penalties
2. Apply logit bias (forced / forbidden tokens)
3. Apply temperature
4. Apply top-k (if set)
5. Apply top-p
6. Renormalize and sample
```

Most APIs expose `temperature`, `top_p`, `top_k`, `presence_penalty`, `frequency_penalty`, `logit_bias`, and a few stop sequences. Some (Claude) don't expose top-k.

## 8. Penalties: Preventing Repetition

- **Frequency penalty** subtracts `α × count(token so far)` from logits. Strong anti-repetition; can push the model into weird directions.
- **Presence penalty** subtracts `β × 1[token seen]`. Gentler; encourages new vocabulary.
- **No-repeat-ngram** forbids any n-gram that already appeared.

Classic generation bug: greedy → infinite repetition. Sampling with modest top-p usually suffices; penalties are for edge cases.

## 9. Constrained / Structured Decoding

Sometimes you need outputs to conform to a schema (JSON) or grammar (SQL).

| Tool | Idea |
| --- | --- |
| `logit_bias` | Forbid or force individual tokens |
| **JSON mode** (OpenAI, Anthropic) | Provider-enforced valid JSON |
| **Outlines / LM Format Enforcer** | Client-side regex / grammar mask on logits |
| **Guidance / DSPy** | Programmatic interleaving of prompts and structured outputs |
| **jsonformer** | Token-level JSON schema enforcement |

Structured decoding sits at the sampling step: at each step, it builds a mask of "tokens that keep the output valid" and zeros out everything else.

Performance note: structured decoding can *increase* quality by constraining the search space, but overly aggressive constraints can damage fluency. Use it when the output shape is non-negotiable.

## 10. Speculative Decoding (Inference Optimization)

A small "draft" model proposes k tokens; the big target model verifies them in parallel. If they match, you skip k forward passes. If not, fall back.

This doesn't change the output distribution (correctness-preserving) but 2–5× throughput is common. Most serving stacks (vLLM, TGI) support it.

Covered in depth in `14-llmops-deployment/03_inference_optimization`.

## 11. Determinism in Production

| Strategy | Deterministic? |
| --- | --- |
| Greedy (`temperature=0`) | Yes, assuming fixed model + hardware |
| Sampling | No, unless you seed |
| Beam search | Yes |

Even `temperature=0` is not perfectly deterministic across batch sizes on GPUs due to non-associative floating-point. If you need bit-exact reproducibility (legal, scientific), you need `seed` support + fixed batch size + fixed hardware.

## 12. Picking Defaults for Your Use Case

| Use case | Temp | top_p | Notes |
| --- | --- | --- | --- |
| Factual Q&A | 0.0–0.3 | 1.0 | Penalize creativity |
| Chat assistant | 0.7 | 0.9 | The standard |
| Creative writing | 0.9–1.2 | 0.95 | More diversity |
| Code generation | 0.2–0.5 | 0.95 | Slight creativity, mostly deterministic |
| Structured JSON | 0.0 + schema mode | 1.0 | Make validity cheap |
| Summarization | 0.3–0.5 | 1.0 | Faithful, not creative |

## 13. Architect Checklist

- Pin `temperature` and `top_p` at the **feature** level, not the model level. Different features want different settings.
- For eval and regression tests, force `temperature=0` or a fixed seed.
- Log decoding params with every inference call — debugging quality regressions requires it.
- Use JSON/grammar mode when output correctness matters more than style.
- Don't use beam search for chat.

## Prereqs / Connections

- `02_transformer_architecture` — logits come from the final linear projection
- `05_pretraining_objectives` — decoding is sampling from the learned distribution
- `06-prompt-engineering/` — prompt + decoding together shape output

## Further Reading

- Holtzman et al. 2019, "The Curious Case of Neural Text Degeneration" (nucleus sampling paper)
- Welleck et al. 2019, "Neural Text Generation with Unlikelihood Training"
- Leviathan et al. 2022, "Fast Inference from Transformers via Speculative Decoding"
- Willard & Louf 2023, "Efficient Guided Generation for LLMs" (outlines paper)
