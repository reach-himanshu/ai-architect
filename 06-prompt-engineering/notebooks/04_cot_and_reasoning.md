# Prompt 04: Chain-of-Thought & Reasoning Prompts

For tasks that require multiple steps — arithmetic, logical inference, planning, debugging — plain next-token sampling often fails. **Chain-of-thought (CoT)** and its descendants teach the model to think out loud. This session covers what works, what doesn't, and when modern "reasoning models" obviate these techniques entirely.

## 1. The Core Idea of CoT

Standard prompting:

```
Q: Alice had 5 apples, gave 2 away, bought 3 more. How many now?
A: 6
```

Chain-of-thought:

```
Q: Alice had 5 apples, gave 2 away, bought 3 more. How many now?
A: Let's think step by step. Alice starts with 5. She gives 2 away, leaving 5-2=3.
   She buys 3 more: 3+3=6. Answer: 6.
```

Wei et al. 2022 showed that prompting this reasoning trace improves accuracy by 10–40 points on multi-step tasks. The model doesn't become smarter; it uses its existing capability more reliably by externalizing intermediate state.

## 2. Zero-Shot CoT

The famous trick (Kojima et al., 2022): just add "Let's think step by step."

```
Prompt: ... Let's think step by step.
```

One sentence, no examples, often massive gains on reasoning benchmarks. Works because the model was trained on enough step-by-step traces that the phrase conditions it to generate one.

Limit: introspection is hit-or-miss. On some tasks the model produces a plausible-looking but wrong chain.

## 3. Few-Shot CoT

Show, don't tell:

```
Q: Bob has 12 cookies. He eats 4 and shares half of the rest. How many left?
A: Bob has 12. He eats 4, leaving 12-4=8. He shares half of 8, so he gives away 4.
   He has 8-4=4 cookies. Answer: 4.

Q: Carla has 7 pencils. She buys 3 more, then loses 2. How many does she have?
A: Carla starts with 7. She buys 3, reaching 7+3=10. She loses 2, leaving 10-2=8. Answer: 8.

Q: Alice had 5 apples, gave 2 away, bought 3 more. How many now?
A:
```

Few-shot CoT outperforms zero-shot CoT on harder reasoning tasks. Expensive in tokens but reliable.

## 4. Self-Consistency

Wang et al. 2022: instead of one CoT, generate **N samples with higher temperature** and take a majority vote on the final answer.

```python
answers = [generate(prompt, temperature=0.7) for _ in range(7)]
final_answer = majority_vote([extract_final_answer(a) for a in answers])
```

Costs N× tokens. Substantial accuracy gains on math and logic. Strict upper bound: the model must be at least sometimes-right.

## 5. Tree of Thoughts (ToT)

Yao et al. 2023: formalize reasoning as a search over a tree of partial solutions.

```
Root: problem statement
├── Branch: try approach A
│   ├── Eval: promising
│   │   └── Expand further...
│   └── Eval: dead end → prune
└── Branch: try approach B
    └── ...
```

Each node = a partial reasoning trace. The model both *generates* branches and *evaluates* them. Then a search algorithm (BFS, DFS, heuristic) explores.

ToT improves accuracy on tasks where you can evaluate partial progress, but it's slow and token-heavy. Use when the problem has a "generate-and-verify" structure.

## 6. Graph of Thoughts, Cumulative Reasoning, etc.

Variants in the literature: nodes that combine multiple ancestors, aggregation of partial results, etc. For most production systems, CoT + self-consistency gets 90% of the value; ToT is worth it for specific tasks (puzzles, constrained generation).

## 7. "Let's First Plan" Patterns

An especially practical variant: make the model **plan** before executing.

```
Step 1: Read the question and list the sub-questions.
Step 2: Answer each sub-question.
Step 3: Combine into a final answer.
```

Or the "decomposition" pattern:

```
Before answering, break the question into simpler sub-questions. Answer each.
Then synthesize.
```

Planning prompts reduce early commitment to bad strategies.

## 8. Reasoning Models Change the Picture

In 2025, OpenAI `o1`, DeepSeek-R1, Anthropic Claude "thinking mode," and Google "Deep Think" exposed models whose internals *already* do chain-of-thought — often for thousands of tokens — before responding.

Implications:

- **Don't add "think step by step" to reasoning models.** Can backfire; they already are.
- **The output you see is post-reasoning.** Don't parse a chain from it.
- **Cost and latency jump dramatically.** A reasoning model call may spend 10K+ "thinking" tokens.
- **Some providers expose the thinking trace** (Anthropic: as a `thinking` content block); some don't.

When to use reasoning models:

- Math, logic, coding competition problems
- Multi-step debugging
- Planning with complex constraints
- Anywhere GPT-4/Sonnet-class models fail

When NOT:

- Chat, summarization, rewriting, classification — regular models are faster and cheaper.
- Latency-sensitive UX.
- Cost-sensitive high-volume features.

Architecturally: route by difficulty (session 12 from module 05).

## 9. Pitfalls of CoT

- **Plausible but wrong traces.** Model generates confident nonsense. Mitigation: verify the final answer with an independent call or rule.
- **Chain length blows up token cost.** Mitigation: set `max_tokens`, include "Answer in at most N steps" in the prompt.
- **Mixing CoT with strict-format output.** The chain-of-thought *is* output — if you want just JSON, don't ask for CoT; or have the model emit `<thinking>...</thinking>` tags and extract the JSON separately.
- **Chain-of-thought on tasks that don't benefit** — summarization, classification. Pure cost.

## 10. Hiding the Chain

Sometimes you want the reasoning benefits but don't want the user seeing the trace:

```
Think through the answer inside <thinking>...</thinking> tags.
Then produce the final answer inside <answer>...</answer> tags.
```

Display only `<answer>`. Log `<thinking>` for debugging. Claude's native thinking mode does this implicitly.

## 11. Measuring CoT Impact

Every feature should check: does CoT actually help on *my* task?

```python
metrics = {}
for mode in ['direct', 'zero_shot_cot', 'few_shot_cot', 'self_consistency']:
    accuracy, avg_tokens = run_eval(mode)
    metrics[mode] = (accuracy, avg_tokens)
```

If direct scores 80% and few-shot CoT scores 83% at 5× tokens, CoT is probably not worth it. If CoT moves you from 60% to 85%, it's essential.

## 12. Architect Takeaways

- **CoT is a technique, not a default.** Apply where measurement supports it.
- **Self-consistency is the highest-ROI enhancement** when quality >> cost.
- **Reasoning models obviate manual CoT** on hard tasks; route to them when they earn their cost.
- **Hide the chain when you ship it**; keep it for logs.
- **Verify CoT outputs** — plausible reasoning can wrap wrong conclusions.

## Prereqs / Connections

- `03_zero_few_n_shot` — few-shot CoT extends classic few-shot
- `05-llm-fundamentals/06_decoding_strategies` — self-consistency uses sampling
- `05-llm-fundamentals/12_token_economics` — CoT is a cost lever
- `10_testing_prompts` — measuring reasoning lift

## Further Reading

- Wei et al. 2022, "Chain-of-Thought Prompting Elicits Reasoning in LLMs"
- Kojima et al. 2022, "Large Language Models are Zero-Shot Reasoners"
- Wang et al. 2022, "Self-Consistency Improves Chain of Thought Reasoning"
- Yao et al. 2023, "Tree of Thoughts: Deliberate Problem Solving with LLMs"
- OpenAI, "Learning to Reason with LLMs" (o1 technical report)
- DeepSeek, "DeepSeek-R1" paper
