# Prompt 02: Anatomy of a Prompt

Before writing better prompts, we need a shared vocabulary for their parts. This session dissects a production-grade prompt into its components, shows how each contributes, and gives you a reusable template.

## 1. The Message List

Modern chat APIs take a **list of messages**, not a single string. The structure:

```python
[
  {"role": "system", "content": "<policies + persona + format + tools>"},
  {"role": "user",   "content": "<few-shot example 1 input>"},
  {"role": "assistant", "content": "<few-shot example 1 output>"},
  {"role": "user",   "content": "<few-shot example 2 input>"},
  {"role": "assistant", "content": "<few-shot example 2 output>"},
  {"role": "user",   "content": "<actual task>"},
]
```

Why multi-message matters:

- The model was trained on chat templates — it responds better to well-formed conversations.
- Few-shot examples as `(user, assistant)` pairs beat in-message examples in most evals.
- System vs user separation survives tokenizer changes.
- Some providers cache the system message separately (Anthropic).

## 2. The System Message

The highest-leverage sentence in your product.

Structure (ordered):

```
1. Identity / persona
2. Task scope (what you do; what you refuse)
3. Output format
4. Constraints (tone, length, language)
5. Tools (if any)
6. Safety boundaries
```

Example:

```
You are PolicyBot, an HR benefits assistant.

SCOPE
- Answer questions about employee benefits, PTO, and healthcare.
- Do not answer medical, legal, or tax questions; refer to professionals.

FORMAT
- Answer in 2-4 sentences.
- Cite the specific policy section in brackets: [Policy 4.3].

CONSTRAINTS
- Do not speculate. If the answer is not in the provided policy excerpts, say so.
- Never include PII in responses.

TOOLS
- lookup_policy(query): search the HR policy knowledge base.
- open_ticket(subject): escalate to HR.
```

Every one of those lines does work. Remove any one and quality drops measurably.

## 3. The Context Block

After the system message, you typically have retrieved or supplied context. Delimit it clearly:

```
<context>
[Policy 4.3] Full-time employees accrue 15 days PTO per year...
[Policy 4.4] PTO may be rolled over up to 5 days...
</context>

Question: How many days can I roll over?
```

Delimiters help the model distinguish context from instructions *and* defend against prompt injection from user content in that context (covered in session 08).

On Claude: **XML tags** (`<context>`, `<policy>`, `<example>`). Trained to respect them.
On GPT/Gemini: **Markdown or three-hash delimiters** (`### CONTEXT ###`) work well.

## 4. Few-Shot Examples

Few-shot in the message list (preferred):

```
user:      Name: Alice, hire: 2022-01-15
assistant: {"name": "Alice", "tenure_years": 4}
user:      Name: Bob, hire: 2019-06-01
assistant: {"name": "Bob", "tenure_years": 6}
user:      Name: Carol, hire: 2015-09-30
```

Few-shot inline (less preferred but sometimes necessary):

```
Examples:
Input: A, output: X
Input: B, output: Y
---
Input: C, output:
```

## 5. The User Message

Most of the craft concentrates here. A good user message has:

- **Task statement** — "Summarize this article in three bullets."
- **Input demarcation** — "ARTICLE: <<<...>>>" or `<article>...</article>`.
- **Output spec reminder** — at the end, re-state the format. The model attends strongly to the final tokens of the input.

Pattern:

```
Summarize the article below in exactly 3 bullets, each under 15 words.

<article>
...
</article>

Return only the 3 bullets.
```

The re-statement at the end compensates for lost-in-the-middle effects.

## 6. The Assistant Prefix (Claude and some GPTs)

You can seed the assistant's response to steer the shape:

```
assistant: {"name":
```

Claude will continue from that prefix, making JSON output much more reliable. OpenAI supports prefill only on some endpoints. Use sparingly but deliberately.

## 7. Tool Schemas as Part of the Prompt

When tools are available, their schemas are part of the context the model attends to. Good schemas include:

- Clear **descriptions** on the tool and every parameter
- **Examples in the description** (not just types)
- Explicit **enum constraints** where possible

Bad tool descriptions are a common cause of "the model never uses the tool."

## 8. The Anatomy Template

A production-ready skeleton:

```
SYSTEM
[identity]
[scope + refusals]
[output format spec]
[tone/length constraints]
[safety policies]

CONTEXT (dynamic, delimited)
<docs>retrieved passages</docs>

FEW-SHOT (2-5 pairs)
user → assistant → user → assistant ...

TASK (final user message)
[task statement]
<input>user-provided content</input>
[output spec reminder]
```

Memorize that skeleton. Every complex feature is a variation of it.

## 9. Order Sensitivity

Position within the prompt changes behavior:

- Early tokens anchor role and constraints.
- End tokens anchor format and task.
- Middle tokens are vulnerable to being skimmed (lost-in-middle).

Heuristics:
- Put the **most important rule** at the start of the system prompt AND repeat it briefly before the task.
- Put **bulk context** in the middle — it's what the model reasons over.
- Put the **task + format** at the end — right next to where generation begins.

## 10. What NOT to Put in Prompts

- **API keys, secrets, credentials.** They'll leak.
- **Unescaped user input concatenated as instructions.** Injection vector.
- **Contradictory rules.** Pick one and be explicit.
- **Implicit assumptions.** If you're thinking it, state it.
- **Prose about how great the model is.** "You are a brilliant expert" often hurts more than it helps on modern models.

## 11. Debugging a Bad Prompt

When output is wrong, diagnose in order:

1. **Is the task statement ambiguous?** Would a new hire understand what you asked?
2. **Is the format spec precise?** "JSON" is not a format; a schema is.
3. **Do the few-shot examples actually match the format?** Inconsistency in examples destroys signal.
4. **Is the context sufficient?** Does the answer actually exist in what you provided?
5. **Is there a contradiction between system and user?** System says "be terse," user says "explain thoroughly."
6. **Did the model get confused by user-provided text being indistinguishable from instructions?** Delimit.

Most "the model is dumb" complaints resolve at step 1 or 2.

## 12. Architect Takeaways

- **Every production prompt has four layers:** system, context, few-shot, task.
- **System prompts are contracts.** Write them like contracts.
- **Delimit untrusted content** with tags or fences.
- **Repeat the task spec at the end** to fight lost-in-middle.
- **Prefix the assistant** when format reliability matters.

## Prereqs / Connections

- `01_why_prompt_engineering` — the motivation
- `05-llm-fundamentals/09_context_windows_and_long_context` — order sensitivity comes from attention
- `07_templates_and_variables` — implementing the anatomy in code
- `12-ai-safety-governance/02_prompt_injection` — why delimiters matter

## Further Reading

- Anthropic "Prompt engineering overview" — canonical Claude anatomy
- OpenAI "Prompt Engineering Guide" — canonical GPT anatomy
- Jeremy Howard, "A Hackers' Guide to Language Models" (video)
- Lilian Weng, "Prompt Engineering" (blog)
