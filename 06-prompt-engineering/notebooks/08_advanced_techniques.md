# Prompt 08: Advanced Techniques

ReAct, self-refine, self-critique, multi-turn orchestration. These techniques layer on top of the basics and are the bread-and-butter of agent systems. Covered here at the prompt level; mechanics in `10-agentic-ai/`.

## 1. ReAct: Thought + Action + Observation

Yao et al. 2022: interleave reasoning (*Thought*) with tool use (*Action*) and tool results (*Observation*):

```
Thought: I need to find the current weather in Paris.
Action: get_weather(city="Paris")
Observation: 18°C, overcast

Thought: The user also asked about tomorrow. I need a forecast.
Action: get_forecast(city="Paris", days=1)
Observation: Tomorrow: 16°C, rain

Thought: I have both; I can now answer.
Final Answer: Paris is 18°C and overcast today; tomorrow expect 16°C with rain.
```

Why ReAct works:
- **Grounds reasoning in external signals** (tools) rather than parametric memory.
- **Externalizes state**, so the model can re-attend instead of re-derive.
- **Debuggable**: trace shows exactly what was searched and why.

ReAct-style prompts look like:

```
You have access to the following tools: {tool_descriptions}.

For each user request:
1. Write a "Thought:" describing your plan.
2. If you need a tool, write "Action: tool_name(arg1=..., arg2=...)".
3. You will receive "Observation: ..." with the result.
4. Continue Thought → Action → Observation until you can answer.
5. When ready, write "Final Answer: <response>".
```

Modern models with native tool use (Claude, GPT-4, Gemini) don't need literal ReAct text — function calling is the productionized form. But the **mental model** is still ReAct.

## 2. Self-Refine

Madaan et al. 2023: let the model critique and improve its own output.

```
Round 1: generate_initial(prompt)
Round 2: critique(initial_answer)       → feedback: "You skipped the edge case of empty input."
Round 3: revise(initial_answer, feedback)
```

Pattern inside a single prompt or across calls. Works well for:

- Code generation (catch compile / logic issues)
- Summarization (improve faithfulness)
- Creative writing (tighten)
- Error messages (more actionable)

Diminishing returns after 1–2 rounds. The model's critique ability ceiling is the ceiling of this technique.

## 3. Self-Consistency (Revisited)

Covered in session 04 in math/reasoning context. Generalizes:

- Generate N candidates with higher temperature.
- Either vote (for categorical) or pick-best-via-judge (for open-ended).

For open-ended: have a judge model or a rubric-based selector pick the winner.

## 4. Multi-Turn Structured Conversation

Some tasks are poorly served by one-shot prompts. Break them into a guided conversation:

```
Turn 1: model proposes plan    → human/system approves/edits plan
Turn 2: model executes step 1 → system verifies
Turn 3: model executes step 2 → system verifies
...
```

Each turn has a focused system prompt. The model doesn't carry the whole task in its head; each turn is narrow and testable.

## 5. "Decompose, Solve, Recompose"

Complex tasks → break into sub-tasks → solve each → combine.

```
Step 1: "Identify 3 sub-questions needed to answer the full question."
Step 2: For each, call a simpler sub-prompt.
Step 3: "Given these sub-answers, synthesize the final answer."
```

Benefits:
- Each sub-call is simpler → higher accuracy per piece.
- Parallelizable.
- Sub-results are debuggable artifacts.

Tradeoffs:
- More API calls = more latency + cost.
- Mistakes compound if sub-steps are wrong.

## 6. Reflection and Critique Patterns

Varieties:

- **Pure critique**: "Review the answer. Is it correct? What's wrong?"
- **Rubric-based**: "Score on [criteria]. 1-5 each."
- **Adversarial**: "Argue against the answer. Find the strongest counter."
- **Calibration**: "Estimate confidence 0-1 for each claim."

Use critique sparingly — it doubles cost per call. Reserve for high-stakes outputs or when evals show quality lift.

## 7. Iterative Deepening

Start broad, iterate to specifics:

```
Turn 1: "Give 5 candidate approaches in one line each."
Turn 2: "Of those, select the best. Give 3 pros and 3 cons."
Turn 3: "Now implement the approach. Show code."
```

Each turn gets a narrower scope. Models work better on narrow tasks than broad ones; this architecture respects that.

## 8. Parallel Exploration

When the model is uncertain, explore in parallel:

```python
responses = [generate(p, temperature=0.8, seed=s) for s in range(5)]
best = judge(responses)
```

Cheap-ish way to buy reliability when N× model calls are affordable. Related to self-consistency and Tree-of-Thoughts.

## 9. Prompt Chaining

A pipeline of specialized prompts:

```
input → [extract_intent] → [plan] → [execute] → [verify] → [explain] → output
```

Each stage:
- Has its own system prompt.
- Is individually testable.
- Can be reused across features.

Libraries (LangChain, DSPy, LlamaIndex) formalize this. Hand-rolled is fine for simple cases.

## 10. Human-in-the-Loop Patterns

- **Confirm before action** — "I plan to send this email. Show me the draft and approve." Before mutating tool calls fire.
- **Escalate on low confidence** — surface a `needs_human_review` flag.
- **Edit-and-resubmit** — user tweaks the model's output; re-evaluate.

Critical for agents that take real-world actions (send email, modify database, make purchases).

## 11. Anti-Patterns

- **Chaining when one call suffices.** You just 10× cost for no benefit.
- **Critique on trivial outputs.** 2-sentence summaries rarely benefit from critique rounds.
- **Self-consistency on tasks with low baseline accuracy.** If the model is wrong 70% of the time, majority voting reinforces wrong.
- **Parallel exploration without a real judge.** A coin-flip judge is worse than the baseline.

## 12. Architect Takeaways

- **ReAct = reasoning + tools; modern native tool use is its productionized form.**
- **Self-refine** is great for code and safety-critical outputs; diminishing returns past round 2.
- **Self-consistency** is the simplest N× compute lever with proven quality gains on reasoning.
- **Decompose** when sub-tasks beat monolithic calls on your evals.
- **Human-in-the-loop** for consequential actions; not optional.

## Prereqs / Connections

- `04_cot_and_reasoning` — foundation for ReAct + self-consistency
- `05_role_and_personas` — personas survive (or don't) across multi-turn
- `10-agentic-ai/01_agent_fundamentals` — ReAct as agent loop
- `11-evaluation/06_agent_evaluation`

## Further Reading

- Yao et al. 2022, "ReAct: Synergizing Reasoning and Acting in Language Models"
- Madaan et al. 2023, "Self-Refine: Iterative Refinement with Self-Feedback"
- Shinn et al. 2023, "Reflexion: Language Agents with Verbal Reinforcement Learning"
- Wang et al. 2023, "Plan-and-Solve Prompting"
- Khattab et al. 2023, "DSPy: Compiling Declarative Language Model Calls"
