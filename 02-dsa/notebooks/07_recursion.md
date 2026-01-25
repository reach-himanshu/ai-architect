# DSA 07: Recursion

Recursion is a method of solving problems where a function calls itself to solve smaller instances of the same problem.

## 1. The Three Laws of Recursion
1.  **A Base Case**: A condition that stops the recursion.
2.  **Move toward the Base Case**: Each step must reduce the problem size.
3.  **Call Itself**: The function must call itself.

## 2. The Call Stack
Every time a function is called, a frame is added to the "Call Stack". If recursion goes too deep without hitting a base case, you get a **Stack Overflow**.

## 3. Complexity
*   **Time**: $O(b^d)$ where $b$ is branching factor and $d$ is depth.
*   **Space**: $O(d)$ for the call stack.
