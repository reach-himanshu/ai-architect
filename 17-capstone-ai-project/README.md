# Capstone AI Project

An end-to-end production AI application that exercises every prior module. Suggested theme: **Agentic RAG over personal knowledge base with feedback-driven fine-tuning**.

## 🚀 Roadmap

### 1. Project Charter
- Problem statement and target users
- Success metrics (quality, latency, cost)
- Non-goals

### 2. Data Ingestion & Chunking Pipeline
- Builds on `07-data-engineering-for-ai/`
- Deduplication and PII redaction wired in

### 3. Vector Store + Retrieval Layer
- Builds on `08-vector-databases/`, `09-rag/`
- Hybrid retrieval + reranking

### 4. Agent Orchestration Layer
- Builds on `10-agentic-ai/`
- Tool use, memory, sub-agents
- Prompting patterns from `06-prompt-engineering/`

### 5. Evaluation Harness
- Builds on `11-evaluation/`
- Offline golden set + online A/B

### 6. Safety & Guardrails
- Builds on `12-ai-safety-governance/`
- Prompt injection defenses, PII filtering, output classifier

### 7. Fine-Tuning from Captured Feedback
- Builds on `13-fine-tuning/`
- Preference pairs from user feedback → DPO

### 8. Deployment
- Builds on `14-llmops-deployment/`
- FastAPI + container + gateway + serving

### 9. Observability, Cost Tracking, Incident Playbook
- Builds on `14-llmops-deployment/`
- Dashboards, alerts, runbooks
- ROI tracking from `16-industry-use-cases-roi/`

### 10. Post-Launch Iteration
- Feedback → data → retrain loop
- Experiment cadence
- Handoff and documentation
