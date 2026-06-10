# Chapter 6: Iterators & Generators

---

## Topic Overview

| Aspect | Details |
|--------|---------|
| **Definition** | An **iterator** produces values one at a time. A **generator** is an easy way to create an iterator using the `yield` keyword. |
| **Why It Exists** | Memory efficiency. Instead of loading 1 million items into memory at once, produce them one at a time. |
| **Where It Is Used** | Reading huge files, streaming data, infinite sequences, data pipelines, ML data loading (PyTorch DataLoader). |
| **Real-World Analogy** | **Iterator**: A vending machine — gives you ONE item each time you press the button. **Generator**: A chef who prepares ONE dish at a time when you order, instead of cooking the entire menu upfront. |

---

## Part 1: Iterators

### What Happens When You Write a `for` Loop?

```python
# When you write this:
for x in [1, 2, 3]:
    print(x)

# Python actually does this behind the scenes:
my_list = [1, 2, 3]
iterator = iter(my_list)       # Step 1: Get an iterator from the list
print(next(iterator))          # Step 2: Get next value → 1
print(next(iterator))          # Step 3: Get next value → 2
print(next(iterator))          # Step 4: Get next value → 3
# next(iterator)               # Step 5: StopIteration! No more values.
```

So a `for` loop is really just calling `next()` over and over until `StopIteration`.

### Iterable vs Iterator — The Difference

```python
my_list = [1, 2, 3]

# ITERABLE: has __iter__() — can PRODUCE iterators
# Lists, strings, dicts, ranges are all iterables

# ITERATOR: has __iter__() AND __next__() — produces values one at a time
# Consumed after one pass!

iterator = iter(my_list)     # Get an iterator from the list

print(list(iterator))  # [1, 2, 3] — consumed!
print(list(iterator))  # [] — empty! Iterator is exhausted.
# You need a NEW iterator to go through again:
iterator = iter(my_list)
print(list(iterator))  # [1, 2, 3] — fresh iterator
```

**Key difference:**
- **Iterable** (list, string) = can be looped over MANY times
- **Iterator** = can be consumed ONCE, then it's empty

### Building Your Own Iterator

```python
class Countdown:
    def __init__(self, start):
        self.start = start

    def __iter__(self):
        self.current = self.start
        return self

    def __next__(self):
        if self.current < 0:
            raise StopIteration      # Signal: no more values
        value = self.current
        self.current -= 1
        return value

# Use it in a for loop
for n in Countdown(5):
    print(n, end=" ")    # 5 4 3 2 1 0

# Use it manually
c = iter(Countdown(3))
print(next(c))   # 3
print(next(c))   # 2
print(next(c))   # 1
print(next(c))   # 0
# next(c)         # StopIteration
```

---

## Part 2: Generators — The Better Way

Writing a whole class with `__iter__` and `__next__` is a lot of work. Generators do the same thing in way less code.

### Your First Generator

```python
def countdown(start):
    while start >= 0:
        yield start        # PAUSE here, give this value, resume later
        start -= 1

# Use exactly like any iterable
for n in countdown(5):
    print(n, end=" ")    # 5 4 3 2 1 0
```

**That's it.** The `yield` keyword is what makes it a generator. No class, no `__iter__`, no `__next__`.

### How `yield` Works (Step by Step)

```python
def simple():
    print("Step 1")
    yield "A"          # Pause here, return "A"
    print("Step 2")
    yield "B"          # Pause here, return "B"
    print("Step 3")    # After last yield

gen = simple()         # Nothing runs yet! Just creates the generator.

print(next(gen))       # Runs Step 1, pauses at yield, returns "A"
# Output: Step 1
#         A

print(next(gen))       # Resumes after yield "A", runs Step 2, returns "B"
# Output: Step 2
#         B

# next(gen)            # Resumes, runs Step 3, function ends → StopIteration
```

**Think of `yield` as a PAUSE button.** The function freezes at `yield`, gives you a value, and waits. Next time you call `next()`, it unfreezes and continues from where it left off.

---

### Generator Expressions (One-Liners)

Just like list comprehensions, but with parentheses:

```python
# List comprehension — creates ALL values at once (uses memory)
squares_list = [x**2 for x in range(1_000_000)]     # ~8 MB in memory

# Generator expression — creates values on demand (tiny memory)
squares_gen = (x**2 for x in range(1_000_000))       # ~200 bytes!

# Generator expressions work in functions that accept iterables
total = sum(x**2 for x in range(100))          # sum directly from generator
has_even = any(x % 2 == 0 for x in [1, 3, 4])  # stops at first True
```

---

## Why Generators Matter — Memory Comparison

```python
import sys

# List: stores ALL values in memory at once
lst = [x for x in range(1_000_000)]
print(f"List:      {sys.getsizeof(lst):>10,} bytes")    # ~8,000,056 bytes

# Generator: stores only the formula, produces values on demand
gen = (x for x in range(1_000_000))
print(f"Generator: {sys.getsizeof(gen):>10,} bytes")    # ~200 bytes

# range is similar — it doesn't store all values either
r = range(1_000_000)
print(f"Range:     {sys.getsizeof(r):>10,} bytes")      # ~48 bytes
```

Generators are essential when:
- Data is too large for memory (reading 10 GB files)
- You only need values one at a time
- You want to build data pipelines

---

## Real-World Examples

### Reading a Huge File

```python
def read_large_file(filepath):
    """Read a file line by line — never loads entire file into memory."""
    with open(filepath, "r") as f:
        for line in f:
            yield line.strip()

# Process a 10 GB log file without crashing
for line in read_large_file("huge_log.txt"):
    if "ERROR" in line:
        print(line)
```

### Infinite Sequence

```python
def fibonacci():
    """Infinite Fibonacci — produces values forever."""
    a, b = 0, 1
    while True:        # Infinite loop is FINE because yield pauses it
        yield a
        a, b = b, a + b

# Get first 10 Fibonacci numbers
fib = fibonacci()
first_10 = [next(fib) for _ in range(10)]
print(first_10)   # [0, 1, 1, 2, 3, 5, 8, 13, 21, 34]

# Get next 5
next_5 = [next(fib) for _ in range(5)]
print(next_5)     # [55, 89, 144, 233, 377]
```

### Data Pipeline

```python
def read_lines(filepath):
    with open(filepath) as f:
        for line in f:
            yield line.strip()

def filter_errors(lines):
    for line in lines:
        if "ERROR" in line:
            yield line

def extract_message(lines):
    for line in lines:
        yield line.split(":")[-1].strip()

# Chain them together — nothing runs until you iterate!
# lines = read_lines("app.log")
# errors = filter_errors(lines)
# messages = extract_message(errors)
# for msg in messages:
#     print(msg)
```

---

## `yield from` — Delegating to Another Generator

```python
def numbers():
    yield from range(3)          # yields 0, 1, 2
    yield from range(10, 13)     # yields 10, 11, 12
    yield from "AB"              # yields "A", "B"

print(list(numbers()))   # [0, 1, 2, 10, 11, 12, 'A', 'B']

# Without yield from, you'd need:
def numbers_manual():
    for n in range(3):
        yield n
    for n in range(10, 13):
        yield n
    for c in "AB":
        yield c
```

---

## Common Mistakes

| Mistake | Why It's Wrong | Fix |
|---------|---------------|-----|
| Trying to reuse an exhausted generator | Generators are consumed after one pass | Create a new generator |
| Using a list when a generator would work | Wastes memory for large datasets | Use `()` instead of `[]` |
| Forgetting generators are lazy | Nothing runs until you iterate | Call `list()` or use in a loop |
| Printing a generator object | Shows `<generator object at 0x...>` | Use `list(gen)` or iterate |

```python
# The "why is my generator empty" mistake
gen = (x**2 for x in range(5))
print(sum(gen))    # 30 — consumed!
print(sum(gen))    # 0  — empty! Already consumed!

# Fix: create a new one each time, or convert to list if small
```

---

## Code Examples

### Easy
```python
# Example 1: Simple generator
def count_up(n):
    i = 1
    while i <= n:
        yield i
        i += 1

print(list(count_up(5)))   # [1, 2, 3, 4, 5]


# Example 2: Generator expression
evens = (x for x in range(20) if x % 2 == 0)
print(list(evens))   # [0, 2, 4, 6, 8, 10, 12, 14, 16, 18]


# Example 3: Using next() with a default
gen = iter([10, 20, 30])
print(next(gen, "done"))   # 10
print(next(gen, "done"))   # 20
print(next(gen, "done"))   # 30
print(next(gen, "done"))   # "done" (no StopIteration!)
```

### Medium
```python
# Example 4: Chunking a list with a generator
def chunks(lst, n):
    """Yield successive n-sized chunks from lst."""
    for i in range(0, len(lst), n):
        yield lst[i:i+n]

data = list(range(1, 11))
for chunk in chunks(data, 3):
    print(chunk)
# [1, 2, 3]
# [4, 5, 6]
# [7, 8, 9]
# [10]


# Example 5: Flatten nested lists
def flatten(nested):
    for item in nested:
        if isinstance(item, list):
            yield from flatten(item)   # recursive yield from
        else:
            yield item

data = [1, [2, [3, 4], 5], [6, 7]]
print(list(flatten(data)))   # [1, 2, 3, 4, 5, 6, 7]
```

### Advanced
```python
# Example 6: Generator-based data pipeline
def integers():
    n = 1
    while True:
        yield n
        n += 1

def squares(nums):
    for n in nums:
        yield n ** 2

def take(n, iterable):
    for i, item in enumerate(iterable):
        if i >= n:
            return
        yield item

# Chain: infinite integers → square each → take first 10
pipeline = take(10, squares(integers()))
print(list(pipeline))  # [1, 4, 9, 16, 25, 36, 49, 64, 81, 100]


# Example 7: send() — sending values INTO a generator
def running_average():
    total = 0
    count = 0
    average = None
    while True:
        value = yield average     # yield current avg, receive new value
        total += value
        count += 1
        average = total / count

avg = running_average()
next(avg)              # "prime" the generator (runs to first yield)
print(avg.send(10))    # 10.0  (avg of [10])
print(avg.send(20))    # 15.0  (avg of [10, 20])
print(avg.send(30))    # 20.0  (avg of [10, 20, 30])
print(avg.send(40))    # 25.0  (avg of [10, 20, 30, 40])
```

---

## Interview Questions (Top 10)

**Q1: What is the difference between an iterable and an iterator?**
> An iterable has `__iter__()` — can produce iterators (list, string, range). An iterator has `__iter__()` AND `__next__()` — produces values one at a time. Iterators are consumed after one pass.

**Q2: What is a generator?**
> A function that uses `yield` instead of `return`. It produces values lazily — one at a time — instead of computing everything at once. Extremely memory-efficient.

**Q3: What is the difference between `yield` and `return`?**
> `return` ends the function and sends ONE value. `yield` pauses the function, sends a value, and remembers where it left off. The function resumes on the next `next()` call.

**Q4: Why are generators memory-efficient?**
> They produce values on demand — only one value exists in memory at a time. A list of 1 million items uses ~8 MB. A generator producing the same values uses ~200 bytes.

**Q5: What is a generator expression?**
> A one-liner generator using parentheses: `(x**2 for x in range(n))`. Like a list comprehension but lazy. Use inside `sum()`, `any()`, `all()`.

**Q6: What is `yield from`?**
> Delegates to another iterable/generator. `yield from range(5)` is equivalent to `for i in range(5): yield i`. Cleaner syntax for sub-generators.

**Q7: What happens when a generator is exhausted?**
> `StopIteration` is raised. You can't reset it — create a new generator instead.

**Q8: Can you use `return` inside a generator?**
> Yes, but it signals the end (raises `StopIteration`). In Python 3, `return value` stores the value in `StopIteration.value`.

**Q9: What is `send()` in generators?**
> `gen.send(value)` sends a value INTO the generator, which becomes the result of the `yield` expression. Used for coroutines and advanced patterns.

**Q10: Write an infinite Fibonacci generator.**
```python
def fibonacci():
    a, b = 0, 1
    while True:
        yield a
        a, b = b, a + b
```

---

*← [Previous: Chapter 5 (Decorators)](05_Decorators.md) | [Next: Chapter 7 (Lambda & Closures) →](07_Lambda_Closures.md)*
