# Chapter 7: Lists

---

## Topic Overview

| Aspect | Details |
|--------|---------|
| **Definition** | A list is an **ordered, mutable, heterogeneous** sequence of elements enclosed in square brackets `[]`. |
| **Why It Exists** | Lists provide a flexible, dynamic container for storing collections of items that may change during program execution. |
| **Where It Is Used** | Data processing, algorithm implementation, ML feature vectors, API responses, batch operations. |
| **Real-World Analogy** | A list is like a **numbered filing cabinet** — you can add drawers, remove drawers, rearrange them, and each drawer can hold anything. |

### How It Works Internally
- CPython implements lists as **dynamic arrays** (not linked lists).
- Internally, a list is a **contiguous array of pointers** (references) to Python objects.
- When the array fills up, Python **over-allocates** (~12.5% extra) to amortize future appends.
- Growth pattern: 0, 4, 8, 16, 24, 32, 40, 52, 64, 76, ...

```python
import sys

# See how memory grows
sizes = []
lst = []
for i in range(20):
    lst.append(i)
    sizes.append((len(lst), sys.getsizeof(lst)))

for length, size in sizes:
    print(f"Length: {length:>3}, Memory: {size:>4} bytes")
# You'll see the size increases in jumps, not one element at a time
```

### Memory Behavior
- Each list element is a **pointer** (8 bytes on 64-bit) to an object.
- The list header is ~56 bytes (for the list object itself).
- The underlying array has the pointers, plus over-allocation space.
- Elements can be **any type** (heterogeneous) because only pointers are stored.

### Time Complexity

| Operation | Time Complexity | Notes |
|-----------|----------------|-------|
| `lst[i]` (index) | O(1) | Direct pointer access |
| `lst[i] = x` (assign) | O(1) | Replace pointer |
| `lst.append(x)` | O(1) amortized | May trigger resize |
| `lst.insert(i, x)` | O(n) | Shift elements right |
| `lst.pop()` | O(1) | Remove last |
| `lst.pop(i)` | O(n) | Shift elements left |
| `lst.remove(x)` | O(n) | Linear search + shift |
| `x in lst` | O(n) | Linear search |
| `lst.sort()` | O(n log n) | Timsort |
| `lst.reverse()` | O(n) | In-place swap |
| `lst + other` | O(n + m) | Creates new list |
| `lst.extend(other)` | O(m) | Appends in-place |
| `lst.copy()` | O(n) | Shallow copy |
| `len(lst)` | O(1) | Stored attribute |

---

## 7.1 Creation

```python
# Empty list
lst = []
lst = list()

# With elements
nums = [1, 2, 3, 4, 5]
mixed = [1, "hello", 3.14, True, None]

# From other iterables
from_string = list("hello")       # ['h', 'e', 'l', 'l', 'o']
from_range = list(range(5))       # [0, 1, 2, 3, 4]
from_tuple = list((1, 2, 3))      # [1, 2, 3]
from_set = list({3, 1, 2})        # [1, 2, 3] (order may vary)

# List comprehension
squares = [x**2 for x in range(10)]  # [0, 1, 4, 9, 16, 25, 36, 49, 64, 81]

# Multiplication (creates references, not copies!)
zeros = [0] * 5                   # [0, 0, 0, 0, 0]
# WARNING with mutable elements:
matrix = [[0] * 3] * 3           # [[0,0,0], [0,0,0], [0,0,0]]
matrix[0][0] = 1                  # ALL rows change! [[1,0,0], [1,0,0], [1,0,0]]
# Correct way:
matrix = [[0] * 3 for _ in range(3)]
```

---

## 7.2 Dynamic Resizing

```python
import sys

lst = []
print(f"Empty: {sys.getsizeof(lst)} bytes")  # ~56 bytes (just the list object)

lst.append(1)
print(f"1 elem: {sys.getsizeof(lst)} bytes")  # ~88 bytes (over-allocated to 4 slots)

# CPython over-allocation formula (simplified):
# new_allocated = (current_size + (current_size >> 3) + 6) & ~3
# This gives roughly 12.5% over-allocation

# Why over-allocate? To make append O(1) amortized.
# Without it, every append would need a full copy → O(n) each time.
```

---

## 7.3 Heterogeneous Data

```python
# Lists can hold ANY type
everything = [
    42,             # int
    3.14,           # float
    "hello",        # str
    True,           # bool
    None,           # NoneType
    [1, 2],         # nested list
    (3, 4),         # tuple
    {"a": 1},       # dict
    {5, 6},         # set
    lambda x: x*2,  # function
]

for item in everything:
    print(f"{type(item).__name__:>10}: {item}")
```

---

## 7.4 Nested Lists

```python
# 2D matrix
matrix = [
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9]
]

# Access element
print(matrix[1][2])    # 6 (row 1, col 2)

# Iterate
for row in matrix:
    for val in row:
        print(f"{val:3}", end="")
    print()

# Flatten
flat = [val for row in matrix for val in row]
print(flat)  # [1, 2, 3, 4, 5, 6, 7, 8, 9]

# Transpose
transposed = [[row[i] for row in matrix] for i in range(len(matrix[0]))]
print(transposed)  # [[1,4,7], [2,5,8], [3,6,9]]

# Or using zip:
transposed = list(map(list, zip(*matrix)))
```

---

## 7.5 List Methods

### append()
```python
lst = [1, 2, 3]
lst.append(4)        # [1, 2, 3, 4]
lst.append([5, 6])   # [1, 2, 3, 4, [5, 6]] — adds list as ONE element
# Returns None! (modifies in-place)
result = lst.append(7)
print(result)  # None
```

### extend()
```python
lst = [1, 2, 3]
lst.extend([4, 5, 6])    # [1, 2, 3, 4, 5, 6] — adds EACH element
lst.extend("abc")         # [1, 2, 3, 4, 5, 6, 'a', 'b', 'c']
lst.extend(range(3))      # [..., 0, 1, 2]

# extend vs append:
a = [1, 2]
a.append([3, 4])   # [1, 2, [3, 4]] ← list as element
b = [1, 2]
b.extend([3, 4])   # [1, 2, 3, 4]   ← elements added individually

# extend is equivalent to: lst += iterable
lst = [1, 2]
lst += [3, 4]  # Same as lst.extend([3, 4])
```

### insert()
```python
lst = [1, 2, 3, 5]
lst.insert(3, 4)      # [1, 2, 3, 4, 5] — insert at index 3
lst.insert(0, 0)       # [0, 1, 2, 3, 4, 5] — insert at beginning
lst.insert(100, 99)    # [..., 99] — index beyond length = append
lst.insert(-1, 98)     # Inserts BEFORE the last element

# Performance: O(n) — all elements after index must shift
# Use collections.deque for efficient front insertion
```

### remove()
```python
lst = [1, 2, 3, 2, 1]
lst.remove(2)          # [1, 3, 2, 1] — removes FIRST occurrence
# lst.remove(99)       # ValueError: list.remove(x): x not in list

# Safe remove:
if 99 in lst:
    lst.remove(99)

# Remove ALL occurrences:
lst = [1, 2, 3, 2, 1]
lst = [x for x in lst if x != 2]  # [1, 3, 1]
```

### pop()
```python
lst = [1, 2, 3, 4, 5]
last = lst.pop()       # 5, lst = [1, 2, 3, 4]
first = lst.pop(0)     # 1, lst = [2, 3, 4]
middle = lst.pop(1)    # 3, lst = [2, 4]

# pop() returns the removed element (unlike remove())
# pop() with no args → O(1)
# pop(0) → O(n) (all elements shift)
# Use collections.deque.popleft() for O(1) front removal

# Empty list pop → IndexError
# [].pop()  # IndexError: pop from empty list
```

### clear()
```python
lst = [1, 2, 3, 4, 5]
lst.clear()            # []
# Equivalent to: del lst[:]
# Different from: lst = [] (creates new object, old references stay)

# Why it matters:
a = [1, 2, 3]
b = a           # b references same list
a.clear()       # Clears the ACTUAL list
print(b)        # [] — b sees the change

a = [1, 2, 3]
b = a
a = []          # a points to NEW empty list
print(b)        # [1, 2, 3] — b still points to old list
```

### sort()
```python
lst = [3, 1, 4, 1, 5, 9, 2, 6]

# In-place sort (returns None!)
lst.sort()                    # [1, 1, 2, 3, 4, 5, 6, 9]
lst.sort(reverse=True)        # [9, 6, 5, 4, 3, 2, 1, 1]

# Custom key function
words = ["banana", "apple", "cherry", "date"]
words.sort()                   # ['apple', 'banana', 'cherry', 'date']
words.sort(key=len)            # ['date', 'apple', 'banana', 'cherry']
words.sort(key=str.lower)      # Case-insensitive sort

# Sort vs sorted:
lst = [3, 1, 2]
sorted_lst = sorted(lst)      # Returns NEW list [1, 2, 3]
print(lst)                     # [3, 1, 2] — unchanged!

lst.sort()                     # Modifies IN-PLACE, returns None
print(lst)                     # [1, 2, 3]

# Stable sort: equal elements maintain original order
data = [("Alice", 85), ("Bob", 92), ("Charlie", 85)]
data.sort(key=lambda x: x[1])
# [('Alice', 85), ('Charlie', 85), ('Bob', 92)] — Alice still before Charlie

# Python uses TIMSORT: hybrid merge sort + insertion sort
# Time: O(n log n), Space: O(n), Stable: Yes
```

### reverse()
```python
lst = [1, 2, 3, 4, 5]
lst.reverse()          # [5, 4, 3, 2, 1] — in-place, returns None

# reverse vs reversed:
lst = [1, 2, 3]
rev_list = list(reversed(lst))  # [3, 2, 1] — new list, original unchanged
rev_slice = lst[::-1]           # [3, 2, 1] — new list via slicing
```

### copy()
```python
# Shallow copy
lst = [1, [2, 3], 4]
shallow = lst.copy()       # Same as lst[:] or list(lst)
print(lst == shallow)      # True (same values)
print(lst is shallow)      # False (different objects)

# But nested objects are SHARED:
shallow[1][0] = 99
print(lst)                 # [1, [99, 3], 4] — inner list affected!

# Deep copy (for nested structures)
import copy
lst = [1, [2, 3], 4]
deep = copy.deepcopy(lst)
deep[1][0] = 99
print(lst)                 # [1, [2, 3], 4] — original unchanged!
```

### count()
```python
lst = [1, 2, 2, 3, 2, 4]
print(lst.count(2))    # 3
print(lst.count(5))    # 0 (not found → 0, no error)
```

### index()
```python
lst = [10, 20, 30, 20, 40]
print(lst.index(20))       # 1 (first occurrence)
print(lst.index(20, 2))    # 3 (search from index 2)
# lst.index(99)            # ValueError

# Safe index:
def safe_index(lst, value, default=-1):
    try:
        return lst.index(value)
    except ValueError:
        return default
```

---

## 7.6 Shallow Copy vs Deep Copy

```python
import copy

# Original
original = [1, [2, 3], [4, [5, 6]]]

# Shallow copy — copies the outer list, shares inner objects
shallow = copy.copy(original)   # or original.copy() or original[:] or list(original)

# Deep copy — recursively copies everything
deep = copy.deepcopy(original)

# Modify nested element
original[1][0] = 99

print(f"Original: {original}")  # [1, [99, 3], [4, [5, 6]]]
print(f"Shallow:  {shallow}")   # [1, [99, 3], [4, [5, 6]]] ← affected!
print(f"Deep:     {deep}")      # [1, [2, 3], [4, [5, 6]]] ← independent!
```

```
Shallow Copy Visualization:
  original: [ptr1, ptr2, ptr3]
  shallow:  [ptr1, ptr2, ptr3]  ← same inner pointers!

Deep Copy Visualization:
  original: [ptr1, ptr2, ptr3]
  deep:     [ptr4, ptr5, ptr6]  ← completely new inner objects!
```

---

## Common Mistakes

| Mistake | Example | Fix |
|---------|---------|-----|
| `sort()` returns None | `x = lst.sort()` → x is None | `x = sorted(lst)` |
| `append` vs `extend` | `lst.append([1,2])` adds list as element | `lst.extend([1,2])` |
| `*` with mutables | `[[0]] * 3` → shared references | `[[0] for _ in range(3)]` |
| Modifying during iteration | `for x in lst: lst.remove(x)` | Iterate over copy: `for x in lst[:]` |
| Shallow copy surprise | `b = a.copy(); b[0][0] = 99` affects a | Use `copy.deepcopy()` |
| `lst = []` vs `lst.clear()` | `lst = []` doesn't affect other refs | `lst.clear()` clears the actual object |

---

## Interview Questions

**Q1: How are Python lists implemented internally?**
> As dynamic arrays (C arrays of pointers). Not linked lists. This gives O(1) index access, O(1) amortized append, but O(n) insert/delete at arbitrary positions.

**Q2: What is the time complexity of `in` operator for lists vs sets?**
> Lists: O(n) — linear scan. Sets: O(1) average — hash lookup. For membership testing with large collections, always prefer sets.

**Q3: What is the difference between `append()` and `extend()`?**
> `append(x)` adds `x` as a single element. `extend(iterable)` adds each element of the iterable individually. `[1].append([2,3])` → `[1, [2,3]]`. `[1].extend([2,3])` → `[1, 2, 3]`.

**Q4: Why does `[[]] * 3` create problems?**
> It creates 3 references to the SAME inner list. Modifying one affects all. Use `[[] for _ in range(3)]` for independent lists.

**Q5: Difference between `sort()` and `sorted()`?**
> `sort()` modifies in-place, returns `None`. `sorted()` returns a new sorted list, original unchanged. Both use Timsort (O(n log n), stable).

---

## Code Examples

### Easy Examples

```python
# Example 1: Basic operations
fruits = ["apple", "banana", "cherry"]
fruits.append("date")
print(fruits)  # ['apple', 'banana', 'cherry', 'date']
print(len(fruits))  # 4
print(fruits[0])    # apple
print(fruits[-1])   # date

# Example 2: List from range
evens = list(range(0, 20, 2))
print(evens)  # [0, 2, 4, 6, 8, 10, 12, 14, 16, 18]

# Example 3: Membership
nums = [10, 20, 30, 40, 50]
print(30 in nums)     # True
print(35 not in nums) # True

# Example 4: Slicing
lst = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
print(lst[2:7])    # [2, 3, 4, 5, 6]
print(lst[::2])    # [0, 2, 4, 6, 8]
print(lst[::-1])   # [9, 8, 7, 6, 5, 4, 3, 2, 1, 0]

# Example 5: Nested list access
matrix = [[1,2,3], [4,5,6], [7,8,9]]
print(matrix[1][2])  # 6
```

### Medium Examples

```python
# Example 6: Remove duplicates preserving order
def remove_duplicates(lst):
    seen = set()
    result = []
    for item in lst:
        if item not in seen:
            seen.add(item)
            result.append(item)
    return result

print(remove_duplicates([1, 3, 2, 3, 1, 4, 2, 5]))  # [1, 3, 2, 4, 5]

# Example 7: Flatten nested list (one level)
nested = [[1, 2], [3, 4], [5, 6]]
flat = [item for sublist in nested for item in sublist]
print(flat)  # [1, 2, 3, 4, 5, 6]

# Example 8: List as stack (LIFO)
stack = []
stack.append("a")  # push
stack.append("b")
stack.append("c")
print(stack.pop())  # c (LIFO)
print(stack.pop())  # b
print(stack)        # ['a']

# Example 9: Sort complex data
students = [
    {"name": "Charlie", "grade": 85},
    {"name": "Alice", "grade": 92},
    {"name": "Bob", "grade": 78},
]
students.sort(key=lambda s: s["grade"], reverse=True)
for s in students:
    print(f"{s['name']}: {s['grade']}")

# Example 10: Zip and unzip
names = ["Alice", "Bob", "Charlie"]
ages = [25, 30, 35]
combined = list(zip(names, ages))
print(combined)  # [('Alice', 25), ('Bob', 30), ('Charlie', 35)]

# Unzip
names2, ages2 = zip(*combined)
print(list(names2))  # ['Alice', 'Bob', 'Charlie']
```

### Advanced Examples

```python
# Example 11: Memory-efficient list vs generator
import sys
lst = [x**2 for x in range(1000)]
gen = (x**2 for x in range(1000))
print(f"List: {sys.getsizeof(lst)} bytes")  # ~8856 bytes
print(f"Generator: {sys.getsizeof(gen)} bytes")  # ~200 bytes

# Example 12: Rotate list
def rotate(lst, k):
    k = k % len(lst) if lst else 0
    return lst[-k:] + lst[:-k]

print(rotate([1,2,3,4,5], 2))   # [4, 5, 1, 2, 3]
print(rotate([1,2,3,4,5], -1))  # [2, 3, 4, 5, 1]

# Example 13: Deep copy demonstration
import copy
original = [1, [2, [3, [4]]]]
shallow = original.copy()
deep = copy.deepcopy(original)

original[1][1][1][0] = 999
print(f"Original: {original}")   # [1, [2, [3, [999]]]]
print(f"Shallow:  {shallow}")    # [1, [2, [3, [999]]]]  ← affected!
print(f"Deep:     {deep}")       # [1, [2, [3, [4]]]]     ← safe!

# Example 14: Custom sort with multiple keys
data = [
    ("Alice", "Math", 95),
    ("Bob", "Math", 92),
    ("Alice", "Science", 88),
    ("Bob", "Science", 95),
    ("Charlie", "Math", 92),
]

# Sort by score (desc), then name (asc)
data.sort(key=lambda x: (-x[2], x[0]))
for name, subject, score in data:
    print(f"{name:>8} | {subject:>8} | {score}")

# Example 15: The * operator trap and fix
# WRONG: All inner lists are the SAME object
bad_matrix = [[0] * 3] * 3
bad_matrix[0][0] = 1
print(f"Bad:  {bad_matrix}")  # [[1,0,0], [1,0,0], [1,0,0]]

# CORRECT: Each inner list is independent
good_matrix = [[0] * 3 for _ in range(3)]
good_matrix[0][0] = 1
print(f"Good: {good_matrix}")  # [[1,0,0], [0,0,0], [0,0,0]]

# Verify:
print(f"Bad same?  {bad_matrix[0] is bad_matrix[1]}")   # True
print(f"Good same? {good_matrix[0] is good_matrix[1]}") # False
```

---

## Edge Cases

1. **Empty list operations**: `[].pop()` → `IndexError`. `[].sort()` → no error (nothing to sort).
2. **`lst * 0`** returns `[]`.
3. **`lst.remove()` with duplicates**: Only removes the **first** occurrence.
4. **Slice assignment**: `lst[1:3] = [10, 20, 30]` can change list length.
5. **`lst.insert(len(lst), x)`** is equivalent to `lst.append(x)`.
6. **`sorted()` works on any iterable**: `sorted("hello")` → `['e', 'h', 'l', 'l', 'o']`.
7. **Negative step slicing**: `lst[5:2:-1]` works; `lst[2:5:-1]` returns `[]`.

---

## Practice Questions

### Easy

**Q1:** Create a list of numbers 1-10, remove all even numbers, and print the result.
```python
# Answer:
nums = list(range(1, 11))
odds = [x for x in nums if x % 2 != 0]
print(odds)  # [1, 3, 5, 7, 9]
```

**Q2:** Given two lists, find their intersection (common elements).
```python
# Answer:
a = [1, 2, 3, 4, 5]
b = [4, 5, 6, 7, 8]
common = list(set(a) & set(b))
print(sorted(common))  # [4, 5]
```

### Medium

**Q3:** Write a function to merge two sorted lists into one sorted list (like merge sort's merge step).
```python
# Answer:
def merge_sorted(lst1, lst2):
    result = []
    i = j = 0
    while i < len(lst1) and j < len(lst2):
        if lst1[i] <= lst2[j]:
            result.append(lst1[i])
            i += 1
        else:
            result.append(lst2[j])
            j += 1
    result.extend(lst1[i:])
    result.extend(lst2[j:])
    return result

print(merge_sorted([1,3,5,7], [2,4,6,8]))  # [1,2,3,4,5,6,7,8]
```

**Q4:** Implement a function that chunks a list into groups of n.
```python
# Answer:
def chunk(lst, n):
    return [lst[i:i+n] for i in range(0, len(lst), n)]

print(chunk([1,2,3,4,5,6,7,8,9], 3))  # [[1,2,3], [4,5,6], [7,8,9]]
print(chunk([1,2,3,4,5], 2))           # [[1,2], [3,4], [5]]
```

### Advanced

**Q5:** Implement a function that finds the k-th largest element without fully sorting the list.
```python
# Answer:
import heapq

def kth_largest(lst, k):
    """O(n log k) using a min-heap of size k."""
    return heapq.nlargest(k, lst)[-1]

# Alternative: Quickselect-inspired O(n) average
def kth_largest_v2(lst, k):
    if k == 1:
        return max(lst)
    # Use sorted partial — still O(n log n) but demonstrates the concept
    lst_sorted = sorted(lst, reverse=True)
    return lst_sorted[k-1]

print(kth_largest([3,1,4,1,5,9,2,6,5,3,5], 3))  # 5
```

**Q6:** Explain why modifying a list while iterating over it causes bugs, and show three safe alternatives.
```python
# Answer:

# THE BUG:
lst = [1, 2, 3, 4, 5, 6]
for x in lst:
    if x % 2 == 0:
        lst.remove(x)
print(lst)  # [1, 3, 5] — but might miss some! 
# The iterator skips elements because indices shift after removal.

# SAFE Alternative 1: Iterate over a copy
lst = [1, 2, 3, 4, 5, 6]
for x in lst[:]:  # lst[:] is a shallow copy
    if x % 2 == 0:
        lst.remove(x)
print(lst)  # [1, 3, 5]

# SAFE Alternative 2: List comprehension (most Pythonic)
lst = [1, 2, 3, 4, 5, 6]
lst = [x for x in lst if x % 2 != 0]
print(lst)  # [1, 3, 5]

# SAFE Alternative 3: Iterate backwards
lst = [1, 2, 3, 4, 5, 6]
for i in range(len(lst) - 1, -1, -1):
    if lst[i] % 2 == 0:
        lst.pop(i)
print(lst)  # [1, 3, 5]
```

---

*← [Previous: Strings](06_Strings.md) | [Next: Tuples →](08_Tuples.md)*
