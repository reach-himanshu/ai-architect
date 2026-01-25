# Python OOP: Encapsulation and Abstraction

These two concepts are about controlling access to data and hiding complexity.

## 1. Access Modifiers (Encapsulation)
Encapsulation is about bundling data and restricting access. Python uses naming conventions to define the visibility of members.

### Visibility Levels
| Type | Convention | Access Level | Description |
| :--- | :--- | :--- | :--- |
| **Public** | `name` | **Everywhere** | Accessible from inside and outside the class. |
| **Protected**| `_name` | **Internal/Subclass** | A hint that it's for internal use; accessible but discouraged. |
| **Private** | `__name` | **Class Only** | Name-mangled; difficult to access from outside the class. |

### Example
```python
class BankAccount:
    def __init__(self, owner, balance):
        self.owner = owner          # Public
        self._account_type = "Gold" # Protected
        self.__balance = balance    # Private

    def get_balance(self):
        return self.__balance # Accessing private member internally

account = BankAccount("Alice", 1000)
print(account.owner)          # OK
print(account._account_type)  # OK, but discouraged
# print(account.__balance)    # Raises AttributeError
```

---

## 2. Abstraction
Abstraction is about hiding the implementation details and only showing the essential features of the object. We use **Abstract Base Classes (ABC)** to define blueprints that *must* be implemented by subclasses.

### Creating an Abstract Class
```python
from abc import ABC, abstractmethod

class Shape(ABC):
    @abstractmethod
    def area(self):
        pass

class Square(Shape):
    def __init__(self, side):
        self.side = side
    
    def area(self): # Must implement this
        return self.side * self.side

# s = Shape() # This would raise an error
```

---

## 3. Summary
- **Encapsulation**: "Protecting data." Keeps the state of an object safe from outside interference.
- **Abstraction**: "Hiding complexity." Focuses on what an object does rather than how it does it.
