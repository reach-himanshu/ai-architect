# Python Standard Data Types

Python has several built-in data types that are used to store and manipulate data. They are categorized as follows:

## 1. Numeric Types
Used to represent numbers.
- **`int`**: Integer values (e.g., `5`, `-10`).
- **`float`**: Floating-point numbers (e.g., `3.14`, `-0.001`).
- **`complex`**: Complex numbers with a real and imaginary part (e.g., `3 + 5j`).

## 2. Sequence Types
Used to store collections of items in a specific order.
- **`str`**: Strings; sequences of Unicode characters (e.g., `"Hello"`).
- **`list`**: Mutable sequences (e.g., `[1, "apple", 3.5]`).
- **`tuple`**: Immutable sequences (e.g., `(10, 20, 30)`).
- **`range`**: Represents an immutable sequence of numbers (e.g., `range(5)`).

## 3. Mapping Type
Used to store data in key-value pairs.
- **`dict`**: Dictionaries (e.g., `{"name": "Alice", "age": 25}`).

## 4. Set Types
Used to store unordered collections of unique items.
- **`set`**: Mutable set (e.g., `{1, 2, 3}`).
- **`frozenset`**: Immutable version of a set.

## 5. Boolean Type
Represents truth values.
- **`bool`**: `True` and `False`.

## 6. Binary Types
Used to manipulate binary data.
- **`bytes`**: Immutable sequences of single bytes.
- **`bytearray`**: Mutable version of `bytes`.
- **`memoryview`**: Allows accessing internal data of an object without copying.

## 7. None Type
- **`NoneType`**: Represents the absence of a value (`None`).

---

### Type Checking
You can check the type of any object using the `type()` function:
```python
x = 5
print(type(x)) # Output: <class 'int'>
```
