# DSA 06: Hash Tables (Dictionaries)

Hash Tables provide fast insertion, deletion, and lookup of key-value pairs. In Python, these are implemented as **Dictionaries** (`dict`) and **Sets** (`set`).

## 1. How it works
*   **Hash Function**: Converts a key (string/int) into an index.
*   **Collision**: When two keys hash to the same index.
    *   **Chaining**: Storing a list at that index.
    *   **Open Addressing**: Looking for the next open slot.

## 2. Complexity
*   **Average Case**: $O(1)$ for insert/delete/lookup.
*   **Worst Case**: $O(n)$ if many collisions occur (rare with good hash functions).

## 3. Common Interview Patterns
*   **Counting Frequencies**: Using a hash map to count occurrences.
*   **Two Sum**: Finding a complement value in $O(1)$ time.
