# Python: Async/Await & Asyncio

Asyncio is Python's built-in library for writing **concurrent** code using the `async`/`await` syntax. It's essential for I/O-bound applications like web servers, APIs, and web scraping.

## 1. Why Async?
- **I/O-Bound Tasks**: Waiting for network, files, or databases.
- **Concurrency**: Handle thousands of connections without threads.
- **Efficiency**: Single thread, no GIL contention.

## 2. Basic Syntax

```python
import asyncio

async def fetch_data():
    print("Fetching...")
    await asyncio.sleep(2)  # Non-blocking wait
    print("Done!")
    return "Data"

asyncio.run(fetch_data())
```

## 3. Key Concepts
- **`async def`**: Defines a coroutine.
- **`await`**: Pauses execution until the awaited coroutine finishes.
- **Event Loop**: The scheduler that runs coroutines.

## 4. Running Multiple Tasks Concurrently

```python
async def task(name, delay):
    await asyncio.sleep(delay)
    return f"{name} finished"

async def main():
    results = await asyncio.gather(
        task("A", 2),
        task("B", 1),
        task("C", 3)
    )
    print(results)  # All finish in ~3s, not 6s

asyncio.run(main())
```

## 5. When NOT to Use Async
- **CPU-Bound Tasks**: Use `multiprocessing` instead.
- **Simple Scripts**: Async adds complexity.
