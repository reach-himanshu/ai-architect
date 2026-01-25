# DSA 03: Linked Lists

A Linked List is a linear data structure where elements are not stored in contiguous memory. Instead, each element (node) points to the next.

## 1. Singly Linked List
*   **Nodes**: Contain `data` and `next`.
*   **Access**: $O(n)$ - Must traverse from the head.
*   **Insertion/Deletion**: $O(1)$ - If you have the reference to the node *before* the target.

## 2. Doubly Linked List
*   **Nodes**: Contain `data`, `next`, and `prev`.
*   **Pros**: Can traverse backwards. Easier deletion (since you have access to `prev` node).
*   **Cons**: Uses more memory.

## 3. Operations
*   **Append**: Add to end ($O(n)$ if no tail pointer, $O(1)$ with tail pointer).
*   **Prepend**: Add to start ($O(1)$).
*   **Delete**: Remove node ($O(n)$ to find, $O(1)$ to remove).

```python
class Node:
    def __init__(self, data):
        self.data = data
        self.next = None
```
