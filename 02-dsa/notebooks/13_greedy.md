# DSA 13: Greedy Algorithms

Greedy algorithms make the locally optimal choice at each step with the hope of finding a global optimum. They are often fast and intuitive but don't work for every problem.

## 1. How it works
At every stage, pick the best option right now. Don't worry about the future.

## 2. Example: Coin Change
Given coins `[1, 5, 10, 25]` (US currency), find the minimum number of coins to make `n` cents.
*   **Greedy Strategy**: Always pick the largest coin $\le$ remaining amount.
*   **Caveat**: This strategy fails for some arbitrary coin sets (e.g., `[1, 3, 4]` to make 6. Greedy picks 4+1+1 (3 coins), but optimal is 3+3 (2 coins)).

## 3. Example: Activity Selection
Given a set of activities with start and end times, pick the maximum number of non-overlapping activities.
*   **Strategy**: Sort by *end time* and pick the first one that fits.
