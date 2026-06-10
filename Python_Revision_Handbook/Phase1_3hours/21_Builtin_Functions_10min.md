# Chapter 21: Useful Built-in Functions

---

## Complete Reference

This chapter covers Python's most important built-in functions with syntax, examples, internal behavior, and interview notes.

---

## len()
```python
# Returns the number of items in an object
# Calls __len__() on the object — O(1) for built-in types
print(len([1, 2, 3]))      # 3
print(len("hello"))         # 5
print(len({"a": 1, "b": 2}))  # 2
print(len(set()))           # 0
print(len(range(100)))      # 100 (O(1)!)
```

## type()
```python
# Returns the type of an object
print(type(42))         # <class 'int'>
print(type("hello"))    # <class 'str'>
print(type([]))         # <class 'list'>

# Type comparison
print(type(42) is int)  # True
# Prefer isinstance() for type checking (handles inheritance)
print(isinstance(True, int))  # True (bool is int subclass)
```

## id()
```python
# Returns the unique identity (memory address in CPython) of an object
a = [1, 2, 3]
b = a
print(id(a) == id(b))  # True (same object)
print(id(a))            # e.g., 140456789
```

## range()
```python
# Covered in Chapter 13 — lazy integer sequence
print(list(range(5)))          # [0, 1, 2, 3, 4]
print(list(range(1, 10, 2)))   # [1, 3, 5, 7, 9]
```

## enumerate()
```python
# Returns (index, value) pairs — essential for indexed iteration
fruits = ["apple", "banana", "cherry"]
for i, fruit in enumerate(fruits):
    print(f"{i}: {fruit}")
# 0: apple, 1: banana, 2: cherry

# Custom start index
for i, fruit in enumerate(fruits, start=1):
    print(f"{i}. {fruit}")
# 1. apple, 2. banana, 3. cherry

# Under the hood — it's a generator of tuples
print(list(enumerate("abc")))  # [(0, 'a'), (1, 'b'), (2, 'c')]
```

## zip()
```python
# Combines iterables element-wise — stops at shortest
names = ["Alice", "Bob", "Charlie"]
ages = [25, 30, 35]
cities = ["NYC", "LA"]

# Basic
pairs = list(zip(names, ages))
# [('Alice', 25), ('Bob', 30), ('Charlie', 35)]

# Multiple iterables
combined = list(zip(names, ages, cities))
# [('Alice', 25, 'NYC'), ('Bob', 30, 'LA')]  — stops at shortest!

# strict=True (Python 3.10+) — raises if lengths differ
# list(zip(names, cities, strict=True))  # ValueError!

# Unzip
names2, ages2 = zip(*pairs)  # Transpose trick

# Dict from zip
d = dict(zip(["a", "b", "c"], [1, 2, 3]))
# {'a': 1, 'b': 2, 'c': 3}

# zip_longest for unequal lengths
from itertools import zip_longest
list(zip_longest(names, cities, fillvalue="N/A"))
```

## sorted()
```python
# Returns a NEW sorted list from any iterable
print(sorted([3, 1, 4, 1, 5]))        # [1, 1, 3, 4, 5]
print(sorted("python"))                # ['h', 'n', 'o', 'p', 't', 'y']
print(sorted({3, 1, 2}))               # [1, 2, 3]

# Reverse
print(sorted([3, 1, 2], reverse=True)) # [3, 2, 1]

# Custom key
words = ["banana", "apple", "Cherry"]
print(sorted(words))                    # ['Cherry', 'apple', 'banana'] (case-sensitive!)
print(sorted(words, key=str.lower))     # ['apple', 'banana', 'Cherry']
print(sorted(words, key=len))           # ['apple', 'Cherry', 'banana']

# Sort dicts by value
d = {"b": 3, "a": 1, "c": 2}
print(sorted(d.items(), key=lambda x: x[1]))
# [('a', 1), ('c', 2), ('b', 3)]

# sorted() vs .sort(): sorted returns new list, .sort() modifies in-place
```

## sum()
```python
print(sum([1, 2, 3, 4, 5]))     # 15
print(sum(range(101)))            # 5050
print(sum([0.1, 0.2, 0.3]))      # 0.6000000000000001

# With start value
print(sum([1, 2, 3], 10))        # 16 (10 + 1 + 2 + 3)

# Don't use for string concatenation — use "".join() instead
# sum(["a", "b"], "")  # TypeError! Use "".join(["a", "b"])

# Precise floating-point sum
import math
print(math.fsum([0.1, 0.2, 0.3]))  # 0.6 (exact!)
```

## min() / max()
```python
print(min(3, 1, 4, 1, 5))          # 1
print(max(3, 1, 4, 1, 5))          # 5
print(min([3, 1, 4]))              # 1

# With key
words = ["banana", "apple", "cherry"]
print(min(words, key=len))          # "apple"
print(max(words, key=len))          # "banana" or "cherry" (first maximal)

# With default (for empty iterables)
print(min([], default=0))           # 0 (avoids ValueError)

# Multiple return for equal values — returns first
print(min(1, 1.0))  # 1 (int, because it appeared first)
```

## abs()
```python
print(abs(-5))       # 5
print(abs(3.14))     # 3.14
print(abs(-3+4j))    # 5.0 (magnitude of complex number)
```

## round()
```python
print(round(3.14159, 2))  # 3.14
print(round(3.145, 2))    # 3.14 (banker's rounding!)
print(round(3.155, 2))    # 3.15
print(round(2.5))          # 2 (rounds to even!)
print(round(3.5))          # 4 (rounds to even!)

# Negative ndigits
print(round(1234, -2))    # 1200
print(round(1250, -2))    # 1200 (banker's rounding!)
print(round(1350, -2))    # 1400
```

## any() / all()
```python
# any() — True if ANY element is truthy
print(any([False, False, True]))   # True
print(any([0, 0, 0]))             # False
print(any([]))                    # False

# all() — True if ALL elements are truthy
print(all([True, True, True]))    # True
print(all([True, False, True]))   # False
print(all([]))                    # True (vacuous truth!)

# Common patterns
nums = [2, 4, 6, 8]
print(all(n % 2 == 0 for n in nums))  # True (all even)
print(any(n > 5 for n in nums))       # True (some > 5)

# Short-circuit: stops at first decisive element
print(any(x > 0 for x in range(1_000_000)))  # True (stops at x=1)
```

## input() / print()
```python
# Covered in Chapter 5
```

---

## Additional Important Built-ins

### map()
```python
# Apply function to every item in iterable — returns iterator
nums = [1, 2, 3, 4]
squares = list(map(lambda x: x**2, nums))  # [1, 4, 9, 16]

# Multiple iterables
sums = list(map(lambda a, b: a + b, [1,2,3], [10,20,30]))  # [11, 22, 33]

# map vs comprehension — comprehension is usually more Pythonic
squares = [x**2 for x in nums]  # Preferred in most cases
```

### filter()
```python
# Filter elements — returns iterator
nums = [1, 2, 3, 4, 5, 6]
evens = list(filter(lambda x: x % 2 == 0, nums))  # [2, 4, 6]

# Equivalent comprehension (preferred)
evens = [x for x in nums if x % 2 == 0]

# filter(None, iterable) removes falsy values
data = [0, 1, "", "hello", None, 42, [], [1]]
truthy = list(filter(None, data))  # [1, 'hello', 42, [1]]
```

### isinstance() / issubclass()
```python
print(isinstance(42, int))           # True
print(isinstance(True, int))         # True (bool is int)
print(isinstance(42, (int, float)))  # True (check multiple types)
print(issubclass(bool, int))         # True
```

### hash()
```python
print(hash(42))           # 42 (for small ints)
print(hash("hello"))      # Some large number
print(hash((1, 2, 3)))    # Hashable
# hash([1, 2])             # TypeError: unhashable type: 'list'
```

### dir() / help() / vars()
```python
print(dir(str))       # List all attributes of str
# help(str.split)     # Interactive help
print(vars())         # Current namespace as dict
```

### callable()
```python
print(callable(print))       # True
print(callable(42))          # False
print(callable(str))         # True (can be called as constructor)
```

### iter() / next()
```python
lst = [10, 20, 30]
it = iter(lst)
print(next(it))    # 10
print(next(it))    # 20
print(next(it))    # 30
# next(it)         # StopIteration

# With default
it = iter([1, 2])
print(next(it, "done"))  # 1
print(next(it, "done"))  # 2
print(next(it, "done"))  # "done" (no StopIteration)
```

### repr() / str() / ascii()
```python
s = "hello\tworld"
print(str(s))     # hello	world (human-readable)
print(repr(s))    # 'hello\tworld' (unambiguous, with quotes)
print(ascii("café"))  # 'caf\\xe9' (ASCII-safe representation)
```

### globals() / locals()
```python
x = 10
print(globals()["x"])  # 10

def f():
    y = 20
    print(locals())  # {'y': 20}
```

---

## Quick Reference Table

| Function | Purpose | Returns | Time Complexity |
|----------|---------|---------|----------------|
| `len(x)` | Count items | `int` | O(1) |
| `type(x)` | Get type | `type` | O(1) |
| `id(x)` | Get identity | `int` | O(1) |
| `range(n)` | Number sequence | `range` | O(1) creation |
| `enumerate(it)` | Index + value | `enumerate` | O(1) creation |
| `zip(a, b)` | Pair elements | `zip` | O(1) creation |
| `sorted(it)` | Sort iterable | `list` | O(n log n) |
| `sum(it)` | Sum elements | Number | O(n) |
| `min(it)` | Minimum value | Element | O(n) |
| `max(it)` | Maximum value | Element | O(n) |
| `abs(x)` | Absolute value | Number | O(1) |
| `round(x, n)` | Round number | Number | O(1) |
| `any(it)` | Any truthy? | `bool` | O(n) worst |
| `all(it)` | All truthy? | `bool` | O(n) worst |
| `map(f, it)` | Transform | `map` | O(1) creation |
| `filter(f, it)` | Filter | `filter` | O(1) creation |
| `isinstance(x, t)` | Type check | `bool` | O(1) |
| `hash(x)` | Hash value | `int` | O(1) |
| `callable(x)` | Is callable? | `bool` | O(1) |
| `iter(x)` | Get iterator | Iterator | O(1) |
| `next(it)` | Next value | Element | O(1) |

---

## Interview Questions

**Q1: What is the difference between `sorted()` and `.sort()`?**
> `sorted()` returns a new list, works on any iterable. `.sort()` modifies in-place, returns `None`, only works on lists.

**Q2: What does `all([])` return?**
> `True`. This is "vacuous truth" — all zero elements satisfy the condition.

**Q3: What is banker's rounding?**
> `round()` uses "round half to even" — `round(0.5)=0`, `round(1.5)=2`, `round(2.5)=2`. This reduces statistical bias in large datasets.

**Q4: What is the difference between `map()` and a list comprehension?**
> `map()` returns a lazy iterator, is slightly faster for simple function calls. List comprehension is more readable, more Pythonic, and can include conditions.

---

## Practice Questions

### Easy
**Q1:** Use `enumerate` and `zip` to create a numbered list pairing names with scores.
```python
names = ["Alice", "Bob", "Charlie"]
scores = [92, 85, 78]
for i, (name, score) in enumerate(zip(names, scores), 1):
    print(f"{i}. {name}: {score}")
```

**Q2:** Use `any` and `all` to check if a list has all positive numbers and if any are even.
```python
nums = [2, 4, 7, 9, 11]
print(f"All positive: {all(n > 0 for n in nums)}")
print(f"Any even: {any(n % 2 == 0 for n in nums)}")
```

### Medium
**Q3:** Use `sorted` with a custom key to sort a list of dictionaries by multiple fields.
```python
students = [
    {"name": "Charlie", "grade": 85, "age": 20},
    {"name": "Alice", "grade": 92, "age": 22},
    {"name": "Bob", "grade": 85, "age": 21},
]
result = sorted(students, key=lambda s: (-s["grade"], s["name"]))
for s in result:
    print(f"{s['name']}: {s['grade']}")
# Alice: 92, Bob: 85, Charlie: 85
```

**Q4:** Combine `filter`, `map`, and `sum` to calculate the sum of squares of even numbers from 1-100.
```python
result = sum(map(lambda x: x**2, filter(lambda x: x % 2 == 0, range(1, 101))))
print(result)  # 171700

# More Pythonic:
result = sum(x**2 for x in range(1, 101) if x % 2 == 0)
```

### Advanced
**Q5:** Implement your own `enumerate()` function.
```python
def my_enumerate(iterable, start=0):
    index = start
    for item in iterable:
        yield index, item
        index += 1

for i, val in my_enumerate(["a", "b", "c"], start=1):
    print(f"{i}: {val}")
```

**Q6:** Implement your own `zip()` that handles iterables of different lengths.
```python
def my_zip(*iterables, strict=False):
    iterators = [iter(it) for it in iterables]
    while True:
        results = []
        for it in iterators:
            try:
                results.append(next(it))
            except StopIteration:
                if strict and results:
                    raise ValueError("zip() has arguments with different lengths")
                return
        yield tuple(results)

print(list(my_zip([1,2,3], "abc", [10,20,30])))
```

---

*← [Previous: File Handling](20_File_Handling.md) | [Next: 100 Interview Questions →](22_Interview_Questions_100.md)*
