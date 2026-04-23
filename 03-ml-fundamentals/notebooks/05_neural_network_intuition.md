# ML 05: Neural Network Intuition

You don't need to implement backpropagation — you need to understand how architecture choices affect behavior.

## 1. The Neural Network Mental Model

A neural network is a composition of functions:

```
Input → [Layer 1: Linear + Activation] → [Layer 2: Linear + Activation] → Output
```

Each layer learns to extract increasingly abstract features:
*   Layer 1: Edges, simple patterns.
*   Layer 2: Shapes, combinations.
*   Layer 3+: Concepts, semantics.

---

## 2. Activation Functions

The activation function adds non-linearity — without it, stacking layers is equivalent to a single linear layer.

| Function | Formula | Range | Use Case |
| :--- | :--- | :--- | :--- |
| **ReLU** | max(0, x) | [0, ∞) | Default for hidden layers |
| **GELU** | x·Φ(x) (approx) | (-∞, ∞) | Transformers, BERT, GPT |
| **Sigmoid** | 1/(1+e⁻ˣ) | (0, 1) | Binary output |
| **Softmax** | eˣⁱ/Σeˣʲ | (0, 1), sum=1 | Multi-class output |
| **Tanh** | (eˣ-e⁻ˣ)/(eˣ+e⁻ˣ) | (-1, 1) | RNNs, some hidden layers |

---

## 3. Forward Pass (What Happens at Inference)

```python
# Simplified single-layer forward pass
def forward(X, W, b, activation="relu"):
    Z = W @ X + b      # Linear transformation
    if activation == "relu":
        A = max(0, Z)  # Apply activation
    elif activation == "sigmoid":
        A = 1 / (1 + exp(-Z))
    return A
```

For inference (prediction), this is all that runs. No gradients needed.

---

## 4. Training: Backpropagation Intuition

Training adjusts weights to minimize loss:

```
Forward pass: Compute prediction → Compute loss
Backward pass: Compute how each weight contributed to loss
Update: Move weights in direction that reduces loss
```

The gradient tells you the slope — which direction to move. Learning rate controls step size.

---

## 5. Architecture Trade-offs

| Choice | Effect |
| :--- | :--- |
| More layers (depth) | More abstract representations, harder to train |
| Wider layers (neurons) | More capacity per layer, more parameters |
| Dropout (0.1–0.5) | Regularization — randomly zero neurons during training |
| Batch normalization | Stabilizes training, allows higher learning rates |
| Residual connections | Skip connections (like in ResNet, Transformers) — helps gradient flow |
| Attention | Each element attends to all others — key for sequences |

---

## 6. Transformer Architecture (Simplified)

The LLM architecture:

```
Input tokens
    ↓
Token Embeddings + Positional Encodings
    ↓
[Self-Attention → Feed-Forward → Layer Norm] × N layers
    ↓
Output logits (vocabulary probabilities)
```

Each Transformer block has:
*   **Multi-Head Self-Attention**: Each token attends to all others.
*   **Feed-Forward Network**: Two linear layers with GELU activation.
*   **Layer Normalization**: Stabilizes training.
*   **Residual Connection**: Adds input to output — helps gradient flow.

---

## 7. Model Size vs. Capability

| Model Size | Parameters | Approximate Capability |
| :--- | :--- | :--- |
| Tiny | 1M–10M | Simple classification, keyword extraction |
| Small | 100M–1B | Good NLP, code completion |
| Medium | 1B–10B | Strong reasoning, following instructions |
| Large | 10B–100B | Near-human on many benchmarks |
| Frontier | 100B+ | State-of-the-art, expensive to run |

Smaller models that are fine-tuned often beat larger general models on specific tasks.
