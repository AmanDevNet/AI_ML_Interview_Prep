# Chapter 14: List Comprehension

---

## Topic Overview

| Aspect | Details |
|--------|---------|
| **Definition** | A concise, Pythonic syntax for creating lists (also dicts, sets, generators) from iterables with optional filtering. |
| **Why It Exists** | Replaces verbose `for` loops with a single expressive line. More readable, often faster (optimized C loop internally). |
| **Where It Is Used** | Data transformation, filtering, matrix operations, feature engineering in ML, data cleaning. |
| **Real-World Analogy** | Like an **assembly line** with a filter — items go in, get transformed, bad ones are rejected, and the result comes out. |

### How It Works Internally
- Comprehensions compile to **specialized bytecode** that is faster than equivalent for loops.
- They avoid the overhead of repeated `list.append()` calls.
- A new **scope** is created for the loop variable (Python 3) — it doesn't leak.

### Performance
```python
import time

# For loop
start = time.time()
result1 = []
for i in range(1_000_000):
    result1.append(i ** 2)
loop_time = time.time() - start

# Comprehension
start = time.time()
result2 = [i ** 2 for i in range(1_000_000)]
comp_time = time.time() - start

print(f"Loop: {loop_time:.3f}s")
print(f"Comp: {comp_time:.3f}s")
# Comprehension is typically 20-30% faster
```

---

## 14.1 Basic List Comprehension

```python
# Syntax: [expression for item in iterable]

# Squares
squares = [x**2 for x in range(10)]
# [0, 1, 4, 9, 16, 25, 36, 49, 64, 81]

# String operations
names = ["alice", "bob", "charlie"]
upper_names = [name.upper() for name in names]
# ['ALICE', 'BOB', 'CHARLIE']

# Type conversions
str_nums = ["1", "2", "3", "4"]
nums = [int(x) for x in str_nums]
# [1, 2, 3, 4]

# Calling functions
lengths = [len(name) for name in names]
# [5, 3, 7]
```

---

## 14.2 Conditional Comprehension

```python
# Syntax: [expression for item in iterable if condition]

# Filter even numbers
evens = [x for x in range(20) if x % 2 == 0]
# [0, 2, 4, 6, 8, 10, 12, 14, 16, 18]

# Filter and transform
long_names = [name.title() for name in ["alice", "bob", "charlie", "dave"] if len(name) > 3]
# ['Alice', 'Charlie', 'Dave']

# if-else in expression (ternary)
labels = ["even" if x % 2 == 0 else "odd" for x in range(6)]
# ['even', 'odd', 'even', 'odd', 'even', 'odd']

# NOTE: 'if' placement matters:
# [expr for x in iterable if cond]     ← filter (no else!)
# [expr_if if cond else expr_else for x in iterable]  ← transform (must have else!)

# Multiple conditions
filtered = [x for x in range(100) if x % 2 == 0 if x % 3 == 0]
# Same as: if x % 2 == 0 and x % 3 == 0
# [0, 6, 12, 18, 24, 30, 36, 42, 48, 54, 60, 66, 72, 78, 84, 90, 96]
```

---

## 14.3 Nested Comprehension

```python
# Syntax: [expression for outer in iterable1 for inner in iterable2]

# Flatten 2D list
matrix = [[1,2,3], [4,5,6], [7,8,9]]
flat = [val for row in matrix for val in row]
# [1, 2, 3, 4, 5, 6, 7, 8, 9]

# Equivalent for loop:
flat = []
for row in matrix:
    for val in row:
        flat.append(val)

# Cartesian product
colors = ["red", "blue"]
sizes = ["S", "M", "L"]
combos = [(color, size) for color in colors for size in sizes]
# [('red','S'), ('red','M'), ('red','L'), ('blue','S'), ('blue','M'), ('blue','L')]

# Create 2D matrix
matrix = [[i * 5 + j for j in range(5)] for i in range(3)]
# [[0,1,2,3,4], [5,6,7,8,9], [10,11,12,13,14]]

# Matrix transpose
original = [[1,2,3], [4,5,6], [7,8,9]]
transposed = [[row[i] for row in original] for i in range(3)]
# [[1,4,7], [2,5,8], [3,6,9]]

# Nested with condition
pairs = [(x, y) for x in range(5) for y in range(5) if x != y]
```

---

## Other Comprehension Types

### Dict Comprehension
```python
# {key_expr: value_expr for item in iterable}
squares = {x: x**2 for x in range(6)}
# {0: 0, 1: 1, 2: 4, 3: 9, 4: 16, 5: 25}

# Invert dict
original = {"a": 1, "b": 2, "c": 3}
inverted = {v: k for k, v in original.items()}
# {1: 'a', 2: 'b', 3: 'c'}

# Filter dict
scores = {"Alice": 85, "Bob": 62, "Charlie": 91}
passed = {k: v for k, v in scores.items() if v >= 70}
# {'Alice': 85, 'Charlie': 91}
```

### Set Comprehension
```python
# {expr for item in iterable}
unique_lengths = {len(word) for word in ["hello", "world", "hi", "hey"]}
# {2, 3, 5}
```

### Generator Expression
```python
# (expr for item in iterable) — lazy, memory-efficient
gen = (x**2 for x in range(1_000_000))
print(type(gen))  # <class 'generator'>

# Use in functions that accept iterables
total = sum(x**2 for x in range(100))  # No extra brackets needed!
```

---

## Common Mistakes

| Mistake | Example | Fix |
|---------|---------|-----|
| Missing `else` in ternary | `[x if x > 0 for x in lst]` | `[x if x > 0 else 0 for x in lst]` or `[x for x in lst if x > 0]` |
| Too complex/unreadable | Triple-nested with conditions | Use regular for loops for complex logic |
| Variable leak (Python 2) | `x` leaks from comprehension | Not an issue in Python 3 |
| Using comprehension for side effects | `[print(x) for x in lst]` | Use a for loop instead |

---

## Interview Questions

**Q1: What is the difference between a list comprehension and a generator expression?**
> List comprehension `[...]` creates the entire list in memory. Generator expression `(...)` is lazy — it produces values one at a time, saving memory for large datasets.

**Q2: Are list comprehensions faster than for loops?**
> Generally yes, ~20-30% faster. They compile to optimized bytecode that avoids the overhead of `list.append()` method lookups and calls.

**Q3: Does the loop variable leak in Python 3?**
> No. In Python 3, comprehensions have their own scope. `[x for x in range(5)]` doesn't affect an outer variable named `x`.

---

## Code Examples

### Easy (5)
```python
# 1: Squares of 1-10
squares = [x**2 for x in range(1, 11)]

# 2: Uppercase words
words = ["hello", "world"]
upper = [w.upper() for w in words]

# 3: Even numbers filtered
evens = [x for x in range(1, 21) if x % 2 == 0]

# 4: Lengths of strings
words = ["Python", "is", "awesome"]
lengths = [len(w) for w in words]

# 5: Convert to float
data = ["1.5", "2.7", "3.9"]
floats = [float(x) for x in data]
```

### Medium (5)
```python
# 6: Flatten nested list
nested = [[1,2], [3,4], [5,6]]
flat = [x for sublist in nested for x in sublist]

# 7: Matrix creation
identity = [[1 if i == j else 0 for j in range(4)] for i in range(4)]

# 8: Filter and transform
data = [("Alice", 85), ("Bob", 62), ("Charlie", 91)]
honors = [name for name, score in data if score >= 80]

# 9: Pythagorean triples
triples = [(a,b,c) for a in range(1,30) for b in range(a,30) 
           for c in range(b,30) if a**2 + b**2 == c**2]
print(triples[:5])

# 10: Word frequency
text = "the cat sat on the mat"
freq = {word: text.split().count(word) for word in set(text.split())}
```

### Advanced (5)
```python
# 11: Prime sieve with comprehension
def primes_up_to(n):
    composites = {j for i in range(2, int(n**0.5)+1) for j in range(i*i, n+1, i)}
    return [x for x in range(2, n+1) if x not in composites]
print(primes_up_to(50))

# 12: Nested dict comprehension
students = {"Alice": [85, 90, 92], "Bob": [78, 82, 80]}
averages = {name: sum(scores)/len(scores) for name, scores in students.items()}

# 13: Walrus operator in comprehension
results = [y for x in range(20) if (y := x**2) > 50 and y < 200]
print(results)  # [64, 81, 100, 121, 144, 169, 196]

# 14: Generator pipeline
import sys
# Memory-efficient pipeline
nums = range(10_000_000)
total = sum(x**2 for x in nums if x % 2 == 0)
# Uses almost no memory — everything is lazy!

# 15: Decode nested JSON-like structure
data = {
    "users": [
        {"name": "Alice", "tags": ["admin", "user"]},
        {"name": "Bob", "tags": ["user"]},
        {"name": "Charlie", "tags": ["admin", "moderator"]},
    ]
}
admins = [u["name"] for u in data["users"] if "admin" in u["tags"]]
all_tags = {tag for u in data["users"] for tag in u["tags"]}
print(f"Admins: {admins}")
print(f"All tags: {all_tags}")
```

---

## Practice Questions

### Easy
**Q1:** Create a list of cubes of even numbers from 1 to 20.
```python
cubes = [x**3 for x in range(1, 21) if x % 2 == 0]
print(cubes)
```

**Q2:** Given a list of words, create a new list containing only words with more than 4 characters.
```python
words = ["Python", "is", "an", "amazing", "programming", "language"]
long_words = [w for w in words if len(w) > 4]
print(long_words)
```

### Medium
**Q3:** Flatten a 3D list into a 1D list using comprehension.
```python
cube = [[[1,2], [3,4]], [[5,6], [7,8]]]
flat = [val for layer in cube for row in layer for val in row]
print(flat)  # [1, 2, 3, 4, 5, 6, 7, 8]
```

**Q4:** Create a dictionary mapping each character in a string to its frequency using dict comprehension.
```python
s = "mississippi"
freq = {c: s.count(c) for c in set(s)}
print(freq)
```

### Advanced
**Q5:** Generate all valid Sudoku pairs (row, col) where neither is the same using comprehension.
```python
pairs = [(r, c) for r in range(9) for c in range(9)]
print(f"Total cells: {len(pairs)}")
same_box = [(r1,c1,r2,c2) for r1 in range(9) for c1 in range(9)
            for r2 in range(9) for c2 in range(9)
            if (r1,c1) != (r2,c2) and r1//3 == r2//3 and c1//3 == c2//3]
print(f"Same-box pairs: {len(same_box)}")
```

**Q6:** Compare memory usage and speed of list comprehension, generator expression, and for loop for summing squares of 10 million numbers.
```python
import time, sys

n = 10_000_000

# List comprehension
start = time.time()
lst = [x**2 for x in range(n)]
total1 = sum(lst)
t1 = time.time() - start
m1 = sys.getsizeof(lst)

# Generator expression
start = time.time()
total2 = sum(x**2 for x in range(n))
t2 = time.time() - start

# For loop
start = time.time()
total3 = 0
for x in range(n):
    total3 += x**2
t3 = time.time() - start

print(f"List comp: {t1:.3f}s, {m1:,} bytes")
print(f"Generator: {t2:.3f}s, ~200 bytes")
print(f"For loop:  {t3:.3f}s")
print(f"All equal: {total1 == total2 == total3}")
```

---

*← [Previous: Range Function](13_Range_Function.md) | [Next: Functions →](15_Functions.md)*
