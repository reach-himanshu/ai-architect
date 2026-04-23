# LLM 13: Hallucination & Guardrail Basics

Hallucination is the single most-cited failure mode of LLMs. It's also the one architects most often under-plan for. This session covers why it happens, the shape of the problem, and the baseline guardrails every production system needs. Deep dives on adversarial safety and governance live in `12-ai-safety-governance/`.

## 1. What Hallucination Actually Is

"Hallucination" is a catch-all for:

| Subtype | Description | Example |
| --- | --- | --- |
| **Factual hallucination** | Made-up facts | "The capital of Australia is Sydney." |
| **Citation fabrication** | Real-looking but non-existent sources | "See Smith et al. 2022, *J. Appl. Linguistics*" (doesn't exist) |
| **Contradiction** | Internal inconsistency | Says X in one sentence, not-X in the next |
| **Grounding failure** | Contradicts provided context | Ignores retrieved doc and answers from parametric memory |
| **Overgeneralization** | True in one domain, false in another | Medical claim true in vitro, false in humans |
| **Confabulation under uncertainty** | Confident answer when it should abstain | "Yes, John Smith was born in 1982." (model doesn't know; guesses) |

Treating all of these as one thing is a mistake. Each has different mitigations.

## 2. Why LLMs Hallucinate

Mechanistically:
- **Training objective** — next-token prediction rewards fluency, not factuality. The model is optimized to produce plausible continuations, not truthful ones.
- **Parametric memory is lossy** — facts stored in weights can be partial, outdated, or entangled with similar facts.
- **No calibration** — base models don't produce well-calibrated uncertainty; they say "Paris" and "Atlantis" with similar surface confidence.
- **Alignment pressure** — SFT/RLHF often trains models to be helpful and definitive, which can discourage "I don't know."

Architecturally:
- **No grounding** — without retrieval, the model is guessing from compressed training data.
- **Distractors in context** — irrelevant retrieved passages cause confusion.
- **Long context** — lost-in-the-middle increases the chance of ignoring grounding evidence.
- **Model size** — smaller models hallucinate more. Not always fixable with prompting.

## 3. The Guardrail Stack

Production systems layer defenses:

```
1. Input validation     →  reject / sanitize user input
2. Retrieval grounding  →  provide evidence in context
3. Prompt constraints   →  system prompt rules
4. Structured output    →  schema-enforced fields
5. Self-critique        →  model checks its own output
6. External validators  →  regex, schema, policy classifiers
7. Human-in-the-loop    →  escalate high-stakes decisions
8. Post-hoc monitoring  →  sample for hallucination rate
```

No single layer is sufficient. Every production system of value uses at least 3–4.

## 4. Retrieval Grounding (Preview of RAG)

The most effective single mitigation: give the model the evidence it needs before asking for an answer.

```
<context>
Paragraph 1 from authoritative source...
Paragraph 2...
</context>

<question>
What did the CEO say about Q3 guidance?
</question>

<instructions>
Answer strictly from the context above. If the context does not
contain the answer, say "I cannot answer from the provided context."
</instructions>
```

Deep dive in `09-rag/`.

## 5. Prompt Patterns That Reduce Hallucination

- **Tell it to abstain.** "If you do not know, say 'I don't know'." Combines with rubric-based eval penalizing wrong-confident answers.
- **Require citations.** "Cite the specific passage from the context that supports your answer." The model's hallucinations drop when it has to point to evidence.
- **Show your work.** Chain-of-thought reduces some hallucination classes.
- **Use constrained output.** A schema that includes `{"confidence": low|medium|high, "source_ids": [...]}` forces acknowledgment of uncertainty.
- **Tools for facts.** For dates, math, current events: a tool call is almost always more reliable than parametric recall.

## 6. Self-Critique and Verification

The model can reduce its own hallucination rate:

```
Round 1: generate answer
Round 2: "Review the answer above. Does every factual claim follow from the context? List any that don't."
Round 3: "Rewrite, removing or marking unsupported claims."
```

Costs 3× tokens. Worthwhile for high-stakes answers (medical, legal, financial).

Alternative: **self-consistency** — sample the same question 5 times with higher temperature; look for agreement. Divergence signals hallucination.

## 7. Rule-Based Validators

After the model produces output, you can:

- **Schema validate** — Pydantic (session 08).
- **Regex-check for forbidden patterns** — PII, profanity, policy violations.
- **Fact-check categorical fields** — does `country` appear in a known list?
- **Cross-check numbers** — do extracted numbers sum to the reported total?
- **Reality-check URLs** — do cited links resolve?

Validation is cheap. Every production system should run it.

## 8. Policy Classifiers (Guardrails)

Dedicated models classify outputs against policies:

| Tool | Focus |
| --- | --- |
| Llama Guard (Meta) | Broad safety taxonomy |
| NeMo Guardrails (NVIDIA) | Programmable, rails-based |
| Prompt Shield (Azure) | Prompt injection detection |
| Anthropic constitutional classifiers | Policy classes |
| Detoxify | Toxicity |
| Presidio (Microsoft) | PII detection |

Architectural pattern: output → classifier → {allow, rewrite, block}. Deep dive in `12-ai-safety-governance/04_output_filtering`.

## 9. Measuring Hallucination Rate

An eval set annotated for factual correctness, run against candidate prompts/models/RAG configs:

```
metrics = {
  'factual_accuracy': fraction of claims correct,
  'abstention_rate': fraction that said "don't know",
  'wrong_confident_rate': fraction wrong and confident,   # the worst category
  'citation_present_rate': fraction with at least one supporting citation,
  'citation_correct_rate': fraction of citations actually supporting the claim,
}
```

**Wrong-confident rate** is the metric to watch. A system that says "I don't know" too often is unhelpful; a system that confidently hallucinates is dangerous.

## 10. Calibration and Reliability

Calibration: does stated confidence match real accuracy?

A well-calibrated system that claims 80% confidence should be right 80% of the time. LLMs are typically *over*confident. Techniques:

- **Token probability as a proxy** — sum log-probs of the answer tokens; use as implicit confidence.
- **Verbalized confidence** — ask the model for a 0–1 confidence; observe calibration.
- **Ensemble voting** — sample N times; agreement ≈ confidence.

Production impact: use calibration to route low-confidence outputs to human review or escalate to stronger models.

## 11. When to Escalate to Humans

High-stakes pipelines need human gates:

- **Medical/legal/financial advice** — output always goes to a domain expert before user.
- **Low-confidence outputs** — schema includes `confidence`; below a threshold, escalate.
- **Policy-triggered outputs** — classifier flags → human review.
- **First-of-a-kind user queries** — anomaly detection on topic/phrasing.

Human-in-the-loop is not a failure; it's a design choice that the economics often justify.

## 12. Architect Checklist

- **Layer defenses.** No single mitigation suffices.
- **Ground with retrieval** for knowledge tasks.
- **Require citations** and validate them.
- **Measure wrong-confident rate** as a primary SLI.
- **Classify outputs** against policies before surfacing to users.
- **Escalate** low-confidence / high-stakes outputs.
- **Treat hallucination as a product bug**, not a model bug — you're unlikely to fix the model, but you can usually change the system.

## Prereqs / Connections

- `07_prompt_engineering_intro` — abstention and citation prompts
- `08_structured_outputs` — schema-based validators
- `11_llm_evaluation_intro` — measuring hallucination
- `09-rag/` — grounding as the primary mitigation
- `12-ai-safety-governance/` — adversarial safety, policy classifiers

## Further Reading

- Ji et al. 2022, "Survey of Hallucination in Natural Language Generation"
- Tian et al. 2023, "Fine-tuning Language Models for Factuality"
- Inan et al. 2023, "Llama Guard: LLM-based Input-Output Safeguard for Human-AI Conversations"
- Anthropic, "Constitutional AI" papers
- Kadavath et al. 2022, "Language Models (Mostly) Know What They Know" (calibration)
