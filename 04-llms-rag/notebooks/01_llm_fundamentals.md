# LLMs 01: LLM Fundamentals

Understanding how Large Language Models work is the foundation for making good architectural decisions. You don't need to know the math — you need the intuition.

## 1. What is a Transformer?

A Transformer is the neural network architecture behind every modern LLM (GPT, Claude, Gemini, Llama). It processes sequences of tokens and predicts the next token.

Key concepts:
*   **Tokens**: The basic unit of text. "Hello world" might be 2–3 tokens. One token ≈ 4 characters.
*   **Context Window**: The maximum tokens a model can "see" at once (e.g., GPT-4: 128K, Claude 3.5: 200K).
*   **Parameters**: The "weights" learned during training. GPT-3 = 175B params. More params ≠ always better.

---

## 2. The Attention Mechanism (Intuition)

Attention lets the model focus on the most relevant parts of the input when generating each token.

```
Input: "The animal didn't cross the street because it was too tired."
```

When predicting what "it" refers to, attention weights will be high on "animal" — not "street". This is **self-attention**.

*   **Multi-Head Attention**: Multiple attention heads run in parallel, each learning different relationships (syntax, coreference, semantics).
*   **Key/Query/Value**: Conceptually — Query is "what am I looking for", Key is "what I offer", Value is "what I contain".

---

## 3. Tokenization

Tokenizers convert text to integers that the model processes.

| Text | Tokens | Count |
| :--- | :--- | :--- |
| "Hello" | [9906] | 1 |
| "Hello world" | [9906, 1917] | 2 |
| "AI architect" | [15836, 23820] | 2 |
| "supercalifragilistic" | [13066, 4416, 333, 1409, 1285, 2233] | 6 |

**Why it matters for architects**:
*   Pricing is per token (not per word).
*   Long documents consume context window fast.
*   Non-English text uses more tokens per word.

---

## 4. How LLMs Generate Text

LLMs are **next-token predictors**. They output a probability distribution over the vocabulary, then sample.

```
Input: "The capital of France is"
Output probabilities: {"Paris": 0.97, "Lyon": 0.01, "Berlin": 0.005, ...}
Sampled token: "Paris"
```

**Key generation parameters**:
| Parameter | What it Does | Typical Range |
| :--- | :--- | :--- |
| `temperature` | Controls randomness. 0 = deterministic, 1 = creative | 0.0 – 2.0 |
| `top_p` | Nucleus sampling. Only sample from top P% probability mass | 0.0 – 1.0 |
| `max_tokens` | Maximum output length | 1 – model limit |
| `stop` | Stop generation at these strings | Any string |

---

## 5. Model Families and Trade-offs

| Model | Strengths | Context | Use Case |
| :--- | :--- | :--- | :--- |
| GPT-4o | Multimodal, fast | 128K | General purpose, vision |
| Claude 3.5 Sonnet | Long context, reasoning | 200K | Documents, analysis |
| Gemini 1.5 Pro | Huge context, Google ecosystem | 1M | Very long docs |
| Llama 3.1 (open) | Self-hosted, no data leaving | 128K | Privacy, cost control |
| Mistral / Phi | Lightweight, fast | 32K | Edge, low latency |

---

## 6. Key Architectural Decisions

As an AI architect, these are the questions you'll be asked:

1.  **Which model?** — Balance cost, latency, context length, and capability.
2.  **Hosted vs. self-hosted?** — Data privacy, compliance, and operational burden.
3.  **How much context?** — Longer context = higher cost + latency.
4.  **Determinism needed?** — Use `temperature=0` for structured extraction, higher for creative tasks.

---

## 7. Summary

*   LLMs predict the next token using learned attention over context.
*   Context window size is a hard architectural constraint.
*   Temperature and sampling parameters control output behavior.
*   Model selection is a trade-off between cost, latency, capability, and privacy.
