# Python OOP: Multiple Inheritance and MRO

Multiple inheritance allows a class to inherit from more than one parent class. While powerful, it can lead to complexity, particularly regarding the order in which methods are resolved.

## 1. Multiple Inheritance
A class can have multiple base classes by listing them in the class definition.

```python
class Flying:
    def move(self):
        print("Flying through the air!")

class Swimming:
    def move(self):
        print("Swimming through the water!")

class Duck(Flying, Swimming):
    pass

d = Duck()
d.move() # Which move() will be called?
```

---

## 2. MRO (Method Resolution Order)
MRO is the search order Python follows to look for a method or attribute in a class hierarchy. Python uses the **C3 Linearization** algorithm to determine this.

### How to check MRO
You can use the `.mro()` method or the `__mro__` attribute.

```python
print(Duck.mro())
# [<class '__main__.Duck'>, <class '__main__.Flying'>, <class '__main__.Swimming'>, <class 'object'>]
```

In the example above, `Flying` comes before `Swimming` because it was listed first in the class definition (`class Duck(Flying, Swimming)`).

---

## 3. The Diamond Problem
The "Diamond Problem" occurs when two subclasses of a single class are both parents to another subclass.

```text
    A
   / \
  B   C
   \ /
    D
```

Python handles this gracefully using MRO, ensuring that each class is visited only once and strictly follows the precedence set by the class definition.

---

## 4. Using `super()` with Multiple Inheritance
In multiple inheritance, `super()` does not just call the "parent." It calls the **next class in the MRO**.

```python
class Base:
    def __init__(self):
        print("Base init")

class A(Base):
    def __init__(self):
        print("A init")
        super().__init__()

class B(Base):
    def __init__(self):
        print("B init")
        super().__init__()

class C(A, B):
    def __init__(self):
        print("C init")
        super().__init__()

c = C() 
# Order: C -> A -> B -> Base
```

---

## 5. Summary
- **Precedence**: Classes listed first in the inheritance tuple have higher priority.
- **MRO**: Always check the MRO if you're unsure which method will execution.
- **Design Tip**: Use multiple inheritance sparingly. Often, **Composition** or **Mixins** are cleaner alternatives.
