# Prompt 10: Testing & Evaluation of Prompts

Prompts without tests are production bugs waiting to happen. This session is about the discipline: what to test, how to structure the tests, and how to run them in CI.

## 1. Why Prompt Tests

A prompt change can:

- Silently regress accuracy on edge cases.
- Double output length, doubling cost.
- Break downstream parsers.
- Introduce subtle hallucinations.
- Mis-behave with non-English inputs.

Unit tests catch none of this. Full evals catch all of it.

## 2. What to Test (The Hierarchy)

From cheapest/fastest to most comprehensive:

1. **Render tests** — does the template produce a valid string for representative inputs?
2. **Format tests** — does output parse, match a schema, include required sections?
3. **Smoke tests** — on 5–10 canonical inputs, does the output look right?
4. **Regression tests** — on a curated golden set, does accuracy hit a threshold?
5. **Drift tests** — on sampled production traffic, does behavior stay stable?
6. **Adversarial tests** — on red-team inputs, does the system refuse appropriately?
7. **Bias tests** — across demographics, locales, formats, is the behavior consistent?

Most teams run 1–4 in CI. 5 runs in monitoring. 6–7 run periodically.

## 3. Render and Format Tests

Cheap, deterministic, fast — run on every commit:

```python
def test_invoice_prompt_renders():
    p = INVOICE_TEMPLATE.render(invoice=sample_invoice(), tenant="acme")
    assert "<invoice>" in p
    assert "acme" in p
    assert "TODO" not in p

def test_invoice_output_matches_schema():
    output = call_llm(INVOICE_TEMPLATE.render(invoice=sample_invoice()))
    obj = json.loads(output)
    assert Invoice(**obj)   # pydantic validation
```

No LLM cost beyond the smoke test. Runs in <10 seconds. Every PR.

## 4. Golden Set Regression Tests

A **golden set** is a curated collection of `(input, expected_output)` examples with known-correct labels. Running a prompt against the golden set gives an accuracy number.

```python
GOLDEN = load_golden_set("prompts/invoice_extract/eval.jsonl")

def test_invoice_prompt_accuracy():
    results = [(ex, call_llm(render(ex.input))) for ex in GOLDEN]
    accuracy = sum(match(r, ex.expected) for ex, r in results) / len(GOLDEN)
    assert accuracy >= BASELINE_ACCURACY - 0.02   # tolerate 2pp noise
```

CI runs this on PRs that touch prompts. A regression fails the build.

Notes:
- **Baseline** is pinned; updated deliberately.
- **Threshold tolerance** accounts for model non-determinism.
- **Golden set grows** over time from production samples.

## 5. Building a Golden Set

Sources:
- **Sampled production traffic** (anonymized) — highest signal.
- **Historical incidents** — cases that went wrong.
- **Adversarial** — try to break it.
- **Edge cases** — empty input, very long input, non-English, nested structures.
- **Synthetic** — LLM-generated variations.

Target sizes:
- **Quick smoke**: 5–10 examples (runs fast, catches big regressions).
- **Standard regression**: 50–100 examples.
- **Comprehensive**: 500+ for critical features.

Quality > quantity: 30 carefully-chosen examples beat 300 random ones.

## 6. LLM-as-Judge in Regression Tests

For open-ended outputs, use judge prompts:

```python
def judge(question, expected, actual) -> dict:
    raw = call_llm(
        JUDGE_SYSTEM,
        f"Q: {question}\nExpected: {expected}\nActual: {actual}\n\n"
        "Is actual correct and consistent with expected? JSON: {correct: bool, reason: str}"
    )
    return json.loads(raw)
```

Cost: a judge call per test example per run. Budget for it explicitly.

Guardrails:
- Pin the judge model. A new judge is a new scoring function.
- Spot-check judge decisions with humans periodically.
- Use rubric judges, not vague "is it good?" judges.

## 7. Structuring Tests (pytest Patterns)

Treat prompts as code, test them with pytest:

```python
# tests/prompts/test_invoice_extract.py
import pytest
from prompts.invoice_extract import render, run

GOLDEN = load_jsonl("tests/data/invoice_golden.jsonl")

@pytest.mark.parametrize("example", GOLDEN, ids=lambda ex: ex["id"])
def test_extract_produces_valid_json(example):
    out = run(example["input"])
    assert parse_json(out) is not None

@pytest.mark.parametrize("example", GOLDEN)
def test_extract_field_accuracy(example):
    out = run(example["input"])
    obj = parse_json(out)
    assert obj["vendor"] == example["expected"]["vendor"]
```

Per-example tests give you fine-grained pass/fail per input. Slower but debuggable.

## 8. Snapshot Tests

For stability tracking: snapshot outputs for a set of inputs, detect when they change.

```python
def test_summary_snapshot():
    new_out = run("a well-known stable input")
    expected = load_snapshot("summary/v1/001.txt")
    assert similar(new_out, expected, threshold=0.85)
```

Useful for detecting subtle drift after a model provider-side update.

## 9. Deterministic Configs for Tests

Production uses `temperature=0.7`. Tests should use `temperature=0` (where supported) for reproducibility. Document the discrepancy; don't let it surprise you.

For truly non-deterministic outputs (creative writing), use:
- Rubric-based judge instead of exact comparison
- Multiple samples with aggregate metrics
- Longer tolerance bands

## 10. CI Integration

```yaml
# .github/workflows/prompts.yml
name: Prompt Tests
on:
  pull_request:
    paths: ["prompts/**", "tests/prompts/**"]

jobs:
  prompt-tests:
    steps:
      - checkout
      - run: pytest tests/prompts/ --tb=short
        env:
          OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY_CI }}
```

- Separate CI key with a budget cap.
- Fail the PR on regressions.
- Post the accuracy diff as a PR comment.

## 11. Monitoring in Production (Drift)

Offline eval catches regressions against a frozen test set. Production data drifts. Complement with:

- **Sampled scoring** — take 1% of production traffic, run judges, store scores.
- **Outlier detection** — prompts whose distribution changes (length, refusal rate) deserve investigation.
- **User signal proxies** — regeneration rate, thumbs-down rate, copy-to-clipboard rate.

Dashboards + alerts, not manual review.

## 12. Human Evals

Ultimately LLM judges approximate human judgment. Periodically:

- **Calibration panels** — humans score a shared set; LLM judge scores the same set; measure agreement.
- **Spot-check failures** — of the tests that fail, do humans agree?
- **Red team** — external testers try to break the system.

Schedule: at least quarterly for customer-facing features.

## 13. Architect Takeaways

- **Prompt tests are CI tests** — mandatory, versioned, automated.
- **Golden sets compound value.** Invest in them.
- **Pin the judge model** when using LLM-as-judge.
- **Snapshot for drift**, regression for change control.
- **Monitor in production** — offline eval is necessary, not sufficient.

## Prereqs / Connections

- `07_templates_and_variables` — tests verify templates render
- `09_prompt_optimization` — optimization depends on reliable evals
- `11-evaluation/` — broader evaluation stack
- `14-llmops-deployment/07_canary_rollout`

## Further Reading

- Braintrust documentation (eval + regression workflows)
- LangSmith / LangEvals (eval + trace integration)
- OpenAI Evals framework
- Anthropic "Inspect" open-source eval framework
- DeepEval (pytest-shaped eval library)
