# Python OOP: Advanced Concepts

This section covers special decorators that change how methods behave and how we access data.

## 1. Class Methods (`@classmethod`)
A class method belongs to the class itself, not a specific instance. It receives `cls` as its first argument.
- **Use Case**: Creating "Factory Methods" (alternative ways to create objects).

```python
class Date:
    def __init__(self, day, month, year):
        self.day, self.month, self.year = day, month, year

    @classmethod
    def from_string(cls, date_str):
        day, month, year = map(int, date_str.split("-"))
        return cls(day, month, year) # Create a new instance

d = Date.from_string("24-01-2026")
```

---

## 2. Static Methods (`@staticmethod`)
A static method doesn't need access to the class (`cls`) or the instance (`self`). It behaves like a plain function that is grouped inside a class for organization.

```python
class MathUtils:
    @staticmethod
    def is_even(n):
        return n % 2 == 0

print(MathUtils.is_even(10)) # True
```

---

## 3. Properties (`@property`)
The `@property` decorator allows you to define methods that can be accessed like attributes. This is the Pythonic way to implement **Getters and Setters**.

```python
class Employee:
    def __init__(self, name):
        self._name = name

    @property
    def name(self): # Getter
        return self._name.upper()

    @name.setter
    def name(self, value): # Setter
        if not value: raise ValueError("Name cannot be empty")
        self._name = value

emp = Employee("Alice")
print(emp.name) # ALICE (accessed as attribute, calls method)
emp.name = "Bob"
```

---

## 4. Summary Table
| Method Type | Argument | Purpose |
| :--- | :--- | :--- |
| **Instance** | `self` | Modify object state. |
| **Class** | `cls` | Modify class state / Factory methods. |
| **Static** | None | Utility functions for the class. |
| **Property** | `self` | Computed attributes / Validation. |
