# LLM 03: Tokenization

Tokens are the unit of LLM economics, latency, and quality. Before a model sees text, the text is chopped into integer IDs. How it's chopped has surprising consequences.

## 1. Why Not Characters or Words?

| Unit | Problem |
| --- | --- |
| Characters | Sequences too long; the model has to rediscover "the" billions of times |
| Words | Vocabulary explodes (inflection, typos, proper nouns); OOV problem |
| Subwords | Best of both — frequent words are one token, rare words decompose into pieces |

Modern LLMs use **subword tokenization**. A vocabulary of 32K–200K subword units covers essentially any text.

## 2. Byte-Pair Encoding (BPE)

BPE starts from bytes (or characters) and iteratively merges the most frequent adjacent pair into a new token, until the vocabulary reaches a target size.

```
corpus: "low lower lowest"
initial: l o w _ l o w e r _ l o w e s t
merge #1 (lo → lo): lo w _ lo w e r _ lo w e s t
merge #2 (low → low): low _ low e r _ low e s t
merge #3 (er → er): low _ low er _ low e s t
...
```

At inference, we greedily apply learned merges to the input.

### Variants

| Name | Used by | Notes |
| --- | --- | --- |
| BPE | GPT-2, GPT-3, GPT-4, Claude | Byte-level BPE handles any Unicode |
| WordPiece | BERT | Like BPE but maximizes likelihood of merges |
| SentencePiece (BPE or unigram) | Llama, T5, Mistral, Gemma | Handles whitespace as a first-class character |
| Tiktoken | OpenAI | Fast Rust BPE implementation; different encodings per model |

## 3. Token Counts Vary Wildly

The same prompt, different models, different token counts.

| Text | GPT-4 tokens | Llama-3 tokens | Comment |
| --- | --- | --- | --- |
| "Hello world" | 2 | 2 | Common phrase, similar |
| "Antidisestablishmentarianism" | 6 | 7 | Rare word, splits |
| Japanese: "こんにちは" | 5 | 7 | Multilingual varies a lot |
| Python code sample | Varies | Varies | Code tokenizers differ |

**Implication for cost:** non-English prompts can be 1.5–3× more tokens than the English equivalent. Pricing is per-token, so this directly affects feature unit economics.

## 4. Special Tokens and Chat Templates

Tokenizers reserve special IDs: BOS (beginning of sequence), EOS (end), PAD, and — critically — **chat template tokens** like `<|im_start|>user`, `<|eot_id|>`, `<|begin_of_text|>`, etc.

The chat template is **model-specific** and wrong templates produce garbage. Example, Llama-3:

```
<|begin_of_text|><|start_header_id|>system<|end_header_id|>
You are a helpful assistant.<|eot_id|><|start_header_id|>user<|end_header_id|>
Hello<|eot_id|><|start_header_id|>assistant<|end_header_id|>
```

When using the HuggingFace `transformers` library, always use `tokenizer.apply_chat_template(messages)` — never roll your own, or you'll silently ship a model that's mis-prompted.

## 5. The Tokenizer Is Part of the Model

You cannot mix tokenizers. If you fine-tune Llama-3 with the Gemma tokenizer, nothing works. When shipping a fine-tune, ship the tokenizer config too.

For embedding/retrieval workflows, the embedding model has its own tokenizer too — usually different from the generator's. Input-token budgets on embeddings are typically lower (~512 tokens for most sentence-transformer models).

## 6. Context Length vs Token Count

Context length is measured in **tokens**, not characters. A 200K-context Claude can hold ~150K English words (avg 1.3 tokens/word), fewer in other languages or code.

Rules of thumb (GPT-4 / Claude):
- English: ~0.75 words per token
- Code: ~2–3 chars per token
- JSON: ~2–3 chars per token (lots of syntax overhead)
- Non-Latin scripts: often 2–4× more tokens than a direct translation

Always measure, never assume.

## 7. Tokenization Pitfalls

1. **Numbers** — `3141592653` might be one token or many; arithmetic is harder for the model when digits are chunked unpredictably. Some models (Llama-3) explicitly tokenize digits individually.
2. **Trailing whitespace** — "Hello" vs "Hello " is *two different token sequences* with different continuations.
3. **Leading space sensitivity** — " the" and "the" are often different tokens. Always include the leading space when crafting few-shot examples.
4. **Rare Unicode** — some tokenizers fall back to byte-level pairs for emoji and rare scripts, inflating token counts.
5. **Tokenizer leakage** — weird tokens from the training corpus (like `<|endoftext|>`, `SolidGoldMagikarp`) can trigger unexpected behavior when passed as input.

## 8. Measuring Token Counts in Production

Before deploying an LLM feature:

1. Count tokens for **median and p99** prompts in expected traffic.
2. Measure input-token growth from chat history accumulation.
3. Set budget alarms at the token level, not request level.
4. Cache computed token counts when possible (tokenization is not free at scale).

## Prereqs / Connections

- `02-dsa/06_hash_tables` — tokenizer vocab is a hash table
- `03-design-patterns/15_batching_paging` — batching tokenization for throughput

## Further Reading

- Sennrich et al. 2015, "Neural Machine Translation of Rare Words with Subword Units" (BPE)
- Kudo & Richardson 2018, "SentencePiece" paper
- OpenAI tiktoken repo (read the code — it's small and instructive)
- Karpathy, "Let's build the GPT Tokenizer" (YouTube)
- "SolidGoldMagikarp" (LessWrong) — anomalous-token phenomenon
