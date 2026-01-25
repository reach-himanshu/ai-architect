# DSA 05: Queues and Buffers

A Queue follows the **FIFO** (First-In-First-Out) principle. Like a line at a ticket counter, the first person to join is the first to be served.

## 1. Core Operations
*   **Enqueue**: Add to rear ($O(1)$).
*   **Dequeue**: Remove from front ($O(1)$).

## 2. Circular Buffers (Ring Buffers)
A fixed-size buffer that wraps around. Useful for streaming data or resource-constrained environments (e.g., keyboard input buffer).

*   **Pointers**: Uses `head` (read) and `tail` (write) indices modulo the size.

## 3. Implementation
*   **`collections.deque`**: Double-ended queue, optimized for appending/popping from both ends.
*   **`list`**: inefficient for dequeue from front ($O(n)$) because all elements shift.
