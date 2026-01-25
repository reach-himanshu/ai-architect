# Python Operators

Operators are symbols that perform operations on variables and values. Python divides operators into the following groups:

## 1. Arithmetic Operators
Used with numeric values to perform common mathematical operations.

| Operator | Name | Example | Result |
| :--- | :--- | :--- | :--- |
| `+` | Addition | `10 + 5` | `15` |
| `-` | Subtraction | `10 - 5` | `5` |
| `*` | Multiplication | `10 * 5` | `50` |
| `/` | Division | `10 / 3` | `3.333...` |
| `%` | Modulus | `10 % 3` | `1` |
| `**` | Exponentiation | `2 ** 3` | `8` |
| `//` | Floor division | `10 // 3` | `3` |

```python
x = 10
y = 3
print(x + y)  # 13
print(x / y)  # 3.3333333333333335
print(x // y) # 3 (removes decimal part)
print(x % y)  # 1 (remainder)
```

## 2. Assignment Operators
Used to assign values to variables.

| Operator | Example | Equivalent to |
| :--- | :--- | :--- |
| `=` | `x = 5` | `x = 5` |
| `+=` | `x += 3` | `x = x + 3` |
| `-=` | `x -= 3` | `x = x - 3` |
| `*=` | `x *= 3` | `x = x * 3` |
| `/=` | `x /= 3` | `x = x / 3` |

```python
x = 10
x += 5  # x is now 15
x *= 2  # x is now 30
print(x) # 30
```

## 3. Comparison Operators
Used to compare two values. They return a boolean (`True` or `False`).

| Operator | Name | Example |
| :--- | :--- | :--- |
| `==` | Equal | `5 == 5` (True) |
| `!=` | Not equal | `5 != 3` (True) |
| `>` | Greater than | `5 > 3` (True) |
| `<` | Less than | `5 < 3` (False) |
| `>=` | Greater than or equal to | `5 >= 5` (True) |
| `<=` | Less than or equal to | `5 <= 4` (False) |

```python
x = 5
y = 10
print(x == y) # False
print(x < y)  # True
```

## 4. Logical Operators
Used to combine conditional statements.

```python
x = 5
# and: True if both are True
print(x > 3 and x < 10) # True

# or: True if at least one is True
print(x > 3 or x < 4)   # True

# not: Inverts the result
print(not(x > 3))       # False
```

## 5. Identity Operators
Used to compare objects, not if they are equal, but if they are actually the same object, with the same memory location.

```python
x = ["apple", "banana"]
y = ["apple", "banana"]
z = x

print(x is z)     # True (same object)
print(x is y)     # False (different objects with same content)
print(x == y)    # True (content is equal)
```

## 6. Membership Operators
Used to test if a sequence is presented in an object.

```python
fruits = ["apple", "banana", "cherry"]

print("banana" in fruits)     # True
print("orange" not in fruits) # True
```

## 7. Bitwise Operators
Used to compare (binary) numbers.

| Operator | Name | Example (Binary) |
| :--- | :--- | :--- |
| `&` | AND | `5 & 1` (0101 & 0001 = 0001) -> `1` |
| `\|` | OR | `5 \| 1` (0101 \| 0001 = 0101) -> `5` |
| `^` | XOR | `5 ^ 1` (0101 ^ 0001 = 0100) -> `4` |
| `~` | NOT | `~5` -> `-6` |
| `<<` | Left shift | `5 << 1` (0101 -> 1010) -> `10` |
| `>>` | Right shift | `5 >> 1` (0101 -> 0010) -> `2` |
