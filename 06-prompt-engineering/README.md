# Prompt Engineering Module

How to get the most out of an LLM without training it. Covers patterns, structure, optimization, and systematic testing.

## 🚀 Roadmap

### 1. Why Prompt Engineering Still Matters
- Prompt vs fine-tune vs RAG tradeoffs
- Portability across models
- Cost and latency implications

### 2. Anatomy of a Prompt
- System, user, assistant roles
- Instruction, context, examples, output format
- Claude/OpenAI/Gemini conventions

### 3. Zero-Shot, Few-Shot, N-Shot
- When examples help vs hurt
- Example selection strategies
- Dynamic few-shot (retrieval-based)

### 4. Chain-of-Thought & Reasoning Prompts
- Basic CoT
- Step-back prompting
- Self-consistency
- Tree-of-thought

### 5. Role Prompting & Personas
- System prompts as contracts
- Role stability over long conversations
- When personas backfire

### 6. Structured Output Prompting
- XML tags (Claude), JSON schema, Markdown
- Prereq: `05-llm-fundamentals/08_structured_outputs`

### 7. Prompt Templates & Variables
- Jinja2, LangChain templates
- Variable escaping and injection safety
- Prereq: `12-ai-safety-governance/02_prompt_injection`

### 8. Advanced Techniques
- ReAct prompting (bridge to `10-agentic-ai/`)
- Self-critique and refine
- Multi-turn prompting patterns

### 9. Prompt Optimization
- DSPy basics
- APE (Automatic Prompt Engineer)
- Human-in-the-loop tuning

### 10. Testing & Evaluation of Prompts
- Snapshot tests
- Regression suites
- Deep dive in `11-evaluation/`

### 11. Prompt Libraries & Versioning
- Prompt registries (PromptLayer, Humanloop)
- Versioning strategy
- Rollout and rollback

### 12. Multi-Modal Prompting
- Image + text prompts
- Audio prompts
- Prompting for code generation
