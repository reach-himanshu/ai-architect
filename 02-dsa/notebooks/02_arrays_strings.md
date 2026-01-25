# DSA 02: Arrays and Strings

Arrays and Strings are the most fundamental data structures, storing elements in contiguous memory locations.

## 1. Arrays (Lists in Python)
*   **Access**: $O(1)$ - Instant access by index.
*   **Search**: $O(n)$ - Linear search (unless sorted).
*   **Insertion/Deletion**: $O(n)$ - Requires shifting elements.

## 2. Strings
Strings in Python are **Immutable**. Modifying a string creates a new one, which costs $O(n)$.

### Common Operations
*   **Slicing**: `text[start:end]` - Very efficient in Python.
*   **Concatenation**: `s1 + s2` - $O(n + m)$.
*   **Join**: `''.join(list)` - More efficient than loop concatenation.

## 3. Two Pointers Technique
A common pattern for array problems. Using two pointers (e.g., typically one at the start, one at the end) to process the collection in $O(n)$ time.

```python
# Reversing an array in-place
def reverse_array(arr):
    left, right = 0, len(arr) - 1
    while left < right:
        arr[left], arr[right] = arr[right], arr[left]
        left += 1
        right -= 1
```
