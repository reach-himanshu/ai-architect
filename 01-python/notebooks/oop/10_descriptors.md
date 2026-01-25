# Python OOP: Descriptors

Descriptors are the mechanism behind `@property`, `@classmethod`, and `@staticmethod`. They let you customize attribute access.

## 1. The Descriptor Protocol
An object is a descriptor if it defines any of:
- `__get__(self, obj, type=None)`: Called when attribute is accessed.
- `__set__(self, obj, value)`: Called when attribute is assigned.
- `__delete__(self, obj)`: Called when attribute is deleted.

## 2. Simple Example: Validation

```python
class PositiveNumber:
    def __set_name__(self, owner, name):
        self.name = name

    def __get__(self, obj, type=None):
        return obj.__dict__.get(self.name, 0)

    def __set__(self, obj, value):
        if value < 0:
            raise ValueError(f"{self.name} must be positive")
        obj.__dict__[self.name] = value

class Product:
    price = PositiveNumber()
    quantity = PositiveNumber()
```

## 3. How `@property` Works
`@property` is a built-in descriptor! It wraps a getter method.

## 4. When to Use Descriptors?
- Reusable validation across multiple classes.
- Building frameworks (like Django ORM fields).
- Lazy loading of expensive attributes.
