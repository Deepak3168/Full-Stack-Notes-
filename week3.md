# Week 3 — Modules, Error Handling, Files & Advanced Python

*From organizing code with modules and packages to exceptions, file handling, JSON/CSV, context managers, iterators, generators, and decorators.*

---

## 1. Modules, Packages, and Imports

As a program grows, keeping everything in one file becomes unmanageable. Python lets you split code across multiple files and folders, then pull pieces back together wherever they're needed.

### Modules

A module is simply one Python file full of reusable code — functions, classes, or variables defined once and used elsewhere.

```python
# math_utils.py
def add(a, b):
    return a + b
```

### Import

Another file can pull that code in using `import`. Importing the whole module means accessing its contents with a dot:

```python
import math_utils
print(math_utils.add(2, 3))
```

Alternatively, you can import just the specific piece you need directly, without the dot:

```python
from math_utils import add
print(add(2, 3))
```

### Packages

A package is a folder containing multiple related modules, letting you organize code hierarchically as a project grows.

```
shapes/
  circle.py
  square.py
```

```python
from shapes import circle
```

> **Key idea:** modules = one file. Packages = a folder full of modules.

---

## 2. pip and Virtual Environments

### pip

Most Python projects rely on code someone else already wrote and published. `pip` is Python's package installer — it fetches published packages from PyPI (the Python Package Index) and installs them so you can import them.

```bash
pip install requests
```

This downloads the `requests` package and makes it importable in your code, just like any module you wrote yourself.

### Virtual Environments

Different projects often need different, sometimes conflicting, versions of the same package. Installing everything globally on your machine eventually causes version clashes between projects. A virtual environment solves this by giving each project its own isolated, self-contained space for installed packages.

```bash
python -m venv env
```

This creates a fresh, isolated environment named `env` for the current project only.

```bash
source env/bin/activate
```

Activating switches your terminal into that isolated environment — any `pip install` commands you run afterward only affect this project, not your whole system or other projects.

> **Rule of thumb:** create and activate a virtual environment at the start of every real project, before installing anything with pip.

---

## 3. Exceptions and try/except/else/finally

When something goes wrong while a program is running — dividing by zero, opening a file that doesn't exist, converting text that isn't actually a number — Python raises an exception. Left unhandled, an exception crashes the program immediately.

### try / except

Wrapping risky code in a `try` block lets you catch the exception and respond, instead of letting the whole program crash.

```python
try:
    x = 10 / 0
except ZeroDivisionError:
    print("Can't divide by zero")
```

The risky line runs inside `try`; if it raises the matching exception type, `except` catches it and the program keeps running.

### else

An `else` block runs only if the `try` block succeeded with no exception at all — useful for code that should run only after a successful attempt.

```python
try:
    x = 10 / 2
except ZeroDivisionError:
    print("Error")
else:
    print("Success:", x)
```

### finally

A `finally` block always runs, whether an exception happened or not — commonly used for cleanup, like closing a file or a network connection.

```python
try:
    x = 10 / 0
except ZeroDivisionError:
    print("Error")
finally:
    print("Done trying")
```

- **try** — the code that might fail
- **except** — runs if a matching error occurs, preventing a crash
- **else** — runs only if try succeeded with no error
- **finally** — always runs, error or not

---

## 4. Custom Exceptions

Python's built-in exceptions don't always describe your specific problem. You can define your own exception type by inheriting from the built-in `Exception` class.

```python
class InvalidAgeError(Exception):
    pass
```

Because it inherits from `Exception`, Python treats it as a genuine error type, usable with `raise` and `except` just like a built-in one.

```python
def set_age(age):
    if age < 0:
        raise InvalidAgeError("Age can't be negative")
```

`raise` triggers the custom error on purpose, carrying your own descriptive message.

```python
try:
    set_age(-5)
except InvalidAgeError as e:
    print(e)
```

> **Why bother:** a custom exception name like `InvalidAgeError` immediately tells the next reader (including future you) exactly what went wrong, instead of a generic `ValueError` with no context.

---

## 5. File Handling

Variables live only in memory — the moment a program ends, they're gone. Files let data survive after the program closes.

### Writing

```python
f = open("notes.txt", "w")
f.write("Hello file!")
f.close()
```

`"w"` opens the file in write mode, creating it if it doesn't exist or overwriting it if it does. Always close a file when you're done, or changes may not actually be saved to disk.

### Reading

```python
f = open("notes.txt", "r")
content = f.read()
print(content)
f.close()
```

`"r"` opens the file in read mode, pulling existing content out.

### File Modes

- `"r"` — read (file must already exist)
- `"w"` — write (creates a new file, or overwrites an existing one)
- `"a"` — append (adds to the end of an existing file, without erasing it)
- `"r+"` — read and write on the same file

---

## 6. JSON

JSON (JavaScript Object Notation) is a plain-text format that looks almost exactly like a Python dictionary or list, but is understood across virtually every programming language — making it the standard way to save structured data to a file, or send it over a network.

```python
import json
data = {"name": "Ravi", "age": 20}
with open("data.json", "w") as f:
    json.dump(data, f)
```

`json.dump()` converts a Python dict into JSON text and writes it directly to the open file.

```python
with open("data.json", "r") as f:
    data = json.load(f)
print(data)
```

`json.load()` reads JSON text back from a file and turns it back into a real Python dict, ready to use again.

---

## 7. CSV

CSV (Comma-Separated Values) is the plain-text format spreadsheets are saved as — each line is one row, and commas separate the columns within it.

```python
import csv
with open("students.csv", "w") as f:
    writer = csv.writer(f)
    writer.writerow(["Ravi", 20])
```

`csv.writer` turns a Python list into one comma-separated line written to the file.

```python
with open("students.csv", "r") as f:
    reader = csv.reader(f)
    for row in reader:
        print(row)
```

`csv.reader` turns each line of the file back into a Python list, ready to loop over.

---

## 8. Context Managers

It's easy to forget to close a file after opening it — and a forgotten `.close()` can leave a file locked or its contents unsaved. The `with` statement is a context manager: it opens a resource, runs the code inside the block, and automatically closes the resource afterward, even if an error happens partway through.

```python
with open("notes.txt", "r") as f:
    content = f.read()
```

- **Manual:** open → use → must remember to close, even on error paths
- **with:** open → use → closes itself automatically, guaranteed

> **Best practice:** prefer `with open(...) as f:` over manually calling `open()`/`close()` for every file you work with.

---

## 9. Iterators

What actually happens when a `for` loop runs over a list? Under the hood, Python uses two related concepts: an **iterable** (something you CAN loop over, like a list) and an **iterator** (the object actually doing the stepping, one item at a time).

```python
nums = [1, 2, 3]
it = iter(nums)
print(next(it))
print(next(it))
```

`iter()` creates the stepping tool from an iterable; `next()` moves it forward one item at a time. When there are no items left, calling `next()` raises a `StopIteration` exception — which is exactly how a `for` loop knows when to stop.

> **Key idea:** a `for` loop is really just Python repeatedly calling `next()` behind the scenes until it runs out of items.

---

## 10. Generators

A regular function that returns a list builds the entire list in memory before handing anything back — which becomes a problem if that list would be huge. A generator function produces values one at a time, on demand, using `yield` instead of `return`.

```python
def count_up(n):
    i = 1
    while i <= n:
        yield i
        i += 1
```

`yield` pauses the function and hands back one value, but the function remembers exactly where it left off — the next call resumes right after that `yield`.

```python
gen = count_up(3)
print(next(gen))
print(next(gen))
```

Each call to `next()` resumes the paused function until it hits the next `yield` (or the function ends).

> **Why it matters:** generators are memory-efficient — instead of building a giant list all at once, values are produced lazily, only when actually needed.

---

## 11. Decorators

Sometimes you want to add behavior around an existing function — logging, timing, access checks — without editing the function's own code. A decorator is a function that takes another function in, and returns a new, wrapped version of it.

```python
def my_decorator(func):
    def wrapper():
        print("Before")
        func()
        print("After")
    return wrapper
```

The decorator accepts the original function, defines a new wrapper function around it, and returns that wrapper in its place.

```python
@my_decorator
def say_hello():
    print("Hello")

say_hello()
```

`@my_decorator` above a function definition is shorthand for `say_hello = my_decorator(say_hello)`. Calling `say_hello()` now actually runs the wrapper, which prints "Before", calls the original function, then prints "After".

> **Real-world use:** decorators are commonly used for logging, timing how long a function takes, or checking permissions — the same wrapping behavior applied to many different functions without duplicating code in each one.