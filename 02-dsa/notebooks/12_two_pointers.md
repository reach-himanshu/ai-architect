# DSA 12: Two Pointers & Sliding Window

These patterns are used to optimize array/string problems from $O(n^2)$ (nested loops) to $O(n)$ (single pass).

## 1. Sliding Window
Used for finding subarrays (contiguous sections) that satisfy a condition (max sum, longest substring, etc.).
*   **Static Window**: Window size `k` is fixed. Slide via `current_sum += arr[i] - arr[i-k]`.
*   **Dynamic Window**: Expand `right` pointer to find valid window, shrink `left` pointer to optimize.

## 2. Two Pointers
Used mostly on sorted arrays.
*   Pointers start at ends (`left`, `right`) and move inward.
*   Pointers start at `0` (`slow`, `fast`) for cycle detection or removing duplicates.
