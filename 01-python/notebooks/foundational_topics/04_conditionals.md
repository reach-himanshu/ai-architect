# Python Conditional Statements

Conditional statements allow you to execute different blocks of code based on certain conditions.

## 1. The `if` Statement
The simplest form of a conditional. If the condition is `True`, the code block is executed.

```python
x = 10
if x > 5:
    print("x is greater than 5")
```

## 2. The `else` Statement
Used to provide an alternative block of code if the `if` condition is `False`.

```python
x = 3
if x > 5:
    print("x is greater than 5")
else:
    print("x is not greater than 5")
```

## 3. The `elif` Statement
Short for "else if". It allows you to check multiple conditions.

```python
x = 5
if x > 5:
    print("x is greater than 5")
elif x == 5:
    print("x is exactly 5")
else:
    print("x is less than 5")
```

## 4. Nested Conditionals
You can place inside another conditional statement.

```python
x = 10
y = 20
if x > 5:
    if y > 15:
        print("Both conditions are met")
```

## 5. Short Hand (Ternary Operator)
Useful for simple if-else assignments in a single line.

```python
# Syntax: value_if_true if condition else value_if_false
x = 10
status = "Adult" if x >= 18 else "Minor"
print(status)
```

## 6. Logical Operators with Conditionals
You can combine conditions using `and`, `or`, and `not`.

```python
x = 7
if x > 5 and x < 10:
    print("x is between 5 and 10")
```
