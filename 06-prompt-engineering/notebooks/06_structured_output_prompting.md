# Prompt 06: Structured Output Prompting

Structured output is covered mechanically in `05-llm-fundamentals/08_structured_outputs`. This session is about the **prompt-level** craft: how to describe the schema, what to put in descriptions, where to place it, how to mix it with free-text, and when schema-less tagged output beats JSON.

## 1. What This Session Adds

`05/08` covered:
- JSON mode, tool use, Pydantic + retry, XML tags.

This session adds:
- How to **describe** fields so the model populates them correctly.
- When to use JSON vs XML vs other tag schemes.
- How to balance **structured + free-text** output.
- Placement within the prompt anatomy.
- Common schema mistakes that look fine but degrade quality.

## 2. Schema Descriptions Are Prompts

Every field description is a mini-prompt the model reads:

```json
{
  "vendor": {
    "type": "string",
    "description": "Legal vendor name as printed on the invoice header. Do NOT abbreviate or translate."
  },
  "amount": {
    "type": "number",
    "description": "Total amount due in USD. Include tax. Numeric only — no currency symbol."
  }
}
```

Rules:

- **Describe edge cases.** "If multiple vendors appear, use the one in the top-right header."
- **Forbid common mistakes.** "Do NOT include the dollar sign." "Do NOT approximate."
- **Include format hints.** `"ISO-8601 date"`, `"snake_case identifier"`.
- **Use examples.** Single phrase examples inside description strings help: *"e.g., 'alice@corp.com'"*.

A mediocre schema with great descriptions beats a great schema with generic descriptions.

## 3. Top-Level Order

Inside a single JSON object, place fields in the order the model should fill them:

```json
{
  "citations_used": [...],    // first: model collects evidence
  "reasoning": "...",          // second: reasons from evidence
  "answer": "...",             // third: produces answer
  "confidence": "..."          // last: assesses own confidence
}
```

The model generates top-to-bottom. Each later field conditions on the earlier ones. This makes the model "think before answering" in the structured output itself — free chain-of-thought without a separate prompt.

Flip side: if you put `"answer"` first and then ask for `"reasoning"`, the reasoning becomes post-hoc rationalization. Almost always worse.

## 4. Structured + Free-Text Hybrid

Sometimes you want a mix:

```
<analysis>
Free-form analysis that a human might read. Use markdown.
</analysis>

<extracted>
{ "vendor": "...", "amount": ... }
</extracted>
```

Prompt:

```
Produce two sections:
1. <analysis>: prose summary of the invoice for a human reviewer.
2. <extracted>: JSON object with the exact fields and no extra text.
```

Parse each tag independently. Use `<extracted>` for downstream systems; show `<analysis>` to the user.

## 5. XML vs JSON in Prompts

Claude is trained on extensive XML-tagged data. For extraction-shaped tasks, XML tags are often more reliable than JSON:

```xml
<invoice>
  <vendor>NovaCore</vendor>
  <date>2026-04-22</date>
  <line_items>
    <item>
      <desc>Widget Pro</desc>
      <qty>3</qty>
      <price>25.00</price>
    </item>
  </line_items>
  <total>109.50</total>
</invoice>
```

Pros:
- More forgiving — no strict quoting, commas, braces.
- Easier to parse with regex on partial output.
- Plays well with hybrid prose + structured output.

Cons:
- No schema enforcement at decode time.
- Less standard; harder to pipe into existing tools.

Use XML when shipping Claude and quick parsing is acceptable. Use JSON schema mode when correctness is critical.

## 6. Delimiters When the Model Won't JSON-Mode

When the provider doesn't offer strict JSON mode or you're rolling your own:

```
Return your answer INSIDE this structure:
BEGIN_ANSWER
<the JSON>
END_ANSWER
```

Parse between the markers. Simple, effective, resistant to surrounding prose.

## 7. Optional Fields and Null Handling

LLMs often fabricate plausible values for optional fields rather than returning `null`. To force explicit null:

```json
{
  "reported_revenue_usd": {
    "type": ["number", "null"],
    "description": "Revenue figure if explicitly stated. If not present in the text, return null. Do NOT infer, estimate, or compute."
  }
}
```

The `null` type union + explicit instruction is the pattern. Without it, you get `"approximately $10M"` or `-1` or made-up numbers.

## 8. Enums: Explicit Beats Inferred

Always enumerate categorical values. Otherwise the model invents new labels:

```json
{
  "status": {
    "type": "string",
    "enum": ["open", "in_progress", "resolved", "escalated"],
    "description": "Current ticket status."
  }
}
```

Even better, describe each enum value:

```
"enum": ["open", "in_progress", "resolved", "escalated"],
"description": "open=newly filed, in_progress=assigned to an agent, resolved=fixed with customer confirmation, escalated=sent to tier-2."
```

This is the single highest-ROI schema improvement.

## 9. Array Length Constraints

LLMs often produce 3 when you asked for 5 items (or vice versa). Force length explicitly:

```
"description": "Return EXACTLY 5 bullet points."
"minItems": 5, "maxItems": 5
```

Or use an object with numbered keys:

```json
{"bullet_1": "...", "bullet_2": "...", "bullet_3": "...", "bullet_4": "...", "bullet_5": "..."}
```

Objects with fixed keys are more reliable than arrays with length constraints.

## 10. Nested Schemas and Reliability

Deeply nested schemas degrade reliability. Each level of nesting adds syntactic burden. Preferences:

| Depth | Reliability | When to use |
| --- | --- | --- |
| Flat | Highest | Default |
| 1 nested (arrays of objects) | High | Line items, bullets |
| 2+ nested | Lower | Only when genuinely hierarchical |
| 3+ nested | Fragile | Almost never; split into multiple schemas |

For deep hierarchies, consider multiple calls (extract top level; extract sub-level for each).

## 11. The "Reason for Missing" Pattern

Instead of just a nullable field, require a reason:

```json
{
  "vendor_name": {"type": ["string", "null"]},
  "vendor_name_source": {"type": "string", "description": "'header', 'footer', 'body', or 'not_found'"}
}
```

This forces the model to demonstrate it actually looked. Dramatically reduces the "invented but nulled" failure mode.

## 12. Format-Coercion Tricks

- **Assistant prefill** — `{"` or `<answer>` — for Claude.
- **Type hints in description** — "(number, two decimal places)".
- **Forbidden tokens** via logit_bias — rarely needed with modern schema modes.
- **Post-validation retry** — re-prompt with Pydantic error.

## 13. Architect Takeaways

- **Descriptions do more work than types.**
- **Order fields** to encourage reasoning-before-answer.
- **XML tags** for Claude extraction; **JSON schema mode** for strict correctness.
- **Enumerate enums.**
- **Flatten schemas** when possible.
- **Require reasons for nulls** to prevent silent fabrication.

## Prereqs / Connections

- `02_anatomy_of_a_prompt` — where structured output sits
- `05-llm-fundamentals/08_structured_outputs` — mechanism
- `08_advanced_techniques` — schema-aware agents
- `11-evaluation/` — structured-output success rate as SLI

## Further Reading

- Anthropic, "Use XML tags to structure your prompts"
- OpenAI, "Structured Outputs" docs
- `instructor` docs: description-first schema design
- `outlines` docs: grammar/regex-constrained generation
