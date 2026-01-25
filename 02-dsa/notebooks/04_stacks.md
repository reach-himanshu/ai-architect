# DSA 04: Stacks

A Stack is a linear data structure that follows the **LIFO** (Last-In-First-Out) principle. Like a stack of plates, you can only add or remove from the top.

## 1. Core Operations
*   **Push**: Add element to top ($O(1)$).
*   **Pop**: Remove element from top ($O(1)$).
*   **Peek**: View element at top ($O(1)$).

## 2. Implementation in Python
*   **List**: `append()` is push, `pop()` is pop. efficient (amortized $O(1)$).
*   **`collections.deque`**: Slightly faster for large stacks.

## 3. Common Use Cases
1.  **Function Call Stack**: Tracking recursive calls.
2.  **Undo Mechanisms**: Storing history (Ctrl+Z).
3.  **Parsing**: Checking valid parentheses `((()))`.
