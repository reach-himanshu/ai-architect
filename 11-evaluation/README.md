# Evaluation Module

Eval is what separates demos from products. This module covers how to measure LLM, RAG, and agent quality offline and online.

## 🚀 Roadmap

### 1. Why LLM Evaluation Is Hard
- Open-ended outputs
- Reference-free settings
- Drift and non-determinism

### 2. Reference-Based Metrics
- BLEU, ROUGE, BERTScore
- Where they help, where they mislead

### 3. LLM-as-Judge
- Pairwise vs pointwise
- Rubric design
- Bias mitigation (position, verbosity, self-preference)

### 4. Golden Datasets & Test Curation
- Sampling from production
- Adversarial and edge cases
- Drift detection

### 5. RAG Eval Deep Dive
- RAGAS (faithfulness, answer relevance, context recall)
- TruLens triad
- Retrieval-only metrics (MRR, nDCG)

### 6. Agent Eval
- Trajectory eval
- Tool-use correctness
- Success rate, cost, latency

### 7. Regression Testing & CI for Prompts
- Prompt version pinning
- Snapshot tests
- Blocking vs warning failures

### 8. A/B Testing & Online Eval
- Online metrics selection
- Guardrail metrics
- Statistical significance at low volume

### 9. Eval Tooling
- Braintrust, Langfuse, LangSmith
- OpenAI Evals, DeepEval
- Building your own harness

### 10. Human Eval Protocols
- Rater calibration
- Inter-rater reliability
- Cost and throughput planning
