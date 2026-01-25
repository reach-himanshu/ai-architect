# Python: Type Hints

Type hints (introduced in Python 3.5+) provide static type information that improves code readability and enables tooling like `mypy` to catch bugs before runtime.

## 1. Basic Syntax

```python
def greet(name: str) -> str:
    return f"Hello, {name}"

age: int = 25
```

## 2. Common Types from `typing`

```python
from typing import List, Dict, Tuple, Optional, Union, Callable

def process(items: List[int]) -> Dict[str, int]:
    return {"count": len(items)}

def maybe_value(x: Optional[int]) -> int:  # int or None
    return x if x else 0

def flexible(x: Union[int, str]) -> str:
    return str(x)
```

## 3. Callable Types
For functions as arguments.

```python
from typing import Callable

def apply(func: Callable[[int, int], int], a: int, b: int) -> int:
    return func(a, b)
```

## 4. Generics (Python 3.9+)
Use built-in types directly:

```python
def first(items: list[int]) -> int:
    return items[0]

config: dict[str, str] = {"key": "value"}
```

## 5. TypeVar (Generic Functions)

```python
from typing import TypeVar

T = TypeVar('T')

def first(items: list[T]) -> T:
    return items[0]
```

## 6. Why Use Type Hints?
1.  **Readability**: Self-documenting code.
2.  **IDE Support**: Better autocomplete and error detection.
3.  **Static Analysis**: `mypy` catches type errors before runtime.
4.  **Refactoring**: Safer large-scale changes.
