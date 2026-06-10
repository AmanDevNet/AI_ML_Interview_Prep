# Chapter 13: Range Function

---

## Topic Overview

| Aspect | Details |
|--------|---------|
| **Definition** | `range()` generates an **immutable sequence of numbers** — it's a lazy, memory-efficient sequence type. |
| **Why It Exists** | Provides an efficient way to represent arithmetic sequences without storing all values in memory. |
| **Where It Is Used** | For loops, index generation, creating number sequences, slicing proxies. |
| **Real-World Analogy** | Like a **recipe** for making numbers — it doesn't store them all, just knows how to produce each one on demand. |

### How It Works Internally
- `range` is a **lazy sequence** — it doesn't create a list of numbers.
- It stores only `start`, `stop`, and `step`, and computes values on demand.
- Memory usage is **O(1)** regardless of range size (not O(n)!).
- Supports O(1) membership testing, indexing, slicing, and length.

```python
import sys

# Memory comparison
lst = list(range(1_000_000))
rng = range(1_000_000)

print(f"List: {sys.getsizeof(lst):,} bytes")  # ~8,000,056 bytes
print(f"Range: {sys.getsizeof(rng)} bytes")    # ~48 bytes!
```

---

## Syntax

```python
range(stop)                # 0 to stop-1
range(start, stop)         # start to stop-1
range(start, stop, step)   # start to stop-1, increment by step
```

### Examples
```python
# range(stop)
list(range(5))         # [0, 1, 2, 3, 4]
list(range(0))         # []
list(range(-5))        # [] (stop < default start of 0)

# range(start, stop)
list(range(1, 6))      # [1, 2, 3, 4, 5]
list(range(5, 10))     # [5, 6, 7, 8, 9]
list(range(3, 3))      # [] (start == stop)

# range(start, stop, step)
list(range(0, 10, 2))  # [0, 2, 4, 6, 8]
list(range(1, 10, 3))  # [1, 4, 7]
list(range(10, 0, -1)) # [10, 9, 8, 7, 6, 5, 4, 3, 2, 1]
list(range(5, -1, -1)) # [5, 4, 3, 2, 1, 0]
list(range(0, 10, -1)) # [] (can't count up with negative step)

# Step cannot be 0
# range(0, 10, 0)      # ValueError: range() arg 3 must not be zero
```

---

## Key Properties

### O(1) Membership Testing
```python
# This does NOT iterate — it uses arithmetic!
print(999_999_999 in range(1_000_000_000))  # True (instant!)
# Checks: start <= value < stop and (value - start) % step == 0

import time
start = time.time()
_ = 999_999_999 in range(1_000_000_000)
print(f"Range `in`: {time.time() - start:.8f}s")  # Nearly instant

start = time.time()
_ = 999_999_999 in list(range(1_000_000_000))  # DON'T DO THIS (creates huge list)
```

### Indexing and Slicing
```python
r = range(0, 100, 3)

print(r[0])      # 0
print(r[5])      # 15
print(r[-1])     # 99
print(len(r))    # 34

# Slicing returns a new range!
print(r[2:5])    # range(6, 15, 3)
print(list(r[2:5]))  # [6, 9, 12]
```

### Equality
```python
# Ranges are equal if they produce the same sequence
print(range(10) == range(0, 10))       # True
print(range(0) == range(5, 5))         # True (both empty)
print(range(1, 10, 2) == range(1, 10, 2))  # True
```

---

## Common Patterns

```python
# Loop n times
for _ in range(5):
    print("Hello!")

# Index-based iteration
lst = ["a", "b", "c"]
for i in range(len(lst)):
    print(f"{i}: {lst[i]}")

# Better: use enumerate
for i, val in enumerate(lst):
    print(f"{i}: {val}")

# Generate even numbers
evens = list(range(0, 20, 2))

# Countdown
for i in range(10, 0, -1):
    print(i, end=" ")

# Create a list of zeros (alternative to [0]*n)
zeros = [0 for _ in range(10)]
```

---

## Interview Questions

**Q1: Why is `in` O(1) for range but O(n) for lists?**
> `range` doesn't store values — it uses arithmetic: `start <= value < stop and (value - start) % step == 0`. Lists must scan through each element.

**Q2: Does `range()` create a list?**
> No. `range()` returns a `range` object — a lazy sequence. It stores only `start`, `stop`, `step` and computes values on demand. Use `list(range(...))` to create a list.

**Q3: Can range handle floats?**
> No. `range()` only works with integers. For float ranges, use `numpy.arange()` or a generator: `def frange(start, stop, step): ...`

---

## Code Examples

### Easy (5)
```python
# 1-5 already covered in syntax section above

# Practical: Generate multiplication table for 7
for i in range(1, 11):
    print(f"7 × {i} = {7*i}")
```

### Medium (5)
```python
# 6: Float range generator
def frange(start, stop, step):
    while start < stop:
        yield round(start, 10)
        start += step
print(list(frange(0, 1, 0.1)))

# 7: Range as memory-efficient way to check bounds
def is_valid_index(lst, idx):
    return idx in range(len(lst))

# 8: Create matrix with range
matrix = [list(range(i*5+1, i*5+6)) for i in range(3)]
print(matrix)  # [[1,2,3,4,5], [6,7,8,9,10], [11,12,13,14,15]]
```

### Advanced (5)
```python
# Custom range class
class FloatRange:
    def __init__(self, start, stop, step=1.0):
        self.start, self.stop, self.step = start, stop, step
    def __iter__(self):
        current = self.start
        while (self.step > 0 and current < self.stop) or \
              (self.step < 0 and current > self.stop):
            yield round(current, 10)
            current += self.step
    def __len__(self):
        import math
        return max(0, math.ceil((self.stop - self.start) / self.step))
    def __contains__(self, value):
        if self.step > 0:
            return self.start <= value < self.stop and \
                   abs((value - self.start) % self.step) < 1e-9
        return False

fr = FloatRange(0, 1, 0.15)
print(list(fr))
print(0.3 in fr)
```

---

## Practice Questions

### Easy
**Q1:** Generate a list of all odd numbers between 1 and 50.
```python
odds = list(range(1, 51, 2))
print(odds)
```

**Q2:** Print numbers from 20 down to 1.
```python
for i in range(20, 0, -1):
    print(i, end=" ")
```

### Medium
**Q3:** Create a function that generates a range of floats.
```python
def float_range(start, stop, step):
    result = []
    current = start
    while current < stop:
        result.append(round(current, 10))
        current += step
    return result
print(float_range(0.0, 1.0, 0.2))
```

**Q4:** Prove that `range` membership is O(1) by timing it with increasing range sizes.
```python
import time
for size in [10**3, 10**6, 10**9, 10**12]:
    target = size - 1
    start = time.time()
    _ = target in range(size)
    elapsed = time.time() - start
    print(f"range({size:>15,}): {elapsed:.8f}s")
# All should be nearly identical (O(1))
```

### Advanced
**Q5:** Implement a custom `Range` class that supports float step values and O(1) `__contains__`.
```python
# See the FloatRange class in advanced examples above
```

**Q6:** Explain why `range(0, 10, 0)` raises ValueError and implement a safe version.
```python
def safe_range(start, stop, step):
    if step == 0:
        raise ValueError("step must not be zero")
    if (step > 0 and start >= stop) or (step < 0 and start <= stop):
        return iter([])  # Empty
    return range(start, stop, step)

print(list(safe_range(0, 10, 2)))   # [0, 2, 4, 6, 8]
print(list(safe_range(10, 0, -2)))  # [10, 8, 6, 4, 2]
print(list(safe_range(0, 10, -1)))  # [] (empty)
```

---

*← [Previous: Loops](12_Loops.md) | [Next: List Comprehension →](14_List_Comprehension.md)*
