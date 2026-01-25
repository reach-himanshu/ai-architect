# DSA 09: Graphs

A Graph consists of a set of vertices (nodes) and edges (connections). They model relationships like social networks, maps, or internet links.

## 1. Representations
*   **Adjacency List**: A dictionary where keys are nodes and values are lists of neighbors. Space efficient $O(V+E)$.
*   **Adjacency Matrix**: A 2D array. fast lookups $O(1)$ but uses $O(V^2)$ space.

## 2. Traversals
*   **BFS (Breadth-First Search)**: Explores level by level using a **Queue**. Used for shortest path.
*   **DFS (Depth-First Search)**: Explores deep down a branch using a **Stack** (or recursion). Used for maze solving, topological sort.
