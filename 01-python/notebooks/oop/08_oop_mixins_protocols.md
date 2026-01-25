# Python OOP: Mixins and Protocols

Mixins and Protocols are advanced patterns for adding functionality to classes without using strict, deep inheritance hierarchies.

## 1. Mixins
A Mixin is a small class that provides specific methods for other classes to use. It's not meant to stand on its own.

### Example: A JSON Serialization Mixin
```python
import json

class JSONMixin:
    def to_json(self):
        return json.dumps(self.__dict__)

class User(JSONMixin):
    def __init__(self, name, age):
        self.name = name
        self.age = age

u = User("Alice", 30)
print(u.to_json()) # Mixin provides this functionality
```

---

## 2. Protocols (Structural Typing)
Introduced in Python 3.8, **Protocols** allow for "Static Duck Typing." Instead of saying a class *must* inherit from a parent, you say it *must* have certain methods.

### Why use Protocols?
It allows for type checking (at development time) without the overhead of `abc` modules.

```python
from typing import Protocol

class Drawable(Protocol):
    def draw(self) -> None:
        ...

class Circle: # No inheritance needed!
    def draw(self):
        print("Drawing a circle")

def render(obj: Drawable):
    obj.draw()

render(Circle()) # Mypy/IDE will confirm this is valid
```

---

## 3. Summary
- **Mixins**: Used to "mix in" common behaviors across unrelated classes.
- **Protocols**: Used to define requirements (interfaces) that other classes must meet, promoting a decoupled design.
