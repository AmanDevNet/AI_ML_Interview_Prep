# Chapter 8: Recursion

---

## Topic Overview

| Aspect | Details |
|--------|---------|
| **Definition** | Recursion is when a function **calls itself** to solve a smaller piece of the same problem. |
| **Why It Exists** | Some problems are naturally recursive — trees, nested data, divide-and-conquer algorithms. |
| **Where It Is Used** | Tree traversal, binary search, sorting (mergesort, quicksort), parsing, file system walking. |
| **Real-World Analogy** | Russian nesting dolls — open one, there's a smaller one inside, open that, another smaller one, until you reach the smallest doll (base case). |

---

## Think of It This Way

**"How do you eat an elephant?"** — One bite at a time.

Recursion works the same way. Break a big problem into a smaller version of the same problem, and keep going until it's tiny enough to solve directly.

**Factorial example in plain English:**
- What is 5! (5 factorial)?
- It's 5 × 4!
- What's 4!? It's 4 × 3!
- What's 3!? It's 3 × 2!
- What's 2!? It's 2 × 1!
- What's 1!? It's just **1**. STOP. (This is the **base case**)
- Now work backwards: 2×1=2, 3×2=6, 4×6=24, 5×24=**120**

---

## Two Rules of Recursion

Every recursive function needs:
1. **Base case** — when to STOP (without this → infinite loop → crash)
2. **Recursive case** — call yourself with a SMALLER input

```python
def factorial(n):
    # BASE CASE — stop here!
    if n <= 1:
        return 1

    # RECURSIVE CASE — smaller problem
    return n * factorial(n - 1)

print(factorial(5))   # 120
print(factorial(0))   # 1
```

### What Happens in Memory (The Call Stack)

```
factorial(5)
  → 5 * factorial(4)
    → 4 * factorial(3)
      → 3 * factorial(2)
        → 2 * factorial(1)
          → returns 1        ← BASE CASE HIT!
        → returns 2 * 1 = 2
      → returns 3 * 2 = 6
    → returns 4 * 6 = 24
  → returns 5 * 24 = 120
```

Each call creates a new frame on the **call stack**. When it returns, the frame is removed.

---

## Classic Examples

### Sum of a List
```python
def sum_list(lst):
    if not lst:           # base case: empty list
        return 0
    return lst[0] + sum_list(lst[1:])   # first element + sum of rest

print(sum_list([1, 2, 3, 4, 5]))   # 15
```

### Fibonacci (with memoization)
```python
# Naive — SLOW! O(2^n) time
def fib(n):
    if n < 2:
        return n
    return fib(n-1) + fib(n-2)

# With memoization — FAST! O(n) time
from functools import lru_cache

@lru_cache(maxsize=None)
def fib_fast(n):
    if n < 2:
        return n
    return fib_fast(n-1) + fib_fast(n-2)

print(fib_fast(100))   # 354224848179261915075 — instant!
```

### Flatten Nested List
```python
def flatten(lst):
    result = []
    for item in lst:
        if isinstance(item, list):
            result.extend(flatten(item))   # recurse into nested list
        else:
            result.append(item)
    return result

print(flatten([1, [2, [3, 4], 5], [6, 7]]))
# [1, 2, 3, 4, 5, 6, 7]
```

### Binary Search
```python
def binary_search(lst, target, low=0, high=None):
    if high is None:
        high = len(lst) - 1
    if low > high:
        return -1             # base case: not found
    mid = (low + high) // 2
    if lst[mid] == target:
        return mid            # base case: found!
    elif lst[mid] < target:
        return binary_search(lst, target, mid + 1, high)
    else:
        return binary_search(lst, target, low, mid - 1)

nums = [1, 3, 5, 7, 9, 11, 13]
print(binary_search(nums, 7))    # 3 (index)
print(binary_search(nums, 4))    # -1 (not found)
```

---

## Recursion Limit in Python

```python
import sys
print(sys.getrecursionlimit())   # 1000 (default)

# You can increase it (but be careful)
sys.setrecursionlimit(5000)

# If you go too deep → RecursionError
```

**Why the limit?** Each call adds a frame to the stack. Too many frames = stack overflow. Python does NOT optimize tail recursion.

---

## Recursion vs Iteration

```python
# RECURSIVE factorial
def fact_recursive(n):
    if n <= 1: return 1
    return n * fact_recursive(n - 1)

# ITERATIVE factorial (same result, no stack frames)
def fact_iterative(n):
    result = 1
    for i in range(2, n + 1):
        result *= i
    return result
```

| Aspect | Recursion | Iteration |
|--------|-----------|-----------|
| Readability | Cleaner for trees/nested problems | Cleaner for simple loops |
| Memory | Each call = new stack frame | Constant memory |
| Speed | Function call overhead | Generally faster |
| Risk | RecursionError if too deep | No stack limit |
| Use when | Trees, nested data, divide-and-conquer | Simple sequences, large inputs |

---

## Interview Questions (Top 5)

**Q1: What is recursion?**
> A function that calls itself to solve smaller subproblems. Needs a base case (to stop) and a recursive case (to progress toward the base case).

**Q2: What is the recursion limit in Python?**
> Default is 1000. Change with `sys.setrecursionlimit()`. Python doesn't optimize tail calls, so deep recursion can crash.

**Q3: What is memoization?**
> Caching results of function calls to avoid recomputing. `@lru_cache` turns exponential Fibonacci O(2^n) into linear O(n).

**Q4: When should you NOT use recursion?**
> When depth could exceed ~1000, or when an iterative solution is simpler. Convert to iteration using an explicit stack if needed.

**Q5: Convert recursive Fibonacci to iterative.**
```python
def fib_iterative(n):
    if n < 2: return n
    a, b = 0, 1
    for _ in range(2, n + 1):
        a, b = b, a + b
    return b
```

---

*← [Previous: Chapter 7 (Lambda & Closures)](07_Lambda_Closures.md) | [Next: Chapter 9 (Regex) →](09_Regex.md)*
