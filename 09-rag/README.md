# Retrieval-Augmented Generation (RAG) Module

How to ground LLMs in your data. Covers ingestion, retrieval, reranking, advanced patterns, and failure modes.

## 🚀 Roadmap

### 1. RAG Architecture Overview & Failure Modes
- Classic RAG flow
- Where RAG breaks (retrieval, grounding, generation)
- When NOT to use RAG

### 2. Document Loaders & Parsers
- Builds on `07-data-engineering-for-ai/03_parsing_extraction`
- LangChain/LlamaIndex loader survey

### 3. Chunking Strategies
- Fixed-size, recursive
- Semantic chunking
- Document-aware (headings, sections)

### 4. Embedding Model Selection
- OpenAI, Cohere, BGE, E5, Voyage
- MTEB leaderboard reading
- Cost vs quality

### 5. Retrieval Patterns
- Dense retrieval
- Sparse / BM25
- Hybrid retrieval

### 6. Reranking
- Cross-encoders
- Cohere Rerank, bge-reranker
- Two-stage retrieval economics

### 7. Query Transformations
- HyDE (hypothetical document embeddings)
- Multi-query expansion
- Step-back prompting

### 8. Context Compression & Packing
- LLMLingua and friends
- Token budget allocation
- Lost-in-the-middle mitigation

### 9. Citations & Source Attribution
- Span-level citations
- Footnote-style references
- Evaluating citation quality

### 10. RAG Evaluation (Intro)
- Faithfulness, relevance, recall
- Deep dive in `11-evaluation/`

### 11. Advanced RAG
- Parent-child chunking
- Contextual retrieval (Anthropic)
- Graph RAG

### 12. Caching & Streaming in RAG
- Embedding cache, result cache, semantic cache
- Prereq: `03-design-patterns/14_caching`
- Streaming answer with streaming retrieval

### 13. Multi-Modal RAG
- Image + text retrieval
- Table and figure extraction
- Multi-modal reranking

### 14. Hands-On: Production RAG End-to-End
- Ingestion → vector store → retrieval → rerank → generation
- Eval harness wired in
