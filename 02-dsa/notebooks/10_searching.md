# DSA 10: Searching Algorithms

Searching is the process of finding a target element within a collection.

## 1. Linear Search
Iterate through every element until the target is found.
*   **Time**: $O(n)$
*   **Precondition**: None (works on unsorted data).

## 2. Binary Search
Divide the search space in half at each step.
*   **Time**: $O(log n)$ - Much faster.
*   **Precondition**: Data **MUST** be sorted.

## 3. Algorithm
1.  Set `low` and `high` pointers.
2.  Calculate `mid`.
3.  If `arr[mid] == target`, return `mid`.
4.  If `target < arr[mid]`, search left half (`high = mid - 1`).
5.  If `target > arr[mid]`, search right half (`low = mid + 1`).
