# LLM 15: Hands-On — Claude + OpenAI + Local

The capstone for module 05. Build a unified provider abstraction that runs the same prompt through Claude (Anthropic), GPT-4o (OpenAI), and a local Ollama model, then compare cost, latency, and quality. This is the pattern every production LLM app eventually needs.

## 1. Why You Need an Abstraction

Vendor risk is the loudest reason, but not the most common. The real reasons:

- **Model routing** — cheap for simple, expensive for hard (session 12).
- **Fallback** — when one provider is down, keep shipping.
- **A/B testing** — compare at the feature level.
- **Cost arbitrage** — pricing changes over quarters, sometimes by 10×.
- **Local fallback** — offline mode, data residency, sensitive data.
- **Consistent observability** — one log format across providers.

The abstraction is small. Not building it is the default mistake.

## 2. What to Abstract (And What Not To)

Abstract:
- `generate(system, user, **params) -> LLMResponse`
- Streaming interface
- Tool use (function calling) — unified schema
- Structured output — map to provider-specific JSON mode
- Usage counters (input/output/cached tokens)
- Cost (computed from a price table)

Do NOT abstract:
- Model-specific features that don't generalize (Claude thinking, Gemini live video)
- Provider-specific retry semantics (use provider's SDK where possible)
- Every possible kwarg — keep the surface minimal

The right abstraction captures 80% of your usage and exposes escape hatches for the rest.

## 3. Shape of the Interface

```python
@dataclass
class LLMResponse:
    text: str
    input_tokens: int
    output_tokens: int
    cached_tokens: int
    cost_usd: float
    latency_ms: float
    provider: str
    model: str
    raw: Any                  # original response for escape-hatch access

class LLMClient(Protocol):
    def generate(self, system: str, user: str, *,
                 temperature: float = 0.2,
                 max_tokens: int = 1024,
                 tools: list[dict] | None = None,
                 response_schema: type[BaseModel] | None = None,
                 stream: bool = False) -> LLMResponse | Iterator[str]: ...
```

## 4. Provider Implementations (Sketch)

Each provider is a thin adapter:

```python
class ClaudeClient(LLMClient):
    def __init__(self, model='claude-sonnet-4-6'):
        from anthropic import Anthropic
        self.c = Anthropic()
        self.model = model
    def generate(self, system, user, **kw):
        t0 = time.perf_counter()
        r = self.c.messages.create(
            model=self.model, system=system,
            messages=[{'role':'user','content':user}],
            max_tokens=kw.get('max_tokens', 1024),
            temperature=kw.get('temperature', 0.2))
        return LLMResponse(
            text=r.content[0].text,
            input_tokens=r.usage.input_tokens,
            output_tokens=r.usage.output_tokens,
            cached_tokens=getattr(r.usage, 'cache_read_input_tokens', 0),
            cost_usd=_price(self.model, r.usage),
            latency_ms=(time.perf_counter() - t0) * 1000,
            provider='anthropic', model=self.model, raw=r)
```

Repeat for OpenAI, Ollama/vLLM (OpenAI-compatible API).

## 5. Benchmarking Across Providers

A useful pattern — run the same prompt set through all three:

```python
results = {}
for name, client in clients.items():
    for ex in eval_set:
        r = client.generate(ex.system, ex.user)
        # record cost, latency, output
        # score with judge
```

Then plot:
- Cost distribution per provider
- p50/p95 latency per provider
- Quality scores per provider
- Pareto frontier: quality vs cost

This is the quarterly homework every team should do: *is our provider mix still optimal?*

## 6. Streaming Across Providers

The interface diverges slightly:
- **Anthropic** — `messages.stream()` yields `text_stream`, typed events.
- **OpenAI** — `stream=True`, iterate over chunks; `delta.content`.
- **Ollama / vLLM (OpenAI-compat)** — same as OpenAI.

Unify via a generator:

```python
def stream(self, ...) -> Iterator[str]:
    ...
    for chunk in provider_stream:
        yield normalize(chunk)
```

## 7. Tool Use Across Providers

| Provider | Representation |
| --- | --- |
| Anthropic | `tools=[{name, description, input_schema}]`, returns `tool_use` block |
| OpenAI | `tools=[{type:'function', function:{name, description, parameters}}]`, returns `tool_calls` |
| Google Gemini | `tools=[{function_declarations:[...]}]` |

Translate to a common form:

```python
ToolDef = {
    'name': str,
    'description': str,
    'parameters': dict,    # JSON Schema
}
```

Each adapter translates to the provider's format and parses responses back to a canonical `ToolCall`.

## 8. Fallback and Resilience

Wrap the abstraction in a decorator that handles:

- Transient errors → retry with backoff (`03-design-patterns/09_retry_backoff`)
- Rate limits → exponential backoff + jitter
- Provider outage → failover to secondary (`03-design-patterns/08_circuit_breaker`)
- Cost budget exceeded → fail fast or route to cheaper tier

```python
def with_fallback(clients: list[LLMClient]):
    def _call(system, user, **kw):
        last = None
        for c in clients:
            try:
                return c.generate(system, user, **kw)
            except TransientError as e:
                last = e
                continue
        raise last
    return _call
```

## 9. Observability

Log every call with:
- Feature/tenant ID
- Model, provider
- Token counts (input / cached / output)
- Cost
- Latency (TTFT if streamed)
- Success / failure
- Prompt hash (for cache analysis)

Ship to your tracer of choice (Langfuse, Braintrust, OpenTelemetry → any backend).

Covered in `14-llmops-deployment/10_observability`.

## 10. Running Models Locally

Why local matters:

- **Cost** — marginal inference is ~$0 after you pay for hardware.
- **Privacy / data residency** — data never leaves your network.
- **Latency** — no round-trip.
- **Offline / edge** — airplane, factory floor, regulated environments.

Options:

| Tool | Use case |
| --- | --- |
| Ollama | Easiest local inference; OpenAI-compat API |
| vLLM | Production serving; throughput-optimized |
| llama.cpp | CPU / small-hardware inference |
| LM Studio | Desktop UI around llama.cpp |
| Text Generation Inference (TGI) | Production serving; HF ecosystem |

All expose OpenAI-compatible APIs; your abstraction targets them the same way.

## 11. When to Add Each Provider

| Phase | Recommended mix |
| --- | --- |
| Prototype | One provider, top-tier model |
| MVP | One provider, right-sized model; prompt caching on |
| Production v1 | Primary + fallback; structured observability |
| Scale | Multi-provider routing; batch API for offline; local for high-volume simple calls |
| Enterprise | Self-host for sensitive data; multiple providers for resilience |

Premature multi-provider is a drag. Premature single-provider is a risk. Know where you are on the curve.

## 12. Architect Takeaways

- **Build the abstraction on day 1.** 100 lines of code; enormous optionality.
- **Benchmark quarterly.** Prices, models, and quality shift.
- **Route by cost AND quality.** Neither alone is enough.
- **Have a local fallback for privacy and offline.** Even if it's worse — it's a capability.
- **Observe everything.** Provider, tokens, cost, quality, latency. Per feature, per tenant.

## Prereqs / Connections

- `07_prompt_engineering_intro` — the abstraction ran the same prompt in three places
- `12_token_economics` — the cost math the abstraction surfaces
- `14-llmops-deployment/05_gateway_routing` — production-grade version
- `03-design-patterns/08_circuit_breaker`, `03-design-patterns/09_retry_backoff`
- `03-design-patterns/07_repository_registry` — the abstraction pattern itself

## Further Reading

- LiteLLM documentation (open-source provider gateway)
- Portkey docs (managed provider gateway)
- vLLM production deployment guide
- Ollama documentation
- Anthropic / OpenAI / Google SDK references
- Claude Agent SDK docs (for agent-shaped apps)
