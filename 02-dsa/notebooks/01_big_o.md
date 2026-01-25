# Big O Notation and Complexity Analysis

Big O notation is used to describe the efficiency of an algorithm in terms of **Time Complexity** (how fast it runs) and **Space Complexity** (how much memory it uses) as the input size `n` grows.

## 1. Common Time Complexities

| Big O | Name | Description | Example |
| :--- | :--- | :--- | :--- |
| **O(1)** | Constant | Time stays the same regardless of `n`. | Accessing array element by index. |
| **O(log n)** | Logarithmic | Time increases slowly; input is halved each step. | Binary Search. |
| **O(n)** | Linear | Time increases proportional to `n`. | Iterating through a list once. |
| **O(n log n)** | Linearithmic | Very common in efficient sorts. | Merge Sort, Quick Sort. |
| **O(n²)** | Quadratic | Nested loops. | Bubble Sort, Nested for-loops. |
| **O(2ⁿ)** | Exponential | Time doubles with each extra input. | Recursive Fibonacci. |

---

## 2. Time Complexity Example (Python)

```python
# O(1) - Constant Time
def get_first_item(items):
    return items[0]

# O(n) - Linear Time
def print_all_items(items):
    for item in items:
        print(item)

# O(n^2) - Quadratic Time
def print_all_pairs(items):
    for item1 in items:
        for item2 in items:
            print(item1, item2)
```

---

## 3. Space Complexity
Space complexity refers to the total amount of memory that an algorithm takes relative to the input size.

```python
# O(1) Space - Only storing a single variable
def sum_list(items):
    total = 0
    for i in items:
        total += i
    return total

# O(n) Space - Creating a new list of size n
def double_list(items):
    new_list = []
    for i in items:
        new_list.append(i * 2)
    return new_list
```

---

## 4. Why Does It Matter?
In small-scale applications, the difference between $O(n)$ and $O(n^2)$ might be milliseconds. However, as `n` grows to millions or billions:
*   $O(n)$ might take a few seconds.
*   $O(n^2)$ could take **days**.

---

## 5. Summary
- Always look for nested loops (likely $O(n^2)$).
- If the problem space is halved at each step, think $O(log n)$.
- Big O describes the **Worst Case Scenario**.
