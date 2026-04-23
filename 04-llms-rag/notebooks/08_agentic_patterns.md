# LLMs 08: Agentic Patterns

Agents extend LLMs from single-turn responders to autonomous systems that plan, act, observe, and iterate.

## 1. What is an Agent?

An agent is an LLM that can:
1.  **Observe** its environment (tools, memory, context).
2.  **Reason** about what to do next.
3.  **Act** by calling tools or other agents.
4.  **Iterate** until the task is complete.

```
┌──────────────────────────────────────┐
│              AGENT LOOP              │
│                                      │
│  Observation → LLM Reasoning         │
│       ↑            ↓                 │
│  Tool Result ← Tool Call             │
│       ↑            ↓                 │
│  Environment    (repeat until done)  │
└──────────────────────────────────────┘
```

---

## 2. ReAct Pattern (Reason + Act)

The most widely used agent pattern. The LLM alternates between reasoning and tool use.

```
Thought: I need to find the population of Tokyo.
Action: search("Tokyo population 2024")
Observation: Tokyo has 13.96 million people in the city proper.

Thought: Now I have the Tokyo population. The user also asked about NYC.
Action: search("New York City population 2024")
Observation: NYC has 8.26 million people.

Thought: I have both numbers. I can answer now.
Final Answer: Tokyo (13.96M) is larger than NYC (8.26M).
```

**How to implement**: Use the model's tool-calling API and loop until `finish_reason == "stop"`.

---

## 3. Tool Design Principles

Good tool design is critical. Tools are the agent's interface with the world.

| Principle | Bad Tool | Good Tool |
| :--- | :--- | :--- |
| **Single responsibility** | `do_everything()` | `search_web()`, `run_code()` |
| **Clear description** | "Search" | "Search the web for current information. Use for facts, news, prices." |
| **Typed parameters** | `query: any` | `query: str, max_results: int = 5` |
| **Returns structured data** | Raw HTML | `{"results": [{"title": ..., "url": ..., "snippet": ...}]}` |
| **Idempotent when possible** | Sends email on every call | Read-only by default, writes require confirmation |
| **Error messages are useful** | `{"error": true}` | `{"error": "API rate limited. Retry in 60s."}` |

---

## 4. Memory Architectures

| Type | Storage | Lifetime | Use Case |
| :--- | :--- | :--- | :--- |
| **In-context** | LLM context window | Current session | Short conversations |
| **External (vector)** | Vector DB | Persistent | Long-term user memory, KB |
| **External (key-value)** | Redis / DB | Persistent | User preferences, facts |
| **Episodic** | Summarized past sessions | Persistent | Agent that remembers past tasks |

---

## 5. Multi-Agent Patterns

| Pattern | Description | Use Case |
| :--- | :--- | :--- |
| **Orchestrator-Worker** | One agent plans, others execute | Complex task decomposition |
| **Parallel Agents** | Multiple agents run simultaneously | Research with multiple queries |
| **Pipeline** | Agent A output → Agent B input | Sequential processing steps |
| **Supervisor** | Parent agent evaluates child agent output | Quality control, retry on failure |
| **Debate** | Multiple agents argue positions | Improving answer quality |

---

## 6. Agent Failure Modes

*   **Infinite loops**: Agent keeps calling tools without making progress. Add iteration limits.
*   **Tool call hallucination**: Model invents tool results. Validate all tool outputs.
*   **Context overflow**: Long agent loops fill the context window. Summarize periodically.
*   **Cascading errors**: One bad tool result leads the agent off course. Add verification steps.
*   **Over-automation**: Agent takes irreversible actions without human confirmation. Use human-in-the-loop checkpoints.

---

## 7. Human-in-the-Loop

For high-stakes actions, require human approval:

```python
def execute_action(action: dict, require_approval: bool = True) -> dict:
    if require_approval and action["type"] in RISKY_ACTIONS:
        print(f"Agent wants to: {action['description']}")
        confirm = input("Approve? (y/n): ")
        if confirm.lower() != "y":
            return {"status": "rejected", "reason": "User declined"}

    return execute(action)
```

Irreversible actions that should always require approval:
- Sending emails/messages
- Making purchases
- Deleting files/records
- Publishing content
