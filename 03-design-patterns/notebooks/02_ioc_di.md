# Design Patterns 02: IoC & Dependency Injection

Inversion of Control (IoC) and Dependency Injection (DI) are techniques to remove dependencies from your code, making it modular and testable.

## 1. The Problem: Tight Coupling
When Class A creates an instance of Class B inside itself, it is **tightly coupled** to Class B.
*   You cannot easily swap Class B for a mock version during testing.
*   You cannot easily replace Class B with Class C (a better implementation) without changing Class A.

## 2. Dependency Injection (DI)
Instead of creating dependencies, they are **injected** (passed in).

### Constructor Injection
Passing dependencies via `__init__`. This is the most common and robust form.

```python
# TIGHT COUPLING (BAD)
class Service:
    def __init__(self):
        self.logger = FileLogger() # Hardcoded dependency!

# DEPENDENCY INJECTION (GOOD)
class Service:
    def __init__(self, logger):
        self.logger = logger # Injected!
```

## 3. Benefits
1.  **Testability**: You can inject a `MockLogger` implementation that does nothing (or records calls) instead of a real `FileLogger`.
2.  **Flexibility**: You can swap `DatabaseRepository` for `InMemoryRepository` locally.
