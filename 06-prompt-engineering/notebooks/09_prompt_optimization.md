# Prompt 09: Prompt Optimization

Writing prompts by hand has a ceiling. **Prompt optimization** treats the prompt itself as a search problem: find the version that maximizes your metric on your eval set. This session covers the techniques, the tooling, and when it's worth the machinery.

## 1. The Core Idea

Given:
- An eval set `{(input, expected)}_N`
- A scoring function `score(output, expected) -> float`
- A prompt template with variables

Find: the prompt (or the few-shot examples, or the instructions, or the structure) that maximizes average score.

The search can be:
- **Manual** — you iterate, rerun eval, keep the best.
- **LLM-assisted** — an optimizer prompt proposes candidate prompts; you eval; repeat.
- **Automated** — frameworks like DSPy optimize prompts + weights jointly.

## 2. When to Invest

Prompt optimization is worth serious effort when:

- You have a **well-defined metric** (accuracy, format validity, judge score).
- You have a **stable eval set** of ≥50 examples.
- You **run the prompt at scale** (thousands+ calls/day) — small gains compound.
- The prompt is **repeated** (production feature, not one-off analysis).

Not worth it when:
- You're prototyping.
- No eval set exists.
- Requirements are shifting weekly.
- The model changes every month and you'd have to re-optimize.

## 3. Human-in-the-Loop Search

The simplest optimization is systematic manual iteration:

```
for variant in [v1, v2, v3, v4]:
    score = run_eval(variant)
    record(variant, score)
keep best
```

Rules for productive manual search:
- **Change one thing per variant.** If you change 3 things and it improves, you don't know which helped.
- **Track every version** with its score.
- **Prefer ablations** — what happens if I remove this sentence?
- **Don't optimize on < 20 examples.** Noise dominates.

## 4. Automatic Prompt Engineer (APE)

Zhou et al. 2022: an LLM proposes candidate prompts; another LLM evaluates them. Essentially:

```
candidates = meta_llm.generate(
    f"Propose 20 prompts for task: {task_description}"
)
for c in candidates:
    scores = [score(task_llm(c, ex.input), ex.expected) for ex in eval_set]
    results[c] = mean(scores)
pick_best(results)
```

Variants:
- Evolutionary: keep the best, mutate/crossover to produce new candidates.
- Beam search: keep top-k, expand each.

Tools: `ape` packages, `AutoPrompt`, DSPy `BootstrapFewShot`.

Budget: ~100–1000 LLM calls for meaningful search. A hundred-dollar investment on a feature that runs millions of times is cheap.

## 5. DSPy: Programs, Not Prompts

Khattab et al. 2023: DSPy treats LLM programs as compilable modules with typed signatures.

```python
import dspy

class Classify(dspy.Signature):
    """Classify a support ticket into one of: billing, account, bug, feature."""
    ticket: str = dspy.InputField()
    category: str = dspy.OutputField(desc="one of: billing|account|bug|feature")

classify = dspy.Predict(Classify)

# Compile: optimize with few-shot examples drawn from train set
compiler = dspy.BootstrapFewShot(metric=exact_match)
compiled = compiler.compile(classify, trainset=examples)
```

What DSPy does:
- Auto-generates few-shot examples from your training set.
- Optimizes instruction wording via heuristic search.
- Can optimize whole pipelines (multi-step prompts), not just single calls.

Strengths: composable, declarative, measurable. Weaknesses: steep learning curve; sometimes produces opaque prompts hard to hand-tune afterward.

Worth it for pipelines with ≥3 stages where manual tuning becomes unmanageable.

## 6. TextGrad: Gradients Over Strings

Yuksekgonul et al. 2024: think of LLM outputs as the forward pass and use **textual "gradients"** from a critic LLM to edit the prompt.

```
prompt = "Answer concisely."
output = model(prompt, input)
grad = critic(output, "Your answer was too verbose.")
prompt = prompt + "\nLimit to two sentences."
```

Research-stage as of early 2026 but promising for prompt + program optimization without hand-curated training data.

## 7. Evaluating Candidate Prompts

The scoring function is where rigor matters:

| Task type | Scoring |
| --- | --- |
| Classification | Exact match |
| Extraction | Per-field F1 |
| Ranking | nDCG, MRR |
| Open-ended | LLM-as-judge with rubric |
| Structured | Schema pass + per-field correctness |

Always run eval on a **held-out set** distinct from the set used for search, to avoid overfitting the prompt to the eval.

## 8. Overfitting Prompts

Yes, prompts can overfit. Signs:

- Huge gain on dev set, regressions on holdout.
- Prompt is unusually long with seemingly-magic phrases.
- Accuracy drops dramatically when you paraphrase input slightly.

Mitigations:
- Hold out ≥30% of examples as final eval.
- Include paraphrased inputs in eval.
- Prefer shorter prompts on ties.
- Re-evaluate after model version changes.

## 9. Stability Across Model Versions

A prompt optimized for GPT-4 may be mediocre on GPT-5 or on Sonnet. Automated optimization tends to find local tricks specific to the current model. Re-run optimization when:

- Provider releases a new model version.
- You migrate providers.
- Your traffic mix changes (e.g., more non-English).

Budget this as a quarterly task for high-value prompts.

## 10. Prompt Optimization as an Eval Investment

The hidden gift of optimization is forcing rigorous evaluation. Teams that run systematic prompt search:
- End up with large, high-quality eval sets.
- Have baseline metrics for every feature.
- Catch regressions quickly.

The optimization itself is often half the value; the discipline it forces is the other half.

## 11. When Optimization Hurts

- On tasks where the model's capability ceiling is below your target — no prompt fixes a hallucinating model; invest in RAG or fine-tuning.
- When the business logic keeps shifting — you keep re-optimizing a moving target.
- Over-optimized prompts become brittle — they work on the exact shape of eval inputs and degrade outside that distribution.

## 12. Architect Takeaways

- **Optimize when the prompt runs at scale and has a clear metric.**
- **Invest in the eval set first.** Optimization is worthless without it.
- **Start simple** — 3 variants by hand beats a sophisticated framework for small tasks.
- **DSPy is the right tool for pipelines**, not single prompts.
- **Budget re-optimization** on model upgrades.
- **Overfitting is real.** Keep holdouts; prefer shorter prompts.

## Prereqs / Connections

- `10_testing_prompts` — the eval infrastructure this depends on
- `11-evaluation/` — the broader evaluation stack
- `03_zero_few_n_shot` — DSPy's few-shot bootstrap is powered from this
- `12-ai-safety-governance/` — don't optimize to the point of brittleness

## Further Reading

- Zhou et al. 2022, "Large Language Models are Human-Level Prompt Engineers" (APE)
- Khattab et al. 2023, "DSPy: Compiling Declarative Language Model Calls"
- Yuksekgonul et al. 2024, "TextGrad: Automatic 'Differentiation' via Text"
- Pryzant et al. 2023, "Automatic Prompt Optimization with Gradient Descent and Beam Search"
- DSPy documentation & tutorials
