# Python Comprehensions

Comprehensions provide a concise way to create new sequences (lists, sets, dictionaries) from existing ones. They are often more readable and faster than traditional loops.

## 1. List Comprehensions
List comprehensions are the most common. They follow a specific syntax:
`[expression for item in iterable if condition]`

### Example: Squaring Numbers
```python
# Traditional Way
squares = []
for x in range(10):
    squares.append(x**2)

# Comprehension Way
squares = [x**2 for x in range(10)]
```

### Example: Filtering with `if`
```python
# Only even squares
even_squares = [x**2 for x in range(10) if x % 2 == 0]
```

## 2. Dictionary Comprehensions
Similar to list comprehensions, but they create a dictionary using key-value pairs.
`{key: value for item in iterable}`

```python
# Creating a dictionary of squares
square_dict = {x: x**2 for x in range(5)}
# Output: {0: 0, 1: 1, 2: 4, 3: 9, 4: 16}
```

## 3. Set Comprehensions
Sets automatically handle unique values and are created using curly braces `{}`.

```python
# Unique initials from a list of names
names = ["Alice", "Bob", "Alice", "Charlie"]
initials = {name[0] for name in names}
# Output: {'A', 'B', 'C'}
```

## 4. Nested Comprehensions
You can even nest comprehensions, though use them sparingly for the sake of readability.

```python
# Creating a 3x3 matrix
matrix = [[j for j in range(3)] for i in range(3)]
# Output: [[0, 1, 2], [0, 1, 2], [0, 1, 2]]
```

## 5. Summary Table
| Type | Syntax | Result Type |
| :--- | :--- | :--- |
| **List** | `[x for x in seq]` | `list` |
| **Set** | `{x for x in seq}` | `set` |
| **Dict** | `{k:v for k,v in seq}` | `dict` |
