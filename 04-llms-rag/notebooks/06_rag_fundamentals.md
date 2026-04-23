# LLMs 06: RAG Fundamentals

RAG (Retrieval-Augmented Generation) grounds LLM responses in your own data. It's the most important pattern for enterprise AI.

## 1. Why RAG?

| Problem | Without RAG | With RAG |
| :--- | :--- | :--- |
| LLM knowledge cutoff | Outdated answers | Live data |
| Hallucinations | Plausible-sounding lies | Grounded in sources |
| Proprietary data | Model doesn't know it | Retrieved from your KB |
| Cost | Fine-tuning is expensive | Retrieval is cheap |
| Transparency | No sources cited | Can cite documents |

---

## 2. The Standard RAG Pipeline

```
User Query
    │
    ▼
[1. RETRIEVE]
Query Embedding → Vector DB Search → Top-K Documents
    │
    ▼
[2. AUGMENT]
Build Context: System Prompt + Retrieved Docs + User Query
    │
    ▼
[3. GENERATE]
LLM → Grounded Response
```

---

## 3. Chunking Strategies

How you split documents dramatically affects retrieval quality.

| Strategy | How it Works | Best For |
| :--- | :--- | :--- |
| **Fixed-size** | Split every N tokens (e.g., 512) | Simple baseline |
| **Sentence** | Split at sentence boundaries | General text |
| **Recursive** | Try `\n\n`, then `\n`, then `.`, etc. | Mixed content |
| **Semantic** | Split where topics change (via embeddings) | Dense, topic-shifting text |
| **Document-aware** | Respect headers, sections (Markdown/PDF) | Structured documents |
| **Parent-Child** | Small chunks for retrieval, large for context | Best of both worlds |

**Rule of thumb**: Chunks should be no larger than 1/4 of your context window.

---

## 4. Retrieval Patterns

**Naive RAG**: Embed query → search → top-k → LLM.

**Multi-Query**: Generate multiple query variants → search each → deduplicate → LLM.
```python
query_variants = llm("Generate 3 search queries for: {user_query}")
# Searches different aspects, better coverage
```

**HyDE (Hypothetical Document Embedding)**: Ask LLM to generate a hypothetical answer → embed that → search.
```python
hypothetical_doc = llm("Write a passage that would answer: {user_query}")
results = vector_search(embed(hypothetical_doc))
```

**Step-back Prompting**: Abstract the query before searching.
```python
abstract_query = llm("What is the broader topic behind: {user_query}?")
results = vector_search(embed(abstract_query))
```

---

## 5. Reranking

First-pass retrieval (ANN) is fast but imprecise. A reranker re-scores candidates with a cross-encoder for better precision.

```
ANN Retrieval (top-20) → Reranker → top-5 → LLM
```

Popular rerankers:
*   **Cohere Rerank**: Best commercial option.
*   **CrossEncoder (sentence-transformers)**: Open source, runs locally.
*   **BGE Reranker**: Strong open-source option.

Reranking adds latency (~100ms) but significantly improves answer quality.

---

## 6. Context Window Assembly

After retrieval, you need to assemble context carefully:

```python
def build_rag_prompt(query: str, retrieved_docs: list[dict]) -> list[dict]:
    context_parts = []
    for i, doc in enumerate(retrieved_docs, 1):
        context_parts.append(f"[Source {i}]\n{doc['text']}\n")

    context = "\n".join(context_parts)

    return [
        {
            "role": "system",
            "content": (
                "You are a helpful assistant. Answer based ONLY on the provided sources. "
                "If the answer is not in the sources, say 'I don't have that information.' "
                "Always cite which source you used (e.g., [Source 1])."
            ),
        },
        {
            "role": "user",
            "content": f"Sources:\n{context}\n\nQuestion: {query}",
        },
    ]
```

---

## 7. Common RAG Failure Modes

| Failure | Cause | Fix |
| :--- | :--- | :--- |
| Wrong chunks retrieved | Poor chunking, bad embedding | Better chunking, reranking |
| Answer ignores retrieved context | Prompt not strict enough | Add "ONLY use the sources" |
| Chunks cut off mid-thought | Fixed-size chunking | Sentence-aware chunking |
| No relevant chunks found | Query-document mismatch | Multi-query, HyDE |
| Hallucinations despite RAG | LLM "knows" the answer already | Stricter grounding prompt |
| Slow retrieval | Large index, no filtering | Pre-filter, HNSW index |
