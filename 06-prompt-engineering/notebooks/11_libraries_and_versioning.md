# Prompt 11: Prompt Libraries & Versioning

Once you have 5+ prompts and 3+ engineers, you need a **registry**. This session is the blueprint: how to structure, version, deploy, and roll back prompts like any other production artifact.

## 1. Why a Registry

Without one:
- Prompts are inline strings scattered across the codebase.
- Tweaks happen via copy-paste.
- No one knows which version is running.
- Rollback requires a code deploy.
- A/B testing is ad-hoc.

With one:
- Prompts live in a single well-known place.
- Each has an ID, version, owner, and test suite.
- Deploys decouple from code.
- Rollbacks are config changes.
- Traffic splitting is first-class.

## 2. Directory Layout

```
prompts/
  registry.yaml                ← metadata index
  invoice_extract/
    v1.jinja
    v2.jinja
    v3.jinja                    ← current stable
    v4.jinja                    ← canary
    schema.py                   ← Pydantic model
    metadata.yaml               ← owner, tags, test set ref
    tests/
      eval_set.jsonl
      test_invoice_extract.py
  support_classifier/
    v1.jinja
    ...
```

Simple, browsable, git-tracked.

## 3. Metadata per Prompt

`metadata.yaml`:

```yaml
id: invoice_extract
owner: team-finance-ai
description: Extract vendor, total, and due date from an invoice text.
versions:
  v1: { status: retired, model: gpt-4-turbo }
  v2: { status: retired, model: gpt-4o-mini }
  v3: { status: stable, model: claude-sonnet-4-6, released: 2026-01-15 }
  v4: { status: canary, model: claude-haiku-4-5-20251001, traffic_pct: 10 }
eval_set: tests/invoice_extract/eval_set.jsonl
baseline_accuracy: 0.92
tags: [finance, extraction]
```

Machine-readable; used by deploy tooling and dashboards.

## 4. Access Pattern

Code doesn't inline prompts. It looks them up:

```python
from prompt_registry import get_prompt

prompt = get_prompt(id="invoice_extract")   # resolves to the active version
rendered = prompt.render(text=invoice_text)
response = prompt.call(rendered)             # may even wrap the LLM call
```

Benefits:
- One place to change the default version.
- Logs include `prompt_id@version`.
- Swap to canary or rollback without code changes.

## 5. Versioning Strategy

Semantic-ish versioning adapted for prompts:

- **Major** (v1 → v2): breaking changes to output schema or intent.
- **Minor**: new examples, refined wording, same schema.
- **Patch**: typo fixes, minor wording, no eval impact.

Enforce: PR to prompts/ requires:
- Bumped version in `metadata.yaml`.
- Eval passes at current baseline.
- Changelog entry.

## 6. Release States

Each version has a lifecycle:

```
draft → canary → stable → deprecated → retired
```

- **Draft** — dev-only, behind a flag.
- **Canary** — small % of production traffic (1–10%).
- **Stable** — 100% of default traffic.
- **Deprecated** — still works, but new consumers are warned.
- **Retired** — no longer served; kept in history for audit.

Canary rollouts for prompts look like canary rollouts for code.

## 7. Feature-Flag Integration

Traffic splitting:

```yaml
rollout:
  default: v3
  rules:
    - match: { tenant: "beta-tester" }
      version: v4
    - match: { traffic_pct: 10 }
      version: v4
```

Tools: LaunchDarkly, Statsig, GrowthBook, Unleash, or home-grown.

## 8. Observability Tied to Version

Every LLM call logs:

- `prompt_id`
- `prompt_version`
- Model + params
- Input/output tokens, cost, latency
- Success/failure
- Tenant/feature

Dashboards cut by `prompt_id@version`. Regression detection: if `v4`'s judge-score p50 drops 5 points below `v3`, alert.

## 9. Rollback Workflow

```
1. Detect regression (monitoring / manual report).
2. Flip default version in registry.yaml or feature-flag config.
3. Deploy config change (seconds, not minutes).
4. Open postmortem PR.
```

No code redeployment. No SRE on-call cascade.

## 10. Cross-Feature Sharing

Common fragments belong in shared partials:

```jinja
{% include "partials/_citation_style.jinja" %}
{% include "partials/_refusal_language.jinja" %}
```

Update once, propagate with next render. Test partials independently.

## 11. Multi-Tenant Considerations

Enterprise apps often have per-tenant prompt variations:

- Default prompt for everyone
- Tenant-specific overrides (language, tone, policy)
- Tenant-specific few-shot examples

Pattern:

```python
prompt = get_prompt(id="invoice_extract", tenant=current_tenant.id)
```

The registry resolves default + tenant-specific. Versioning applies per-tenant too.

## 12. Auditability

Regulatory and debugging needs:

- Log **full rendered prompt** (hashed if sensitive) with every LLM call.
- Retain prompt versions indefinitely — retiring ≠ deleting.
- Tie prompt version to outputs stored for user-visible features (so you can answer "what prompt produced this output?").

## 13. Tooling (2026 Ecosystem)

- **Braintrust** — prompt + eval + version management, SaaS.
- **LangSmith** — LangChain-native, traces included.
- **Langfuse** — open-source, self-hostable.
- **PromptLayer** — prompt versioning + metrics.
- **Humanloop** — prompt + eval + deployment.
- **Helicone** — observability + prompt versioning.

Or roll your own — the directory structure above is 80% of what the SaaS tools provide.

## 14. Anti-Patterns

- **Inline prompt strings** scattered across code.
- **Silent prompt changes** that don't bump versions.
- **No test requirement on prompt PRs.**
- **Deleting old versions** — you lose audit trail.
- **Shared prompts across unrelated features** — they drift into unowned complexity.

## 15. Architect Takeaways

- **Prompts are first-class artifacts** — give them a registry.
- **Metadata includes owner, version, status, eval set, baseline.**
- **Separate deploy from code release.** Feature-flag rollouts.
- **Log prompt_id@version on every call.**
- **Rollback is a config change, not a redeploy.**
- **Retire, don't delete.** Audit matters.

## Prereqs / Connections

- `07_templates_and_variables` — the template format
- `10_testing_prompts` — the test requirement
- `14-llmops-deployment/06_versioning_and_registry` — deploy patterns
- `14-llmops-deployment/07_canary_rollout`
- `11-evaluation/` — the eval sets the registry references

## Further Reading

- Braintrust docs on prompt versioning
- Humanloop, "Prompt Management Best Practices"
- OpenTelemetry LLM conventions (for prompt_id attribution)
- Langfuse documentation
