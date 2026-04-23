# Cloud 01: Cloud AI Services

## 1. The Three Major Clouds for AI

| Capability | AWS | GCP | Azure |
| :--- | :--- | :--- | :--- |
| **Managed LLMs** | Amazon Bedrock (Claude, Llama, Titan) | Vertex AI (Gemini, Llama) | Azure OpenAI (GPT-4, o1) |
| **Training Platform** | SageMaker | Vertex AI Training | Azure ML |
| **Vision** | Rekognition | Vision AI | Computer Vision |
| **Speech** | Transcribe, Polly | Speech-to-Text, TTS | Azure Speech |
| **Vector Search** | OpenSearch (kNN) | Vertex Matching Engine | Azure Cognitive Search |
| **Feature Store** | SageMaker Feature Store | Vertex Feature Store | Azure ML Feature Store |
| **Serverless Inference** | Lambda + SageMaker Serverless | Cloud Run + Vertex | Azure Functions + ACI |

---

## 2. Managed LLMs: Hosted APIs

**Amazon Bedrock**:
*   Access Claude, Llama, Mistral, Amazon Titan through one API.
*   Data stays in your AWS VPC — key for compliance.
*   Pay per token; no GPU management.

**Google Vertex AI (Gemini)**:
*   Gemini 1.5 Pro/Flash, Llama, Gemma.
*   Tightly integrated with BigQuery and Google Workspace.
*   Model Garden for open model hosting.

**Azure OpenAI Service**:
*   GPT-4o, o1, DALL-E — same models as OpenAI but inside Azure.
*   Private endpoint, no data sent to OpenAI infrastructure.
*   HIPAA, SOC 2, FedRAMP compliant.

---

## 3. Managed vs. Self-Hosted Models

| Factor | Managed API | Self-Hosted (e.g., vLLM on GPU) |
| :--- | :--- | :--- |
| **Setup time** | Minutes | Days–weeks |
| **Ops burden** | None | High (patching, scaling, monitoring) |
| **Cost at low volume** | Lower | Higher (idle GPU) |
| **Cost at high volume** | Higher (per-token) | Lower (reserved GPU) |
| **Data privacy** | Shared infra (unless VPC) | Full control |
| **Model selection** | Limited to provider's models | Any open model |
| **Customization** | Prompt + fine-tune only | Full model control |

**Decision rule**: Start with managed APIs. Self-host only when: privacy requires it, volume justifies GPU cost, or you need a specific open model.

---

## 4. Vendor Lock-in Considerations

High lock-in risks:
*   Proprietary vector databases (Pinecone → can't switch without re-embedding).
*   SageMaker Pipelines (AWS-specific DSL).
*   Vertex Feature Store (GCP-specific).

Low lock-in strategies:
*   Use OpenAI-compatible APIs (many providers support this).
*   Abstract LLM calls behind an interface.
*   Use portable formats (Parquet, ONNX, HuggingFace).
*   Prefer open models + vLLM for critical workloads.

---

## 5. Multi-Cloud AI Architecture

Some organizations use multiple clouds for AI:
*   AWS for primary compute and storage.
*   Azure OpenAI for GPT-4 compliance (no Azure data leaves Azure).
*   GCP for BigQuery analytics + Vertex for certain models.

The overhead of multi-cloud is significant. Only do it when regulatory or vendor risk justifies the complexity.
