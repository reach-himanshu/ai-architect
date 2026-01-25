# Python OOP: Abstract Base Classes (ABC)

Abstract Base Classes define interfaces that subclasses **must** implement. They enforce a contract.

## 1. The `abc` Module

```python
from abc import ABC, abstractmethod

class Shape(ABC):
    @abstractmethod
    def area(self):
        pass

    @abstractmethod
    def perimeter(self):
        pass
```

## 2. Cannot Instantiate Abstract Classes

```python
# This will raise TypeError!
s = Shape()  # TypeError: Can't instantiate abstract class
```

## 3. Subclasses Must Implement All Abstract Methods

```python
class Rectangle(Shape):
    def __init__(self, width, height):
        self.width = width
        self.height = height

    def area(self):
        return self.width * self.height

    def perimeter(self):
        return 2 * (self.width + self.height)
```

## 4. Why Use ABCs?
1.  **Contract Enforcement**: Guarantee that subclasses have specific methods.
2.  **Polymorphism**: Write code that works with any `Shape`.
3.  **Documentation**: Clear interface definition.
