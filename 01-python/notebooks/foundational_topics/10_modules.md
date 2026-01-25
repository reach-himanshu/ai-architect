# Python Modules and Packages

Modules and packages allow you to organize your code into separate files and directories, making it reusable and easier to maintain.

## 1. What is a Module?
A module is a file containing Python definitions and statements. Any `.py` file is a module.

### Importing a Module
```python
import math
print(math.sqrt(16)) # 4.0
```

### Importing Specific Items
```python
from math import pi, sin
print(pi)
```

### Using Aliases
```python
import pandas as pd
import datetime as dt
```

---

## 2. Creating Your Own Module
Suppose you have a file named `my_utils.py`:
```python
def say_hello():
    return "Hello from my_utils!"
```
You can use it in another file:
```python
import my_utils
print(my_utils.say_hello())
```

---

## 3. What is a Package?
A package is a directory that contains multiple modules and a special file named `__init__.py` (which tells Python this directory should be treated as a package).

### Package Structure
```text
my_app/
├── __init__.py
├── database/
│   ├── __init__.py
│   └── connection.py
└── utils/
    ├── __init__.py
    └── helper.py
```

### Importing from a Package
```python
from my_app.database import connection
connection.connect()
```

---

## 4. The `__name__ == "__main__"` Pattern
This is used to ensure code only runs when the script is executed directly, not when it is imported as a module.

```python
def main():
    print("This script is running directly!")

if __name__ == "__main__":
    main()
```

---

## 5. Built-in Modules You Should Know
- `os`: Interacting with the operating system.
- `sys`: Accessing system-specific parameters.
- `json`: Parsing and creating JSON data.
- `random`: Generating random numbers.
- `datetime`: Working with dates and times.
