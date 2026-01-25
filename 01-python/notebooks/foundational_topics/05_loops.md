# Python Loops

Loops are used to repeat a block of code multiple times. Python has two primitive loop commands: `for` and `while`.

## 1. The `for` Loop
A `for` loop is used for iterating over a sequence (list, tuple, dictionary, set, or string).

```python
fruits = ["apple", "banana", "cherry"]
for fruit in fruits:
    print(fruit)
```

### The `range()` Function
To loop through a set of code a specified number of times, we can use the `range()` function.

```python
for i in range(5):
    print(i) # Prints 0 to 4
```

---

## 2. The `while` Loop
A `while` loop executes a set of statements as long as a condition is true.

```python
i = 1
while i < 6:
    print(i)
    i += 1
```

---

## 3. Loop Control Statements

### `break`
Used to stop the loop even if the condition is true.

```python
for i in range(10):
    if i == 5:
        break
    print(i)
```

### `continue`
Used to stop the current iteration and continue with the next one.

```python
for i in range(5):
    if i == 3:
        continue
    print(i)
```

---

## 4. The `else` Block in Loops
Python allows an `else` block at the end of a loop. The `else` block is executed when the loop finishes naturally (i.e., not stopped by `break`).

```python
for i in range(3):
    print(i)
else:
    print("Loop finished successfully!")
```

---

## 5. Nested Loops
A loop inside another loop.

```python
adj = ["red", "big"]
fruits = ["apple", "banana"]

for a in adj:
    for f in fruits:
        print(a, f)

---

## 6. The `pass` Statement
The `pass` statement is a null operation; nothing happens when it executes. It is used as a placeholder when a statement is syntactically required but you don't want any command or code to execute.

```python
for x in [0, 1, 2]:
    pass # Placeholder for future code
```
```
