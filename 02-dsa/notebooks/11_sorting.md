# DSA 11: Sorting Algorithms

Sorting is critical for optimized searching and data presentation.

## 1. $O(n^2)$ Sorts (Bad)
*   **Bubble Sort**: Repeatedly swaps adjacent elements if they are in wrong order.
*   **Insertion Sort**: Builds the final sorted array one item at a time. Good for small or nearly sorted lists.

## 2. $O(n log n)$ Sorts (Good)
*   **Merge Sort**: A "Divide and Conquer" algorithm. Splits list in half, sorts halves recursively, then merges them. Guaranteed $O(n log n)$.
*   **Quick Sort**: Picks a pivot elements and partitions array around it. Fast average case $O(n log n)$ but worst case $O(n^2)$.

## 3. Python's Timsort
Python's built-in `sort()` uses **Timsort**, a hybrid of Merge Sort and Insertion Sort. It is stable and highly optimized for real-world data.
