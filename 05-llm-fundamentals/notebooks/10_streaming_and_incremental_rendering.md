# LLM 10: Streaming & Incremental Rendering

Streaming is the difference between a usable chat product and one users abandon at 4 seconds. This session covers the server-side mechanics, parsing partial outputs, and client-side UX patterns.

## 1. Why Stream

Time-to-first-token (TTFT) is the user-perceived latency. Even a 5-second total response feels fast if the first token arrives in 200 ms. Streaming buys you:

- Perceived performance
- Early abort (cancel expensive generation)
- Progressive UI (show reasoning, partial JSON, step-by-step tool calls)
- Back-pressure opportunities (rate-limit by token)

## 2. Transport

| Transport | Pros | Cons | Used by |
| --- | --- | --- | --- |
| **Server-Sent Events (SSE)** | HTTP-native, simple, one-way | No binary, one-way only | OpenAI, Anthropic, most providers |
| **WebSocket** | Two-way, binary | More complex, firewall-unfriendly | Some realtime/voice APIs |
| **gRPC streams** | Typed, efficient | Not browser-native without proxy | Internal services |
| **Chunked HTTP** | Simplest | No event framing | Fallback |

SSE is the default for LLM APIs in 2026. The client reads a `text/event-stream` response; each event is a line like:

```
data: {"type": "content_block_delta", "delta": {"text": "Hello"}}
```

## 3. What Gets Streamed

LLM APIs emit a typed stream of events, not just token text:

```
message_start         → metadata, usage counters
content_block_start   → content block opened (text, tool_use, thinking, image)
content_block_delta   → incremental content (text delta, JSON delta, etc.)
content_block_stop    → block closed
message_delta         → usage updates
message_stop          → final event
```

Different block types stream differently:

- **Text blocks** emit `text_delta` — just append.
- **Tool-use blocks** emit `input_json_delta` — partial JSON; parse incrementally.
- **Thinking blocks** (reasoning models) emit `thinking_delta` — may or may not be shown to the user.

## 4. Parsing Partial JSON

A 5K-character JSON response is invalid until the closing `}`. You can't wait.

Libraries:

- **`partial-json-parser`** (JS, Python) — returns the best valid subset.
- **`jsonstreamparse`** — event-driven parser.
- **Manual incremental parse** — track bracket depth, parse when it returns to 0.

Pattern for streaming structured output:

```python
buf = ''
for chunk in response:
    buf += chunk
    obj = partial_json_parse(buf)    # returns {} early, grows to full object
    render(obj)                      # UI shows progressively filled fields
```

## 5. Cancellation

Streaming enables real cancellation. If the user clicks away or the result is obviously wrong, you should stop generating and save cost.

```python
with client.messages.stream(...) as stream:
    for text in stream.text_stream:
        if user_cancelled.is_set():
            stream.close()
            break
```

Server-side: your streaming proxy must honor client-disconnect to propagate cancellation to the model provider. Many naive proxies buffer and keep generating — a silent cost leak.

## 6. UX Patterns

### 6a. Token-by-token

The default ChatGPT look. Simple, familiar. Works for chat, Q&A, natural language.

### 6b. Chunked (word- or sentence-level)

Buffer until whitespace or period, then render. Reduces flicker, feels smoother in some typographic contexts.

### 6c. Progressive structured

Stream a form, populating fields as the model emits them. Example:

```
Name:       Alice█
Email:      ⏳
Phone:      ⏳
```

### 6d. Reasoning + Answer

For reasoning models: show "Thinking..." collapsible, then stream the final answer prominently.

### 6e. Tool calls inline

For agents: show each tool invocation as a card in the stream.

```
🔍 Searching for "paris weather tomorrow"...
✓ Found 3 results
💬 Based on the forecast, Paris will be 18°C and clear.
```

## 7. Backend Architecture

A streaming LLM endpoint is different from a typical REST endpoint. Things to get right:

- **Buffering** — disable proxy and server buffering (Nginx: `X-Accel-Buffering: no`, `proxy_buffering off`).
- **Timeouts** — read timeouts must accommodate long streams. Connection timeout stays short.
- **Load balancer** — sticky sessions aren't needed (SSE is stateless after connection) but long-lived connections need healthy LB behavior.
- **Backpressure** — if the client is slow, the server can pause reading from the provider.
- **Retry** — retries with streaming are tricky; usually you restart from scratch.

## 8. Error Handling Mid-Stream

Streams can fail partway through. Patterns:

- **Stream errors as a special event type** — `{"type": "error", "message": "..."}`.
- **Client state machine** — normal → mid-stream → error → recovery.
- **Show partial output** — don't discard; offer to retry or accept.
- **Log the prefix** that was successfully generated before the failure.

## 9. Streaming + Caching

Anthropic prompt caching works with streaming. The cached prefix is counted in the `cache_read_input_tokens` metric of the first stream event.

Pattern: cache the system prompt + static context → stream only the dynamic portion. Big win on cost and TTFT for repeat-heavy prefixes.

## 10. Token-by-Token Cost Accounting

With streaming, you can:

- Start showing output before you know total cost
- Halt early if you hit a per-user budget
- Record usage counters on `message_delta` / `message_stop`

Per-feature token budgets become enforceable, not just observable.

## 11. Architect Checklist

- **Stream everything that takes >1 second**, no exceptions.
- **Disable all intermediate buffering.**
- **Implement cancellation end-to-end**, not just client-visible.
- **Show partial output on error.**
- **Parse partial JSON with a real library.**
- **Cache + stream together** where possible.

## Prereqs / Connections

- `06_decoding_strategies` — streaming emits tokens at sampling time
- `08_structured_outputs` — streaming structured output needs partial parsing
- `14-llmops-deployment/08_caching` — caching + streaming interplay

## Further Reading

- MDN, "Using Server-Sent Events"
- Anthropic, "Streaming Messages" docs
- OpenAI, "Streaming Chat Completions" docs
- Vercel AI SDK docs (client-side streaming patterns)
- `partial-json-parser` library (Python / JS)
