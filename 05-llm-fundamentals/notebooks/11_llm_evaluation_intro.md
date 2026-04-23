# LLM 11: LLM Evaluation (Intro)

Short session — the full treatment is in `11-evaluation/`. Here we establish the vocabulary and the mental model so decoding, prompting, and structured-output decisions can be justified by measurement instead of vibes.

## 1. Why Evaluation Is Hard for LLMs

Classical ML: predicted_label vs ground_truth_label, compute accuracy, done. LLMs:

- Outputs are open-ended text.
- Many correct answers exist.
- Quality has multiple axes (factuality, helpfulness, safety, format, length, tone).
- Reference answers can be wrong, stale, or not exist.
- The evaluator is often another LLM, with its own biases.

Treat LLM evaluation as **product measurement**, not model measurement. You're evaluating the system (prompt + model + retrieval + post-processing) against a specific use case.

## 2. Types of Evaluation

| Type | What it measures | Example |
| --- | --- | --- |
| Intrinsic benchmarks | Raw model capability | MMLU, HellaSwag, GSM8K, HumanEval |
| Task-specific accuracy | Correct output on a task | Extract invoice fields correctly |
| Reference-based | Similarity to a gold answer | BLEU, ROUGE, BERTScore |
| Reference-free | Judge without a gold | LLM-as-judge, factuality classifiers |
| Rubric-based | Multiple quality axes | Helpfulness, safety, format (each scored) |
| Preference-based | A vs B pairwise | Human or LLM picks a winner |
| Production SLIs | Online metrics | Acceptance rate, regeneration rate |

## 3. Benchmarks: Useful and Misleading

| Benchmark | What it tests | 2026 utility |
| --- | --- | --- |
| MMLU | 57 subjects, multiple choice | Saturated; every strong model is 80+% |
| HumanEval / MBPP | Python function synthesis | Widely gamed; recent models overfit |
| GSM8K | Grade school math | Near-saturated |
| MATH | Competition math | Useful signal for reasoning models |
| BIG-Bench Hard | Mixed reasoning | Still discriminating |
| IFEval | Instruction-following | Still discriminating |
| LiveCodeBench | Recent coding problems | Hard to game |
| Arena (LMSYS) | Crowd preference | Broadest real-world signal |

Use benchmarks to *rule out* weak models, not to pick a winner. Task-level eval matters more than a leaderboard number.

## 4. LLM-as-Judge: the Workhorse

Pattern:

```
1. Generate output with candidate model.
2. Show (input, output, rubric) to a judge model.
3. Judge returns scores per rubric dimension, or a pairwise winner.
4. Aggregate.
```

Strengths: scales without humans. Works for open-ended outputs. Can be fast and cheap.

Biases to be aware of:

- **Position bias** (A-vs-B): judge favors position A. Mitigation: randomize or evaluate both orders.
- **Verbosity bias**: judge favors longer answers. Mitigation: rubric penalizes padding.
- **Self-preference**: judge prefers outputs from its own family. Mitigation: use a different family as judge.
- **Style over substance**: judge rewards polish. Mitigation: explicit rubric dimensions, anchor examples.

## 5. A Minimal Eval Loop

```python
def eval_prompt(prompt_tmpl, dataset, judge_model):
    scores = []
    for ex in dataset:
        prediction = call_model(prompt_tmpl.format(input=ex.input))
        judgment = call_judge(judge_model,
            input=ex.input, expected=ex.expected, actual=prediction,
            rubric=RUBRIC)
        scores.append(judgment.score)
    return mean(scores), std(scores), scores
```

With 50 labeled examples and a good judge prompt, you can A/B-test prompts, models, and retrieval settings rigorously.

## 6. Golden Datasets

Investment in a labeled eval set compounds. Build it from:

1. **Production traffic (sampled, anonymized).** Highest signal.
2. **Synthetic adversarial cases.** LLMs generate edge cases you wouldn't think of.
3. **Historical incidents.** Anything that went wrong once should be in the eval set.
4. **Competitor examples.** Known-good inputs from similar products.

50 well-chosen examples > 5000 generic ones.

## 7. Production SLIs

Offline eval is not enough. In production, monitor:

- **Acceptance rate** — fraction of outputs kept (not edited, not regenerated).
- **Regeneration rate** — users hitting "try again".
- **Latency p50/p95/p99** — for streaming: TTFT and total.
- **Structured-output success rate** — fraction passing schema validation.
- **Guardrail trigger rate** — fraction routed to blocking/rewriting.
- **Cost per successful interaction.**

Covered further in `14-llmops-deployment/10_observability`.

## 8. Eval Tooling (2026)

| Tool | Strength |
| --- | --- |
| Braintrust | Prompt/eval versioning + dashboards |
| LangSmith | Deep LangChain/LangGraph integration |
| Langfuse | Open-source tracing + eval |
| Weights & Biases Weave | Experiment tracking + LLM-specific |
| OpenAI Evals | Free, provider-specific |
| DeepEval | pytest-style eval harness |
| Ragas / TruLens | RAG-specific metrics |
| Inspect | Open-source, rigorous (Anthropic Research) |

Pick one early and stick with it; migration is painful.

## 9. Architect Heuristics

- **Build the eval set before the product.** You can't iterate on quality without it.
- **Pin the judge model.** Swapping judges changes your scores.
- **Treat eval-set drift as a first-class concern.** Update the set as the product evolves.
- **Run regression evals in CI** for prompt changes.
- **Sample production outputs** into the eval set continuously.

## Prereqs / Connections

- `07_prompt_engineering_intro` — eval drives prompt iteration
- `08_structured_outputs` — validation as an eval signal
- `11-evaluation/` — full module with RAGAS, agent eval, A/B testing, tooling
- `14-llmops-deployment/10_observability` — production SLIs

## Further Reading

- Zheng et al. 2023, "Judging LLM-as-a-Judge with MT-Bench and Chatbot Arena"
- Liang et al. 2022, "Holistic Evaluation of Language Models (HELM)"
- Anthropic, "Inspect" open-source eval framework
- LMSYS Chatbot Arena leaderboard methodology
- Braintrust / LangSmith / Langfuse docs
