# LLMs 03: LLM APIs

Production systems must call LLMs reliably — with retries, cost tracking, streaming, and structured outputs. This notebook covers the patterns that matter.

## 1. The Major APIs

| Provider | Models | SDK | Key Features |
| :--- | :--- | :--- | :--- |
| OpenAI | GPT-4o, GPT-4o-mini | `openai` | Structured outputs, vision, function calling |
| Anthropic | Claude 3.5 Sonnet/Haiku | `anthropic` | 200K context, system prompts, streaming |
| Google | Gemini 1.5 Pro/Flash | `google-generativeai` | 1M context, multimodal |
| Together/Groq | Llama, Mistral, etc. | `openai` (compatible) | Open models, fast inference |

---

## 2. The Standard Request Pattern

All modern LLM APIs follow a similar structure:

```python
# OpenAI pattern (also works for compatible APIs)
from openai import OpenAI

client = OpenAI(api_key="...")

response = client.chat.completions.create(
    model="gpt-4o",
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "What is RAG?"},
    ],
    temperature=0.7,
    max_tokens=500,
)

text = response.choices[0].message.content
usage = response.usage  # prompt_tokens, completion_tokens, total_tokens
```

```python
# Anthropic pattern
import anthropic

client = anthropic.Anthropic(api_key="...")

response = client.messages.create(
    model="claude-opus-4-6",
    max_tokens=500,
    system="You are a helpful assistant.",
    messages=[{"role": "user", "content": "What is RAG?"}],
)

text = response.content[0].text
usage = response.usage  # input_tokens, output_tokens
```

---

## 3. Retry Logic with Exponential Backoff

Rate limit errors (429) are common. Always wrap API calls with retry logic.

```python
import time
import random
from openai import RateLimitError, APIError

def call_with_retry(func, max_retries=3, base_delay=1.0):
    for attempt in range(max_retries):
        try:
            return func()
        except RateLimitError:
            if attempt == max_retries - 1:
                raise
            delay = base_delay * (2 ** attempt) + random.uniform(0, 0.5)
            print(f"Rate limited. Retrying in {delay:.1f}s...")
            time.sleep(delay)
        except APIError as e:
            if e.status_code >= 500:  # Server errors are retryable
                if attempt == max_retries - 1:
                    raise
                time.sleep(base_delay * (2 ** attempt))
            else:
                raise  # 4xx errors are not retryable
```

---

## 4. Streaming Responses

For UX, stream tokens as they are generated rather than waiting for the full response.

```python
# OpenAI streaming
with client.chat.completions.create(
    model="gpt-4o",
    messages=[{"role": "user", "content": "Explain RAG in 3 sentences."}],
    stream=True,
) as stream:
    for chunk in stream:
        delta = chunk.choices[0].delta.content
        if delta:
            print(delta, end="", flush=True)
```

---

## 5. Cost Tracking

Always track costs in production. Every token costs money.

```python
COST_PER_1M_TOKENS = {
    "gpt-4o": {"input": 2.50, "output": 10.00},
    "gpt-4o-mini": {"input": 0.15, "output": 0.60},
    "claude-opus-4-6": {"input": 15.00, "output": 75.00},
    "claude-sonnet-4-5-20250929": {"input": 3.00, "output": 15.00},
}

def calculate_cost(model: str, input_tokens: int, output_tokens: int) -> float:
    prices = COST_PER_1M_TOKENS.get(model, {"input": 3.0, "output": 15.0})
    return (
        (input_tokens / 1_000_000) * prices["input"] +
        (output_tokens / 1_000_000) * prices["output"]
    )
```

---

## 6. Function Calling / Tool Use

LLMs can decide which tools to call. This is the foundation of agents.

```python
tools = [{
    "type": "function",
    "function": {
        "name": "get_weather",
        "description": "Get current weather for a city.",
        "parameters": {
            "type": "object",
            "properties": {
                "city": {"type": "string", "description": "City name"},
                "units": {"type": "string", "enum": ["celsius", "fahrenheit"]},
            },
            "required": ["city"],
        },
    }
}]

response = client.chat.completions.create(
    model="gpt-4o",
    messages=[{"role": "user", "content": "What's the weather in Tokyo?"}],
    tools=tools,
    tool_choice="auto",
)

# Check if model wants to call a tool
if response.choices[0].finish_reason == "tool_calls":
    tool_call = response.choices[0].message.tool_calls[0]
    print(f"Tool: {tool_call.function.name}")
    print(f"Args: {tool_call.function.arguments}")
```

---

## 7. Structured Output (Pydantic + OpenAI)

```python
from pydantic import BaseModel

class PersonInfo(BaseModel):
    name: str
    age: int
    occupation: str

response = client.beta.chat.completions.parse(
    model="gpt-4o",
    messages=[
        {"role": "user", "content": "Alice Johnson is 34 and works as a software engineer."}
    ],
    response_format=PersonInfo,
)

person = response.choices[0].message.parsed
print(person.name)  # Alice Johnson
print(person.age)   # 34
```
