# DSA 14: Dynamic Programming (DP)

Dynamic Programming is a method for solving complex problems by breaking them down into simpler subproblems and storing their solutions to avoid re-computing them.

## 1. Key Properties
1.  **Overlapping Subproblems**: The problem recurses into the exact same sub-problems (e.g., Fibonacci(5) calls Fibonacci(3) multiple times).
2.  **Optimal Substructure**: The optimal solution to the problem can be constructed from optimal solutions of its subproblems.

## 2. Techniques
*   **Memoization (Top-Down)**: Write the recursive solution, but store the result of each function call in a map/array.
*   **Tabulation (Bottom-Up)**: Start from the base case (e.g., `dp[0]` and `dp[1]`) and iteratively fill up a table to reach `dp[n]`.

## 3. Comparison with Recursion
*   Naive Recursion: $O(2^n)$ (Exponential) for Fibonacci.
*   DP: $O(n)$ (Linear).
