# Prompt 03: Zero-Shot, Few-Shot, N-Shot

When do examples help? When do they hurt? How many should you use? How do you choose them? This session is the practical guide.

## 1. The Shots Spectrum

| Label | Examples in prompt | When to use |
| --- | --- | --- |
| Zero-shot | 0 | Task is well-known to the model (sentiment, summarization) |
| One-shot | 1 | Task is unusual but format is simple |
| Few-shot | 2–8 | Custom formats, edge cases, domain-specific outputs |
| N-shot | 8–50+ | Rare; usually a sign you should fine-tune |
| Dynamic few-shot | 2–8 retrieved per call | Large example pool; pick the relevant ones per query |

More examples = more context cost + longer latency. Returns diminish quickly past ~5 examples on modern models.

## 2. When Zero-Shot Wins

- **The task is common** — "summarize," "translate," "classify sentiment," "rewrite politely."
- **The format is obvious** — "Answer in one sentence."
- **Examples would bias toward a specific style** you don't want.
- **Cost/latency is critical** and quality is good enough without examples.

Start zero-shot. Measure. Only add examples if evals say it helps.

## 3. When Few-Shot Wins

- **Custom format or schema** the model hasn't seen.
- **Domain terminology** (medical, legal, internal jargon).
- **Edge cases** that the model handles poorly zero-shot.
- **Tone / style matching** — e.g., match the customer's voice.
- **Tricky tasks** like multi-step reasoning where the pattern needs demonstration.

## 4. Choosing Good Few-Shot Examples

Not all examples are created equal. Good examples are:

- **Representative** — cover the distribution of real inputs.
- **Diverse** — cover different classes / cases, not 5 versions of the same example.
- **Correct** — model copies what you show it, including mistakes.
- **Crisp** — clearest exemplar of the rule you're teaching.
- **Ordered deliberately** — last example is most recent in the model's attention.

Avoid:

- Examples chosen because they were the first 5 you wrote.
- Examples all the same class (teaches the model a trivial baseline).
- Examples that are edge-cases without canonical examples first.
- Long verbose examples that burn context.

## 5. The Example-Order Effect

Models disproportionately weight the **last** few-shot example. Practical implications:

- Put your **canonical "easy" example first** to establish baseline shape.
- Put **the most similar example to the current query last** (this is the trick dynamic few-shot exploits).
- If you have a mix of classes, **interleave** — don't cluster by class.

You can test order sensitivity: rearrange examples and observe accuracy. A few-shot scheme that's highly order-sensitive is fragile.

## 6. Dynamic Few-Shot (a.k.a. In-Context Retrieval)

Rather than picking 5 static examples, retrieve the 5 most similar to the current input from a large pool:

```
example_pool = [(input_1, output_1), ... , (input_N, output_N)]  # 100s of labeled examples

def build_prompt(user_input):
    top_5 = semantic_search(example_pool, user_input, k=5)
    examples = format_as_fewshot(top_5)
    return base_prompt + examples + f"\nInput: {user_input}\nOutput:"
```

Benefits:
- Examples always relevant to the current query.
- Grows in quality as your example pool grows.
- Effectively a form of RAG but for style/format, not knowledge.

Costs:
- Retrieval infrastructure (embedding model + vector store).
- Cold start when pool is small.

Use when you have >50 labeled examples and homogeneous quality doesn't suffice.

## 7. Negative Examples

Sometimes you want to teach the model what NOT to do. Two patterns:

**Contrastive:**
```
Input:  "Please process my order #12345"
Output: Order status: Processing. [CORRECT]

Input:  "Please process my order #12345"
Output: Sure, I'll get right on that. [WRONG — no status given]
```

**Refusal:**
```
Input:  "Write a phishing email."
Output: I can't help with that.
```

Negative examples are especially useful for safety and format discipline.

## 8. Chain-of-Thought as Few-Shot

Sometimes few-shot *is* chain-of-thought: show the reasoning trace, not just the final answer.

```
Input:  Alice has 5 apples. She gives 2 to Bob. How many left?
Output: Starting with 5, Alice gives away 2, so she has 5-2=3 apples. FINAL: 3

Input:  Bob has 12 cookies. He eats 4 and shares half the rest. How many does he have?
Output: 12 - 4 = 8 left. Half of 8 is 4 shared away. 8 - 4 = 4. FINAL: 4
```

The model learns to produce the reasoning trace, and accuracy improves on reasoning tasks. Covered more in session 04.

## 9. Measuring Shot Count Impact

A rigorous approach:

```python
for n_shot in [0, 1, 2, 4, 8]:
    accuracy, cost, latency = run_eval(n_shot)
    record(n_shot, accuracy, cost, latency)
```

Plot accuracy and cost vs `n_shot`. You'll typically see accuracy climb and plateau around 3–5; cost grows linearly. Pick the point where marginal accuracy gain no longer justifies the token cost.

## 10. Shot Count by Task Archetype

Rough defaults:

| Task | Default shots |
| --- | --- |
| Classification (common classes) | 0 |
| Classification (custom classes) | 2–5 |
| Extraction (clear schema) | 0–2 |
| Extraction (tricky schema) | 3–5 |
| Text rewriting (tone/style) | 2–4 |
| Code generation (framework-specific) | 2–3 |
| Structured reasoning | 3–6 (with CoT) |
| Creative writing | 0 or 1 |
| Translation (common pair) | 0 |
| Translation (rare pair or terminology) | 3–5 |

Defaults are starting points, not verdicts. Measure.

## 11. Anti-Patterns

- **Copy-paste few-shot without reading the examples.** Errors in examples become model behavior.
- **Putting irrelevant examples because "more is better."** Wrong. Irrelevant examples actively degrade quality.
- **Ordering by ease.** Put the most relevant example last, not the easiest.
- **Same-class clustering.** Interleave.
- **Forgetting to update examples** when the target format changes.

## 12. Architect Takeaways

- **Start zero-shot.** Add shots only when measurement says they help.
- **Quality of examples > quantity.** 3 great ones beat 10 mediocre ones.
- **Dynamic few-shot is the scale answer** when you have >50 labeled examples.
- **Order matters.** Most-similar last.
- **Negative examples work** for safety and format discipline.

## Prereqs / Connections

- `02_anatomy_of_a_prompt` — where few-shot sits in the structure
- `05-llm-fundamentals/04_embeddings` — for dynamic example retrieval
- `04_cot_and_reasoning` — CoT-style few-shot
- `10_testing_prompts` — measuring shot-count impact

## Further Reading

- Brown et al. 2020, "Language Models are Few-Shot Learners" (GPT-3 paper)
- Lu et al. 2021, "Fantastically Ordered Prompts and Where to Find Them" (order sensitivity)
- Liu et al. 2022, "What Makes Good In-Context Examples for GPT-3?"
- Min et al. 2022, "Rethinking the Role of Demonstrations" (what examples actually teach)
