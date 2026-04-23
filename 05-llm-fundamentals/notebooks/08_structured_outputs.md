# LLM 08: Structured Outputs

The gap between "demo" and "product" often comes down to this: demos parse free-form text; products need **reliable, typed, machine-readable** output. This session covers the mechanisms and patterns.

## 1. Why Structured Output

- Downstream systems need typed data (DB inserts, API calls, UI rendering).
- Free-text parsing with regex is brittle; even the best LLM misses edge cases.
- Typed contracts let you validate, version, and monitor.
- Structured output unlocks tool use and agentic behavior (next session).

## 2. The Mechanisms, Ranked

From most reliable to least:

### 2a. Provider-enforced JSON/schema modes

- **OpenAI `response_format={"type": "json_schema", ...}`** — the serving stack masks invalid tokens at decode time, guaranteeing schema-valid output.
- **Anthropic `tool_use`** — Claude's function calling returns a typed object (validated against the JSON Schema you supply).
- **Google Gemini `response_schema`** — similar.
- **Ollama/vLLM `format="json"`** — guarantees valid JSON (not a specific schema).
- **Outlines / LM Format Enforcer** — open-source, applies grammar masks client-side to any model.

**Preference:** use these when available. They're the only way to guarantee schema validity.

### 2b. Function / Tool calling

Define a function with JSON Schema; the model returns the chosen function + typed args:

```json
{
  "name": "create_ticket",
  "arguments": {"title": "DB down", "priority": "high"}
}
```

Every major model supports this. Ideal when the model's job is to choose from a set of structured actions.

### 2c. Pydantic + retry (instructor pattern)

```python
class Invoice(BaseModel):
    vendor: str
    amount: float
    due: date

client.chat.completions.create(
    response_model=Invoice,
    messages=[...],
)
```

Libraries (`instructor`, `marvin`, `outlines`) wrap JSON mode with Pydantic validation and automatic retries on validation failure. Clean developer experience, slightly less robust than provider-enforced schema mode.

### 2d. JSON-mode-only (no schema validation)

```python
response_format={"type": "json_object"}
```

Guarantees valid JSON syntax but not your schema. You still need client-side validation.

### 2e. XML tags (Claude-preferred fallback)

```
Return the extracted fields inside tags:
<vendor>...</vendor>
<amount>...</amount>
```

No guarantee, but Claude's training makes it reliable at tag-structured output. Easier than JSON when human-readability matters.

### 2f. Prompting alone ("Return JSON")

The least reliable. Works most of the time; fails on edge cases. Don't ship it without validation.

## 3. Schema Design

Good LLM schemas are flat and explicit:

```json
{
  "type": "object",
  "properties": {
    "vendor":  {"type": "string"},
    "amount":  {"type": "number"},
    "due":     {"type": "string", "format": "date"},
    "line_items": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "description": {"type": "string"},
          "qty":  {"type": "integer"},
          "price": {"type": "number"}
        },
        "required": ["description", "qty", "price"]
      }
    }
  },
  "required": ["vendor", "amount", "due"]
}
```

Tips:

- **Prefer `string` with `format`** over custom date parsing. The model is better at dates than at Unix timestamps.
- **Use `enum`** to constrain categorical fields.
- **Use `required`** aggressively — optional fields invite inconsistency.
- **Use `description` fields** on every property. The model reads them.
- **Keep it flat-ish.** Deeply nested schemas degrade reliability.

## 4. Pydantic in Practice

```python
from pydantic import BaseModel, Field
from datetime import date

class LineItem(BaseModel):
    description: str
    qty: int = Field(ge=1)
    price: float = Field(ge=0)

class Invoice(BaseModel):
    vendor: str = Field(description="Legal vendor name")
    amount: float = Field(ge=0)
    due: date
    line_items: list[LineItem] = Field(default_factory=list)
```

Pydantic gives you:

1. JSON Schema generation (`Invoice.model_json_schema()`).
2. Post-model validation (types, ranges, custom validators).
3. Round-trip: parse → validate → use. Serialize → log → replay.

`instructor` takes this and adds automatic retry-with-error-feedback on invalid outputs.

## 5. When JSON Mode Isn't Enough

Some cases where even schema-enforced JSON fails:

- **Long outputs hit token limits mid-JSON** → truncated, unparseable. Mitigation: set `max_tokens` wide, stream and progressively parse, or split the task.
- **Correct JSON, wrong semantics** → model returns valid structure with hallucinated values. Mitigation: add examples, add validation, add grounding (RAG), add an eval harness.
- **Schema forces the model into bad choices** → too-restrictive enums miss real variants. Mitigation: add a `"other"` or `"reason_if_none"` field.

## 6. XML Tags for Extraction (Claude)

When schema mode is overkill or you want human-readable output:

```
Extract:
<vendor>...</vendor>
<amount>...</amount>
<line_items>
  <item>
    <desc>...</desc>
    <qty>...</qty>
  </item>
</line_items>
```

Parse with a simple regex or `lxml`. Good for extraction tasks inside chat applications where you also want a natural-language explanation alongside the structured fields.

## 7. Streaming Structured Output

Problem: JSON isn't valid until the closing `}`. How do you stream it?

- **Stream-aware parsers** (`partial-json-parser`, `jsonstreamparse`) accept in-progress JSON and return `{"key": "partial va..."}`.
- **Progressive rendering** — parse up to the last valid comma, render what you have.
- **Send each top-level field separately** for chat UIs.

Covered in `10_streaming_and_incremental_rendering`.

## 8. Error Handling

| Failure | Remedy |
| --- | --- |
| Invalid JSON | Re-prompt with the parse error; retry up to N=2 |
| Valid JSON, schema violation | Re-prompt with Pydantic error; retry up to N=2 |
| Max tokens hit mid-generation | Increase `max_tokens`; consider splitting |
| Repeated failures | Log, alert, fall back to unstructured + human review |
| Model picks non-existent enum | Constrain via schema; audit training data quality |

Never retry infinitely. Cap at 2–3 attempts; log and surface failures.

## 9. Architect Patterns

1. **Never parse by regex in production.** Always typed.
2. **Validate at the boundary.** Pydantic at every LLM→app interface.
3. **Log the raw response** before parsing, so you can debug later.
4. **Version your schemas.** Breaking schema changes need rollout discipline.
5. **Measure structured-output success rate** as a SLI, alongside latency and cost.

## Prereqs / Connections

- `06_decoding_strategies` — structured decoding is grammar-masked sampling
- `07_prompt_engineering_intro` — schema mode still benefits from a good system prompt
- `10-agentic-ai/02_tool_use_basics` — tool use *is* schema-constrained output
- `11-evaluation/` — measure structured-output success rate

## Further Reading

- OpenAI, "Structured Outputs" blog + docs
- Anthropic, "Tool use" docs
- Willard & Louf 2023, "Efficient Guided Generation for LLMs"
- `instructor` library docs (Python)
- `outlines` library docs
