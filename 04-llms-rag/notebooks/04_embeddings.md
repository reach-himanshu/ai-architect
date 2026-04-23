# LLMs 04: Embeddings

Embeddings are the bridge between text and math. They are the foundation of semantic search, RAG, clustering, and recommendation systems.

## 1. What is an Embedding?

An embedding is a dense vector (list of numbers) that represents the semantic meaning of text in a high-dimensional space.

```
"king"    → [0.23, -0.14, 0.87, ..., 0.45]  (1536 dimensions)
"queen"   → [0.21, -0.12, 0.85, ..., 0.47]  (similar!)
"car"     → [-0.54, 0.33, -0.12, ..., 0.89] (very different)
```

Key property: **semantically similar text has similar vectors**.

---

## 2. Similarity Metrics

| Metric | Formula | Use Case |
| :--- | :--- | :--- |
| **Cosine Similarity** | `cos(θ) = A·B / (‖A‖‖B‖)` | Text similarity (most common) |
| **Dot Product** | `A · B` | When vectors are normalized |
| **Euclidean Distance** | `‖A - B‖` | Geometric distance |
| **Manhattan Distance** | `Σ|Aᵢ - Bᵢ|` | Sparse vectors |

Cosine similarity ranges from -1 (opposite) to 1 (identical).

---

## 3. Popular Embedding Models

| Model | Dimensions | Context | Best For |
| :--- | :--- | :--- | :--- |
| `text-embedding-3-small` (OpenAI) | 1536 | 8191 tokens | General, cheap |
| `text-embedding-3-large` (OpenAI) | 3072 | 8191 tokens | High accuracy |
| `text-embedding-ada-002` (OpenAI) | 1536 | 8191 tokens | Legacy, still good |
| `embed-english-v3.0` (Cohere) | 1024 | 512 tokens | Retrieval-optimized |
| `all-MiniLM-L6-v2` (Sentence Transformers) | 384 | 256 tokens | Local, fast, free |
| `nomic-embed-text` (Nomic) | 768 | 8192 tokens | Open, large context |

---

## 4. Choosing an Embedding Model

Consider these factors:

1.  **Dimensionality**: Higher = more expressive, more storage, slower.
2.  **Context length**: Must fit your document chunks.
3.  **Multilingual**: Needed if documents are not English.
4.  **Latency**: Local models (SentenceTransformers) have zero API latency.
5.  **Cost**: OpenAI charges per token; local models are free after setup.

---

## 5. The Embedding Pipeline

```
Raw Text
    ↓
Preprocessing (clean, normalize)
    ↓
Chunking (split to fit context window)
    ↓
Embedding Model
    ↓
Vector [0.23, -0.14, ..., 0.45]
    ↓
Vector Store (store with metadata)
```

---

## 6. Embedding Gotchas

*   **Chunk boundary problem**: Splitting mid-sentence loses context. Use sentence-aware splitters.
*   **Embedding drift**: If you change models, you must re-embed all documents.
*   **Query vs. document asymmetry**: Some models use different encoders for queries vs. documents (bi-encoders).
*   **Stale embeddings**: Embeddings don't update when documents change — rebuild pipeline is needed.
*   **Normalization**: Always normalize vectors if using dot product similarity.

---

## 7. Dimensionality Reduction for Visualization

High-dimensional embeddings can be reduced to 2D/3D using:
*   **PCA**: Fast, linear, preserves global structure.
*   **UMAP**: Non-linear, preserves local clusters. Best for visualization.
*   **t-SNE**: Good for clusters, slow on large datasets.
