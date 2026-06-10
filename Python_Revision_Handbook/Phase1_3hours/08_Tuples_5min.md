# Chapter 8: Tuples

---

## Topic Overview

| Aspect | Details |
|--------|---------|
| **Definition** | A tuple is an **ordered, immutable, heterogeneous** sequence enclosed in parentheses `()`. |
| **Why It Exists** | Tuples provide an immutable sequence for data that should not change — function return values, dictionary keys, record-like structures. |
| **Where It Is Used** | Function returns, dict keys, named records, database rows, coordinates, constant sequences. |
| **Real-World Analogy** | A tuple is like a **sealed envelope** — once you put items in and seal it, you can read them but can't change them. |

### How It Works Internally
- Tuples are stored as **fixed-size arrays of pointers** (similar to lists but without over-allocation).
- No extra space is allocated — the memory footprint is smaller than lists.
- Immutability allows Python to **optimize**: tuples can be cached, hashed, and shared safely.
- CPython caches small empty tuples and reuses them.

### Memory Comparison
```python
import sys
lst = [1, 2, 3, 4, 5]
tup = (1, 2, 3, 4, 5)
print(f"List:  {sys.getsizeof(lst)} bytes")  # ~96 bytes
print(f"Tuple: {sys.getsizeof(tup)} bytes")  # ~80 bytes
# Tuples are smaller because no over-allocation needed
```

### Time Complexity
| Operation | Tuple | List |
|-----------|-------|------|
| Create | O(n) | O(n) |
| Index `t[i]` | O(1) | O(1) |
| Iteration | O(n) | O(n) |
| `in` | O(n) | O(n) |
| `len()` | O(1) | O(1) |
| `count()` | O(n) | O(n) |
| `index()` | O(n) | O(n) |
| Hashable | ✅ (if all elements hashable) | ❌ |

---

## 8.1 Creation

```python
# Parentheses
t = (1, 2, 3)

# Without parentheses (packing)
t = 1, 2, 3          # Same as (1, 2, 3)

# Single element — MUST have trailing comma!
single = (42,)        # tuple with one element
not_tuple = (42)      # This is just int 42 with parentheses!
print(type(single))   # <class 'tuple'>
print(type(not_tuple))# <class 'int'>

# Empty tuple
empty = ()
empty = tuple()

# From other iterables
from_list = tuple([1, 2, 3])      # (1, 2, 3)
from_string = tuple("hello")      # ('h', 'e', 'l', 'l', 'o')
from_range = tuple(range(5))      # (0, 1, 2, 3, 4)

# Nested tuples
nested = (1, (2, 3), (4, (5, 6)))
print(nested[1][0])   # 2
print(nested[2][1][1])# 6
```

---

## 8.2 Packing

```python
# Packing: combining values into a tuple
coordinates = 3, 4, 5    # Packing
print(coordinates)        # (3, 4, 5)
print(type(coordinates))  # <class 'tuple'>

# Function returning multiple values uses packing
def get_stats(numbers):
    return min(numbers), max(numbers), sum(numbers) / len(numbers)

result = get_stats([10, 20, 30])
print(result)        # (10, 30, 20.0)
print(type(result))  # <class 'tuple'>
```

---

## 8.3 Unpacking

```python
# Basic unpacking
x, y, z = (10, 20, 30)
print(x, y, z)  # 10 20 30

# Extended unpacking (star)
first, *rest = (1, 2, 3, 4, 5)
print(first)  # 1
print(rest)   # [2, 3, 4, 5] — note: rest is a list!

*beginning, last = (1, 2, 3, 4, 5)
print(beginning)  # [1, 2, 3, 4]
print(last)       # 5

# Swap using unpacking
a, b = 10, 20
a, b = b, a  # Swap!

# Ignore values
_, important, _ = (1, 42, 3)
print(important)  # 42

# Nested unpacking
(a, b), (c, d) = (1, 2), (3, 4)
print(a, b, c, d)  # 1 2 3 4

# Unpacking in for loops
pairs = [(1, "a"), (2, "b"), (3, "c")]
for num, letter in pairs:
    print(f"{num}: {letter}")
```

---

## 8.4 Immutability

```python
t = (1, 2, 3)

# Cannot modify elements
# t[0] = 10  # TypeError: 'tuple' object does not support item assignment

# Cannot add or remove elements
# t.append(4)  # AttributeError: 'tuple' object has no attribute 'append'

# BUT: mutable elements INSIDE a tuple CAN be modified!
t = (1, [2, 3], 4)
t[1].append(5)
print(t)  # (1, [2, 3, 5], 4) — the list inside changed!
# The tuple still references the same list object — that doesn't change.

# This means:
# - The tuple's structure (which objects it references) is immutable
# - The objects themselves may be mutable
```

### Hashability
```python
# Tuples of immutables are hashable → can be dict keys
point = (3, 4)
distances = {(0, 0): 0, (3, 4): 5, (1, 1): 1.41}
print(distances[(3, 4)])  # 5

# Tuple containing mutable element is NOT hashable
t = (1, [2, 3])
# hash(t)  # TypeError: unhashable type: 'list'
```

---

## 8.5 Tuple Methods

Tuples have only **2 methods** (because they're immutable):

```python
t = (1, 2, 3, 2, 4, 2)

# count() — count occurrences
print(t.count(2))    # 3
print(t.count(5))    # 0

# index() — find first occurrence
print(t.index(2))    # 1 (first occurrence)
print(t.index(2, 2)) # 3 (search from index 2)
# t.index(99)        # ValueError
```

---

## Tuple vs List

| Feature | Tuple | List |
|---------|-------|------|
| Syntax | `(1, 2, 3)` | `[1, 2, 3]` |
| Mutable | ❌ | ✅ |
| Hashable | ✅ (if elements hashable) | ❌ |
| Can be dict key | ✅ | ❌ |
| Methods | 2 (`count`, `index`) | 11+ |
| Memory | Less (no over-allocation) | More |
| Speed | Slightly faster to create/access | Slightly slower |
| Use case | Fixed data, records | Dynamic collections |
| Iteration | Same speed | Same speed |

### When to Use Tuples vs Lists
- **Tuple**: Data that shouldn't change (coordinates, RGB values, database rows, function returns, dict keys).
- **List**: Data that needs to grow/shrink/change (user input, processing queues, results).

---

## Named Tuples (Bonus)

```python
from collections import namedtuple

# Define a named tuple type
Point = namedtuple("Point", ["x", "y"])

p = Point(3, 4)
print(p.x, p.y)     # 3 4 (access by name)
print(p[0], p[1])   # 3 4 (access by index)
print(p)             # Point(x=3, y=4)

# Immutable like regular tuples
# p.x = 10  # AttributeError

# Named tuple is a subclass of tuple
print(isinstance(p, tuple))  # True

# Replace (creates new instance)
p2 = p._replace(x=10)
print(p2)  # Point(x=10, y=4)

# Convert to dict
print(p._asdict())  # {'x': 3, 'y': 4}

# Real-world example
Student = namedtuple("Student", "name age grade")
s = Student("Alice", 20, "A")
print(f"{s.name} got grade {s.grade}")
```

---

## Common Mistakes

| Mistake | Example | Fix |
|---------|---------|-----|
| Single element tuple | `t = (42)` → int, not tuple | `t = (42,)` |
| Trying to modify | `t[0] = 1` → TypeError | Create new tuple: `t = (1,) + t[1:]` |
| Mutable inside tuple | `(1, [2]).count(...)` fine, but `hash((1, [2]))` fails | Don't use tuples with mutable elements as keys |
| Forgetting comma in function return | `return value` vs `return value,` | `return (val1, val2)` explicitly |

---

## Interview Questions

**Q1: Why are tuples faster than lists?**
> Tuples are faster to create (no over-allocation), smaller in memory, and can be cached by Python. CPython also optimizes tuple operations since it knows they won't change.

**Q2: Can a tuple be a dictionary key?**
> Yes, if all its elements are hashable (immutable). `{(1,2): "point"}` is valid. `{(1, [2]): "bad"}` raises `TypeError`.

**Q3: What is the difference between `(42)` and `(42,)`?**
> `(42)` is just the integer `42` in parentheses. `(42,)` is a tuple containing one element. The comma makes the tuple, not the parentheses.

**Q4: If tuples are immutable, how can `t = (1, [2, 3])` allow `t[1].append(4)`?**
> The tuple is immutable in terms of its references — it always points to the same list object. But the list itself is mutable. The tuple doesn't change (same reference), but the referenced object does.

---

## Code Examples

### Easy Examples

```python
# Example 1: Basic tuple operations
coords = (3, 4, 5)
print(f"x={coords[0]}, y={coords[1]}, z={coords[2]}")
print(f"Length: {len(coords)}")

# Example 2: Tuple unpacking
name, age, city = ("Alice", 25, "NYC")
print(f"{name} is {age}, lives in {city}")

# Example 3: Tuple as function return
def divmod_custom(a, b):
    return a // b, a % b  # Returns tuple

quotient, remainder = divmod_custom(17, 5)
print(f"17 ÷ 5 = {quotient} remainder {remainder}")

# Example 4: Tuple comparison
print((1, 2, 3) < (1, 2, 4))   # True (element-by-element)
print((1, 2, 3) < (1, 3, 0))   # True (2 < 3)
print((1, 2) < (1, 2, 0))      # True (shorter is "less")

# Example 5: Tuple as dict key
locations = {
    (40.7128, -74.0060): "New York",
    (51.5074, -0.1278): "London",
    (35.6762, 139.6503): "Tokyo",
}
print(locations[(40.7128, -74.0060)])  # New York
```

### Medium Examples

```python
# Example 6: Tuple concatenation
t1 = (1, 2, 3)
t2 = (4, 5, 6)
t3 = t1 + t2      # (1, 2, 3, 4, 5, 6) — new tuple
t4 = t1 * 3        # (1, 2, 3, 1, 2, 3, 1, 2, 3)

# Example 7: Sorting list of tuples
students = [("Charlie", 85), ("Alice", 92), ("Bob", 78)]
# Sort by name
print(sorted(students))
# Sort by grade
print(sorted(students, key=lambda x: x[1], reverse=True))

# Example 8: Named tuple for readable code
from collections import namedtuple

Color = namedtuple("Color", ["red", "green", "blue"])
white = Color(255, 255, 255)
black = Color(red=0, green=0, blue=0)
print(f"White RGB: ({white.red}, {white.green}, {white.blue})")

# Example 9: Tuple with mutable element
t = (1, [2, 3], 4)
print(id(t[1]))
t[1].append(5)     # OK — modifying the list, not the tuple
print(t)            # (1, [2, 3, 5], 4)
print(id(t[1]))     # Same id — tuple still references same list

# But augmented assignment on tuple element fails:
try:
    t[1] += [6]     # This BOTH succeeds AND fails!
except TypeError as e:
    print(f"Error: {e}")
    print(f"But t is now: {t}")  # (1, [2, 3, 5, 6], 4) — list WAS modified!
# __iadd__ modifies the list, THEN assignment to t[1] fails.

# Example 10: enumerate returns tuples
for i, char in enumerate("hello"):
    print(f"({i}, '{char}')", end=" ")
# (0, 'h') (1, 'e') (2, 'l') (3, 'l') (4, 'o')
```

### Advanced Examples

```python
# Example 11: Tuple caching in CPython
# CPython caches the empty tuple
a = ()
b = ()
print(a is b)  # True (same object — cached)

# Small tuples may also be cached
a = (1, 2, 3)
b = (1, 2, 3)
print(a is b)  # May be True in CPython (constant folding)

# Example 12: Tuple as a lightweight record type
from typing import NamedTuple

class Employee(NamedTuple):
    name: str
    age: int
    department: str
    salary: float

emp = Employee("Alice", 30, "Engineering", 95000.0)
print(f"{emp.name} ({emp.department}): ${emp.salary:,.0f}")
print(isinstance(emp, tuple))  # True

# Example 13: Tuple for matrix representation
def matrix_from_tuples(*rows):
    return tuple(tuple(row) for row in rows)

def matrix_multiply(A, B):
    rows_A, cols_A = len(A), len(A[0])
    rows_B, cols_B = len(B), len(B[0])
    assert cols_A == rows_B
    
    result = [[0] * cols_B for _ in range(rows_A)]
    for i in range(rows_A):
        for j in range(cols_B):
            for k in range(cols_A):
                result[i][j] += A[i][k] * B[k][j]
    return matrix_from_tuples(*result)

A = ((1, 2), (3, 4))
B = ((5, 6), (7, 8))
print(matrix_multiply(A, B))  # ((19, 22), (43, 50))

# Example 14: The += TypeError paradox
t = ([1, 2],)
try:
    t[0] += [3, 4]
except TypeError:
    pass
print(t)  # ([1, 2, 3, 4],) — the list WAS modified despite the error!
# This is because:
# 1. t[0].__iadd__([3, 4]) modifies the list (succeeds)
# 2. t[0] = <result> tries to assign back (fails — tuple is immutable)

# Example 15: Using tuples for multiple return values with type hints
from typing import Tuple

def analyze_data(data: list) -> Tuple[float, float, float, int]:
    """Returns (mean, median, std_dev, count)."""
    n = len(data)
    mean = sum(data) / n
    sorted_data = sorted(data)
    median = sorted_data[n // 2] if n % 2 else (sorted_data[n//2-1] + sorted_data[n//2]) / 2
    variance = sum((x - mean) ** 2 for x in data) / n
    std_dev = variance ** 0.5
    return mean, median, std_dev, n

mean, median, std_dev, count = analyze_data([10, 20, 30, 40, 50])
print(f"Mean: {mean}, Median: {median}, StdDev: {std_dev:.2f}, Count: {count}")
```

---

## Edge Cases

1. **Empty tuple**: `()` — falsy, `len()` = 0, hashable.
2. **Single-element tuple needs comma**: `(42,)` not `(42)`.
3. **`t[0] += [1]` paradox**: List inside tuple gets modified, but assignment raises `TypeError`.
4. **Tuple comparison**: `(1, 2) < (1, 3)` compares element by element.
5. **`tuple()` with generator**: `tuple(x for x in range(5))` → `(0, 1, 2, 3, 4)`.

---

## Practice Questions

### Easy

**Q1:** Create a tuple of 5 colors, print the first and last, and check if "red" is in it.
```python
colors = ("red", "blue", "green", "yellow", "purple")
print(f"First: {colors[0]}, Last: {colors[-1]}")
print(f"'red' in colors: {'red' in colors}")
```

**Q2:** Write a function that returns both the quotient and remainder of two numbers as a tuple.
```python
def divide(a, b):
    return a // b, a % b

q, r = divide(17, 5)
print(f"17 ÷ 5 = {q} remainder {r}")  # 3 remainder 2
```

### Medium

**Q3:** Write a function that groups consecutive duplicate elements in a list into tuples.
```python
# Answer: [1,1,2,3,3,3,4] → [(1,2), (2,1), (3,3), (4,1)] where (value, count)
def group_consecutive(lst):
    if not lst:
        return []
    result = []
    current, count = lst[0], 1
    for item in lst[1:]:
        if item == current:
            count += 1
        else:
            result.append((current, count))
            current, count = item, 1
    result.append((current, count))
    return result

print(group_consecutive([1,1,2,3,3,3,4]))
# [(1, 2), (2, 1), (3, 3), (4, 1)]
```

**Q4:** Demonstrate the `t[0] += [1]` paradox and explain why it both succeeds and fails.
```python
# Answer:
t = ([1, 2],)
print(f"Before: {t}")

try:
    t[0] += [3]  # Raises TypeError but modifies the list!
except TypeError as e:
    print(f"Error: {e}")

print(f"After: {t}")  # ([1, 2, 3],) — list was modified!

# Explanation:
# Step 1: t[0].__iadd__([3]) → modifies list in-place → succeeds
# Step 2: t[0] = <result> → tries to assign to tuple index → TypeError
# The list is already modified when the error occurs!
# Lesson: Never use += on mutable elements inside tuples.
```

### Advanced

**Q5:** Implement a function that generates all permutations of a tuple using recursion.
```python
# Answer:
def permutations(t):
    if len(t) <= 1:
        return [t]
    result = []
    for i, elem in enumerate(t):
        remaining = t[:i] + t[i+1:]
        for perm in permutations(remaining):
            result.append((elem,) + perm)
    return result

perms = permutations((1, 2, 3))
for p in perms:
    print(p)
# (1, 2, 3), (1, 3, 2), (2, 1, 3), (2, 3, 1), (3, 1, 2), (3, 2, 1)
```

**Q6:** Compare the creation time and memory usage of tuples vs lists for large datasets.
```python
# Answer:
import sys
import time

n = 1_000_000

# Creation time
start = time.time()
lst = list(range(n))
list_time = time.time() - start

start = time.time()
tup = tuple(range(n))
tuple_time = time.time() - start

print(f"List creation:  {list_time:.4f}s, Memory: {sys.getsizeof(lst):,} bytes")
print(f"Tuple creation: {tuple_time:.4f}s, Memory: {sys.getsizeof(tup):,} bytes")

# Access time
start = time.time()
for i in range(n):
    _ = lst[i]
list_access = time.time() - start

start = time.time()
for i in range(n):
    _ = tup[i]
tuple_access = time.time() - start

print(f"List access:  {list_access:.4f}s")
print(f"Tuple access: {tuple_access:.4f}s")
# Tuples are generally slightly faster and use less memory.
```

---

*← [Previous: Lists](07_Lists.md) | [Next: Sets →](09_Sets.md)*
