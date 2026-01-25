# DSA 08: Trees

A Tree is a hierarchical structure with a root node and children.

## 1. Key Terminology
*   **Root**: Top node.
*   **Parent/Child**: Direct relationship.
*   **Leaf**: Node with no children.
*   **Height**: Max depth from root to leaf.

## 2. Binary Tree
Each node has at most two children: `left` and `right`.

## 3. Binary Search Tree (BST)
A binary tree with a special property:
*   **Left Child** < Parent
*   **Right Child** > Parent
*   This allows $O(log n)$ search, insert, and delete (if balanced).

## 4. Traversals
*   **In-Order**: Left -> Root -> Right (Yields sorted values in BST).
*   **Pre-Order**: Root -> Left -> Right.
*   **Post-Order**: Left -> Right -> Root.
