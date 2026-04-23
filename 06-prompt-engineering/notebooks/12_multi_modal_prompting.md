# Prompt 12: Multi-Modal Prompting

The capstone for module 06. Prompting vision, audio, and code-generation models requires adaptations on top of everything covered so far. This session covers those adaptations.

## 1. What Changes with Multi-Modal

Text prompting assumes the prompt *is* the signal. Multi-modal prompts interleave modalities:

- **Images** — screenshots, documents, charts, faces (with care).
- **Audio** — calls, meetings, voice commands.
- **Video** — recorded or streamed.
- **Code** — treated distinct because of its syntactic structure.

Each modality changes:
- **Token economics** (images/audio are dense in tokens).
- **Order sensitivity** (where you place the image in the message matters).
- **Schema patterns** (extraction from images has different best practices).
- **Hallucination profile** (different failure modes than text-only).

## 2. Prompting with Images

Two placement patterns:

**Image first, then text question:**
```
[image]
"What is the total amount on this invoice?"
```
Fast, focused. Good for single-image tasks.

**Text context, then image, then question:**
```
"You will extract structured data from an invoice."
[image]
"Return JSON: {vendor, total, due}."
```
Better when the question has rich instructions.

Avoid: instructions AFTER the image with no preamble. Models sometimes don't "remember" the image content well enough to follow later instructions.

## 3. Prompt Patterns for Documents and OCR

When the image is a document:

- **Start with role:** "You extract fields from scanned documents."
- **Specify layout hints:** "The vendor name is usually in the top-left."
- **Force a schema:** JSON or XML-tagged.
- **Ask for bounding-box references** when exact provenance matters: "For each field, include the approximate region: top|middle|bottom, left|center|right."
- **Handle multi-page:** send pages as separate images; ask the model to aggregate.

## 4. Chart and Figure Prompts

For data visualizations:

```
This is a bar chart. Extract:
1. The axis labels.
2. The title.
3. Each series: label and estimated values.
4. Any annotations or callouts.

Return as JSON matching: {title, x_axis, y_axis, series: [{label, data: [...]}], annotations: [...]}
```

Followed by: "Explain what the chart shows in one sentence."

Models are surprisingly good at this. Don't expect exact values for dense charts — ballparks.

## 5. Screenshot Agents

For UI navigation:

```
You are an automation agent. You see screenshots and decide actions.

Available actions:
- click(x, y)
- type(text)
- scroll(direction)
- wait

Emit ONE action. State your reasoning.
```

Best practices:
- **Show screenshot before prompt.** The model's "plan" conditions on the visual.
- **Include coordinate grid** as overlay or reference.
- **Limit to one action** per turn to make behavior debuggable.
- **Save trajectory** of screenshots + actions for replay.

Deep dive in `10-agentic-ai/02_tool_use_basics` and screenshot-driven agents.

## 6. Prompting Image Generation

Different model class (diffusion/flow-matching), different prompt style. Generally:

- **Long, descriptive prompts** — unlike text LLMs, verbose helps.
- **Explicit style, lighting, composition** — "cinematic lighting, 35mm, golden hour."
- **Negative prompts** (where supported) — "no text, no watermarks, no extra limbs."
- **Reference images** (ControlNet, IP-Adapter, Stable-Diffusion variants) for precise control.

Modern image APIs (DALL·E-3, Imagen 3, Flux 1) expand prompts internally — your prompt is "lifted" to a richer one. Test exact vs. liberal phrasing.

## 7. Audio Prompts

For ASR (Whisper and friends):
- **Provide language hint** when the audio is short.
- **Provide domain glossary** for technical content (medical, legal) — Whisper accepts an `initial_prompt` with specialized terminology.

For realtime voice (GPT-4o-Realtime, Gemini Live):
- **Persona stays in system** as usual.
- **Turn-taking** is typically automatic but you can force pauses with explicit silence.
- **Interruption** needs explicit handling.

## 8. Code Generation Prompts

Code is text, but the patterns differ:

- **Specify language, version, framework.** "Python 3.11, FastAPI 0.110."
- **Specify style.** "Type hints on all signatures. Use `pytest`, not `unittest`."
- **Provide imports.** Often improves correctness.
- **Describe the surrounding code.** "This function is called from a Celery task."
- **Ask for tests alongside.** Catches many silent bugs.
- **Constrain to single-file or single-function** for cleaner outputs.
- **Specify edge cases:** "Handle: empty input, negative numbers, duplicates."

For "edit this code" tasks, use anchored diffs:

```
Here is the function. Make it handle empty input by returning None.
Return ONLY the updated function.

<function>
def mean(xs):
    return sum(xs) / len(xs)
</function>
```

## 9. Hallucination Profiles Differ

| Modality | Common hallucinations |
| --- | --- |
| Image extraction | Misread digits; invent vendors not shown |
| Chart reading | Fabricate series not in the image; guess values |
| Screenshots | Click on imagined elements |
| Audio | Hallucinate speech not spoken |
| Code | Imaginary APIs, wrong package versions |

Mitigations:
- **Ground with retrieval** (API docs, product manual) where applicable.
- **Require confidence flags** and `not_found` values.
- **Pair with deterministic validators** (regex, API call, compile, AST check).
- **Ask the model to cite its evidence** ("which region of the image shows the vendor name?").

## 10. Cost Implications

- A single high-resolution image can cost ~1K–2K tokens.
- A 10-minute video in Gemini 1.5 can cost 100K+ tokens.
- Audio routed through ASR then text LLM is two inference costs.
- Realtime voice is charged per minute of audio, typically 10–100× text per-token cost.

Budget accordingly. For high-volume pipelines, consider:
- OCR → text LLM instead of vision LLM for documents.
- Audio → ASR → text LLM instead of realtime voice.
- Frame sampling instead of full video.

## 11. Structured Outputs with Images

Combine vision extraction with strict schema:

```
<task>
Extract from the invoice image: vendor, total, due.
</task>

<instructions>
Return JSON matching:
{vendor: string, total_usd: number, due: "YYYY-MM-DD" or null}

For each field, include confidence: "high" if clearly visible, "medium" if partially obscured, "low" if unclear.
</instructions>
```

Same schema patterns (enums, ordering, null handling) from session 06 apply.

## 12. Architect Takeaways

- **Image placement matters.** Lead with image for single-image; with context for instructions.
- **Document / chart / screenshot have distinct patterns** — know them.
- **Image generation has its own prompt language.** Verbose is good.
- **Audio realtime is a different cost curve.**
- **Code prompts need language + version + style + tests.**
- **Multi-modal hallucinations are different** — pair with validators.

## Prereqs / Connections

- `05-llm-fundamentals/14_multi_modal_basics` — the mechanisms
- `06_structured_output_prompting` — schemas apply here too
- `09-rag/13_multi_modal_rag`
- `10-agentic-ai/02_tool_use_basics` — screenshot agents

## Further Reading

- OpenAI, "Vision guide" in API docs
- Anthropic, "Vision capabilities" docs
- Google, "Gemini 1.5 / 2 multimodal" docs
- Dang et al. 2024, "How to Prompt Vision Language Models"
- AnthropicQuickstarts: `computer-use-demo` (screenshot agent reference)
