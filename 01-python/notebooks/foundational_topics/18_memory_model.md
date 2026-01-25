# Python: Memory Model & Internals

Understanding Python's memory model prevents subtle bugs related to mutability, references, and copying.

## 1. Variables are References
In Python, variables are **names** that point to objects in memory. They don't "contain" the value.

```python
a = [1, 2, 3]
b = a       # b points to the SAME list
b.append(4)
print(a)    # [1, 2, 3, 4] - a is also modified!
```

## 2. `is` vs `==`
- `==`: Compares **values** (content).
- `is`: Compares **identity** (same object in memory).

```python
a = [1, 2]
b = [1, 2]
print(a == b)  # True (same value)
print(a is b)  # False (different objects)
```

## 3. Shallow vs Deep Copy

```python
import copy

original = [[1, 2], [3, 4]]
shallow = copy.copy(original)    # Inner lists are still shared
deep = copy.deepcopy(original)   # Fully independent
```

## 4. Mutable Default Arguments (Pitfall!)

```python
# BAD!
def add_item(item, lst=[]):
    lst.append(item)
    return lst

add_item(1)  # [1]
add_item(2)  # [1, 2] - Unexpected!

# GOOD
def add_item(item, lst=None):
    if lst is None:
        lst = []
    lst.append(item)
    return lst
```

## 5. The GIL (Global Interpreter Lock)
- CPython has a GIL that prevents true parallelism in threads.
- **I/O-bound**: Threads still help (GIL released during I/O).
- **CPU-bound**: Use `multiprocessing` for parallelism.
