# DSA: Union-Find (Disjoint Set)

Union-Find is a data structure for tracking a set of elements partitioned into disjoint (non-overlapping) subsets. It supports "union" and "find" operations efficiently.

## 1. Operations
- **Find(x)**: Returns the representative (root) of x's set.
- **Union(x, y)**: Merges the sets containing x and y.

## 2. Optimizations
- **Path Compression**: Flatten the tree during Find.
- **Union by Rank**: Attach smaller tree under root of larger tree.

With both optimizations, operations are nearly $O(1)$ (amortized).

## 3. Use Cases
- **Graph Connectivity**: Check if two nodes are connected.
- **Kruskal's MST**: Efficiently detect cycles.
- **Image Processing**: Connected components.
