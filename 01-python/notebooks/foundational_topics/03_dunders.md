# Python Dunder (Magic) Methods and Attributes

"Dunder" stands for **Double UNDERscore**. These are special methods and attributes that Python uses to implement core language features like iteration, operator overloading, and metadata management.

## 1. Dunder Methods (Behavior)

Dunder methods allow your custom objects to behave like built-in Python types. They are usually triggered by syntax rather than direct calls.

### Core Lifecycle
| Method | Trigger | Purpose |
| :--- | :--- | :--- |
| `__init__` | `obj = MyClass()` | The constructor; initializes a new instance. |
| `__new__` | `obj = MyClass()` | The method that actually creates the instance (rarely overridden). |
| `__del__` | `del obj` | The destructor; called when an object is about to be destroyed. |

### Representations
| Method | Trigger | Purpose |
| :--- | :--- | :--- |
| `__str__` | `str(obj)`, `print(obj)` | User-friendly string representation. |
| `__repr__` | `repr(obj)` | Official string representation (for debugging). |
| `__format__` | `f"{obj:spec}"` | Custom formatting logic. |

### Operators
| Method | Trigger | Operator |
| :--- | :--- | :--- |
| `__add__` | `a + b` | Addition |
| `__sub__` | `a - b` | Subtraction |
| `__mul__` | `a * b` | Multiplication |
| `__eq__` | `a == b` | Equality |
| `__lt__` | `a < b` | Less than |

### Containers & Iteration
| Method | Trigger | Purpose |
| :--- | :--- | :--- |
| `__len__` | `len(obj)` | Returns size. |
| `__getitem__` | `obj[key]` | Access by index/key. |
| `__iter__` | `for x in obj` | Returns an iterator. |
| `__contains__` | `item in obj` | Implements `in` keyword. |

---

## 2. Dunder Attributes (Metadata)

Attributes store internal state or documentation about modules, classes, and functions.

| Attribute | Purpose |
| :--- | :--- |
| `__name__` | Name of the current namespace (e.g., `"__main__"` or the module name). |
| `__doc__` | The docstring defined at the top of the object. |
| `__file__` | Absolute path to the source file. |
| `__annotations__`| Dictionary containing type hints for parameters and return values. |
| `__dict__` | Dictionary containing the object's writable attributes. |
| `__all__` | List of public names exported by a module. |

## 3. Practical Example

```python
class Vector:
    """A simple 2D Vector class."""
    
    def __init__(self, x: int, y: int):
        self.x = x
        self.y = y

    def __str__(self):
        return f"Vector({self.x}, {self.y})"

    def __add__(self, other):
        return Vector(self.x + other.x, self.y + other.y)

    def __len__(self):
        return 2

# Usage
v1 = Vector(1, 2)
v2 = Vector(3, 4)

print(v1)              # Vector(1, 2) -> triggered by __str__
print(v1 + v2)         # Vector(4, 6) -> triggered by __add__
print(Vector.__doc__)  # "A simple 2D Vector class." -> __doc__
print(v1.__dict__)     # {'x': 1, 'y': 2} -> __dict__
```
