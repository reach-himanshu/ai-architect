# LLM 12: Token Economics

A feature that costs $0.02 per call at 100 calls a day is cheap. At 10 million calls a day it's $73M/year. Architects who don't model token cost early ship features that can't scale or can't be priced. This session teaches the math and the levers.

## 1. How LLM Pricing Works (2026)

Pricing is per million input / output tokens, with asymmetric rates (outputs cost more, sometimes 3–5×). Representative snapshot:

| Model | Input $/M | Output $/M |
| --- | --- | --- |
| Claude Haiku 4.5 | 1 | 5 |
| Claude Sonnet 4.6 | 3 | 15 |
| Claude Opus 4.7 | 15 | 75 |
| GPT-4o-mini | 0.15 | 0.60 |
| GPT-4o | 2.50 | 10 |
| Gemini 1.5 Flash | 0.075 | 0.30 |
| Llama-3.3 70B (self-hosted, rough) | 0.10–0.50 | 0.10–0.50 |

(Use these for arithmetic practice, not procurement.)

Additional dimensions that change the bill:

- **Cache reads** — 10–100× cheaper than fresh input tokens.
- **Batch API** — 50% discount, slower turnaround.
- **Fine-tuned models** — often 2–5× base rates.
- **Vision tokens** — images converted to equivalent tokens, priced as input.

## 2. The Cost Equation

Per-call cost:

```
cost = (input_tokens × input_price) + (output_tokens × output_price)
     + (cached_tokens × cache_read_price)
     + (image_tokens × input_price)
```

Per-feature daily cost:

```
daily = cost_per_call × calls_per_user × daily_active_users × success_rate_multiplier
```

Good architects model this **before** the feature is built, not after the invoice lands.

## 3. Levers Ordered by Impact

1. **Cache prefixes.** A 50K-token system/context prefix repeated across calls is a cache bullseye. Savings: up to 90%.
2. **Switch model tier per task.** Route simple calls to cheap models, hard calls to strong ones. Savings: 3–10× on mixed traffic.
3. **Reduce input tokens.** Trim system prompts, retrieval passages, chat history. Savings: directly linear.
4. **Reduce output tokens.** Enforce concise outputs via system prompt, `max_tokens`, or structured schema. Savings: linear + output-tokens cost more.
5. **Batch when latency allows.** Analytics / offline jobs → Batch API, 50% off.
6. **Self-host small models** for high-volume, low-complexity calls (classification, reranking).

## 4. Cost vs Latency vs Quality: the Triangle

| Lever | Cost | Latency | Quality |
| --- | --- | --- | --- |
| Smaller model | ↓↓ | ↓ | ↓ |
| Shorter output | ↓ | ↓ | ↓ |
| Longer context | ↑↑ | ↑↑ | ↑ |
| Prompt caching | ↓↓ | ↓ | = |
| Batching | ↓ | ↑↑ | = |
| Few-shot examples | ↑ | ↑ | ↑ |
| Reranking | ↑ | ↑ | ↑↑ |
| Retrieval (vs long context) | ↓↓ | ↓↓ | = or ↑ |

You can't optimize all three. Pick the feature-appropriate tradeoff.

## 5. Model Routing

Modern production systems route calls by difficulty:

```
user_request → classifier (cheap) → {simple → Haiku, complex → Sonnet, research-grade → Opus}
```

Typical savings: 50–80% vs always using the top tier. Examples:

- **Embedding the classification in the first call** — let Sonnet handle easy ones, escalate only on uncertainty signals.
- **Dual-call with confidence check** — run cheap model, run expensive only if confidence low.
- **Provider gateways** (LiteLLM, Portkey) automate routing with rules + fallbacks.

Covered further in `14-llmops-deployment/05_gateway_routing`.

## 6. Caching: the #1 Optimization

Anthropic, Gemini, and OpenAI all offer prompt caching. Semantics differ but the idea is the same: the long static prefix is cached server-side; re-use it cheaply.

Payoff calculation:
```
savings = (input_tokens_cached × (1 - cache_read_multiplier)) × call_volume
```

A 50K-token system prompt, 5K calls/day, 90% cache savings → **~$7/day → $50/day saved** depending on model. Compounds across features.

Caveats:
- Cache has a TTL (minutes, sometimes longer with write-cache endpoints).
- First call incurs a cache-write cost (usually 25% premium over base input).
- Works best when prefixes are stable across users (tenant-level).

## 7. The Batch API

50% discount, best-effort completion within 24 hours. Perfect for:

- Nightly classification / tagging
- Embeddings refresh
- Eval runs
- Analytics enrichment

Architecturally: keep a queue of async-eligible jobs and a submitter that builds batches up to the provider's size limit.

## 8. Unit Economics Playbook

1. **Model every feature per call, per user, per day.** Spreadsheet or code — doesn't matter.
2. **Know the p95 input and output token counts.** Not just averages.
3. **Set a per-user daily budget at the infrastructure level.** Not just an alert.
4. **Forecast against growth.** 10× users → ?× cost.
5. **Revisit quarterly.** Prices drop, model capabilities shift; recalc.

## 9. Monitoring in Production

Record on every call:
- Input/output token counts (separate)
- Cached token counts
- Model and version
- Feature ID / tenant ID
- Latency
- Cost (computed at emit time)

Dashboards must include:
- Cost per feature, per tenant, per call
- p50/p95/p99 costs
- Cache hit rate
- Anomaly detection on cost-per-call (promotes visibility on regressions)

Covered in `14-llmops-deployment/10_observability`.

## 10. Architect Heuristics

- **Model cost before you model retention.** Unprofitable features are worse than unlaunched ones.
- **Start cheap, scale up.** Easier to upgrade Haiku→Sonnet than to explain an invoice.
- **Cache aggressively.** It's the single highest-ROI optimization.
- **Separate input and output token budgets.** Output is where cost hides.
- **Benchmark quality at each tier.** Routing rules need measurable justification.

## Prereqs / Connections

- `03_tokenization` — tokens are the unit of cost
- `09_context_windows_and_long_context` — context is a cost lever
- `11-evaluation/` — quality-at-cost tradeoffs need measurement
- `14-llmops-deployment/08_caching`, `14-llmops-deployment/09_cost_guardrails`
- `16-industry-use-cases-roi/` — ROI framing for leadership

## Further Reading

- Anthropic, "Prompt caching" docs + pricing page
- OpenAI, "Pricing" + "Batch API" docs
- A16Z, "LLM API Economics" analysis series
- Artificial Analysis (comparative pricing + quality dashboards)
