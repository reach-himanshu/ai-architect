# Python File I/O

File I/O (Input/Output) allows your Python programs to interact with files on your computer.

## 1. Opening and Closing Files
The safest way to open a file is using the `with` statement. It ensures the file is automatically closed, even if an error occurs.

### Syntax
```python
with open("filename.txt", "mode") as file:
    # Perform file operations
```

---

## 2. File Modes
| Mode | Description |
| :--- | :--- |
| **`'r'`** | **Read** (Default): Opens a file for reading; error if it doesn't exist. |
| **`'w'`** | **Write**: Opens a file for writing; creates it or truncates the existing one. |
| **`'a'`** | **Append**: Opens a file for appending; creates it if it doesn't exist. |
| **`'x'`** | **Create**: Creates a new file; error if it already exists. |
| **`'b'`** | **Binary**: For non-text files (e.g., images). |

---

## 3. Reading Files
- **`read()`**: Reads the entire file content.
- **`readline()`**: Reads a single line.
- **`readlines()`**: Reads all lines into a list.
- **Iteration**: The most memory-efficient way.

```python
with open("sample.txt", "r") as f:
    for line in f:
        print(line.strip())
```

---

## 4. Writing Files
- **`write()`**: Writes a string to the file.
- **`writelines()`**: Writes a list of strings.

```python
lines = ["First line\n", "Second line\n"]
with open("output.txt", "w") as f:
    f.writelines(lines)
```

---

## 5. Working with Paths
For cleaner and cross-platform path handling, use the `pathlib` module.

```python
from pathlib import Path

# Create a path object
path = Path("data/settings.json")

# Check if file exists
if path.exists():
    print("File found!")
```

---

## 6. Summary: The Golden Rule
> [!IMPORTANT]
> **Always use the `with` statement.** Manually calling `.open()` and `.close()` is prone to bugs (like leaving a file locked if your script crashes).
