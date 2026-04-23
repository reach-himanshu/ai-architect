# LLM 05: Pretraining Objectives

Every LLM was born by minimizing a loss over enormous text corpora. The choice of loss — the **pretraining objective** — determines what the base model is good at before any fine-tuning. This session unpacks the main families.

## 1. What Pretraining Produces

Pretraining doesn't produce a chatbot. It produces a **base model**: a probability distribution over the next token given any prefix. Every useful behavior downstream — instruction-following, reasoning, tool use — is built on top of that distribution, usually via SFT and RLHF / DPO (covered in `13-fine-tuning`).

## 2. Causal Language Modeling (CLM)

The objective:

```
L = -Σ_t log P(x_t | x_<t)
```

Predict each token given only the tokens before it. Used by GPT, Llama, Claude, Mistral, Gemini — the entire decoder-only family.

**Strengths:** one objective, any task via next-token prediction; natural fit for generation; scales indefinitely.
**Weakness:** bidirectional context at training time is wasted; may need more data than bidirectional equivalents for some classification tasks.

## 3. Masked Language Modeling (MLM)

The objective (BERT-style):

```
Randomly mask 15% of tokens → predict them from bidirectional context
```

Used by BERT, RoBERTa, DeBERTa, modernBERT.

**Strengths:** bidirectional context = strong sentence-level representations; excellent for embeddings, classification, and NER.
**Weakness:** can't generate natively (masks are artificial); less useful for chat.

The `[MASK]` token is itself a distribution issue: it appears in training but not downstream, causing a small distribution shift. RoBERTa addressed this with dynamic masking; DeBERTa added disentangled attention.

## 4. Span Corruption / Denoising (T5)

The objective:

```
Corrupt spans of tokens with unique sentinel tokens → predict the corrupted content
```

Example:
```
input:  "Thank you <X> me to your party <Y> week."
target: "<X> for inviting <Y> last"
```

Used by T5, BART (similar), UL2.

**Strengths:** unifies generation and understanding in one framework; sometimes outperforms CLM at the same scale on mixed tasks.
**Weakness:** encoder-decoder architecture is heavier; the LLM field largely moved to decoder-only for scaling efficiency.

## 5. Prefix LM and FIM (Fill-in-the-Middle)

**Prefix LM** (GLM, PaLM-2): bidirectional attention over a prefix, causal over a suffix. Hybrid of CLM and MLM.

**Fill-in-the-Middle** (OpenAI Codex, DeepSeek-Coder, StarCoder): restructure training examples so the model learns to fill gaps:

```
prompt: <PREFIX> def add(a, b): <SUFFIX>    return a + b<MIDDLE>
target: the code that goes between PREFIX and SUFFIX
```

Essential for IDE completion use cases. Any code model worth using was FIM-trained.

## 6. Contrastive and Retrieval-Aware Objectives

**SimCSE, GTE, BGE, E5** are encoder models trained with contrastive objectives: pull paraphrases together and push unrelated sentences apart. These are the dominant embedding models.

**Retrieval-augmented pretraining** (REALM, Atlas, RETRO) incorporates a retrieved passage into the loss, encouraging the model to use external memory. Re-emerging in 2025–2026 as long-context + retrieval hybrids.

## 7. Summary Table

| Objective | Architecture | Training data orientation | Best for | Example models |
| --- | --- | --- | --- | --- |
| CLM | Decoder-only | Left-to-right | Generation, chat, reasoning | GPT, Claude, Llama |
| MLM | Encoder-only | Bidirectional | Embeddings, classification | BERT, RoBERTa |
| Span corruption | Encoder-decoder | Bidirectional + autoregressive | Summarization, translation | T5, BART |
| Prefix LM | Hybrid | Bi + causal | Mixed tasks | GLM, PaLM-2 |
| FIM | Decoder-only | Causal w/ rearranged examples | Code completion | StarCoder, DeepSeek-Coder |
| Contrastive | Encoder-only | Sentence pairs | Retrieval embeddings | SimCSE, BGE, E5 |

## 8. Data Mix Is Half the Story

As of 2026, pretraining corpora are not "just the internet." They're curated blends:

| Category | Share | Notes |
| --- | --- | --- |
| Web (CommonCrawl, C4) | 50–70% | Heavily filtered and deduped |
| Books | 5–10% | Long-form coherence |
| Code (GitHub, Stack) | 10–20% | Even non-code LLMs benefit |
| Wikipedia / refs | 2–5% | High-quality factual |
| Math / reasoning traces | 5–15% | Critical for reasoning LLMs |
| Synthetic (LLM-generated) | 0–30% | Growing rapidly |

Quality-over-quantity has become the dominant discipline. FineWeb, DCLM, and Nemotron datasets showed careful curation beats raw scale at a fixed token budget.

## 9. Chinchilla and the Token:Param Ratio

Kaplan et al. 2020 suggested model size matters more than data. **Chinchilla (Hoffmann et al., 2022)** corrected that: for a fixed compute budget, the optimal ratio is roughly **20 training tokens per parameter**.

- Llama-3 8B was trained on 15T tokens → ratio ~1875:1 (vastly over-trained, by design, because inference cost matters more than training cost).
- GPT-4 class models: ratios we don't know publicly.

**Architect takeaway:** when picking open-weight base models, higher tokens:params usually → better model at the same size. Llama-3 beats Llama-2 at every size partly because it saw 10× more tokens.

## 10. What Objective Means for Your App

| Your use case | Pick this pretraining heritage |
| --- | --- |
| Chat / generation / reasoning | Decoder-only CLM (Claude, GPT, Llama, Mistral) |
| Embeddings / retrieval | Encoder with contrastive obj. (BGE, E5, Voyage, Cohere) |
| Classification (with latency budget) | Encoder MLM fine-tuned (BERT, DeBERTa) |
| Code completion | Decoder w/ FIM (DeepSeek-Coder, StarCoder2) |
| Structured output / extraction | Instruction-tuned decoder + constrained decoding |

## Prereqs / Connections

- `02_transformer_architecture` — decoder vs encoder blocks
- `04_embeddings` — downstream of encoder-style pretraining
- `13-fine-tuning/` — SFT and preference optimization on top of a base model

## Further Reading

- Radford et al. 2018/2019/2020, GPT-1/2/3 papers
- Devlin et al. 2018, BERT paper
- Raffel et al. 2019, T5 paper ("Exploring the Limits of Transfer Learning")
- Hoffmann et al. 2022, Chinchilla paper
- Bavarian et al. 2022, "Efficient Training of Language Models to Fill in the Middle"
- Penedo et al. 2024, "FineWeb: Decanting the Web for the Finest Text Data"
