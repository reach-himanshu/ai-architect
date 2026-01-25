# Python OOP: Object Lifecycle & Slots

Understanding how objects are created, used, and destroyed.

## 1. `__new__` vs `__init__`
- **`__new__`**: Creates the instance (allocates memory). Rarely overridden.
- **`__init__`**: Initializes the instance (sets attributes).

```python
class MyClass:
    def __new__(cls, *args, **kwargs):
        print("Creating instance")
        instance = super().__new__(cls)
        return instance

    def __init__(self, value):
        print("Initializing instance")
        self.value = value
```

## 2. `__del__` (Destructor)
Called when an object is garbage collected. **Avoid relying on it** for cleanup—use context managers instead.

## 3. `__slots__` (Memory Optimization)
By default, Python stores attributes in a `__dict__`. Using `__slots__` avoids this, saving memory.

```python
class Point:
    __slots__ = ['x', 'y']  # Only these attributes allowed

    def __init__(self, x, y):
        self.x = x
        self.y = y
```

## 4. Benefits of `__slots__`
- **Memory Savings**: Significant for millions of objects.
- **Faster Attribute Access**: Slightly faster.
- **Prevents Typos**: Can't add random attributes.

## 5. When NOT to Use `__slots__`
- When you need dynamic attributes.
- With multiple inheritance (complex).
