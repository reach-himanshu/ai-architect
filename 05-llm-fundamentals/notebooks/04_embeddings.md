# LLM 04: Embeddings

An embedding is a dense vector representation of text. Embeddings power semantic search, RAG, clustering, classification, and recommendation. For an AI Architect, embeddings are the single most important primitive after the LLM itself.

## 1. What an Embedding Is

Given text, an embedding model returns a vector `v ∈ R^d` such that semantically similar texts have vectors close together (usually by cosine similarity).

```python
embed("dog")   → [0.12, -0.04, 0.88, ..., 0.31]   # d=1536 typically
embed("puppy") → [0.10, -0.02, 0.91, ..., 0.29]
cosine(embed("dog"), embed("puppy"))  # ≈ 0.95
cosine(embed("dog"), embed("bank"))   # ≈ 0.22
```

That's the whole idea. Everything else is tradeoffs.

## 2. Static vs Contextual

| Era | Examples | Idea |
| --- | --- | --- |
| Static (pre-2018) | word2vec, GloVe, fastText | One vector per word, context-free |
| Contextual (2018+) | ELMo, BERT, sentence-transformers, OpenAI embeddings | Vector depends on surrounding context |

Static embeddings are a historical curiosity for retrieval. Every production system uses contextual embeddings, usually from a transformer encoder.

## 3. How Contextual Embeddings Are Produced

From a bidirectional encoder (BERT-style) or decoder-only model:

1. Tokenize text → IDs.
2. Run through the transformer → one vector per token.
3. **Pool** into a single sentence vector:
   - CLS token (BERT default)
   - Mean of token vectors (sentence-transformers default)
   - Last-token pooling (decoder-only models)
   - Weighted pooling (SIF, attentive)

Pooling matters. The same model with different pooling strategies produces different retrieval quality.

## 4. The Modern Embedding Provider Landscape (2026)

| Provider / Model | Dim | Notes |
| --- | --- | --- |
| OpenAI `text-embedding-3-small` | 1536 (scalable) | Matryoshka; cheap baseline |
| OpenAI `text-embedding-3-large` | 3072 | Strong quality |
| Cohere `embed-v3` | 1024 | Multilingual, retrieval-tuned |
| Voyage `voyage-3-large` | 1024 | Strong on domain-specific tasks |
| BGE (`bge-m3`, BAAI) | 1024 | Open-weight, multilingual, dense + sparse |
| E5 (`intfloat/e5-mistral-7b-instruct`) | 4096 | Strong but heavy |
| `all-MiniLM-L6-v2` | 384 | Tiny, fast, good enough for many use cases |
| Anthropic Claude | (no embedding API as of 2026) | — |

**Selection heuristic:**
- Start with `text-embedding-3-small` or `BGE-M3`. Good enough for >80% of use cases.
- Move to `-large` / `voyage-3-large` only if retrieval eval shows you need more.
- Self-host small models (`all-MiniLM`, `bge-small`) for cost-sensitive or privacy-sensitive workflows.
- Always evaluate on *your* data; leaderboard rank does not predict your rank.

## 5. MTEB: the Benchmark

The Massive Text Embedding Benchmark ([MTEB](https://huggingface.co/spaces/mteb/leaderboard)) covers 56+ tasks across retrieval, clustering, classification, STS, summarization. Treat it as a filter (to eliminate bad models) not an oracle. Leaders on MTEB can be middling on your specific data.

## 6. Dimensionality and the Matryoshka Trick

Higher dimension = better quality (usually) = more storage + compute.

**Matryoshka Representation Learning (MRL)** trains a single model whose first `k` dimensions are still useful — you can truncate `d=3072` down to `d=256` with a controlled quality drop. `text-embedding-3-*` supports this via the `dimensions` parameter. Huge win for storage-bound vector DBs.

Storage math:
```
1B vectors × 1536 dim × 4 bytes (float32) = 6 TB
Truncated to 256 dim × 4 bytes = 1 TB
Quantized to int8 (256 dim) = 256 GB
```

That difference is the difference between "one machine" and "cluster".

## 7. Normalization and Similarity

Most embedding models return L2-normalized vectors (`||v|| = 1`). If so, cosine similarity, dot product, and (negative) Euclidean distance are all equivalent:

```
cos(a, b) = (a · b) / (||a|| ||b||) = a · b      # if normalized
```

Check your provider's docs. If vectors aren't pre-normalized, normalize yourself before inserting into a vector DB, otherwise dot-product search ranks by magnitude.

## 8. Query vs Passage Asymmetry

State-of-the-art retrievers train with asymmetric inputs — queries and passages are encoded with slightly different instructions:

```
query:   "What are symptoms of diabetes?"
passage: "Diabetes is a chronic condition characterized by ..."
```

OpenAI/Cohere/BGE/E5 all expect specific prefixes for query vs document. Mixing them destroys retrieval quality silently. Read your embedding model's card — this is the most common "my RAG doesn't work" bug.

## 9. Fine-Tuning Embeddings

Sometimes off-the-shelf embeddings don't separate your domain (legal, medical, enterprise jargon). Options:

1. **Adapter fine-tuning** — train LoRA on a base embedding model with your (query, positive, negative) triples.
2. **Full fine-tuning** — full parameters, requires more data.
3. **Instruction prefixing** — sometimes a custom prompt prefix is enough.

Ask: *can I get sufficient hard-negatives?* If not, fine-tuning won't help — focus on reranking (session `09-rag/06_reranking`).

## 10. Sparse, Dense, and Hybrid

| Representation | Example | Strength |
| --- | --- | --- |
| Sparse | BM25, SPLADE | Keyword precision; explainability |
| Dense | Any embedding model | Semantic recall |
| Hybrid | BGE-M3, ColBERT, SPLADE+dense fusion | Best of both |

In practice, hybrid retrieval (dense + BM25 with reciprocal rank fusion) is the default production pattern in 2026.

## 11. Architect Decisions

1. **Choose embedding dim by storage budget first, quality second.**
2. **Always normalize.**
3. **Respect query/passage asymmetry.**
4. **Evaluate on your own data with your own eval set — not MTEB.**
5. **Revisit annually.** Embedding quality keeps improving; a 2-year-old embedding is an incumbency risk.

## Prereqs / Connections

- `03_tokenization` — embeddings live downstream of tokens
- `02-dsa/10_searching` — k-NN is the search primitive
- `08-vector-databases/03_similarity_metrics` — cosine vs dot vs L2

## Further Reading

- Reimers & Gurevych 2019, "Sentence-BERT" (the paper that started sentence-transformers)
- OpenAI, "New embedding models" (v3 release blog)
- Nussbaum et al. 2024, "Nomic Embed: Training a Reproducible Long Context Text Embedder"
- Chen et al. 2023, "BGE M3-Embedding" paper
- MTEB paper and leaderboard
