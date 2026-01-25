# Python: Dataclasses & Named Tuples

Modern Python provides cleaner ways to create data-holding classes without boilerplate.

## 1. Dataclasses (Python 3.7+)
The `@dataclass` decorator auto-generates `__init__`, `__repr__`, `__eq__`, and more.

```python
from dataclasses import dataclass

@dataclass
class Point:
    x: float
    y: float

p = Point(1.0, 2.0)
print(p)  # Point(x=1.0, y=2.0)
```

## 2. Optional Features

```python
from dataclasses import dataclass, field

@dataclass(frozen=True)  # Immutable
class Config:
    name: str
    values: list = field(default_factory=list)  # Mutable default
```

## 3. Named Tuples
Lightweight, immutable data structures.

```python
from typing import NamedTuple

class Coordinate(NamedTuple):
    x: float
    y: float

c = Coordinate(3.0, 4.0)
print(c.x, c[0])  # Both work
```

## 4. When to Use What?
| Feature | Dataclass | NamedTuple |
|:---|:---|:---|
| Mutability | By default | Immutable |
| Methods | Yes | Yes |
| Inheritance | Yes | Yes |
| Memory | Normal | Smaller |
| Unpacking | No | Yes (`x, y = coord`) |
