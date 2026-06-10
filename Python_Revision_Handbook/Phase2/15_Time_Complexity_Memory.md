# Chapter 15: Time Complexity & Memory Optimization — "Writing High-Performance Python"

---

## Topic Overview

| Aspect | Details |
|--------|---------|
| **Definition** | **Time Complexity** measures the execution time relative to input size (Big O). **Memory Optimization** is the practice of reducing Python's RAM usage. |
| **Why It Exists** | Python is dynamically typed and has a lot of object wrapper overhead. Writing naive loops or storing too much in memory will make scripts slow and crash servers (out of memory). |
| **Where It Is Used** | Large data processing pipelines, web backend optimization (handling concurrent users), embedded systems, and LeetCode/coding interviews. |
| **Real-World Analogy** | **Time Complexity:** Looking up a word in a dictionary. Scanning alphabetically (Binary Search, `O(log n)`) is much faster than flipping page-by-page from the start (Linear Search, `O(n)`). **Memory Optimization:** Packing a suitcase tightly using vacuum bags (`__slots__`) rather than throwing clothes in loosely. |

---

## Think of It This Way

### 1. Time Complexity: Use the right collection.
If you have a list of 10 million elements:
* Checking `item in my_list` requires Python to scan all 10 million items one-by-one from index 0. This is **Linear Time: `O(n)`** (slow).
* Checking `item in my_set` or `item in my_dict` uses a hash table lookup. It calculates the memory address instantly. This is **Constant Time: `O(1)`** (near-instant!).

### 2. Memory Optimization: `__slots__`
Normally, every custom class instance in Python stores its attributes inside a dynamic dictionary called `__dict__`. Dicts are highly fast, but they consume a lot of memory because they must allocate buffer space to prevent hash collisions.
By defining `__slots__`, you tell Python: *"This class will only ever have these specific attributes."* Python drops the dynamic `__dict__` and allocates a tiny, fixed-size array in memory. This can reduce memory usage by **50% to 70%**!

---

## Step 1: Python Built-In Operation Complexity

Here is a cheat sheet of built-in Python operations:

| Collection | Operation | Average Time Complexity | Details |
|---|---|---|---|
| **List** | Append | `O(1)` | Adding to the end is fast. |
| **List** | Pop (last) | `O(1)` | Removing from the end is fast. |
| **List** | Insert/Pop (first) | `O(n)` | Slow, has to shift all other elements in memory. |
| **List** | Search (`item in lst`) | `O(n)` | Scans linearly. |
| **Set** / **Dict** | Lookup / Insert | `O(1)` | Uses hash table. Fast. |
| **Deque** | Append/Pop (first) | `O(1)` | Double-ended queue is optimized for front operations. |

---

## Step 2: Memory Optimization Techniques

### 1. Saving memory with `__slots__`
```python
import sys

# Normal class
class NormalPlayer:
    def __init__(self, name, score):
        self.name = name
        self.score = score

# Slotted class
class SlottedPlayer:
    __slots__ = ['name', 'score']  # Fixed attributes only!
    def __init__(self, name, score):
        self.name = name
        self.score = score

p1 = NormalPlayer("Alice", 100)
p2 = SlottedPlayer("Alice", 100)

# Check size of the instances (sys.getsizeof doesn't fully measure nested __dict__ size, but reveals class size)
print(sys.getsizeof(p1.__dict__))  # ~104 bytes (just for the dictionary container)
# print(p2.__dict__)               # AttributeError: 'SlottedPlayer' object has no attribute '__dict__'
```

### 2. Generators vs Lists (Lazy Loading)
If you need to process a file containing 10 million rows:
* **List approach:** Loading all 10 million rows into a list allocates gigabytes of RAM.
* **Generator approach:** Yielding one row at a time processes the entire file using only a few kilobytes of RAM.

```python
# List comprehension (allocates memory for 1,000,000 items immediately)
squares_list = [x**2 for x in range(1_000_000)]

# Generator expression (allocates 0 memory for items, computes on the fly)
squares_gen = (x**2 for x in range(1_000_000))

print(sys.getsizeof(squares_list)) # ~8,448,728 bytes
print(sys.getsizeof(squares_gen))  # ~208 bytes!
```

---

## Step 3: Python Garbage Collection (Brief)

Python manages memory using two main strategies:
1. **Reference Counting:** Every object keeps track of how many variables/places reference it. When this count hits `0`, Python deletes the object immediately.
2. **Generational Cyclic Garbage Collector:** Reference counting cannot detect circular references (e.g. `A` points to `B` and `B` points to `A`, with no outer variable referencing either). Python's cyclic GC periodically runs to find and destroy these cycles.

---

## Code Examples

### Easy Examples
```python
# 1. Check memory size of basic objects
print(sys.getsizeof(0))       # ~28 bytes (ints have wrapper overhead!)
print(sys.getsizeof(""))      # ~49 bytes (empty string)
print(sys.getsizeof([]))      # ~56 bytes (empty list)

# 2. Timing lookups: List vs Set
import time
large_list = list(range(1_000_000))
large_set = set(large_list)

# List lookup (O(n))
start = time.perf_counter()
999_999 in large_list
print(f"List search: {time.perf_counter() - start:.6f} sec")

# Set lookup (O(1))
start = time.perf_counter()
999_999 in large_set
print(f"Set search: {time.perf_counter() - start:.6f} sec")

# 3. Memory usage with slots on 10,000 objects (conceptual)
# Normal instances would take ~1.5 MB; slots reduce it to ~500 KB

# 4. Range vs List range in memory
print(sys.getsizeof(range(1_000_000)))  # 48 bytes (range is a generator-like object)
print(sys.getsizeof(list(range(1_000_000)))) # ~8,000,056 bytes

# 5. String joins vs concatenation
# Concatenating inside a loop creates a new string in memory every time
# Join is O(n); concatenation inside a loop is O(n^2)
words = ["word"] * 1000
# Bad:
s = ""
for w in words: s += w
# Good:
s = "".join(words)
```

### Medium Examples
```python
# 6. Avoiding memory leak in closures / cycles using weakref
import weakref

class ExpensiveObject:
    def __del__(self):
        print("Object deleted!")

obj = ExpensiveObject()
# Create a weak reference (does not increment the reference count)
weak_ref = weakref.ref(obj)

print(weak_ref())  # <__main__.ExpensiveObject object at 0x...>
del obj            # "Object deleted!" (Triggered immediately!)
print(weak_ref())  # None (Object is gone)

# 7. Monitoring active garbage collector generations
import gc
print(gc.get_threshold())  # Default thresholds for gen 0, 1, 2 e.g. (700, 10, 10)

# 8. Lazy line-by-line reading of a huge file
def read_huge_file(file_path):
    with open(file_path, "r") as f:
        for line in f:
            yield line.strip() # Keeps only ONE line in memory at a time

# 9. List pre-allocation optimization
# If you know list size, appending sequentially causes Python to constantly resize and reallocate the underlying array.
# Pre-allocating elements is slightly faster and prevents reallocation cycles.
size = 10000
# Appending:
lst = []
for i in range(size): lst.append(0)
# Pre-allocated:
lst_pre = [0] * size

# 10. Counting frequencies efficiently
# Bad: Using list.count() inside a loop is O(n^2)
# Good: Using collections.Counter is O(n)
items = ["apple", "banana", "apple"]
# Bad:
counts = {x: items.count(x) for x in items}
# Good:
from collections import Counter
counts_good = Counter(items)
```

### Advanced Examples
```python
# 11. Creating custom container with __slots__ and inheritance
# Child classes do not automatically inherit slots unless they define __slots__ = ()
class Parent:
    __slots__ = ('x',)
    def __init__(self, x):
        self.x = x

class Child(Parent):
    __slots__ = ('y',) # Inherits x from Parent, adds y. Still lacks __dict__!
    def __init__(self, x, y):
        super().__init__(x)
        self.y = y

c = Child(1, 2)
# c.z = 3  # AttributeError! No __dict__ available.

# 12. Profiling memory with tracemalloc
import tracemalloc

tracemalloc.start()
# Execute memory intensive code
temp_list = [x for x in range(100_000)]
current, peak = tracemalloc.get_traced_memory()
print(f"Current memory usage: {current / 10**6:.2f} MB; Peak: {peak / 10**6:.2f} MB")
tracemalloc.stop()

# 13. Deep dive into standard string interning
# Interning reuses the same memory address for identical strings.
# Python automatically interns short string constants/identifiers.
s1 = "hello_world"
s2 = "hello_world"
print(s1 is s2) # True (interned automatically)

# Dynamic strings are NOT interned by default
s3 = "".join(["hello", "_", "world"])
print(s1 is s3) # False

# Manually intern to optimize memory and change comparison from O(n) string comparison to O(1) pointer comparison
import sys
s4 = sys.intern("".join(["hello", "_", "world"]))
print(s1 is s4) # True

# 14. Clearing references to avoid memory leaks in long-running processes
# If a list is stored as a global or class variable, it will never be garbage collected.
class DataProcessor:
    def __init__(self):
        self.cache = []

    def process(self, data):
        self.cache.extend(data)
        # Process data...
        # Clear cache when done to free up RAM
        self.cache.clear() 

# 15. Disabling GC for performance during massive computations
# If allocating millions of objects that you know will clean up at the end, turning off GC temporarily avoids overhead.
def high_performance_task():
    gc.disable()
    try:
        # Create millions of short-lived objects
        temp = {x: str(x) for x in range(1_000_000)}
    finally:
        gc.enable() # Re-enable GC
        gc.collect() # Force manual sweep
```

---

## Common Mistakes & Edge Cases

### 1. Slotted classes can break inheritance
If a subclass inherits from a slotted class but does not declare `__slots__ = ()` (an empty tuple), Python will automatically create a `__dict__` for the subclass, instantly losing all memory optimization benefits.

### 2. Using slots on classes with dynamic attributes
You cannot assign arbitrary new attributes to an instance of a slotted class:
```python
class Point:
    __slots__ = ('x', 'y')

p = Point()
p.x = 1
# p.z = 2  # AttributeError!
```

### 3. List `.count()` in a loop
A classic coding interview mistake is calculating elements inside a loop using `list.count(x)`. Because `.count()` scans the list linearly, calling it in a loop of size `n` results in an `O(n^2)` time complexity. Use a `Counter` or standard `dict` to index elements first (`O(n)`).

---

## Interview Questions (Top 5)

**Q1: How does declaring `__slots__` save memory in Python?**
> By default, every instance of a Python class allocates a dictionary (`__dict__`) to store its attributes. Dictionaries have significant memory overhead because they allocate extra space to avoid hash collisions. Declaring `__slots__` tells Python to allocate a small, fixed-size array in memory for the listed attributes instead, eliminating the `__dict__` dictionary entirely and saving up to 70% of memory per instance.

**Q2: What is the time complexity of lookup, insertion, and deletion in a list versus a set?**
> * **List:** Lookup is `O(n)` because Python has to scan the array sequentially. Insertion (using `insert()`) and deletion (using `pop(0)`) at the beginning are `O(n)` because all remaining elements must be shifted in memory. Append and pop at the end are `O(1)`.
> * **Set:** Lookup, insertion, and deletion are all average-case `O(1)` because sets are implemented as hash tables.

**Q3: How does Python's Garbage Collection system handle circular references?**
> Python uses reference counting as its primary memory management system. However, reference counts of objects in a circular reference (e.g. `A` references `B`, and `B` references `A`) never reach `0`. To resolve this, Python runs a background Generational Cyclic Garbage Collector. It periodically scans objects (categorized into three generations based on survival age), detects isolated cycles of objects that are unreachable from the root namespace, and deletes them.

**Q4: Why is adding strings in a loop using `+` slow in Python, and what is the alternative?**
> Strings are immutable. Every time you concatenate strings using `+` (e.g., `s += char`), Python allocates a completely new string object in memory and copies the data of both strings into it. In a loop of size `n`, this leads to `O(n^2)` time complexity. The optimized alternative is to append strings to a list and combine them using `''.join(list)` at the end, which takes `O(n)` time.

**Q5: What is the difference between a generator expression and a list comprehension in terms of memory and execution?**
> A list comprehension creates the entire list and loads all elements into RAM immediately, which takes `O(n)` memory. A generator expression returns an iterator that computes each element lazily on-demand when iterated over, using constant `O(1)` memory. While list comprehensions are slightly faster when you need to access all elements repeatedly, generator expressions are highly recommended for one-time iterations over large or infinite datasets to prevent memory exhaustion.

---

*← [Previous: Chapter 14 (Copy Module)](14_Copy_Module.md) | [Next: Chapter 16 (Multithreading & Multiprocessing) →](16_Multithreading_Multiprocessing.md)*
