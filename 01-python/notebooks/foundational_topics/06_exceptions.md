# Python Exception Handling

Exception handling allows you to handle errors gracefully without crashing your program. Use `try`, `except`, `else`, and `finally` blocks.

## 1. The `try...except` Block
The `try` block lets you test a block of code for errors, and the `except` block handles the error.

```python
try:
    print(10 / 0)
except ZeroDivisionError:
    print("Error: Cannot divide by zero!")
```

## 2. Catching Any Exception
You can use a generic `except` block to catch all errors (though specific catching is preferred).

```python
try:
    print(x)
except Exception as e:
    print(f"An error occurred: {e}")
```

## 3. Multiple Exceptions
You can handle multiple specific exceptions separately.

```python
try:
    num = int(input("Enter a number: "))
    result = 10 / num
except ValueError:
    print("That's not a valid number!")
except ZeroDivisionError:
    print("You can't divide by zero!")
```

## 4. The `else` Block
The `else` block runs if **no exceptions** were raised in the `try` block.

```python
try:
    print("Trying something safe...")
except:
    print("Something went wrong.")
else:
    print("Everything went perfectly!")
```

## 5. The `finally` Block
The `finally` block runs **no matter what**, regardless of whether an exception occurred or was handled. This is often used for cleanup (like closing files).

```python
try:
    f = open("sample.txt", "r")
    # Perform file operations
except FileNotFoundError:
    print("File not found.")
finally:
    print("Executing cleanup (this always runs).")
```

## 6. Raising Exceptions
You can manually trigger an exception using the `raise` keyword.

```python
x = -1
if x < 0:
    raise ValueError("Only positive numbers are allowed!")
```
