# ML 02: Embeddings & Representations

Embeddings are the universal currency of modern AI — they turn any data (text, images, code, users) into numbers that models can reason about.

## 1. What Problem Do Embeddings Solve?

Raw data is categorical and sparse. ML models need dense, continuous representations.

```
"cat" → one-hot: [0,0,1,0,...,0]  (10,000 dims, mostly zeros, no semantic info)
"cat" → embedding: [0.3, -0.8, 0.5, ..., 0.2]  (300 dims, semantically meaningful)
```

Embeddings encode:
*   **Semantic similarity**: "cat" and "kitten" have similar vectors.
*   **Analogies**: king - man + woman ≈ queen (famous Word2Vec result).
*   **Relationships**: "Paris" - "France" ≈ "Tokyo" - "Japan".

---

## 2. How Embeddings Are Learned

Embeddings emerge as a side effect of training on a task:
*   **Word2Vec**: Predict surrounding words → similar words get similar vectors.
*   **BERT**: Predict masked tokens → context-aware representations.
*   **LLM**: Predict next token → rich semantic representations at every layer.

The key insight: **the model learns to compress meaning into dense vectors** because that's what helps minimize the training loss.

---

## 3. Types of Embeddings

| Type | Model Example | Use Case |
| :--- | :--- | :--- |
| **Word** | Word2Vec, GloVe | Legacy NLP, fast |
| **Sentence** | Sentence-BERT | Semantic similarity, clustering |
| **Document** | OpenAI ada-002 | RAG, search |
| **Code** | CodeBERT, StarCoder | Code search |
| **Image** | CLIP | Multi-modal search |
| **User** | Collaborative filtering | Recommendations |
| **Graph** | Node2Vec, GraphSAGE | Link prediction |

---

## 4. Transfer Learning

Pre-training embeddings on large corpora, then using them for downstream tasks:

```
Large Corpus (Wikipedia, Books)
    ↓
Pre-train (self-supervised)
    ↓
General-purpose embedding model
    ↓ Fine-tune on your task
Task-specific model (better with less data)
```

**Why it matters**: You don't need millions of labeled examples. Transfer learning lets you leverage billions of training samples from the internet.

---

## 5. Fine-Tuning for Better Embeddings

When general embeddings don't capture your domain vocabulary:

*   **Contrastive learning**: Train with positive/negative pairs — similar items pulled together, dissimilar pushed apart.
*   **Sentence pair training**: "This document answers this query" → better retrieval embeddings.
*   **LoRA on embedding layers**: Efficient adaptation without full retraining.

---

## 6. Dimensionality Reduction

High-dimensional embeddings can be compressed for speed or visualization:

| Method | Output | Preserves | Speed |
| :--- | :--- | :--- | :--- |
| **PCA** | Linear projection | Global variance | Fast |
| **UMAP** | Non-linear 2D/3D | Local + global structure | Medium |
| **t-SNE** | Non-linear 2D | Local clusters | Slow |
| **Matryoshka** | Variable-length | Semantic meaning | Training-time |

**Matryoshka Representation Learning (MRL)**: A single model can output embeddings at multiple sizes (e.g., 256, 512, 1536) — the smaller prefix retains most of the information.

---

## 7. When Embeddings Fail

*   **Domain shift**: Medical embeddings trained on Wikipedia miss clinical jargon.
*   **Polysemy**: "bank" (river vs. financial) — context-free embeddings fail.
*   **Rare tokens**: Uncommon words get poor embeddings due to sparse training signal.
*   **Language gap**: Cross-lingual search requires multilingual models.
