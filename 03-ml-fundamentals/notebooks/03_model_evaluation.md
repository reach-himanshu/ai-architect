# ML 03: Model Evaluation

Good models are measured, not assumed. This is the toolkit for evaluating AI systems honestly.

## 1. Classification Metrics

Given a binary classifier (spam/not spam), the confusion matrix shows:

```
                  Predicted
                  Pos    Neg
Actual  Pos  [ TP  |  FN ]
        Neg  [ FP  |  TN ]
```

| Metric | Formula | Meaning |
| :--- | :--- | :--- |
| **Accuracy** | (TP+TN) / Total | % correct overall |
| **Precision** | TP / (TP+FP) | When it says positive, how often right? |
| **Recall** | TP / (TP+FN) | Of all actual positives, how many caught? |
| **F1 Score** | 2 × P×R / (P+R) | Harmonic mean — balances P and R |
| **AUC-ROC** | Area under ROC curve | Overall discriminability (0.5=random, 1=perfect) |

**Precision vs. Recall trade-off**:
*   Medical diagnosis: High recall (miss no cancers), accept low precision (some false alarms).
*   Spam filter: High precision (don't delete real email), accept some spam slipping through.

---

## 2. Regression Metrics

| Metric | Formula | Units | Sensitivity to Outliers |
| :--- | :--- | :--- | :--- |
| **MAE** | mean(|y - ŷ|) | Same as y | Low |
| **MSE** | mean((y - ŷ)²) | y² | High |
| **RMSE** | √MSE | Same as y | High |
| **R²** | 1 - SS_res/SS_tot | Unitless [0,1] | Medium |

**Use RMSE when large errors are especially bad. Use MAE for robust evaluation.**

---

## 3. Common Evaluation Pitfalls

| Pitfall | Description | Fix |
| :--- | :--- | :--- |
| **Class imbalance** | 99% negative class → 99% accuracy for trivial model | Use F1, AUC, or balanced accuracy |
| **Data leakage** | Test data contaminated by train data | Strict temporal/group splits |
| **Metric gaming** | Optimize for metric but not real outcome | Multiple metrics + human eval |
| **Distribution shift** | Test set doesn't match production | Stratified splits, production monitoring |
| **P-hacking** | Run many experiments, report best | Hold-out test set, statistical tests |

---

## 4. Cross-Validation

Instead of a single train/val split, use k-fold:

```
Data: [fold1 | fold2 | fold3 | fold4 | fold5]

Round 1: Train=[2,3,4,5]  Val=[1]
Round 2: Train=[1,3,4,5]  Val=[2]
Round 3: Train=[1,2,4,5]  Val=[3]
...

Final score = mean(scores across all rounds)
```

Better estimate of generalization, especially with small datasets.

---

## 5. Evaluation for LLMs

Traditional metrics don't always work for LLMs:

| LLM Task | Metric | Tool |
| :--- | :--- | :--- |
| Classification | Accuracy, F1 | Standard |
| Generation | BLEU, ROUGE | Weak proxy for quality |
| RAG faithfulness | Faithfulness score | RAGAS |
| Open-ended QA | LLM-as-judge | GPT-4o, Claude |
| Code generation | Pass@k | HumanEval |
| Reasoning | Exact match on CoT | BIG-bench, MMLU |

---

## 6. A/B Testing for ML Models

When deploying a new model, use A/B testing:

```python
def route_request(user_id: str, traffic_split: float = 0.1) -> str:
    """Route x% of traffic to new model."""
    import hashlib
    hash_val = int(hashlib.md5(user_id.encode()).hexdigest(), 16)
    if hash_val % 100 < traffic_split * 100:
        return "model_v2"  # Challenger
    return "model_v1"  # Champion
```

*   Start with 5-10% traffic.
*   Monitor real metrics (not just loss).
*   Use statistical significance testing before full rollout.
