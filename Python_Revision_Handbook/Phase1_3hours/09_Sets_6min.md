# Chapter 9: Sets

---

## Topic Overview

| Aspect | Details |
|--------|---------|
| **Definition** | A set is an **unordered, mutable collection of unique, hashable elements** enclosed in curly braces `{}`. |
| **Why It Exists** | Sets provide O(1) membership testing, deduplication, and mathematical set operations (union, intersection, difference). |
| **Where It Is Used** | Removing duplicates, membership testing, finding common elements, graph algorithms, data cleaning. |
| **Real-World Analogy** | A set is like a **bag of unique marbles** — no duplicates, no specific order, and you can quickly check if a marble is in the bag. |

### How It Works Internally
- Sets are implemented as **hash tables** (similar to dict but without values).
- Each element is hashed using `hash()`, and stored in a bucket.
- Collisions are handled using **open addressing** (probing).
- Elements must be **hashable** (immutable types: int, float, str, tuple of immutables).
- The hash table is resized when it's about 2/3 full.

### Time Complexity

| Operation | Average | Worst |
|-----------|---------|-------|
| `x in s` | O(1) | O(n) |
| `s.add(x)` | O(1) | O(n) |
| `s.remove(x)` | O(1) | O(n) |
| `s.discard(x)` | O(1) | O(n) |
| `s \| t` (union) | O(len(s) + len(t)) | — |
| `s & t` (intersection) | O(min(len(s), len(t))) | — |
| `s - t` (difference) | O(len(s)) | — |
| `s ^ t` (symmetric diff) | O(len(s) + len(t)) | — |
| `len(s)` | O(1) | O(1) |

---

## 9.1 Creation

```python
# Curly braces (non-empty)
s = {1, 2, 3, 4, 5}

# IMPORTANT: {} creates an EMPTY DICT, not a set!
empty_dict = {}       # dict!
empty_set = set()     # set!

# From iterable
s = set([1, 2, 3, 2, 1])     # {1, 2, 3} — duplicates removed
s = set("hello")               # {'h', 'e', 'l', 'o'} — unique chars
s = set(range(5))              # {0, 1, 2, 3, 4}

# Set comprehension
squares = {x**2 for x in range(10)}  # {0, 1, 4, 9, 16, 25, 36, 49, 64, 81}

# Frozenset (immutable set — can be dict key or element of another set)
fs = frozenset([1, 2, 3])
# fs.add(4)  # AttributeError — frozenset is immutable
```

---

## 9.2 Unique Elements & Hashing

```python
# Sets automatically deduplicate
numbers = [1, 2, 2, 3, 3, 3, 4, 4, 4, 4]
unique = set(numbers)
print(unique)  # {1, 2, 3, 4}

# Order is NOT preserved (implementation detail: may appear ordered in small sets)
s = {3, 1, 4, 1, 5, 9}
print(s)  # {1, 3, 4, 5, 9} — order not guaranteed

# Only hashable elements allowed
valid_set = {1, 2.0, "hello", (1, 2), True, None, frozenset([3])}

# Unhashable elements → TypeError
# {[1, 2]}       # TypeError: unhashable type: 'list'
# {{1: 2}}       # TypeError: unhashable type: 'dict'
# {{1, 2}}       # TypeError: unhashable type: 'set'
# {(1, [2])}     # TypeError: unhashable type: 'list' (inside tuple)

# Interesting: True == 1 and hash(True) == hash(1)
s = {True, 1, 1.0}
print(s)  # {True} — all three are "equal" and have same hash!
```

---

## 9.3 Set Operations

```python
a = {1, 2, 3, 4, 5}
b = {4, 5, 6, 7, 8}

# Union: elements in either set
print(a | b)             # {1, 2, 3, 4, 5, 6, 7, 8}
print(a.union(b))        # Same

# Intersection: elements in both sets
print(a & b)             # {4, 5}
print(a.intersection(b)) # Same

# Difference: elements in a but not in b
print(a - b)             # {1, 2, 3}
print(a.difference(b))   # Same

# Symmetric difference: elements in either but not both
print(a ^ b)             # {1, 2, 3, 6, 7, 8}
print(a.symmetric_difference(b))  # Same

# Subset / Superset
print({1, 2} <= {1, 2, 3})    # True (subset)
print({1, 2}.issubset({1, 2, 3}))  # True
print({1, 2, 3} >= {1, 2})    # True (superset)
print({1, 2, 3}.issuperset({1, 2}))  # True

# Proper subset (strict)
print({1, 2} < {1, 2, 3})     # True (proper subset)
print({1, 2, 3} < {1, 2, 3})  # False (equal, not proper)

# Disjoint (no common elements)
print({1, 2}.isdisjoint({3, 4}))  # True
print({1, 2}.isdisjoint({2, 3}))  # False (2 is common)
```

### Venn Diagram

```
    Set A          Set B
  ┌───────┐    ┌───────┐
  │ 1 2 3 │    │ 6 7 8 │
  │       │4 5 │       │
  │       │    │       │
  └───────┘    └───────┘

  A | B  = {1,2,3,4,5,6,7,8}  (Union)
  A & B  = {4,5}              (Intersection)
  A - B  = {1,2,3}            (Difference)
  B - A  = {6,7,8}            (Difference)
  A ^ B  = {1,2,3,6,7,8}      (Symmetric Difference)
```

---

## 9.4 Set Methods

### add()
```python
s = {1, 2, 3}
s.add(4)      # {1, 2, 3, 4}
s.add(3)      # {1, 2, 3, 4} — no error, no duplicate added
# Returns None
```

### update()
```python
s = {1, 2, 3}
s.update([3, 4, 5])       # {1, 2, 3, 4, 5}
s.update({6, 7}, [8, 9])  # Can accept multiple iterables
# Equivalent to |=
s |= {10, 11}
```

### remove()
```python
s = {1, 2, 3, 4}
s.remove(3)    # {1, 2, 4}
# s.remove(99)  # KeyError: 99
```

### discard()
```python
s = {1, 2, 3, 4}
s.discard(3)   # {1, 2, 4}
s.discard(99)  # No error! (unlike remove)
```

### pop()
```python
s = {1, 2, 3}
element = s.pop()  # Removes and returns an ARBITRARY element
print(element)      # Could be any element
# set().pop()       # KeyError: pop from an empty set
```

### clear()
```python
s = {1, 2, 3}
s.clear()      # set()
```

### Summary: remove vs discard vs pop

| Method | Not Found Behavior | Returns |
|--------|--------------------|---------|
| `remove(x)` | `KeyError` | None |
| `discard(x)` | No error | None |
| `pop()` | `KeyError` (empty set) | Arbitrary element |

---

## Common Mistakes

| Mistake | Example | Fix |
|---------|---------|-----|
| `{}` creates dict, not set | `s = {}` → dict | `s = set()` |
| Unhashable elements | `{[1,2]}` → TypeError | Use tuples: `{(1,2)}` |
| Assuming order | `for x in {3,1,2}` — order varies | Use `sorted(s)` if order needed |
| `True` and `1` are same in sets | `{True, 1}` → `{True}` | Be aware of bool-int equivalence |
| Using `remove` without checking | `s.remove(x)` may raise KeyError | Use `discard(x)` or check first |

---

## Interview Questions

**Q1: How are sets implemented in Python?**
> Hash tables using open addressing. Elements are hashed, and stored in buckets. This gives O(1) average lookup. Only hashable (immutable) elements are allowed.

**Q2: When would you use a set over a list?**
> When you need: (1) O(1) membership testing (`in`), (2) automatic deduplication, (3) mathematical set operations. Lists are O(n) for membership testing.

**Q3: What is the difference between `remove()` and `discard()`?**
> `remove(x)` raises `KeyError` if `x` not found. `discard(x)` does nothing if `x` not found.

**Q4: Can a set contain another set?**
> No, because sets are unhashable (mutable). Use `frozenset` to have a set inside a set: `{frozenset([1,2]), frozenset([3,4])}`.

**Q5: Why does `{True, 1, 1.0}` result in `{True}`?**
> Because `True == 1 == 1.0` and `hash(True) == hash(1) == hash(1.0)`. They are treated as duplicates. The first one added (`True`) is kept.

---

## Code Examples

### Easy Examples

```python
# Example 1: Remove duplicates
nums = [1, 2, 2, 3, 3, 3, 4, 4, 4, 4]
unique = list(set(nums))
print(sorted(unique))  # [1, 2, 3, 4]

# Example 2: Membership testing
valid_users = {"alice", "bob", "charlie"}
user = "alice"
print(f"Access granted: {user in valid_users}")  # True

# Example 3: Set from string
letters = set("mississippi")
print(sorted(letters))  # ['i', 'm', 'p', 's']

# Example 4: Common elements
list1 = [1, 2, 3, 4, 5]
list2 = [4, 5, 6, 7, 8]
common = set(list1) & set(list2)
print(common)  # {4, 5}

# Example 5: Basic set operations
evens = {2, 4, 6, 8, 10}
primes = {2, 3, 5, 7}
print(f"Even primes: {evens & primes}")   # {2}
print(f"Even or prime: {evens | primes}")  # {2, 3, 4, 5, 6, 7, 8, 10}
```

### Medium Examples

```python
# Example 6: Remove duplicates preserving order
def unique_ordered(lst):
    seen = set()
    result = []
    for item in lst:
        if item not in seen:
            seen.add(item)
            result.append(item)
    return result

print(unique_ordered([3, 1, 4, 1, 5, 9, 2, 6, 5, 3]))
# [3, 1, 4, 5, 9, 2, 6]

# Example 7: Find missing numbers
def find_missing(nums, n):
    """Find numbers 1..n missing from the list."""
    return set(range(1, n+1)) - set(nums)

print(find_missing([1, 3, 5, 7, 9], 10))  # {2, 4, 6, 8, 10}

# Example 8: Set of frozensets (set of sets)
power_set_elements = {
    frozenset(),
    frozenset([1]),
    frozenset([2]),
    frozenset([1, 2])
}
print(power_set_elements)

# Example 9: Performance comparison: list vs set lookup
import time
n = 1_000_000
data_list = list(range(n))
data_set = set(range(n))

# Search for last element
target = n - 1

start = time.time()
_ = target in data_list
list_time = time.time() - start

start = time.time()
_ = target in data_set
set_time = time.time() - start

print(f"List lookup: {list_time:.6f}s")
print(f"Set lookup:  {set_time:.6f}s")
print(f"Set is {list_time/max(set_time, 1e-9):.0f}x faster")

# Example 10: Symmetric difference for finding unique elements
alice_skills = {"Python", "Java", "SQL", "Docker"}
bob_skills = {"Python", "Go", "SQL", "Kubernetes"}

only_alice = alice_skills - bob_skills      # {"Java", "Docker"}
only_bob = bob_skills - alice_skills        # {"Go", "Kubernetes"}
unique_to_one = alice_skills ^ bob_skills   # {"Java", "Docker", "Go", "Kubernetes"}
shared = alice_skills & bob_skills          # {"Python", "SQL"}

print(f"Only Alice: {only_alice}")
print(f"Only Bob: {only_bob}")
print(f"Shared: {shared}")
```

### Advanced Examples

```python
# Example 11: Power set using sets
def power_set(s):
    """Generate all subsets of a set."""
    s = list(s)
    result = [set()]
    for elem in s:
        result += [subset | {elem} for subset in result]
    return [frozenset(sub) for sub in result]

ps = power_set({1, 2, 3})
print(f"Power set size: {len(ps)}")  # 8 (2^3)
for subset in sorted(ps, key=len):
    print(subset)

# Example 12: Hash collision demonstration
class BadHash:
    """Object with intentionally bad hash for demonstration."""
    def __init__(self, value):
        self.value = value
    def __hash__(self):
        return 1  # All objects hash to same value!
    def __eq__(self, other):
        return self.value == other.value
    def __repr__(self):
        return f"BadHash({self.value})"

# All elements collide — set still works but degrades to O(n)
s = {BadHash(i) for i in range(5)}
print(s)  # Works but performance is O(n) for lookups

# Example 13: Frozenset as dictionary key
# Representing graph edges
graph = {
    frozenset({"A", "B"}): 5,
    frozenset({"B", "C"}): 3,
    frozenset({"A", "C"}): 7,
}
# Order doesn't matter — frozenset{"A","B"} == frozenset{"B","A"}
print(graph[frozenset({"B", "A"})])  # 5

# Example 14: Set algebra for database-like operations
users = {"alice", "bob", "charlie", "dave", "eve"}
premium_users = {"bob", "eve"}
active_users = {"alice", "bob", "charlie", "eve"}
banned_users = {"dave"}

# Active, non-banned, premium users
target = active_users & premium_users - banned_users
print(f"Target users: {target}")  # {'bob', 'eve'}

# Free active users
free_active = active_users - premium_users - banned_users
print(f"Free active: {free_active}")  # {'alice', 'charlie'}

# Example 15: Efficient two-set operations
import sys

# Memory of set vs list for large data
n = 100_000
lst = list(range(n))
st = set(range(n))

print(f"List memory: {sys.getsizeof(lst):,} bytes")
print(f"Set memory:  {sys.getsizeof(st):,} bytes")
# Sets use more memory (hash table overhead) but provide O(1) lookup
```

---

## Edge Cases

1. **`{}`** creates a dict, not a set. Use `set()`.
2. **`{True, 1}`** → `{True}` because `True == 1`.
3. **Empty set is falsy**: `bool(set())` → `False`.
4. **`set.pop()`** is non-deterministic — don't assume which element.
5. **Modifying a set during iteration** raises `RuntimeError`.
6. **`frozenset` can be a set element or dict key**, regular `set` cannot.

---

## Practice Questions

### Easy

**Q1:** Given two lists, find all elements that appear in both lists.
```python
a = [1, 2, 3, 4, 5, 6]
b = [4, 5, 6, 7, 8, 9]
common = sorted(set(a) & set(b))
print(common)  # [4, 5, 6]
```

**Q2:** Remove all duplicate characters from a string while maintaining order.
```python
def unique_chars(s):
    seen = set()
    return "".join(c for c in s if not (c in seen or seen.add(c)))

print(unique_chars("programming"))  # "progamin"
```

### Medium

**Q3:** Write a function that checks if two strings are anagrams using sets and counting.
```python
from collections import Counter
def are_anagrams(s1, s2):
    return Counter(s1.lower().replace(" ", "")) == Counter(s2.lower().replace(" ", ""))

print(are_anagrams("listen", "silent"))   # True
print(are_anagrams("hello", "world"))     # False
```

**Q4:** Implement a function that finds elements unique to each of N lists.
```python
def unique_to_each(*lists):
    sets = [set(lst) for lst in lists]
    all_elements = set().union(*sets)
    result = {}
    for elem in all_elements:
        appears_in = [i for i, s in enumerate(sets) if elem in s]
        if len(appears_in) == 1:
            idx = appears_in[0]
            result.setdefault(idx, set()).add(elem)
    return result

r = unique_to_each([1,2,3], [3,4,5], [5,6,7])
print(r)  # {0: {1, 2}, 1: {4}, 2: {6, 7}}
```

### Advanced

**Q5:** Implement the power set of a set using both iterative and recursive approaches.
```python
# Iterative
def power_set_iter(s):
    result = [set()]
    for elem in s:
        result += [sub | {elem} for sub in result]
    return result

# Recursive
def power_set_rec(s):
    s = list(s)
    if not s:
        return [set()]
    rest = power_set_rec(s[1:])
    return rest + [{s[0]} | sub for sub in rest]

ps = power_set_iter({1, 2, 3})
print(f"Power set ({len(ps)} subsets):")
for sub in sorted(ps, key=lambda x: (len(x), sorted(x))):
    print(f"  {sub}")
```

**Q6:** Compare the performance of membership testing across list, set, dict, and sorted list with bisect.
```python
import time
import bisect

n = 500_000
data = list(range(n))
data_set = set(data)
data_dict = dict.fromkeys(data)
data_sorted = sorted(data)

target = n - 1
iterations = 1000

def benchmark(name, func):
    start = time.time()
    for _ in range(iterations):
        func()
    elapsed = time.time() - start
    print(f"{name:<20}: {elapsed:.6f}s")

benchmark("list (in)", lambda: target in data)
benchmark("set (in)", lambda: target in data_set)
benchmark("dict (in)", lambda: target in data_dict)
benchmark("bisect", lambda: data_sorted[bisect.bisect_left(data_sorted, target)] == target)
# Set and dict are O(1), list is O(n), bisect is O(log n)
```

---

*← [Previous: Tuples](08_Tuples.md) | [Next: Dictionaries →](10_Dictionaries.md)*
