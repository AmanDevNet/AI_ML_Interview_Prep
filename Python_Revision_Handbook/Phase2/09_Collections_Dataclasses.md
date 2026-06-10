# Chapter 9: Collections & Dataclasses — "Data Structures on Steroids"

---

## Topic Overview

| Aspect | Details |
|--------|---------|
| **Definition** | The `collections` module provides specialized container datatypes. **Dataclasses** (via `@dataclass`) provide a clean way to write classes designed primarily to hold data. |
| **Why It Exists** | Standard `dict`, `tuple`, and `list` can be verbose or error-prone. Standard classes require writing boilerplate like `__init__`, `__repr__`, and `__eq__` manually. |
| **Where It Is Used** | Data processing, tracking frequencies, structured data representations, parsing APIs, high-performance queues. |
| **Real-World Analogy** | **Collections:** Using a specialized toolbox (e.g., organizer box for screws) instead of a single giant bucket. **Dataclasses:** A pre-printed form where you only fill in the fields, rather than drawing the layout from scratch. |

---

## Think of It This Way

### 1. Collections: Built-in structures, but smarter.
* **Counter:** Instead of writing a loop to count items and check if they exist in a dictionary, a `Counter` does it in one line.
* **defaultdict:** No more `KeyError`. If a key isn't there, it creates it automatically with a default value (like an empty list or `0`).
* **deque:** A double-ended queue. Standard lists are slow (`O(n)`) when inserting or removing from the beginning. `deque` is lightning fast (`O(1)`) from both ends.
* **namedtuple:** Tuples are great, but reading `person[2]` is unreadable. `namedtuple` lets you access fields by name (`person.age`) without class overhead.

### 2. Dataclasses: Class boilerplate is dead.
Normally, to write a class that stores data, you must write:
```python
class User:
    def __init__(self, username, email):
        self.username = username
        self.email = email
    def __repr__(self):
        return f"User({self.username}, {self.email})"
    def __eq__(self, other):
        return self.username == other.username and self.email == other.email
```
With `@dataclass`, Python writes all of that for you automatically.

---

## Step 1: The Collections Module

### Counter
```python
from collections import Counter

# Counts occurrences of each item automatically
counts = Counter("abracadabra")
print(counts)  # Counter({'a': 5, 'b': 2, 'r': 2, 'c': 1, 'd': 1})

# Get the most common items
print(counts.most_common(2))  # [('a', 5), ('b', 2)]
```

### defaultdict
```python
from collections import defaultdict

# Avoids KeyError by providing a default value factory
grouped_words = defaultdict(list)
words = ["apple", "banana", "apricot", "blueberry"]

for word in words:
    first_letter = word[0]
    grouped_words[first_letter].append(word)

print(grouped_words)  # defaultdict(<class 'list'>, {'a': ['apple', 'apricot'], 'b': ['banana', 'blueberry']})
```

### deque (Double-Ended Queue)
```python
from collections import deque

queue = deque(["Alice", "Bob"])
queue.append("Charlie")      # Add to right
queue.appendleft("Boss")     # Add to left (fast!)

print(queue)                 # deque(['Boss', 'Alice', 'Bob', 'Charlie'])
print(queue.popleft())       # Remove from left -> 'Boss'
```

### namedtuple
```python
from collections import namedtuple

# Create the type
Point = namedtuple("Point", ["x", "y"])

# Create an instance
p = Point(10, 20)
print(p.x, p.y)  # 10 20
# p.x = 30       # AttributeError! namedtuples are immutable like normal tuples
```

---

## Step 2: Dataclasses

```python
from dataclasses import dataclass, field

@dataclass
class Book:
    title: str
    author: str
    price: float
    # Default values work out of the box
    in_stock: bool = True
    # For mutable defaults (lists/dicts), we must use default_factory
    tags: list = field(default_factory=list)

b1 = Book("Python Basics", "AI Mentor", 29.99)
b2 = Book("Python Basics", "AI Mentor", 29.99)

print(b1)       # Book(title='Python Basics', author='AI Mentor', price=29.99, in_stock=True, tags=[])
print(b1 == b2) # True (Comparison logic generated automatically!)
```

### Post-Initialization (`__post_init__`)
Use `__post_init__` to perform validation or calculate properties after initialization:
```python
@dataclass
class Student:
    name: str
    score: int
    grade: str = field(init=False)  # We will calculate this ourselves

    def __post_init__(self):
        if self.score >= 90:
            self.grade = "A"
        elif self.score >= 80:
            self.grade = "B"
        else:
            self.grade = "C"

s = Student("Alice", 88)
print(s)  # Student(name='Alice', score=88, grade='B')
```

---

## Code Examples

### Easy Examples
```python
# 1. Counter from list
c = Counter([1, 2, 2, 3, 3, 3])
print(c[2])  # 2

# 2. defaultdict with int (defaults to 0)
scores = defaultdict(int)
scores["Alice"] += 10
print(scores["Bob"])  # 0 (does not raise KeyError)

# 3. Simple dataclass
@dataclass
class Coord:
    lat: float
    lon: float

c = Coord(37.77, -122.41)
print(c)  # Coord(lat=37.77, lon=-122.41)

# 4. Deque rotation
d = deque([1, 2, 3])
d.rotate(1)
print(d)  # deque([3, 1, 2])

# 5. namedtuple with defaults
Car = namedtuple("Car", ["make", "model", "year"])
# In Python 3.7+ we can add defaults
Car.__new__.__defaults__ = ("Toyota", "Corolla", 2020)
print(Car())  # Car(make='Toyota', model='Corolla', year=2020)
```

### Medium Examples
```python
# 6. Read-only (frozen) dataclass
@dataclass(frozen=True)
class Configuration:
    host: str
    port: int

config = Configuration("localhost", 8080)
# config.port = 9000  # FrozenInstanceError!

# 7. Finding matching bracket using deque
def check_brackets(s):
    d = deque()
    pairs = {')': '(', '}': '{', ']': '['}
    for char in s:
        if char in "({[":
            d.append(char)
        elif char in ")}]":
            if not d or d.pop() != pairs[char]:
                return False
    return len(d) == 0

print(check_brackets("({[]})")) # True

# 8. Counter arithmetic
c1 = Counter(a=3, b=1)
c2 = Counter(a=1, b=2)
print(c1 + c2)  # Counter({'a': 4, 'b': 3})
print(c1 - c2)  # Counter({'a': 2}) (keeps positive values only)

# 9. namedtuple representation subclassing
class UserProfile(namedtuple("UserProfile", ["username", "role"])):
    def is_admin(self):
        return self.role == "admin"

user = UserProfile("alice", "admin")
print(user.is_admin())  # True

# 10. Comparing Dataclasses
@dataclass(order=True)
class Player:
    # 'order=True' generates comparison methods (__lt__, __gt__, etc.)
    # Comparisons are done field by field, in order
    score: int
    name: str = field(compare=False) # Skip name during comparison

p1 = Player(100, "Alice")
p2 = Player(200, "Bob")
print(p1 < p2)  # True
```

### Advanced Examples
```python
# 11. Custom default_factory for nested collections in Dataclasses
@dataclass
class Library:
    name: str
    # Cannot use default=[] because it is shared across instances
    books: list = field(default_factory=list)

lib1 = Library("Central")
lib2 = Library("West")
lib1.books.append("Python Handbook")
print(lib2.books)  # [] (Correctly isolated!)

# 12. Deque sliding window maximum (O(N))
def sliding_window_max(nums, k):
    q = deque() # Stores indices of useful elements
    result = []
    for i, num in enumerate(nums):
        # Remove elements out of the current window
        if q and q[0] < i - k + 1:
            q.popleft()
        # Remove elements smaller than current element from back
        while q and nums[q[-1]] < num:
            q.pop()
        q.append(i)
        if i >= k - 1:
            result.append(nums[q[0]])
    return result

print(sliding_window_max([1, 3, -1, -3, 5, 3, 6, 7], 3))
# [3, 3, 5, 5, 6, 7]

# 13. Counter for word frequency parsing in text file
import re
text = "The quick brown fox jumps over the lazy dog. The dog barked."
words = re.findall(r'\w+', text.lower())
word_counts = Counter(words)
print(dict(word_counts.most_common(3)))
# {'the': 3, 'dog': 2, 'quick': 1}

# 14. Dataclass as_dict and as_tuple conversion
from dataclasses import asdict, astuple

@dataclass
class Coordinate:
    x: float
    y: float

coord = Coordinate(12.5, 45.6)
print(asdict(coord))  # {'x': 12.5, 'y': 45.6}
print(astuple(coord)) # (12.5, 45.6)

# 15. Inheritance in Dataclasses
@dataclass
class Person:
    name: str
    age: int

@dataclass
class Employee(Person):
    salary: float

emp = Employee("Alice", 30, 85000.0)
print(emp)  # Employee(name='Alice', age=30, salary=85000.0)
```

---

## Common Mistakes & Edge Cases

### 1. The Mutable Default Value Trap in Dataclasses
Just like function arguments, you cannot pass a mutable object (like a list or dict) as a default value directly in a dataclass declaration:
```python
# ❌ WRONG (will raise ValueError)
@dataclass
class User:
    roles: list = []

# ✅ CORRECT
@dataclass
class User:
    roles: list = field(default_factory=list)
```

### 2. Time Complexity of lists vs deques
Adding/removing to the front of a standard Python `list` is `O(n)` because all other elements have to be shifted. `deque.appendleft()` and `deque.popleft()` are `O(1)`.

### 3. Comparing dataclasses with different fields
If `order=True` is enabled, Python compares dataclasses like tuples: `(field1, field2, ...)`. If you don't want a field to be used in comparison, use `field(compare=False)`.

---

## Interview Questions (Top 5)

**Q1: What is the difference between `defaultdict` and a standard `dict`'s `setdefault()` method?**
> `defaultdict` automatically generates a default value when trying to access a missing key (`d[key]`). `setdefault(key, default)` requires you to pass the default value explicitly at call time, which can be verbose if done repeatedly. Furthermore, `defaultdict` is faster when handling many insertions of missing keys.

**Q2: What is the benefit of using a `deque` over a standard `list`?**
> A `deque` is a double-ended queue implemented as a doubly linked list under the hood. Appending or popping elements from either the start or end is `O(1)`. A standard Python `list` is a dynamic array; appending/popping from the end is `O(1)`, but inserting or deleting from the start is `O(n)` as all other elements must be shifted in memory.

**Q3: How does `@dataclass` generate the `__init__` method? Can you customize it?**
> `@dataclass` examines the class annotations and creates an `__init__` using those parameters in the order declared. You can customize initialization using `__post_init__(self)` to perform validation, compute derived fields, or set fields that were marked `init=False`.

**Q4: How do you make a dataclass immutable, and what are the benefits?**
> By using `@dataclass(frozen=True)`. This auto-generates `__setattr__` and `__delattr__` methods that raise a `FrozenInstanceError` upon modifications. It makes the instances hashable (meaning they can be keys in a dictionary or stored in a set) and prevents accidental modification of critical configurations.

**Q5: Can you change fields on a `namedtuple` after instantiation?**
> No, `namedtuple` is a subclass of the built-in `tuple`, which means it is immutable. To change a value, you must use the `_replace()` method, which returns a *new* copy of the namedtuple with the specified field changed.
> ```python
> p = Point(1, 2)
> p = p._replace(x=10) # returns new Point(10, 2)
> ```

---

*← [Previous: Chapter 8 (Recursion)](08_Recursion.md) | [Next: Chapter 10 (Regex) →](10_Regex.md)*
