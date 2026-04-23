# LLMOps & Deployment Module

Serving, routing, observability, and incident response for LLM systems in production.

## 🚀 Roadmap

### 1. LLMOps Landscape vs MLOps
- What carries over from MLOps
- What's genuinely new
- Team shape and ownership

### 2. Model Serving
- vLLM, TGI, Ollama, Triton
- OpenAI-compatible APIs
- Self-host vs managed tradeoffs

### 3. Inference Optimization
- Continuous batching
- PagedAttention
- Speculative decoding

### 4. Quantization in Production
- GPTQ, AWQ, GGUF
- Quality vs latency vs VRAM
- When quantization hurts

### 5. Gateway & Routing
- LiteLLM, Portkey, Cloudflare AI Gateway
- Provider failover routing
- Cost-based routing

### 6. Model Versioning & Registry
- Prompt + model version pinning
- Shadow deployments
- Rollback procedures

### 7. Canary & A/B Rollout
- Prompt rollouts
- Model rollouts
- Guardrail metrics during rollout

### 8. Caching
- Prompt cache
- Semantic cache
- Prereq: `03-design-patterns/14_caching`

### 9. Cost Guardrails & Budget Alerts
- Per-user, per-feature budgets
- Rate limits at the token level
- Prereq: `03-design-patterns/11_rate_limiting`

### 10. Observability
- Token usage, latency, error taxonomies
- Tracing (OpenTelemetry, Langfuse)
- Dashboards that matter

### 11. Incident Response for LLM Failures
- Detection (quality drift, error spikes)
- Playbooks
- Post-mortem discipline

### 12. Provider Failover & Disaster Recovery
- Multi-provider strategies
- Prereq: `03-design-patterns/08_circuit_breaker`, `03-design-patterns/09_retry_backoff`
- Data residency and regional DR
