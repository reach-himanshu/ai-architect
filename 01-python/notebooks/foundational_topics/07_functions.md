# Python Functions and Recursion

Functions are reusable blocks of code that perform a specific task. They help make your code more modular and readable.

## 1. Defining a Function
Use the `def` keyword to define a function.

```python
def greet(name):
    """This function greets the person passed in as a parameter."""
    print(f"Hello, {name}!")

greet("Alice")
```

## 2. Parameters vs. Arguments
- **Parameters**: The variables listed in the function definition (e.g., `name`).
- **Arguments**: The values sent to the function when it is called (e.g., `"Alice"`).

### Argument Types
- **Positional Arguments**: Assigned based on their position.
- **Keyword Arguments**: Assigned using the name of the parameter.
- **Default Parameters**: Used if no argument is provided.

```python
def describe_pet(name, animal_type="dog"):
    print(f"I have a {animal_type} named {name}.")

describe_pet("Willie")            # Uses default "dog"
describe_pet(name="Rex", animal_type="hamster") # Keyword arguments
```

## 3. Variable-Length Arguments
Sometimes you don't know how many arguments a user will pass. Python provides two special symbols for this:

### `*args` (Non-Keyword Arguments)
- **What it is**: Collects extra positional arguments into a **tuple**.
- **Use Case**: Creating functions that can take any number of inputs (like a calculator that adds multiple numbers).

```python
def sum_all(*numbers):
    total = 0
    for num in numbers:
        total += num
    return total

print(sum_all(1, 2, 3, 4)) # Result: 10
```

### `**kwargs` (Keyword Arguments)
- **What it is**: Collects extra keyword arguments (key=value) into a **dictionary**.
- **Use Case**: Handling named settings, configurations, or when you want to pass a dynamic set of named parameters (like a user profile creator).

```python
def build_profile(first, last, **user_info):
    profile = {'first': first, 'last': last}
    for key, value in user_info.items():
        profile[key] = value
    return profile

user = build_profile('Albert', 'Einstein', location='Princeton', field='Physics')
print(user) # Result: {'first': 'Albert', 'last': 'Einstein', 'location': 'Princeton', 'field': 'Physics'}
```

### Combined Use Case: Decorators and Wrappers
The most common professional use case for `*args` and `**kwargs` is in **Wrappers** or **Decorators**, where you need to pass all arguments from one function directly into another without knowing what they are.

```python
def wrapper_function(func, *args, **kwargs):
    print("Executing function...")
    return func(*args, **kwargs)
```

## 4. Lambda Functions
Small, anonymous functions defined with the `lambda` keyword. They are often used as arguments to higher-order functions.

```python
# Syntax: lambda arguments: expression
square = lambda x: x * x
print(square(5)) # 25
```

## 5. Higher-Order Functions
Higher-order functions are functions that take other functions as arguments or return them. They are frequently used with lambdas.

- **`map(func, seq)`**: Applies a function to all items in a sequence.
- **`filter(func, seq)`**: Filters a sequence based on a function that returns `True` or `False`.
- **`reduce(func, seq)`**: Performs a rolling computation on a sequence (requires `functools`).

```python
numbers = [1, 2, 3, 4, 5]

# map: Square all numbers
squares = list(map(lambda x: x**2, numbers))

# filter: Only even numbers
evens = list(filter(lambda x: x % 2 == 0, numbers))
```

## 6. Recursion
A process where a function calls itself. 

### Importance of the Base Case
Every recursive function **must** have a base case—a condition that stops the recursion. Without it, the function will call itself infinitely until the program crashes.

### Avoiding Infinite Looping
- **Define a clear base case**: Ensure the recursion eventually hits it.
- **Progress towards the base case**: Each recursive call should move the input closer to the base case (e.g., decrementing `n`).
- **Recursion Limit**: Python has a default limit (usually 1000) to prevent stack overflow. You can check it with `sys.getrecursionlimit()`.

### Example: Factorial
```python
def factorial(n):
    if n <= 1: # Base case: handling 1 and 0
        return 1
    else:
        return n * factorial(n - 1) # Recursive case: n moves closer to 1
```
