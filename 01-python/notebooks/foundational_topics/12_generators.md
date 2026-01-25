# Python Generators and the `yield` Keyword

Generators are a simple and powerful tool for creating iterators. They are written like regular functions but use the `yield` statement whenever they want to return data.

## 1. What is a Generator?
A generator function returns an **iterator object** which we can iterate over (one value at a time).

### Basic Syntax
```python
def count_up_to(n):
    count = 1
    while count <= n:
        yield count
        count += 1

counter = count_up_to(5)
print(next(counter)) # 1
print(next(counter)) # 2
```

---

## 2. Why Use Generators?

### Memory Efficiency
Unlike lists, which store all their elements in memory, generators produce items **only when requested**. This makes them ideal for working with massive datasets.

```python
# List way (stores 1 million items)
squares_list = [x**2 for x in range(1000000)]

# Generator way (stores logic, not items)
squares_gen = (x**2 for x in range(1000000))
```

### Infinite Sequences
You can create "infinite" loops that don't crash your computer, because the values are generated on the fly.

```python
def infinite_sequence():
    num = 0
    while True:
        yield num
        num += 1
```

---

## 3. The `next()` Function and `StopIteration`
When a generator is exhausted (it runs out of `yield` statements), it raises a `StopIteration` exception. Most of the time, we use a `for` loop, which handles this automatically.

```python
for num in count_up_to(3):
    print(num)
```

---

## 4. Generator Expressions
Just like list comprehensions, but using parentheses `()`.

```python
# Generator expression
my_gen = (x * 2 for x in range(10) if x % 2 == 0)
```

---

## 5. Summary: Generator vs List
| Feature | List | Generator |
| :--- | :--- | :--- |
| **Storage** | Entire collection in memory | One item at a time |
| **Performance** | Fast access by index | Fast to start (no pre-calculation) |
| **Use Case** | Small data, need to re-use | Large data, single pass |
