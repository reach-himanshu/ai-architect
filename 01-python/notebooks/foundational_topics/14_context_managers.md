# Python: Context Managers

Context managers provide a clean way to manage resources like files, network connections, or locks—ensuring they are properly acquired and released.

## 1. The `with` Statement
The `with` statement guarantees that cleanup code runs, even if an exception occurs.

```python
# Without context manager (BAD)
f = open("file.txt")
data = f.read()
f.close()  # Might not run if exception occurs!

# With context manager (GOOD)
with open("file.txt") as f:
    data = f.read()
# File is automatically closed here
```

## 2. The Protocol: `__enter__` and `__exit__`
Any object can be a context manager if it implements these two methods.

```python
class MyContext:
    def __enter__(self):
        print("Entering context")
        return self  # Value assigned to `as` variable

    def __exit__(self, exc_type, exc_val, exc_tb):
        print("Exiting context")
        # Return True to suppress exceptions
        return False
```

## 3. The `@contextmanager` Decorator
A simpler way to create context managers using a generator.

```python
from contextlib import contextmanager

@contextmanager
def open_file(name):
    f = open(name)
    try:
        yield f  # Control returns to `with` block
    finally:
        f.close()  # Cleanup guaranteed
```

## 4. Common Use Cases
- **File I/O**: `open()`
- **Database Connections**: Commit/Rollback
- **Locks**: `threading.Lock()` acquisition/release
- **Timing**: Measure code execution time
- **Temporary State**: Change and restore settings
