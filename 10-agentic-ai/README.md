# Agentic AI Module

LLMs that use tools, plan, remember, and collaborate. Covers single-agent loops, multi-agent systems, MCP, and the Claude Agent SDK.

## 🚀 Roadmap

### 1. Agent Fundamentals
- The agent loop
- ReAct pattern
- When an "agent" is overkill

### 2. Tool Use Basics
- Schema design for tools
- Parallel tool calls
- Error handling and retries

### 3. Function Calling in Practice
- Claude tool use API
- OpenAI function calling
- Structured tool results

### 4. Planning Patterns
- Plan-and-execute
- Tree-of-thought
- Self-consistency

### 5. Memory Systems
- Short-term (context window)
- Long-term (vector store backed)
- Episodic memory

### 6. Reflection & Self-Critique
- Self-refine loop
- Critic models
- When reflection hurts

### 7. Multi-Agent Orchestration
- Supervisor pattern
- Router pattern
- Swarm / handoff pattern

### 8. Framework Survey
- LangGraph
- AutoGen
- CrewAI
- Claude Agent SDK

### 9. MCP Fundamentals
- Protocol overview
- Transport types (stdio, SSE, HTTP)
- When to build an MCP server

### 10. Building MCP Servers & Tools
- Server skeleton
- Tool registration
- Resources and prompts

### 11. Claude Agent SDK Deep Dive
- Session lifecycle
- Hooks and permissions
- Sub-agents

### 12. Sub-Agents & Delegation
- Delegation patterns
- Context passing
- Cost control

### 13. Safety, Sandboxing, Permissions
- Permission modes
- Sandboxing execution
- Deep dive in `12-ai-safety-governance/`

### 14. Agent Evaluation (Intro)
- Trajectory eval
- Tool-use correctness
- Deep dive in `11-evaluation/`

### 15. Observability & Tracing
- LangSmith, Langfuse
- OpenTelemetry for agents
- Cost and latency tracing
