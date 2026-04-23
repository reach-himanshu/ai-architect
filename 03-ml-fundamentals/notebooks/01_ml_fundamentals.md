# ML 01: ML Fundamentals

The vocabulary and mental models every AI architect needs.

## 1. Types of Machine Learning

| Type | Description | Examples |
| :--- | :--- | :--- |
| **Supervised** | Learn from labeled examples (X → Y) | Classification, regression |
| **Unsupervised** | Find patterns in unlabeled data | Clustering, dimensionality reduction |
| **Self-supervised** | Labels derived from data itself | LLM pre-training (predict next token) |
| **Reinforcement** | Learn from reward signals | Game playing, RLHF |

Most production AI is supervised or self-supervised.

---

## 2. The ML Workflow

```
Raw Data
   ↓
Data Collection & Cleaning
   ↓
Exploratory Data Analysis (EDA)
   ↓
Feature Engineering
   ↓
Model Selection
   ↓
Training (on train set)
   ↓
Evaluation (on val set) ← tune hyperparameters
   ↓
Final Test (on test set — one time only!)
   ↓
Deployment & Monitoring
```

---

## 3. The Train/Val/Test Split

| Split | Purpose | Typical % |
| :--- | :--- | :--- |
| **Train** | Model learns from this | 70–80% |
| **Validation** | Tune hyperparameters, select model | 10–15% |
| **Test** | Final, unbiased evaluation | 10–15% |

**Critical rule**: Never touch the test set until final evaluation. Otherwise you get an optimistic, misleading score.

---

## 4. Overfitting and Underfitting

```
        Underfitting     Good Fit     Overfitting
Error:
Train:    High             Low           Very Low
Val:      High             Low           High
```

*   **Overfitting**: Model memorizes training data. Fails on new data.
*   **Underfitting**: Model too simple to learn the pattern.

**Fixes for overfitting**:
*   More training data.
*   Regularization (L1, L2, dropout).
*   Simpler model.
*   Data augmentation.

---

## 5. Bias-Variance Trade-off

*   **Bias**: Error from wrong assumptions (underfitting). High-bias models are too simple.
*   **Variance**: Error from sensitivity to training data noise (overfitting). High-variance models memorize noise.

The goal is to minimize total error = Bias² + Variance + Irreducible Noise.

Complex models → low bias, high variance.
Simple models → high bias, low variance.

---

## 6. Key Hyperparameters

| Hyperparameter | What it Controls |
| :--- | :--- |
| **Learning rate** | Step size during gradient descent |
| **Batch size** | Samples per gradient update |
| **Epochs** | Full passes over the training data |
| **Regularization (λ)** | Penalty for model complexity |
| **Number of layers/neurons** | Model capacity |

---

## 7. The Model as a Function

Any ML model is fundamentally a function: `f(X) → Y`.

*   **Linear regression**: `y = wX + b`
*   **Logistic regression**: `y = sigmoid(wX + b)` — for classification
*   **Neural network**: Composition of linear + non-linear functions
*   **LLM**: `y = P(next_token | tokens_1..n; θ)` — a giant probabilistic function

Understanding this framing helps debug model behavior and set expectations.
