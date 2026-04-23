# LLMs 09: LLM Evaluation

You cannot improve what you cannot measure. LLM evaluation is the discipline that makes AI systems reliable.

## 1. Why Evaluation is Hard

LLMs produce free-form text. Unlike traditional software:
*   There's no single "correct" output.
*   Outputs are contextual and probabilistic.
*   Human evaluation is expensive and inconsistent.
*   Metrics can be gamed.

**LLM-as-Judge**: Use a powerful LLM (GPT-4o, Claude) to evaluate outputs. The current industry standard for scalable evaluation.

---

## 2. RAG Evaluation Framework (RAGAS)

RAGAS evaluates RAG pipelines on four dimensions:

| Metric | Question | Range |
| :--- | :--- | :--- |
| **Faithfulness** | Is the answer grounded in the retrieved context? | 0–1 |
| **Answer Relevancy** | Does the answer address the question? | 0–1 |
| **Context Precision** | Were the retrieved chunks actually useful? | 0–1 |
| **Context Recall** | Were all relevant chunks retrieved? | 0–1 |

```python
from ragas import evaluate
from ragas.metrics import faithfulness, answer_relevancy, context_precision, context_recall

results = evaluate(
    dataset=eval_dataset,
    metrics=[faithfulness, answer_relevancy, context_precision, context_recall],
)
print(results)
# {'faithfulness': 0.85, 'answer_relevancy': 0.92, 'context_precision': 0.78, ...}
```

---

## 3. Faithfulness Evaluation (LLM-as-Judge)

```python
FAITHFULNESS_PROMPT = """
You are evaluating an AI answer for faithfulness.

Context (retrieved documents):
{context}

Question: {question}
Answer: {answer}

Score the answer from 0 to 1:
- 1.0: Every claim in the answer is directly supported by the context.
- 0.5: Some claims are supported, others are extrapolated.
- 0.0: The answer contains claims not found in the context (hallucination).

Return only a JSON: {{"score": float, "reasoning": "brief explanation"}}
"""
```

---

## 4. Hallucination Detection Patterns

| Pattern | Description |
| :--- | :--- |
| **Self-check** | Ask model: "Does your answer contain any unsupported claims?" |
| **Entailment check** | NLI model checks if answer is entailed by context |
| **Atomic fact check** | Break answer into atomic facts, verify each against sources |
| **LLM-as-Judge** | Separate judge LLM evaluates for groundedness |
| **Perplexity** | High perplexity for certain claims may indicate confabulation |

---

## 5. Building an Eval Dataset

A good eval dataset has:
*   **Questions**: Real or representative user queries.
*   **Ground truth answers**: Human-verified correct answers.
*   **Reference contexts**: The ideal retrieved documents.
*   **Metadata**: Difficulty, category, domain.

Sources for eval data:
*   Sample from production queries (with user consent).
*   Use LLMs to generate synthetic Q&A pairs from documents.
*   Human annotation (most accurate, most expensive).

---

## 6. Regression Testing for LLMs

LLMs behave differently across versions. Treat eval like unit tests:

```python
EVAL_BASELINE = {
    "faithfulness": 0.85,
    "answer_relevancy": 0.90,
    "context_precision": 0.78,
}

def run_regression_check(new_results: dict, baseline: dict, tolerance: float = 0.03) -> bool:
    passed = True
    for metric, baseline_score in baseline.items():
        new_score = new_results.get(metric, 0)
        if new_score < baseline_score - tolerance:
            print(f"REGRESSION: {metric} dropped {baseline_score:.2f} → {new_score:.2f}")
            passed = False
    return passed
```

---

## 7. The Eval Stack

| Layer | Tool | Purpose |
| :--- | :--- | :--- |
| **Offline eval** | RAGAS, Deepeval, Promptfoo | Batch evaluation before deployment |
| **Online monitoring** | LangSmith, Arize, Helicone | Production monitoring |
| **Human review** | Label Studio, Argilla | Ground truth, edge cases |
| **A/B testing** | Custom or platform | Compare model versions |
| **Tracing** | LangSmith, Phoenix | Debug individual traces |
