# Python OOP: Inheritance and Polymorphism

Inheritance and Polymorphism are two of the four pillars of Object-Oriented Programming that allow for code reuse and flexibility.

## 1. Inheritance
Inheritance allows a class (Child/Subclass) to inherit attributes and methods from another class (Parent/Superclass).

### Basic Syntax
```python
class Animal:
    def speak(self):
        print("Animal makes a sound")

class Dog(Animal): # Dog inherits from Animal
    def bark(self):
        print("Woof!")

my_dog = Dog()
my_dog.speak() # Inherited method
my_dog.bark()  # Own method
```

### The `super()` Function
The `super()` function is used to call methods from the parent class, most commonly used in `__init__`.

```python
class Animal:
    def __init__(self, name):
        self.name = name

class Dog(Animal):
    def __init__(self, name, breed):
        super().__init__(name) # Call parent __init__
        self.breed = breed
```

---

## 2. Polymorphism
Polymorphism (meaning "many forms") allows different classes to be treated as instances of the same general class through the same interface.

### Method Overriding
A child class can provide a specific implementation of a method that is already defined in its parent class.

```python
class Bird:
    def fly(self):
        print("Most birds can fly")

class Penguin(Bird):
    def fly(self):
        print("Penguins cannot fly, they swim!")

class Eagle(Bird):
    pass

def flight_test(bird):
    bird.fly()

flight_test(Eagle())   # Most birds can fly
flight_test(Penguin()) # Penguins cannot fly, they swim!
```

### Polymorphism with Functions
You can create functions that take an object and call a method on it, regardless of the object's type, as long as it has that method.

```python
class Cat:
    def sound(self): return "Meow"

class Dog:
    def sound(self): return "Woof"

def make_sound(animal):
    print(animal.sound())

make_sound(Cat()) # Meow
make_sound(Dog()) # Woof
```

---

## 3. Summary
- **Inheritance**: "is-a" relationship (A Dog *is an* Animal). Promotes code reuse.
- **Polymorphism**: "one interface, multiple forms". Allows for flexible and interchangeable code.
