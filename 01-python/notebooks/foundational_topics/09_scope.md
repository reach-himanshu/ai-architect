# Python Variable Scope

Scope determines the visibility and lifetime of variables in your code. Python uses the **LEGB** rule to resolve variable names.

## 1. Local Scope
A variable created inside a function belongs to the **local scope** of that function and can only be used there.

```python
def my_func():
    x = 10 # Local variable
    print(x)

my_func()
# print(x) # This would raise a NameError
```

## 2. Global Scope
A variable created in the main body of the script is a **global variable** and belongs to the global scope. It is accessible from anywhere in the code.

```python
x = 300 # Global variable

def my_func():
    print(x) # Accessing global variable inside a function

my_func()
```

## 3. The `global` Keyword
If you need to **modify** a global variable inside a function, you must use the `global` keyword.

```python
count = 0

def increment():
    global count
    count += 1

increment()
print(count) # Output: 1
```

## 4. Enclosing Scope & `nonlocal`
This applies to **nested functions** (a function inside another function). A variable in the outer function is in the "enclosing" scope of the inner function.

```python
def outer():
    x = "original"
    def inner():
        nonlocal x # Refers to the 'x' in the outer function
        x = "modified"
    inner()
    print(x) # Output: modified
```

---

## 5. Why nested functions? (Use Cases)

### Scenario A: Closures (Data Hiding)
A **closure** is a function that "remembers" the variables from its enclosing scope even after the outer function has finished executing. This is great for maintaining state without using global variables.

```python
def make_counter():
    count = 0 # Enclosing variable
    def increment():
        nonlocal count
        count += 1
        return count
    return increment

my_counter = make_counter()
print(my_counter()) # 1
print(my_counter()) # 2
```

### Scenario B: Decorators
Decorators are functions that take another function and extend its behavior. They *must* use nested functions to wrap the logic.

```python
def my_decorator(func):
    def wrapper():
        print("Something before.")
        func()
        print("Something after.")
    return wrapper
```

### Scenario C: Encapsulation
If a helper function is only useful inside one specific function, nesting it hides it from the rest of the program, keeping your global namespace clean.

## 6. LEGB Rule (Resolution Order)
Resolution order for variable names:
1.  **L**ocal: Inside the current function.
2.  **E**nclosing: Inside any nested functions.
3.  **G**lobal: At the top level of the script.
4.  **B**uilt-in: Python's pre-defined names (like `len`, `int`).
