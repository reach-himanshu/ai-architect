# LLM Fundamentals Module

The mental model every AI Architect needs: how transformers work, how to prompt them, how to measure them, and what they cost.

## 🚀 Roadmap

### 1. Neural Net Refresher for LLMs
- Forward pass, backprop, gradients (just enough)
- Why scale matters
- Pretrained vs instruction-tuned vs RLHF-tuned

### 2. Transformer Architecture
- Self-attention, multi-head attention
- Encoder vs decoder vs decoder-only
- Positional encoding (absolute, rotary/RoPE)

### 3. Tokenization
- BPE, SentencePiece, tiktoken
- Why token counts differ across models
- Special tokens, chat templates

### 4. Embeddings
- Static (word2vec) vs contextual (BERT, OpenAI, Cohere)
- Embedding spaces and geometry
- When to use which embedding model

### 5. Pretraining Objectives
- Causal LM (GPT family)
- Masked LM (BERT family)
- Prefix / T5 style

### 6. Decoding Strategies
- Greedy, beam search
- Top-k, top-p (nucleus), temperature
- Repetition penalties, logit bias

### 7. Prompt Engineering (Intro)
- Zero-shot, few-shot, chain-of-thought
- Role prompts, system prompts
- Deep dive in `06-prompt-engineering/`

### 8. Structured Outputs
- JSON mode, tool schemas, Pydantic
- Function calling vs text parsing
- Retry on invalid output

### 9. Context Windows & Long Context
- Effective vs advertised context
- Needle-in-haystack limits
- Compression and chunking strategies

### 10. Streaming & Incremental Rendering
- SSE and token events
- Partial JSON parsing
- UX patterns for streaming apps

### 11. LLM Evaluation (Intro)
- Benchmarks and their limits
- LLM-as-judge preview
- Deep dive in `11-evaluation/`

### 12. Token Economics
- Input vs output pricing
- Latency vs throughput tradeoffs
- Budget modeling for features

### 13. Hallucination & Guardrail Basics
- Why LLMs hallucinate
- Grounding techniques preview
- Deep dive in `12-ai-safety-governance/`

### 14. Multi-Modal Basics
- Vision (image inputs, captioning)
- Audio (Whisper, TTS)
- Architectural implications

### 15. Hands-On: Claude + OpenAI + Local
- Side-by-side comparison
- Provider abstraction pattern
- Prereq: `03-design-patterns/07_repository_registry`
