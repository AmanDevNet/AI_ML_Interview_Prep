# Chapter 10: Dictionaries

---

## Topic Overview

| Aspect | Details |
|--------|---------|
| **Definition** | A dictionary (`dict`) is an **ordered (Python 3.7+), mutable collection of key-value pairs**. Keys must be hashable and unique. |
| **Why It Exists** | Dicts provide O(1) key-based lookup — the most efficient way to map keys to values. |
| **Where It Is Used** | JSON processing, API responses, caching, configuration, databases, counting, grouping, ML feature maps. |
| **Real-World Analogy** | A dictionary is like a **phone book** — you look up a name (key) to find a phone number (value). |

### How It Works Internally
- Implemented as a **hash table** with open addressing.
- Since Python 3.6 (CPython) / 3.7 (language guarantee): **insertion order is preserved**.
- Hash table stores hashes, keys, and values in a compact array.
- Resizes when ~2/3 full to maintain performance.
- Keys must be **hashable** (immutable: int, str, tuple of immutables).

### Time Complexity

| Operation | Average | Worst |
|-----------|---------|-------|
| `d[key]` | O(1) | O(n) |
| `d[key] = value` | O(1) | O(n) |
| `del d[key]` | O(1) | O(n) |
| `key in d` | O(1) | O(n) |
| `d.get(key)` | O(1) | O(n) |
| `len(d)` | O(1) | O(1) |
| `d.keys()` / `.values()` / `.items()` | O(1) view creation | — |
| Iteration | O(n) | O(n) |

---

## Creation

```python
# Curly braces
d = {"name": "Alice", "age": 25, "city": "NYC"}

# dict() constructor
d = dict(name="Alice", age=25, city="NYC")

# From list of tuples
d = dict([("name", "Alice"), ("age", 25)])

# From keys with default value
d = dict.fromkeys(["a", "b", "c"], 0)   # {'a': 0, 'b': 0, 'c': 0}

# Dictionary comprehension
squares = {x: x**2 for x in range(6)}  # {0:0, 1:1, 2:4, 3:9, 4:16, 5:25}

# Empty dict
d = {}
d = dict()

# Nested dict
users = {
    "alice": {"age": 25, "role": "admin"},
    "bob": {"age": 30, "role": "user"},
}
print(users["alice"]["role"])  # "admin"

# Merge dicts (Python 3.9+)
d1 = {"a": 1, "b": 2}
d2 = {"b": 3, "c": 4}
merged = d1 | d2    # {'a': 1, 'b': 3, 'c': 4} — d2 wins on conflicts
d1 |= d2            # In-place merge
```

---

## Dictionary Methods

### get()
```python
d = {"name": "Alice", "age": 25}

# Direct access — raises KeyError if missing
print(d["name"])     # "Alice"
# d["email"]         # KeyError!

# get() — returns default if missing
print(d.get("name"))           # "Alice"
print(d.get("email"))          # None (default)
print(d.get("email", "N/A"))   # "N/A" (custom default)

# get() does NOT add the key to the dict
# Use setdefault() to get AND set if missing
```

### keys(), values(), items()
```python
d = {"name": "Alice", "age": 25, "city": "NYC"}

# These return VIEW objects (dynamic — reflect changes to dict)
print(d.keys())    # dict_keys(['name', 'age', 'city'])
print(d.values())  # dict_values(['Alice', 25, 'NYC'])
print(d.items())   # dict_items([('name', 'Alice'), ('age', 25), ('city', 'NYC')])

# Views are dynamic:
keys = d.keys()
d["email"] = "alice@example.com"
print(keys)  # dict_keys(['name', 'age', 'city', 'email']) — updated!

# Iteration patterns
for key in d:              # Iterate over keys (default)
    print(key)

for value in d.values():   # Iterate over values
    print(value)

for key, value in d.items():  # Iterate over key-value pairs
    print(f"{key}: {value}")
```

### update()
```python
d = {"a": 1, "b": 2}

# Update with another dict
d.update({"b": 20, "c": 3})
print(d)  # {'a': 1, 'b': 20, 'c': 3}

# Update with keyword arguments
d.update(d=4, e=5)
print(d)  # {'a': 1, 'b': 20, 'c': 3, 'd': 4, 'e': 5}

# Update with list of tuples
d.update([("f", 6)])
```

### pop()
```python
d = {"a": 1, "b": 2, "c": 3}

# Remove and return value
val = d.pop("b")     # 2, d = {'a': 1, 'c': 3}

# With default (no error if missing)
val = d.pop("z", "not found")  # "not found"

# Without default — KeyError if missing
# d.pop("z")  # KeyError
```

### popitem()
```python
d = {"a": 1, "b": 2, "c": 3}

# Remove and return LAST inserted pair (Python 3.7+)
pair = d.popitem()   # ('c', 3)
print(d)             # {'a': 1, 'b': 2}

# On empty dict → KeyError
```

### clear()
```python
d = {"a": 1, "b": 2}
d.clear()   # {}
```

### setdefault()
```python
d = {"a": 1}

# Get value if key exists, otherwise SET and return default
val = d.setdefault("a", 99)   # 1 (key exists, no change)
val = d.setdefault("b", 99)   # 99 (key added: d = {'a': 1, 'b': 99})

# Common pattern: building lists of grouped items
groups = {}
for name, dept in [("Alice", "Eng"), ("Bob", "Sales"), ("Charlie", "Eng")]:
    groups.setdefault(dept, []).append(name)
print(groups)  # {'Eng': ['Alice', 'Charlie'], 'Sales': ['Bob']}
```

### copy()
```python
d = {"a": 1, "b": [2, 3]}
shallow = d.copy()
shallow["b"].append(4)
print(d)  # {'a': 1, 'b': [2, 3, 4]} — nested list shared!

# Deep copy
import copy
deep = copy.deepcopy(d)
deep["b"].append(5)
print(d)  # {'a': 1, 'b': [2, 3, 4]} — unchanged
```

---

## Advanced Dict Patterns

```python
# Counting with dict
text = "hello world"
freq = {}
for char in text:
    freq[char] = freq.get(char, 0) + 1
print(freq)

# Better: Counter
from collections import Counter
freq = Counter(text)
print(freq.most_common(3))

# defaultdict
from collections import defaultdict
dd = defaultdict(list)
dd["fruits"].append("apple")   # No KeyError — auto-creates empty list
dd["fruits"].append("banana")
print(dd)  # {'fruits': ['apple', 'banana']}

# OrderedDict (preserved order before 3.7, still useful for reordering)
from collections import OrderedDict
od = OrderedDict()
od["c"] = 3
od["a"] = 1
od["b"] = 2
od.move_to_end("a")  # Move 'a' to end
print(list(od.keys()))  # ['c', 'b', 'a']

# Dict merging (Python 3.9+)
d1 = {"a": 1, "b": 2}
d2 = {"b": 3, "c": 4}
print(d1 | d2)   # {'a': 1, 'b': 3, 'c': 4}

# Pre-3.9 merging
merged = {**d1, **d2}  # Same result
```

---

## Common Mistakes

| Mistake | Example | Fix |
|---------|---------|-----|
| Direct access missing key | `d["missing"]` → KeyError | `d.get("missing", default)` |
| Mutable default in `fromkeys` | `dict.fromkeys(keys, [])` — shared list! | Use dict comprehension: `{k: [] for k in keys}` |
| Modifying dict during iteration | `for k in d: del d[k]` → RuntimeError | Iterate over `list(d.keys())` |
| Unhashable keys | `{[1,2]: "val"}` → TypeError | Use tuple: `{(1,2): "val"}` |
| Assuming order in old Python | Before 3.7, dict order wasn't guaranteed | Use Python 3.7+ |

---

## Interview Questions

**Q1: How are dictionaries implemented in Python?**
> Hash tables with open addressing. Keys are hashed, and stored alongside their values. Since Python 3.7, insertion order is preserved. Lookups are O(1) average.

**Q2: Why must dict keys be hashable?**
> Because the dict uses `hash(key)` to compute the storage location. Mutable objects (like lists) can change, which would change their hash, breaking the dict's internal structure.

**Q3: What is the difference between `d[key]` and `d.get(key)`?**
> `d[key]` raises `KeyError` if key is missing. `d.get(key)` returns `None` (or a custom default).

**Q4: Are Python dicts ordered?**
> Yes, since Python 3.7 (language guarantee). They maintain insertion order. In CPython 3.6, this was an implementation detail.

**Q5: What is `defaultdict`?**
> A dict subclass that calls a factory function (e.g., `list`, `int`, `set`) to provide default values for missing keys. `defaultdict(list)` auto-creates an empty list for any new key.

---

## Code Examples

### Easy Examples

```python
# Example 1: Basic CRUD
person = {"name": "Alice", "age": 25}
person["email"] = "alice@example.com"  # Create
print(person["name"])                   # Read
person["age"] = 26                      # Update
del person["email"]                     # Delete

# Example 2: Iterate
for key, val in person.items():
    print(f"{key}: {val}")

# Example 3: Check key existence
print("name" in person)    # True
print("email" in person)   # False

# Example 4: Dict from lists
keys = ["a", "b", "c"]
values = [1, 2, 3]
d = dict(zip(keys, values))
print(d)  # {'a': 1, 'b': 2, 'c': 3}

# Example 5: Nested access
config = {"db": {"host": "localhost", "port": 5432}}
print(config["db"]["host"])
```

### Medium Examples

```python
# Example 6: Word frequency counter
def word_count(text):
    words = text.lower().split()
    freq = {}
    for word in words:
        freq[word] = freq.get(word, 0) + 1
    return dict(sorted(freq.items(), key=lambda x: -x[1]))

print(word_count("the cat sat on the mat the cat"))

# Example 7: Invert a dictionary
d = {"a": 1, "b": 2, "c": 3}
inverted = {v: k for k, v in d.items()}
print(inverted)  # {1: 'a', 2: 'b', 3: 'c'}

# Example 8: Group by
from collections import defaultdict
students = [
    ("Alice", "Math"), ("Bob", "Science"),
    ("Charlie", "Math"), ("Dave", "Science"), ("Eve", "Math")
]
groups = defaultdict(list)
for name, dept in students:
    groups[dept].append(name)
print(dict(groups))

# Example 9: Safe nested access
def deep_get(d, *keys, default=None):
    for key in keys:
        if isinstance(d, dict):
            d = d.get(key, default)
        else:
            return default
    return d

config = {"db": {"host": "localhost", "port": 5432}}
print(deep_get(config, "db", "host"))      # localhost
print(deep_get(config, "db", "password"))   # None
print(deep_get(config, "cache", "host"))    # None

# Example 10: Dict comprehension filtering
scores = {"Alice": 85, "Bob": 62, "Charlie": 91, "Dave": 45, "Eve": 78}
passing = {k: v for k, v in scores.items() if v >= 70}
print(passing)  # {'Alice': 85, 'Charlie': 91, 'Eve': 78}
```

### Advanced Examples

```python
# Example 11: LRU Cache with OrderedDict
from collections import OrderedDict

class LRUCache:
    def __init__(self, capacity):
        self.cache = OrderedDict()
        self.capacity = capacity
    
    def get(self, key):
        if key in self.cache:
            self.cache.move_to_end(key)
            return self.cache[key]
        return -1
    
    def put(self, key, value):
        if key in self.cache:
            self.cache.move_to_end(key)
        self.cache[key] = value
        if len(self.cache) > self.capacity:
            self.cache.popitem(last=False)

cache = LRUCache(3)
cache.put("a", 1)
cache.put("b", 2)
cache.put("c", 3)
cache.get("a")      # Access 'a', making it recently used
cache.put("d", 4)   # Evicts 'b' (least recently used)
print(list(cache.cache.keys()))  # ['c', 'a', 'd']

# Example 12: fromkeys trap
# BAD — all keys share the SAME list!
bad = dict.fromkeys(["a", "b", "c"], [])
bad["a"].append(1)
print(bad)  # {'a': [1], 'b': [1], 'c': [1]} — all affected!

# GOOD — independent lists
good = {k: [] for k in ["a", "b", "c"]}
good["a"].append(1)
print(good)  # {'a': [1], 'b': [], 'c': []}

# Example 13: Merge multiple dicts with conflict resolution
def merge_dicts(*dicts, conflict="last"):
    result = {}
    for d in dicts:
        for k, v in d.items():
            if k in result:
                if conflict == "last":
                    result[k] = v
                elif conflict == "first":
                    pass  # keep existing
                elif conflict == "sum":
                    result[k] += v
                elif conflict == "list":
                    if isinstance(result[k], list):
                        result[k].append(v)
                    else:
                        result[k] = [result[k], v]
            else:
                result[k] = v
    return result

d1 = {"a": 1, "b": 2}
d2 = {"b": 3, "c": 4}
d3 = {"c": 5, "d": 6}
print(merge_dicts(d1, d2, d3, conflict="sum"))
# {'a': 1, 'b': 5, 'c': 9, 'd': 6}

# Example 14: Dict as switch/dispatch
def add(a, b): return a + b
def sub(a, b): return a - b
def mul(a, b): return a * b
def div(a, b): return a / b if b else None

operations = {"+": add, "-": sub, "*": mul, "/": div}

def calculate(expr):
    parts = expr.split()
    a, op, b = float(parts[0]), parts[1], float(parts[2])
    func = operations.get(op)
    if func:
        return func(a, b)
    return "Unknown operator"

print(calculate("10 + 5"))   # 15.0
print(calculate("10 * 3"))   # 30.0

# Example 15: Comparing dict views
d1 = {"a": 1, "b": 2, "c": 3}
d2 = {"b": 2, "c": 4, "d": 5}

# Key views support set operations
common_keys = d1.keys() & d2.keys()    # {'b', 'c'}
all_keys = d1.keys() | d2.keys()       # {'a', 'b', 'c', 'd'}
only_d1 = d1.keys() - d2.keys()        # {'a'}

# Items view with set operations
common_items = d1.items() & d2.items()  # {('b', 2)} — same key AND value
print(f"Common keys: {common_keys}")
print(f"Common items: {common_items}")
```

---

## Edge Cases

1. **`{}` is a dict, not a set**: Use `set()` for empty sets.
2. **`dict.fromkeys(keys, [])` shares the same list**: Use comprehension instead.
3. **`None` can be a key**: `{None: "value"}` is valid.
4. **`True` and `1` are the same key**: `{True: "a", 1: "b"}` → `{True: "b"}`.
5. **Modifying dict during iteration**: Raises `RuntimeError`. Iterate over `list(d.keys())`.
6. **Dict views are dynamic**: `d.keys()` reflects subsequent changes to `d`.

---

## Practice Questions

### Easy

**Q1:** Create a dictionary from two lists (one of keys, one of values) using `zip`.
```python
keys = ["name", "age", "city"]
values = ["Alice", 25, "NYC"]
d = dict(zip(keys, values))
print(d)  # {'name': 'Alice', 'age': 25, 'city': 'NYC'}
```

**Q2:** Write a function that counts the frequency of each character in a string.
```python
def char_freq(s):
    return {c: s.count(c) for c in set(s)}
print(char_freq("mississippi"))
```

### Medium

**Q3:** Write a function that flattens a nested dictionary.
```python
def flatten_dict(d, parent_key="", sep="."):
    items = []
    for k, v in d.items():
        new_key = f"{parent_key}{sep}{k}" if parent_key else k
        if isinstance(v, dict):
            items.extend(flatten_dict(v, new_key, sep).items())
        else:
            items.append((new_key, v))
    return dict(items)

nested = {"a": 1, "b": {"c": 2, "d": {"e": 3}}, "f": 4}
print(flatten_dict(nested))
# {'a': 1, 'b.c': 2, 'b.d.e': 3, 'f': 4}
```

**Q4:** Write a function that finds the two most common words in a text.
```python
from collections import Counter
def top_words(text, n=2):
    words = text.lower().split()
    return Counter(words).most_common(n)

print(top_words("the cat sat on the mat the cat likes the mat"))
# [('the', 4), ('cat', 2)]
```

### Advanced

**Q5:** Implement a dictionary that keeps track of the order items were last accessed.
```python
from collections import OrderedDict

class AccessOrderDict(OrderedDict):
    def __getitem__(self, key):
        self.move_to_end(key)
        return super().__getitem__(key)
    
    def get(self, key, default=None):
        if key in self:
            self.move_to_end(key)
            return super().__getitem__(key)
        return default

d = AccessOrderDict(a=1, b=2, c=3)
d["a"]    # Access 'a'
d["c"]    # Access 'c'
print(list(d.keys()))  # ['b', 'a', 'c']
```

**Q6:** Implement `deep_merge` that recursively merges nested dicts.
```python
def deep_merge(base, override):
    result = base.copy()
    for key, value in override.items():
        if key in result and isinstance(result[key], dict) and isinstance(value, dict):
            result[key] = deep_merge(result[key], value)
        else:
            result[key] = value
    return result

base = {"db": {"host": "localhost", "port": 5432}, "debug": False}
override = {"db": {"port": 3306, "name": "mydb"}, "debug": True}
print(deep_merge(base, override))
# {'db': {'host': 'localhost', 'port': 3306, 'name': 'mydb'}, 'debug': True}
```

---

*← [Previous: Sets](09_Sets.md) | [Next: Conditional Statements →](11_Conditional_Statements.md)*
