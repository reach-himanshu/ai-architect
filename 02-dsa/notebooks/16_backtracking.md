# DSA: Backtracking

Backtracking is an algorithmic technique for solving problems recursively by trying to build a solution incrementally, and abandoning ("backtracking") a path as soon as it fails.

## 1. The Pattern
1.  **Choose**: Pick an option.
2.  **Explore**: Recursively explore with that choice.
3.  **Unchoose**: Undo the choice (backtrack) and try the next option.

## 2. Template

```python
def backtrack(path, choices):
    if is_solution(path):
        record_solution(path)
        return

    for choice in choices:
        if is_valid(choice, path):
            path.append(choice)       # Choose
            backtrack(path, choices)  # Explore
            path.pop()                # Unchoose
```

## 3. Classic Problems
- **N-Queens**: Place N queens on an NxN board with no attacks.
- **Sudoku Solver**: Fill grid with valid numbers.
- **Permutations**: Generate all orderings.
- **Subsets / Combinations**: Generate all subsets.
