# LLM 07: Prompt Engineering (Intro)

Brief overview — the full treatment lives in `06-prompt-engineering/`. This session establishes the mental model and common patterns so later LLM-fundamentals sessions (structured outputs, context windows, evals) can reference them.

## 1. Why Prompt Engineering Still Matters in 2026

Every year someone declares prompt engineering dead because "the next model will just do what I mean." Then the next model ships, and we're still writing prompts. Reasons:

1. **Unit economics** — prompt choice is a direct lever on token cost.
2. **Reliability** — well-structured prompts reduce variance.
3. **Portability** — prompts that state constraints explicitly survive model swaps.
4. **Auditability** — explicit instructions are easier to review than emergent behavior.

Prompt engineering isn't going away; it's becoming more disciplined.

## 2. Anatomy of a Prompt

Modern chat models take a list of messages, not a single string. Each message has a role:

```
system:    "You are a helpful assistant that answers in one sentence."
user:      "What is the capital of France?"
assistant: "Paris."
user:      "And of Germany?"
```

The **system** message sets persistent context (persona, policies, tools, format). **user** and **assistant** alternate to form the conversation. The model completes from the last turn.

Inside a single message, the convention for architects:

```
<instruction>
<context / reference material>
<examples (few-shot)>
<task input>
<output format spec>
```

## 3. Zero-Shot vs Few-Shot

**Zero-shot** — no examples, just task description. Works when the task is well-known to the model.

**Few-shot** — include 1–5 worked examples. Works when the task is unusual, has a specific format, or you want to show style.

Rule of thumb: start zero-shot, add few-shot only if quality is insufficient.

## 4. Chain-of-Thought (CoT)

Adding "Let's think step by step" — or equivalent — makes the model produce intermediate reasoning before the final answer. On reasoning tasks (math, logic, planning) CoT improves accuracy substantially.

```
user: A shop has 3 apples. 5 more are added. Then 2 are sold. How many remain?
user: Let's think step by step.
```

For reasoning models (o1, Claude thinking mode, DeepSeek-R1) the model does CoT internally; you don't need to ask. Over-using "step by step" on these can hurt.

## 5. Role and System Prompts

A strong system prompt is worth more than elaborate user messages. Example patterns:

```
You are an expert SQL analyst. Always:
- Use the provided schema.
- Return a single SQL query in a ```sql block.
- If the request is ambiguous, ask one clarifying question.
```

Claude-specific: use XML tags (`<instructions>...</instructions>`, `<example>...</example>`) — the model was trained to attend to them more reliably than markdown.

## 6. Structured Outputs

When you need machine-readable output:

- **JSON mode / response_format** (OpenAI, Anthropic, most providers) — the serving stack enforces valid JSON.
- **Pydantic + `response_model`** (via `instructor`, `anthropic-tools`) — ties a schema to validation and retry.
- **XML tags** (Claude) — reliable for extraction without strict JSON.

Deep dive in `08_structured_outputs`.

## 7. Patterns Worth Knowing

| Pattern | What | When |
| --- | --- | --- |
| Role prompt | "You are …" | Always |
| Few-shot | In-context examples | Unusual tasks / formats |
| CoT | "Think step by step" | Non-reasoning models on reasoning tasks |
| Self-consistency | Sample N times, vote | Hard reasoning with small compute budget |
| ReAct | Thought → Action → Observation loop | Tool-using agents (`10-agentic-ai/`) |
| Retrieval prompt | Append grounded passages | Knowledge Q&A (`09-rag/`) |
| XML tags | `<context>...</context>` | Claude; any model with multi-section prompts |
| Delimiters | Triple-backticks or `===` | Separating untrusted input from instructions |

## 8. Pitfalls

- **Prompt injection** — user input is not code; if you concatenate it into the system prompt, you've just handed them the steering wheel. Deep dive in `12-ai-safety-governance/02_prompt_injection`.
- **Over-steering** — too many constraints cause the model to ignore or contradict them.
- **Model-specific quirks** — what works on GPT-4 may fail on Claude and vice versa; portability takes effort.
- **Invisible whitespace** — trailing spaces, newlines, or BOM characters can change output. Canonicalize inputs.

## 9. How to Iterate

1. **Start with a minimal prompt.** One sentence of instruction, the task.
2. **Add the output format** explicitly.
3. **Run on 10+ representative inputs.** See the failures.
4. **Add the smallest thing** that fixes the biggest failure class.
5. **Track versions.** A/B evaluate. Don't ship via vibes.

## Prereqs / Connections

- `02_transformer_architecture` — why position & attention matter for long prompts
- `06_decoding_strategies` — decoding params interact with prompt quality
- `06-prompt-engineering/` — full module (templates, optimization, testing)
- `11-evaluation/` — how to measure prompt changes

## Further Reading

- OpenAI Prompt Engineering Guide
- Anthropic Prompt Engineering Docs (especially the XML-tags section)
- Wei et al. 2022, "Chain-of-Thought Prompting Elicits Reasoning in LLMs"
- Wang et al. 2022, "Self-Consistency Improves Chain of Thought Reasoning"
- Yao et al. 2022, "ReAct: Synergizing Reasoning and Acting in Language Models"
