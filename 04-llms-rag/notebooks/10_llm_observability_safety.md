# LLMs 10: LLM Observability & Safety

Production AI systems need the same observability rigor as any distributed system, plus AI-specific safety controls.

## 1. Why Observability Matters

LLMs are non-deterministic and expensive. Without observability you cannot:
*   Debug why an answer was wrong.
*   Track costs before they explode.
*   Detect when the system degrades.
*   Build confidence to deploy new models.

---

## 2. The Observability Stack

```
┌────────────────────────────────────────────────┐
│               LLM Application                   │
│  User → API → [TRACE START]                     │
│           ↓                                     │
│    Prompt Template Fill                         │
│           ↓                                     │
│    Retrieval (vector DB) ──→ [SPAN: retrieval]  │
│           ↓                                     │
│    Reranking ──────────────→ [SPAN: reranking]  │
│           ↓                                     │
│    LLM Call ───────────────→ [SPAN: llm_call]   │
│           ↓                                     │
│    Response ←── [TRACE END]                     │
└────────────────────────────────────────────────┘
              ↓ Export
   [Traces, Metrics, Logs]
              ↓
   LangSmith / Arize / Phoenix / Custom
```

---

## 3. Key Metrics to Track

| Metric | Description | Alert Threshold |
| :--- | :--- | :--- |
| **Latency (p50, p95, p99)** | Time to first token, total time | p99 > 10s |
| **Cost per request** | Input + output tokens × price | Budget alert |
| **Error rate** | API errors, parsing failures | > 1% |
| **Token usage** | Input/output tokens per request | Trending up |
| **Cache hit rate** | Prompt cache utilization | < 30% = investigate |
| **Hallucination rate** | From eval pipeline | > 5% |
| **Retrieval relevancy** | Context precision per query | < 0.7 |
| **User satisfaction** | Thumbs up/down if collected | Drop > 5% |

---

## 4. Structured Logging for LLM Calls

Every LLM call should be logged with:

```python
log_entry = {
    "trace_id": "abc123",
    "timestamp": "2025-01-15T10:30:00Z",
    "model": "claude-sonnet-4-5-20250929",
    "input_tokens": 1234,
    "output_tokens": 456,
    "latency_ms": 2300,
    "cost_usd": 0.00823,
    "user_id": "user_789",  # For multi-tenant systems
    "session_id": "sess_456",
    "prompt_template": "rag_v3",
    "retrieved_doc_count": 5,
    "cache_hit": False,
    "error": None,
    "metadata": {"feature": "search", "ab_group": "control"},
}
```

---

## 5. Safety Controls

### Input Validation
*   Reject inputs exceeding token limits.
*   Detect and block prompt injection attempts.
*   PII detection (names, emails, SSNs) before sending to external APIs.

### Output Validation
*   JSON schema validation for structured outputs.
*   Toxicity classification (Perspective API, custom classifier).
*   Factual grounding check (faithfulness score).
*   Format validation (expected response structure).

### Rate Limiting
*   Per-user: prevent abuse.
*   Per-tenant: isolate resource usage.
*   Global: protect downstream LLM quotas.

---

## 6. Guardrails Architecture

```python
class GuardrailsPipeline:
    def process(self, user_input: str, llm_response: str) -> dict:
        # Input guardrails
        if self.detect_pii(user_input):
            return {"blocked": True, "reason": "PII detected in input"}
        if self.detect_injection(user_input):
            return {"blocked": True, "reason": "Prompt injection detected"}

        # Generate response (LLM call here)
        response = self.call_llm(user_input)

        # Output guardrails
        if not self.validate_json(response):
            return {"blocked": True, "reason": "Invalid output format"}
        if self.detect_toxicity(response) > 0.7:
            return {"blocked": True, "reason": "Toxic content detected"}

        return {"response": response, "blocked": False}
```

---

## 7. Cost Optimization Strategies

| Strategy | Savings | Complexity |
| :--- | :--- | :--- |
| **Prompt caching** | 50-90% on repeated prefixes | Low |
| **Model routing** | Use cheap model for simple queries | Medium |
| **Response caching** | Cache identical queries | Low |
| **Batching** | Group requests when latency allows | Medium |
| **Streaming** | Reduces perceived latency, not cost | Low |
| **Prompt compression** | LLMLingua reduces prompts 2-4× | High |
| **Context pruning** | Remove irrelevant retrieved chunks | Medium |
