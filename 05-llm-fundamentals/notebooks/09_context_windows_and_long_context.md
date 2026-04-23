# LLM 09: Context Windows & Long Context

The context window is the single most architecturally-significant constraint on LLM applications. It dictates how much input, memory, and retrieved evidence you can fit, and it drives the cost curve of nearly every feature.

## 1. What "Context Window" Means

Context = all tokens the model sees in one forward pass: system prompt + chat history + user turn + retrieved docs + tool outputs + generated output so far.

Measured in **tokens**, not characters. See `03_tokenization`.

Context is a shared budget. Whatever you spend on one category, you take away from the rest.

## 2. The State of the Art (2026)

| Model | Context |
| --- | --- |
| GPT-4o family | 128K |
| Claude 4 / 4.5 | 200K (some variants 1M) |
| Gemini 1.5 / 2 | 1M (up to 10M in research) |
| Llama-3.3 | 128K |
| Mistral Large 3 | 128K |
| o1 / o3 reasoning models | 200K (thinking is part of it) |

Context length roughly quadrupled every 18 months through 2025, then plateaued as architectural (KV cache size) and economic limits kicked in.

## 3. Effective vs Advertised Context

A model advertised at `N` tokens is usually only *fully useful* over a smaller range. Empirical tests:

- **Needle-in-a-haystack (NIAH)** — hide a fact in a long document; ask for it.
- **RULER** — multi-needle, multi-hop, and variable-task benchmarks.
- **LongBench** — real tasks at long context.

**Typical finding:** at the advertised limit, factual recall drops and reasoning over long context degrades. The useful context length is often 30–70% of the advertised one. Measure on your task.

## 4. Why Long Context Is Expensive

Attention is **O(n²)** in sequence length. Training and prefill cost scale quadratically; generation cost is dominated by the KV cache, which is **O(n)** in length and **O(n_heads × d_head)** per layer.

For a 70B-parameter model with GQA and 32 layers, the KV cache at 128K tokens is multiple GB per sequence. This is why long-context inference serving is latency-heavy and memory-heavy.

Cost implications:

| Concern | Effect of context length |
| --- | --- |
| Input cost | Linear in input tokens |
| Prefill latency | Roughly linear (with flash-attention); quadratic historically |
| Generation latency per token | Grows with KV cache size |
| Memory per request | Linear in context |
| Throughput (batching) | Drops as contexts grow |

## 5. Lost in the Middle

Liu et al. 2023: LLMs attend to the start and end of the context more than the middle. A key fact buried in the middle of a 100K-token context is often missed.

Implications:

- **Put the most important content near the top and bottom**, not the middle.
- **Explicit instructions go at the top** (system prompt) and can be repeated briefly just before the final user turn.
- **Retrieval should produce a small number of highly relevant passages**, not a stack of maybe-relevant ones.

## 6. Strategies When Your Task Exceeds Context

### 6a. Chunking + retrieval (RAG)

Full deep dive in `09-rag/`. Summary: split documents into chunks, retrieve only what's relevant, include 3–10 passages rather than the whole corpus.

### 6b. Summarization

Compress upstream. Two patterns:

- **Map-reduce** — summarize each chunk; combine summaries. Good for factual extraction.
- **Refine** — summarize chunk 1; iteratively update the summary with each next chunk. Good for narrative coherence.

### 6c. Hierarchical retrieval

Retrieve section-level summaries first; then pull full text only for the most relevant sections.

### 6d. Long-context with smart packing

If you genuinely have 100K tokens of relevant material:

1. Place a concise instruction + schema at the top.
2. Include the full context in the middle, structured with section headers.
3. Repeat the key instruction just before the final user turn.
4. Ask for citations back to section numbers — forces the model to actually attend.

### 6e. Memory systems (agents)

Long-running agents externalize context into a vector store and retrieve only what's relevant each turn. See `10-agentic-ai/05_memory_systems`.

### 6f. Caching

Modern APIs cache system prompts and long documents across requests. Anthropic prompt caching can cut long-context cost by 90% for repeated prefixes. Covered in `14-llmops-deployment/08_caching`.

## 7. Context Packing Tradeoffs

At a fixed budget of `B` tokens:

| Strategy | Pros | Cons |
| --- | --- | --- |
| Whole doc | No info loss; zero pipeline complexity | Lost-in-middle; expensive; slow |
| Retrieval (5 chunks) | Cheap, fast, often better quality | Retrieval must work |
| Summary of doc | Cheap, fast | Summary loses details |
| Doc + TOC | Good for navigation-style tasks | Still expensive on the doc |
| Hybrid (summary + retrieval) | Often best | Most pipeline complexity |

Default: retrieval. Move to whole-doc only when retrieval eval shows you need it.

## 8. Architect Heuristics

- **Keep system prompts compact.** Reuse caching friendly-prefixes.
- **Order matters.** Instruction → context → input → re-instruction works well.
- **Measure at 25%, 50%, 75%, 100% of advertised context** to find your effective ceiling.
- **Monitor the p99 context usage in production**; surprising growth indicates a prompt leak or unbounded chat history.
- **Bound chat history** explicitly. Don't let it grow forever.
- **Long-context prompts are a last resort.** Cheaper options first.

## 9. When Long Context Actually Wins

- Single-document reasoning (analyze this 100-page report)
- Code understanding across files
- Legal / compliance where every clause matters
- Agent scratchpads that genuinely reference earlier turns

For chat-with-your-docs, RAG usually beats long-context on cost and often on quality (better recall from targeted retrieval than lost-in-middle with everything loaded).

## Prereqs / Connections

- `02_transformer_architecture` — attention is O(n²)
- `03_tokenization` — context is measured in tokens
- `09-rag/` — retrieval as a context strategy
- `14-llmops-deployment/08_caching` — prompt caching

## Further Reading

- Liu et al. 2023, "Lost in the Middle: How Language Models Use Long Contexts"
- Hsieh et al. 2024, "RULER: What's the Real Context Size of Your Long-Context Language Models?"
- Anthropic, "Prompt Caching" docs
- Google DeepMind, "Gemini 1.5 Technical Report" (long-context architecture)
- Dao et al., "FlashAttention" papers (v1, v2, v3)
