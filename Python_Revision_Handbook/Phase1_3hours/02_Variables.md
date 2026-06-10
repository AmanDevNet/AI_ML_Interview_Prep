# Chapter 2: Variables

---

## Topic Overview

| Aspect | Details |
|--------|---------|
| **Definition** | A variable is a **named reference** to an object stored in memory. It's a label that points to a memory location containing a value. |
| **Why It Exists** | Variables allow us to store, retrieve, and manipulate data throughout a program's execution without remembering raw memory addresses. |
| **Where It Is Used** | Every Python program — from simple scripts to complex ML pipelines — uses variables to hold data. |
| **Real-World Analogy** | A variable is like a **sticky note** on a box. The sticky note (variable name) points to the box (object in memory). You can peel the sticky note off one box and stick it on another (reassignment). |

---

## 2.1 Variables

### How It Works Internally

In Python, variables are **not boxes that contain values**. They are **references (pointers)** to objects in memory.

```
Variable name → Reference → Object in memory
    x         →   ptr   →  [int object: 42]
```

When you write `x = 42`:
1. Python creates an `int` object with value `42` on the **heap**.
2. Python creates a **reference** named `x` that points to this object.
3. The reference is stored in a **namespace** (a dictionary mapping names to objects).

```python
x = 42
print(id(x))    # Memory address of the object
print(type(x))  # <class 'int'>

# Internally, the namespace looks like: {'x': <reference to int object 42>}
```

### Memory Behavior
- Small integers (-5 to 256) are **cached/interned** by CPython for performance.
- String literals are also **interned** in many cases.
- Variables themselves don't have types — **objects** have types.
- When no variable references an object, it becomes eligible for **garbage collection**.

```python
# Integer caching demonstration
a = 256
b = 256
print(a is b)  # True — same cached object

a = 257
b = 257
print(a is b)  # False (in REPL) — different objects (may vary by context)
```

### Syntax
```python
# Basic assignment
name = "Python"
age = 30
pi = 3.14159
is_valid = True
nothing = None
```

### Important Rules
1. Variable names must start with a **letter** or **underscore** (`_`).
2. Can contain letters, digits, and underscores.
3. **Cannot** start with a digit.
4. **Cannot** use Python **reserved keywords** (35 keywords in Python 3.11+).
5. Variable names are **case-sensitive**.
6. Convention: Use `snake_case` for variables and functions.
7. Assignment creates a **new reference**, not a copy of the value.

---

## 2.2 Naming Rules

### Valid vs Invalid Names

| Name | Valid? | Reason |
|------|--------|--------|
| `my_var` | ✅ | Letters and underscore |
| `_private` | ✅ | Starts with underscore |
| `__dunder__` | ✅ | Dunder name (convention for special methods) |
| `myVar2` | ✅ | Contains digit (not at start) |
| `MY_CONSTANT` | ✅ | All caps (convention for constants) |
| `2ndPlace` | ❌ | Starts with digit |
| `my-var` | ❌ | Contains hyphen |
| `my var` | ❌ | Contains space |
| `class` | ❌ | Reserved keyword |
| `for` | ❌ | Reserved keyword |

### Python Reserved Keywords (Python 3.12+)
```python
import keyword
print(keyword.kwlist)
# ['False', 'None', 'True', 'and', 'as', 'assert', 'async', 'await',
#  'break', 'class', 'continue', 'def', 'del', 'elif', 'else', 'except',
#  'finally', 'for', 'from', 'global', 'if', 'import', 'in', 'is',
#  'lambda', 'nonlocal', 'not', 'or', 'pass', 'raise', 'return',
#  'try', 'while', 'with', 'yield']
```

### Naming Conventions (PEP 8)
| Type | Convention | Example |
|------|-----------|---------|
| Variable | `snake_case` | `user_name` |
| Constant | `UPPER_SNAKE_CASE` | `MAX_RETRIES` |
| Function | `snake_case` | `calculate_total()` |
| Class | `PascalCase` | `UserProfile` |
| Private | `_leading_underscore` | `_internal_value` |
| Name Mangling | `__double_leading` | `__secret` |
| Dunder/Magic | `__double_both__` | `__init__` |
| Module | `short_lowercase` | `utils.py` |

---

## 2.3 Dynamic Typing

### Definition
Dynamic typing means the **type of a variable is determined at runtime**, not at compile time. A variable can refer to objects of different types during its lifetime.

### How It Works Internally
- In Python, **variables don't have types** — **objects** have types.
- A variable is just a name tag. You can reassign it to any type.
- Type checking happens at **runtime**, not before execution.

```python
x = 42          # x references an int object
print(type(x))  # <class 'int'>

x = "hello"     # x now references a str object (int 42 may be garbage collected)
print(type(x))  # <class 'str'>

x = [1, 2, 3]   # x now references a list object
print(type(x))  # <class 'list'>
```

### Comparison: Static vs Dynamic Typing

| Feature | Static Typing (Java, C++) | Dynamic Typing (Python) |
|---------|--------------------------|------------------------|
| Type Declaration | Required | Not required |
| Type Checking | At compile time | At runtime |
| Variable Type | Fixed after declaration | Can change |
| Error Detection | Before running | While running |
| Development Speed | Slower (more code) | Faster (less boilerplate) |
| Safety | More type-safe | More flexible |

### Interview Trap
**Q: Can a variable in Python change its type?**
> Trick answer: Variables don't have types. The name `x` is just a reference. When you write `x = "hello"` after `x = 42`, you're making `x` point to a new object of a different type. The variable itself has no type — the objects do.

---

## 2.4 Strong Typing

### Definition
Strong typing means Python does **not implicitly convert between incompatible types**. Operations on mismatched types raise `TypeError`.

### How It Works
```python
# Python is STRONGLY typed
"hello" + 5        # TypeError: can only concatenate str to str
"hello" + str(5)   # "hello5" — explicit conversion required

# JavaScript is WEAKLY typed (for comparison)
# "hello" + 5      → "hello5" (implicit conversion)
```

### Strong vs Weak Typing

| Operation | Python (Strong) | JavaScript (Weak) |
|-----------|----------------|-------------------|
| `"5" + 3` | `TypeError` | `"53"` |
| `"5" * 3` | `"555"` (valid! str repetition) | `15` |
| `True + 1` | `2` (valid! bool is subclass of int) | `2` |
| `None + 1` | `TypeError` | `1` |

### Important Nuance
Python is strongly typed but has some implicit conversions:
- `bool` → `int`: `True + 1 = 2` (because `bool` is a subclass of `int`)
- `int` → `float`: `5 + 3.0 = 8.0` (numeric tower promotion)

```python
print(True + True)    # 2 (True is 1)
print(False + 10)     # 10 (False is 0)
print(5 + 3.14)       # 8.14 (int promoted to float)
print(isinstance(True, int))  # True!
```

---

## 2.5 Memory References

### Definition
Every object in Python has a unique identity (memory address). The `id()` function returns this identifier.

### How It Works Internally
```python
a = [1, 2, 3]
b = a           # b and a point to the SAME object

print(id(a))    # e.g., 140234567890
print(id(b))    # Same: 140234567890
print(a is b)   # True — same object in memory

b.append(4)     # Modifying through b...
print(a)        # [1, 2, 3, 4] — a is also modified!
```

### Memory Model Visualization
```
Assignment:  a = [1, 2, 3]
             b = a

Memory:
  a ──────┐
          ├──→ [list object: [1, 2, 3]]  at id: 0x7f...
  b ──────┘

After:  b.append(4)

  a ──────┐
          ├──→ [list object: [1, 2, 3, 4]]  at id: 0x7f... (same!)
  b ──────┘
```

### Reference Counting
```python
import sys

a = "hello"
print(sys.getrefcount(a))  # Usually 2+ (a + the getrefcount argument)

b = a
print(sys.getrefcount(a))  # 3+ (a + b + getrefcount argument)

del b
print(sys.getrefcount(a))  # Back to 2+
```

---

## 2.6 Multiple Assignment

### Syntax
```python
# Assign same value to multiple variables
x = y = z = 0
print(x, y, z)  # 0 0 0

# All three reference the SAME object
print(id(x) == id(y) == id(z))  # True

# Assign different values in one line
a, b, c = 1, 2, 3
print(a, b, c)  # 1 2 3

# Swap variables (Pythonic way)
a, b = b, a
print(a, b)  # 2 1
```

### How Swap Works Internally
```python
a, b = b, a
# 1. Python evaluates the RIGHT side first, creating a tuple (b, a) → (2, 1)
# 2. Then unpacks the tuple into a, b
# No temporary variable needed!
```

---

## 2.7 Unpacking

### Definition
Unpacking is the process of extracting values from iterables and assigning them to individual variables.

### Types of Unpacking

```python
# Basic unpacking
x, y, z = [10, 20, 30]
print(x, y, z)  # 10 20 30

# Unpacking with * (extended unpacking)
first, *rest = [1, 2, 3, 4, 5]
print(first)  # 1
print(rest)   # [2, 3, 4, 5]

*beginning, last = [1, 2, 3, 4, 5]
print(beginning)  # [1, 2, 3, 4]
print(last)       # 5

first, *middle, last = [1, 2, 3, 4, 5]
print(first)   # 1
print(middle)  # [2, 3, 4]
print(last)    # 5

# Unpacking in function calls
def add(a, b, c):
    return a + b + c

values = [1, 2, 3]
print(add(*values))  # 6 — unpacks list as positional arguments

data = {"a": 1, "b": 2, "c": 3}
print(add(**data))   # 6 — unpacks dict as keyword arguments

# Nested unpacking
(a, b), (c, d) = [1, 2], [3, 4]
print(a, b, c, d)  # 1 2 3 4

# Ignoring values with _
_, important, _ = (1, 42, 3)
print(important)  # 42
```

### Important Rules for Unpacking
1. Number of variables must match number of values (unless using `*`).
2. Only **one** starred expression (`*`) allowed per assignment.
3. `_` is a valid variable name, conventionally used for "don't care" values.
4. Unpacking works with any **iterable** (list, tuple, string, set, dict, generator).

---

## Common Mistakes

| Mistake | Example | Fix |
|---------|---------|-----|
| Using reserved keyword as variable | `class = "A"` | `student_class = "A"` |
| Starting with digit | `2name = "Bob"` | `name2 = "Bob"` |
| Confusing `=` and `==` | `if x = 5:` | `if x == 5:` |
| Assuming `b = a` copies the object | `b = a; b.append(1)` changes `a` | `b = a.copy()` or `b = list(a)` |
| Unpack count mismatch | `a, b = [1, 2, 3]` → `ValueError` | `a, b, c = [1, 2, 3]` or `a, *b = [1,2,3]` |
| Using mutable default in assignment | `x = y = []` both point to same list | `x, y = [], []` |
| Case sensitivity | `Name != name` | Be consistent |

---

## Interview Questions

**Q1: Are Python variables boxes or labels?**
> Labels (references). In Python, variables are names that reference objects in memory. Multiple variables can reference the same object. This is fundamentally different from C where variables are memory locations.

**Q2: What is the difference between `==` and `is`?**
> `==` compares **values** (calls `__eq__`). `is` compares **identity** (memory addresses / `id()`). Two objects can be equal (`==`) without being identical (`is`).
```python
a = [1, 2, 3]
b = [1, 2, 3]
print(a == b)   # True (same value)
print(a is b)   # False (different objects)
```

**Q3: What happens when you do `x = y = []`?**
> Both `x` and `y` reference the **same** list object. Modifying through one affects the other:
```python
x = y = []
x.append(1)
print(y)  # [1] — same object!
```

**Q4: How does Python swap `a, b = b, a` without a temp variable?**
> Python evaluates the right side first, creating a tuple `(b, a)`, then unpacks it. The tuple creation captures the original values before assignment.

**Q5: What is the output of `id(256) == id(256)` vs `id(257) == id(257)`?**
> `id(256) == id(256)` is `True` because integers -5 to 256 are cached (interned). `id(257) == id(257)` may be `True` or `False` depending on context (in a script, CPython may optimize; in REPL, they may be different objects).

**Q6: What is the difference between dynamic typing and weak typing?**
> Dynamic typing means types are checked at runtime. Weak typing means implicit type conversions happen. Python is **dynamically typed AND strongly typed** — types are checked at runtime, but incompatible types are not implicitly converted.

**Q7: What is name mangling?**
> Variables starting with `__` (double underscore) inside a class are "mangled" — Python changes `__var` to `_ClassName__var` to avoid name conflicts in subclasses.

---

## Code Examples

### Easy Examples

**Example 1: Basic variable assignment and types**
```python
name = "Alice"
age = 25
height = 5.6
is_student = True

print(name, type(name))       # Alice <class 'str'>
print(age, type(age))         # 25 <class 'int'>
print(height, type(height))   # 5.6 <class 'float'>
print(is_student, type(is_student))  # True <class 'bool'>
```

**Example 2: Variable reassignment (dynamic typing)**
```python
x = 10
print(x, type(x))   # 10 <class 'int'>

x = "ten"
print(x, type(x))   # ten <class 'str'>

x = [10]
print(x, type(x))   # [10] <class 'list'>
```

**Example 3: Multiple assignment**
```python
a, b, c = "Python", 3.12, True
print(a)  # Python
print(b)  # 3.12
print(c)  # True
```

**Example 4: Swapping variables**
```python
x, y = 10, 20
print(f"Before: x={x}, y={y}")  # Before: x=10, y=20

x, y = y, x
print(f"After:  x={x}, y={y}")  # After:  x=20, y=10
```

**Example 5: Checking variable identity**
```python
a = 100
b = 100
print(a == b)   # True (same value)
print(a is b)   # True (cached integer)
print(id(a), id(b))  # Same id
```

### Medium Examples

**Example 6: Reference sharing with mutable objects**
```python
list_a = [1, 2, 3]
list_b = list_a        # Both point to same object

list_b.append(4)
print(list_a)           # [1, 2, 3, 4] — modified via list_b!
print(list_a is list_b) # True
```

**Example 7: Extended unpacking**
```python
scores = [95, 87, 92, 78, 85, 90, 88]

highest, *remaining, lowest = sorted(scores, reverse=True)
print(f"Highest: {highest}")    # 95
print(f"Lowest: {lowest}")      # 78
print(f"Others: {remaining}")   # [92, 90, 88, 87, 85]
```

**Example 8: Nested unpacking**
```python
data = [("Alice", 25), ("Bob", 30), ("Charlie", 35)]

for name, age in data:
    print(f"{name} is {age} years old")
# Alice is 25 years old
# Bob is 30 years old
# Charlie is 35 years old
```

**Example 9: Integer caching boundaries**
```python
# Test the integer caching range
import sys

for val in [0, 100, 256, 257, -5, -6]:
    a = val
    b = val
    # Note: In a script, CPython may optimize this
    # The canonical range is -5 to 256
    print(f"{val}: id(a)==id(b) → {id(a) == id(b)}")
```

**Example 10: Reference counting**
```python
import sys

a = "interview_prep"
print(f"Ref count after creation: {sys.getrefcount(a)}")

b = a
print(f"Ref count after b=a: {sys.getrefcount(a)}")

c = a
print(f"Ref count after c=a: {sys.getrefcount(a)}")

del c
print(f"Ref count after del c: {sys.getrefcount(a)}")

del b
print(f"Ref count after del b: {sys.getrefcount(a)}")
```

### Advanced Examples

**Example 11: Demonstrating that variables are namespace entries**
```python
x = 42
y = "hello"

# The local namespace is a dictionary
print(locals())
# {'x': 42, 'y': 'hello', ...}

# You can even create variables dynamically (not recommended)
globals()['dynamic_var'] = 999
print(dynamic_var)  # 999
```

**Example 12: Augmented assignment and identity**
```python
# With immutable types (int)
x = 5
print(f"Before +=: id={id(x)}")
x += 1  # Creates a NEW int object!
print(f"After  +=: id={id(x)}")  # Different id!

# With mutable types (list)
lst = [1, 2]
print(f"Before +=: id={id(lst)}")
lst += [3]  # Modifies IN-PLACE (calls __iadd__)
print(f"After  +=: id={id(lst)}")  # Same id!
```

**Example 13: Unpacking edge cases**
```python
# Unpacking with single-element tuple
(a,) = [42]
print(a)  # 42

# Unpacking dictionary (gets keys)
x, y, z = {"a": 1, "b": 2, "c": 3}
print(x, y, z)  # a b c

# Unpacking string
a, b, c = "Hi!"
print(a, b, c)  # H i !

# Star unpacking into empty list
first, *rest = [1]
print(first)  # 1
print(rest)   # [] — empty list, not an error!
```

**Example 14: Variable scope and mutability interaction**
```python
# Immutable — rebinding doesn't affect outer scope
def modify_int(n):
    n = n + 1  # Creates new object, local reference
    return n

x = 10
modify_int(x)
print(x)  # 10 — unchanged!

# Mutable — modification through reference affects outer scope
def modify_list(lst):
    lst.append(4)  # Modifies the SAME object

my_list = [1, 2, 3]
modify_list(my_list)
print(my_list)  # [1, 2, 3, 4] — changed!
```

**Example 15: Walrus operator (:=) — Assignment expression (Python 3.8+)**
```python
# Traditional way
data = input("Enter value: ") if False else "hello"  # simulated
if len(data) > 3:
    print(f"Long input: {data}")

# With walrus operator
# Assigns and evaluates in one expression
if (n := len("hello world")) > 5:
    print(f"Length {n} is greater than 5")  # Length 11 is greater than 5

# Useful in while loops
import io
buffer = io.StringIO("line1\nline2\nline3")
while (line := buffer.readline()):
    print(line.strip())

# Useful in list comprehensions
values = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
results = [y for x in values if (y := x ** 2) > 20]
print(results)  # [25, 36, 49, 64, 81, 100]
```

---

## Edge Cases

1. **`_` as a valid variable**: `_ = 5; print(_)` works. In REPL, `_` holds the last result.
2. **Unicode variable names**: Python 3 allows `café = "coffee"` (PEP 3131), but avoid in production.
3. **`del` doesn't delete objects**: `del x` removes the name `x` from namespace. The object is garbage collected only when its reference count reaches 0.
4. **Chained assignment creates shared references**: `a = b = []` means `a is b` is `True`.
5. **Augmented assignment on immutable types**: `x += 1` where `x` is an `int` creates a **new** object (immutable). But `lst += [1]` where `lst` is a list modifies **in-place** (mutable).

---

## Comparison with Similar Concepts

| Concept | Python | C | Java | JavaScript |
|---------|--------|---|------|------------|
| Variable declaration | Not needed | `int x = 5;` | `int x = 5;` | `let x = 5;` |
| Type binding | Dynamic | Static | Static | Dynamic |
| Variable as | Reference to object | Memory location | Reference (objects) / Value (primitives) | Reference / Value |
| Reassignment to different type | Allowed | Not allowed | Not allowed | Allowed |
| Memory management | Automatic (GC) | Manual | Automatic (GC) | Automatic (GC) |

---

## Memory Tips

- **"Variables are sticky notes, not boxes"** — They label objects, they don't contain them.
- **"Python = Dynamic + Strong"** — Types checked at runtime, but no implicit conversions.
- **"256 is the magic number"** — CPython caches integers from -5 to 256.
- **"= assigns, == compares, is identifies"** — Three levels of checking.
- **"LEGB"** — Local, Enclosing, Global, Built-in (variable lookup order).
- **"Mutable shares, Immutable copies"** — `b = a` with lists shares; with ints, it's safe because ints can't change.

---

## Practice Questions

### Easy

**Q1:** Create variables for your name, age, and favorite programming language. Print them with their types.
```python
# Answer:
name = "Aman"
age = 25
fav_language = "Python"

print(f"Name: {name} (type: {type(name).__name__})")
print(f"Age: {age} (type: {type(age).__name__})")
print(f"Language: {fav_language} (type: {type(fav_language).__name__})")
# Output:
# Name: Aman (type: str)
# Age: 25 (type: int)
# Language: Python (type: str)
```

**Q2:** Swap two variables `a = 10` and `b = 20` using Python's tuple unpacking. Verify with print.
```python
# Answer:
a, b = 10, 20
print(f"Before: a={a}, b={b}")  # Before: a=10, b=20

a, b = b, a
print(f"After: a={a}, b={b}")   # After: a=20, b=10
```

### Medium

**Q3:** Given the list `[10, 20, 30, 40, 50]`, use extended unpacking to get the first element, the last element, and everything in between as a separate list.
```python
# Answer:
data = [10, 20, 30, 40, 50]
first, *middle, last = data

print(f"First: {first}")     # First: 10
print(f"Middle: {middle}")   # Middle: [20, 30, 40]
print(f"Last: {last}")       # Last: 50
```

**Q4:** Demonstrate that `a = b = []` creates a shared reference, while `a, b = [], []` creates independent lists.
```python
# Answer:
# Shared reference
a = b = []
a.append(1)
print(f"Shared — a: {a}, b: {b}")      # a: [1], b: [1]
print(f"Same object? {a is b}")         # True

# Independent objects
a, b = [], []
a.append(1)
print(f"Independent — a: {a}, b: {b}") # a: [1], b: []
print(f"Same object? {a is b}")         # False
```

### Advanced

**Q5:** Write a function that takes any number of key-value pairs as arguments and creates variables in the global namespace. Then demonstrate accessing those variables.
```python
# Answer:
def create_globals(**kwargs):
    for key, value in kwargs.items():
        globals()[key] = value

create_globals(language="Python", version=3.12, is_fun=True)

print(language)   # Python
print(version)    # 3.12
print(is_fun)     # True

# WARNING: Dynamically creating globals is generally bad practice!
# This demonstrates namespace manipulation for interview understanding.
```

**Q6:** Explain and demonstrate the difference between `x += [1]` and `x = x + [1]` when `x` is a list. Show the `id()` before and after each operation.
```python
# Answer:
# Using += (in-place modification via __iadd__)
x = [0]
print(f"Before +=: id={id(x)}, value={x}")
x += [1]
print(f"After  +=: id={id(x)}, value={x}")
# Same id! += modifies the list in-place

print()

# Using + (creates new object via __add__)
y = [0]
print(f"Before +: id={id(y)}, value={y}")
y = y + [1]
print(f"After  +: id={id(y)}, value={y}")
# Different id! + creates a new list

# This matters when other variables reference the same list:
a = [1, 2]
b = a

a += [3]
print(f"After a +=, b = {b}")  # b = [1, 2, 3] — b sees the change!

a = [1, 2]
b = a

a = a + [3]
print(f"After a = a +, b = {b}")  # b = [1, 2] — b doesn't see the change!
```

---

*← [Previous: Python Introduction](01_Python_Introduction.md) | [Next: Data Types →](03_Data_Types.md)*
