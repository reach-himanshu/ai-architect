# Python OOP: Special (Dunder) Methods and Operator Overloading

Special methods (also known as "Dunder" methods because they start and end with double underscores) allow you to define how your objects behave with built-in Python operations like addition, subtraction, printing, and length.

## 1. Lifecycle and Representation
We've already seen `__init__` and `__del__`. Here are the representation dunders:
- **`__str__`**: Returns a user-friendly string representation (used by `print()`).
- **`__repr__`**: Returns an "official" string representation (for developers/debugging).

```python
class Book:
    def __init__(self, title, author):
        self.title = title
        self.author = author

    def __str__(self):
        return f"'{self.title}' by {self.author}"

    def __repr__(self):
        return f"Book(title='{self.title}', author='{self.author}')"

book = Book("Python 101", "Himanshu")
print(str(book))  # 'Python 101' by Himanshu
print(repr(book)) # Book(title='Python 101', author='Himanshu')
```

---

## 2. Operator Overloading (Binary Operators)
You can make your objects support arithmetic operators by implementing specific dunder methods.

| Operator | Dunder Method |
| :--- | :--- |
| `+` | `__add__(self, other)` |
| `-` | `__sub__(self, other)` |
| `*` | `__mul__(self, other)` |
| `/` | `__truediv__(self, other)` |

```python
class Vector:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __add__(self, other):
        return Vector(self.x + other.x, self.y + other.y)

    def __repr__(self):
        return f"Vector({self.x}, {self.y})"

v1 = Vector(1, 2)
v2 = Vector(3, 4)
print(v1 + v2) # Vector(4, 6)
```

---

## 3. Comparison Operators
| Operator | Dunder Method |
| :--- | :--- |
| `==` | `__eq__(self, other)` |
| `<` | `__lt__(self, other)` |
| `>` | `__gt__(self, other)` |
| `!=` | `__ne__(self, other)` |

```python
class Product:
    def __init__(self, name, price):
        self.name = name
        self.price = price

    def __lt__(self, other):
        return self.price < other.price

p1 = Product("Laptop", 1000)
p2 = Product("Mouse", 20)
print(p1 < p2) # False
```

---

## 4. Collection Dunders
- **`__len__`**: Returns the length (used by `len()`).
- **`__getitem__`**: Allows indexing (used by `obj[key]`).

```python
class Playlist:
    def __init__(self, songs):
        self.songs = songs

    def __len__(self):
        return len(self.songs)

    def __getitem__(self, index):
        return self.songs[index]

my_list = Playlist(["Song A", "Song B", "Song C"])
print(len(my_list)) # 3
print(my_list[1])   # Song B
```

---

## 5. Summary
Operator overloading makes your classes feel like "First Class Citizens" in Python. It allows you to use intuitive syntax for complex objects instead of calling clumsy method names like `add_vectors()` or `is_price_lower_than()`.
