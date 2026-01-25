# Design Patterns 16: Streaming vs Batching

How to handle data that doesn't fit in RAM.

## 1. Batch Processing
Load all data into memory, process it, write it out.
*   **Risk**: `MemoryError` if dataset grows.

## 2. Streaming (Generators)
Process data one item (or chunk) at a time.
*   **Python**: `yield` keyword makes this trivial.
*   **Pattern**: Pipes and Filters.
