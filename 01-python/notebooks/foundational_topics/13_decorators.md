# Python: Decorators Deep Dive

Decorators are one of Python's most powerful features. They allow you to modify or enhance functions without changing their source code.

## 1. Closures (The Foundation)
A closure is a function that remembers values from its enclosing scope even after that scope has finished executing.

```python
def outer(msg):
    def inner():
        print(msg)  # msg is "closed over"
    return inner

greet = outer("Hello")
greet()  # Prints "Hello"
```

## 2. Basic Decorator Pattern
A decorator is a function that takes a function and returns a new function.

```python
def my_decorator(func):
    def wrapper(*args, **kwargs):
        print("Before")
        result = func(*args, **kwargs)
        print("After")
        return result
    return wrapper

@my_decorator
def say_hello():
    print("Hello!")
```

## 3. Preserving Metadata with `functools.wraps`
Without `@wraps`, the decorated function loses its `__name__`, `__doc__`, etc.

```python
from functools import wraps

def my_decorator(func):
    @wraps(func)  # CRITICAL for production code
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper
```

## 4. Decorator Factories (Decorators with Arguments)
A decorator factory is a function that **returns** a decorator.

```python
def repeat(n):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for _ in range(n):
                result = func(*args, **kwargs)
            return result
        return wrapper
    return decorator

@repeat(3)
def greet():
    print("Hi!")
```

## 5. Class-Based Decorators
Using `__call__` to make a class act as a decorator.

```python
class CountCalls:
    def __init__(self, func):
        self.func = func
        self.calls = 0

    def __call__(self, *args, **kwargs):
        self.calls += 1
        print(f"Call {self.calls}")
        return self.func(*args, **kwargs)

@CountCalls
def say():
    print("Hello")
```

## 6. Common Use Cases
- **Logging**: `@log_calls`
- **Timing**: `@timeit`
- **Caching**: `@functools.lru_cache`
- **Authentication**: `@login_required` (Flask/Django)
- **Retry Logic**: `@retry(max_attempts=3)`
