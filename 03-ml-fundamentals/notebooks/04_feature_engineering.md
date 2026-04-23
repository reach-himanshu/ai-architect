# ML 04: Feature Engineering

Features are the inputs to ML models. The quality of your features determines the ceiling of your model's performance. "Garbage in, garbage out."

## 1. Feature Types

| Type | Examples | Encoding |
| :--- | :--- | :--- |
| **Numerical continuous** | Age, price, temperature | Normalize/standardize |
| **Numerical discrete** | Count, year | May treat as categorical |
| **Categorical nominal** | Color, country, category | One-hot, label, target encoding |
| **Categorical ordinal** | Size (S/M/L), rating (1-5) | Label encoding (preserves order) |
| **Text** | Reviews, documents | TF-IDF, embeddings |
| **Date/time** | Timestamp | Extract components, cyclical encoding |
| **Boolean** | Has_feature, is_active | 0/1 |

---

## 2. Normalization vs. Standardization

| Method | Formula | When to Use |
| :--- | :--- | :--- |
| **Min-Max (Normalization)** | (x - min) / (max - min) → [0,1] | When you know the range, neural networks |
| **Z-score (Standardization)** | (x - μ) / σ → mean=0, std=1 | When distribution matters, most ML models |
| **Robust scaling** | (x - median) / IQR | When outliers are present |
| **Log transform** | log(x + 1) | Right-skewed distributions |

---

## 3. Categorical Encoding

**One-Hot Encoding**: Create binary column for each category.
```
Color: red  → [1, 0, 0]
Color: blue → [0, 1, 0]
Color: green→ [0, 0, 1]
```
Problem: High cardinality → too many columns.

**Target Encoding**: Replace category with mean of target variable.
```
Country: "US" → mean(revenue for US customers) = 450
Country: "UK" → mean(revenue for UK customers) = 320
```
Problem: Data leakage if done on training set — use cross-validated target encoding.

**Embedding**: For LLMs, categorical variables go directly into embeddings.

---

## 4. Handling Missing Data

| Strategy | When to Use | Notes |
| :--- | :--- | :--- |
| **Drop rows** | Few missing, random | Loses data |
| **Drop column** | > 50% missing | Column not useful |
| **Mean/Median imputation** | Numerical, missing at random | Simple, distorts distribution |
| **Mode imputation** | Categorical | Simple |
| **Model imputation** | Complex patterns | Use another model to predict missing |
| **Indicator + impute** | When missingness is informative | Add "is_missing" binary column |

---

## 5. Feature Selection

Too many features → curse of dimensionality, overfitting, slow training.

| Method | Type | Description |
| :--- | :--- | :--- |
| **Variance threshold** | Filter | Remove near-constant features |
| **Correlation** | Filter | Remove highly correlated features |
| **Mutual information** | Filter | Measure feature-target dependency |
| **L1 regularization** | Embedded | Model zeroes out irrelevant weights |
| **Feature importance** | Embedded | Tree models rank features |
| **Recursive Feature Elimination** | Wrapper | Iteratively remove least important |

---

## 6. Feature Engineering for LLM Inputs

When building RAG/LLM pipelines, "feature engineering" means:

*   **Chunk metadata**: Add title, section, date, source to each chunk.
*   **Query reformulation**: Expand or clarify queries before retrieval.
*   **Context selection**: Choose which retrieved chunks to include.
*   **Prompt features**: Format, persona, constraints in the system prompt.
*   **Structured schemas**: Define the exact JSON shape you want out.

---

## 7. Data Leakage — The Silent Model Killer

Data leakage occurs when information from the test set contaminates training.

Common causes:
*   **Temporal leakage**: Using future data to predict the past (normalize on all data before splitting).
*   **Group leakage**: The same user/entity appears in both train and test.
*   **Target encoding**: Computing target statistics on the full dataset.
*   **Scaling**: Fitting scaler on all data before splitting.

Always: **Fit preprocessing on train set only. Apply to val/test.**
