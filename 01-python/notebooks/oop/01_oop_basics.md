# Python Object-Oriented Programming (OOP)

OOP is a programming paradigm based on the concept of **"objects"**, which can contain data (attributes) and code (methods).

## 1. Classes and Objects
- **Class**: A blueprint for creating objects (e.g., a "Car" blueprint).
- **Object**: An instance of a class (e.g., your specific red Tesla).

```python
class Car:
    pass

my_car = Car()
```

## 2. Constructors and Destructors
Python uses special "dunder" methods to manage the lifecycle of an object.

### The Constructor: `__init__`
The `__init__` method is automatically called when a new object is created. It is used to initialize the object's attributes.

```python
class Car:
    def __init__(self, brand):
        self.brand = brand
```

### The Destructor: `__del__`
The `__del__` method is called when an object is about to be destroyed (garbage collected). This is rarely used in simple scripts but is helpful for closing files or database connections.

```python
class DatabaseConnection:
    def __init__(self):
        print("Connecting to DB...")

    def __del__(self):
        print("Closing the DB connection safely.")

db = DatabaseConnection()
del db # Manually trigger destruction
```

## 3. Instance Methods
Functions defined inside a class that operate on the instance. They must always take `self` as the first argument.

```python
class Car:
    def __init__(self, brand, model):
        self.brand = brand
        self.model = model

    def drive(self):
        return f"The {self.brand} {self.model} is now driving!"

my_car = Car("Tesla", "Model 3")
print(my_car.drive())
```

## 4. Class vs Instance Attributes
- **Instance Attributes**: Unique to each object (e.g., `brand`).
- **Class Attributes**: Shared by all objects of the class (e.g., `number_of_wheels = 4`).

```python
class Car:
    wheels = 4 # Class attribute

    def __init__(self, brand):
        self.brand = brand

car1 = Car("Audi")
car2 = Car("BMW")
print(car1.wheels) # 4
print(car2.wheels) # 4
```

## 5. The Four Pillars of OOP
1.  **Encapsulation**: Bundling data and methods together (keeping things private).
2.  **Inheritance**: Creating new classes from existing ones.
3.  **Polymorphism**: Using a single interface to represent different types.
4.  **Abstraction**: Hiding complex implementation details.
