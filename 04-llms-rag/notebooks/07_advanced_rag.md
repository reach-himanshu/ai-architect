# LLMs 07: Advanced RAG

Basic RAG fails on complex queries. Advanced RAG techniques dramatically improve retrieval precision, coverage, and latency.

## 1. Why Basic RAG Falls Short

| Problem | Example | Naive RAG Fails Because |
| :--- | :--- | :--- |
| Multi-hop question | "Who is the CEO of the company that acquired OpenAI?" | Single retrieval can't chain facts |
| Ambiguous query | "Tell me about Python" (language or snake?) | Retrieves random chunks |
| Vocabulary mismatch | Docs say "myocardial infarction", user asks "heart attack" | Low embedding similarity |
| Missing context | Answer spans 3 paragraphs | Chunk boundary cuts off key info |

---

## 2. Query Transformation Techniques

### Multi-Query Retrieval
Generate multiple query variants → search each → merge results.

```python
system = "You are a query expansion expert."
prompt = f"""
Generate 3 different search queries to retrieve documents for:
"{user_query}"

Return only the queries, one per line.
"""
variants = llm(system, prompt).strip().split("\n")
all_results = [vector_search(q) for q in variants]
merged = deduplicate(flatten(all_results))
```

### HyDE (Hypothetical Document Embedding)
Ask LLM to write a hypothetical answer, then embed and search.

```python
prompt = f"""
Write a 2-sentence passage that would directly answer:
"{user_query}"
"""
hypothetical = llm(prompt)
results = vector_search(embed(hypothetical))  # Embed the hypothetical doc
```

**Why it works**: The hypothetical answer lives in the same embedding space as real documents, bridging vocabulary gaps.

### Step-Back Prompting
Abstract the question before searching.

```python
prompt = f"""
Given the specific question: "{user_query}"
What is the general topic or concept that would need to be understood to answer it?
"""
abstract = llm(prompt)
results = vector_search(embed(abstract))
```

---

## 3. Parent-Child Chunking

Split documents into two levels:
*   **Child chunks** (small, ~100 tokens): Used for precise retrieval.
*   **Parent chunks** (large, ~500 tokens): Retrieved and sent to LLM for full context.

```python
def parent_child_index(document, parent_size=500, child_size=100):
    parents = chunk(document, size=parent_size)
    for parent in parents:
        children = chunk(parent.text, size=child_size)
        for child in children:
            child.parent_id = parent.id  # Link child → parent
    return parents, children

# At query time:
# 1. Search children (precise)
# 2. Retrieve their parent chunks (full context)
```

---

## 4. Contextual Retrieval (Anthropic)

Before chunking, prepend context to each chunk using an LLM.

```python
def contextualize_chunk(document: str, chunk: str) -> str:
    prompt = f"""
Document:
{document}

Chunk:
{chunk}

Provide a short (1-2 sentence) context that situates this chunk within the document.
Output ONLY the context, nothing else.
"""
    context = llm(prompt)
    return f"{context}\n\n{chunk}"
```

This is expensive (1 LLM call per chunk) but can be done at index time using batch/cheap models.

---

## 5. Reranking

First-pass retrieval (ANN) gets recall. Reranking gets precision.

```
ANN Top-20 → Cross-Encoder Reranker → Top-5 → LLM
```

*   **Bi-encoder** (ANN): Fast, approximate. Each doc encoded independently.
*   **Cross-encoder** (reranker): Slow, precise. Query and doc encoded together — sees interactions.

```python
from sentence_transformers import CrossEncoder
reranker = CrossEncoder("cross-encoder/ms-marco-MiniLM-L-6-v2")

pairs = [(query, doc["text"]) for doc in candidates]
scores = reranker.predict(pairs)
reranked = sorted(zip(scores, candidates), reverse=True)
top_5 = [doc for _, doc in reranked[:5]]
```

---

## 6. RAPTOR (Recursive Abstractive Processing)

For very long documents: cluster → summarize → index at multiple levels.

```
Leaves (raw chunks)
    ↓ cluster
Layer 1 summaries
    ↓ cluster
Layer 2 summaries
    ↓ cluster
Root summary
```

Search across all layers, then synthesize.

---

## 7. Evaluation Metrics

| Metric | What it Measures | Tool |
| :--- | :--- | :--- |
| **Faithfulness** | Is answer grounded in retrieved context? | RAGAS |
| **Answer Relevancy** | Does answer address the question? | RAGAS |
| **Context Precision** | Were retrieved chunks actually useful? | RAGAS |
| **Context Recall** | Were all relevant chunks retrieved? | RAGAS |
| **MRR** | Mean Reciprocal Rank of correct chunk | Manual |
| **nDCG** | Normalized Discounted Cumulative Gain | Manual |
