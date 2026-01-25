# Python OOP: Composition vs. Inheritance

A common design challenge in software engineering is deciding whether to use **Inheritance** ("Is-a" relationship) or **Composition** ("Has-a" relationship).

## 1. Inheritance ("Is-a")
Inheritance is used when a subclass is a specific version of a parent class.

```python
class Engine:
    def start(self): return "Engine started"

class Car(Engine): # Car IS AN Engine? No, this is bad design.
    pass
```

## 2. Composition ("Has-a")
Composition is used when a class "contains" or "uses" another class to perform its duties. This is generally preferred in modern software design (Design Principle: *"Favor composition over inheritance"*).

```python
class Engine:
    def start(self): return "Vroom!"

class Car:
    def __init__(self):
        self.engine = Engine() # Car HAS AN Engine

    def start(self):
        return self.engine.start()

my_car = Car()
print(my_car.start())
```

---

## 3. Why Composition is Often Better
1.  **Flexibility**: You can change the behavior at runtime (e.g., swapping a `GasEngine` for an `ElectricEngine`).
2.  **Encapsulation**: Internals of the "component" class are hidden from the "container" class.
3.  **Low Coupling**: Changes in the parent class won't accidentally break the child's implementation.

---

## 4. Comparison Table
| Aspect | Inheritance | Composition |
| :--- | :--- | :--- |
| **Relationship** | "Is-a" | "Has-a" |
| **Visibility** | Child sees parent internals | Container sees component public API |
| **Flexibility** | Static (Fixed at compile time) | Dynamic (Can change at runtime) |
| **Complexity** | Can lead to deep, messy trees | Flat and manageable |

---

## 5. Summary
Use **Inheritance** only when the relationship is a strict hierarchy. Use **Composition** when you want to build complex behaviors by combining simple, reusable components.
