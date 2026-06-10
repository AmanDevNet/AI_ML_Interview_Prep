# Chapter 14: Copy Module — "Shallow vs Deep Copying"

---

## Topic Overview

| Aspect | Details |
|--------|---------|
| **Definition** | The `copy` module provides operations to copy objects. It distinguishes between **shallow copy** (copies container, references nested items) and **deep copy** (recursively copies everything). |
| **Why It Exists** | Simply assigning an object to a new variable (`a = b`) does **not** copy the object; it only creates a new reference. Modifying one affects the other. |
| **Where It Is Used** | Preserving original state before transformations, copying configuration objects, undo/redo state management, recursive data structure handling. |
| **Real-World Analogy** | **Assignment:** Creating a shortcut on your desktop. Both open the exact same file. **Shallow Copy:** Printing a photocopy of a page containing links to websites. You get a new page, but the links still point to the same global websites. **Deep Copy:** Building a complete replica of a house, down to every custom piece of furniture inside. |

---

## Think of It This Way

In Python, everything is an object. When you write:
```python
list_a = [1, 2, 3]
list_b = list_a  # Reference Assignment
```
You have **not** created two lists. You have created one list that has **two names** (`list_a` and `list_b`).

```python
list_b.append(4)
print(list_a)  # [1, 2, 3, 4]  <- list_a modified too!
print(list_a is list_b)  # True (same memory ID)
```

To create a brand new copy of `list_a`, you need to copy it:

* **Shallow Copy:** Copies the list itself, but if the list has nested lists (like `[[1, 2], [3, 4]]`), only the outer list is copied. The inner lists are still shared!
* **Deep Copy:** Copies the list *and* recursively copies all nested lists, so nothing is shared.

---

## Step 1: Assignment vs Shallow Copy vs Deep Copy

Let's see what happens to a nested list `original = [[1, 2], 3]`:

```python
import copy

original = [[1, 2], 3]

# 1. Assignment (Shared reference)
assigned = original

# 2. Shallow Copy
shallow = copy.copy(original) 
# (You can also do shallow = original.copy() or shallow = original[:])

# 3. Deep Copy
deep = copy.deepcopy(original)

# --- Perform modifications ---
original[0][0] = 99  # Modify nested item
original[1] = 100    # Modify flat item

# --- Check results ---
print(assigned)  # [[99, 2], 100]  <- Both nested and flat items changed!
print(shallow)   # [[99, 2], 3]    <- Nested item changed, but flat item is safe!
print(deep)      # [[1, 2], 3]     <- Completely untouched!
```

---

## Identity (`is`) vs Equality (`==`)

* **`==` (Equality):** Checks if the **values** of the objects are the same.
* **`is` (Identity):** Checks if they point to the **exact same memory address** (i.e. `id(a) == id(b)`).

```python
a = [1, 2]
b = [1, 2]

print(a == b)  # True (values are equal)
print(a is b)  # False (different objects in memory)

c = a
print(a is c)  # True (points to the same object)
```

---

## Code Examples

### Easy Examples
```python
# 1. Slicing as shallow copy
lst1 = [1, 2, 3]
lst2 = lst1[:]
lst2[0] = 99
print(lst1)  # [1, 2, 3] (safe!)

# 2. Dictionary .copy() is shallow
dict1 = {"name": "Alice", "skills": ["Python"]}
dict2 = dict1.copy()
dict2["skills"].append("SQL")
print(dict1["skills"])  # ['Python', 'SQL'] (nested list shared!)

# 3. copy.copy on flat custom object
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

p1 = Point(1, 2)
p2 = copy.copy(p1)
print(p1 is p2)  # False (new instance)
print(p1.x == p2.x) # True

# 4. Copying immutable objects (Optimization)
# Python optimizes memory by returning the original object when copying immutable types
s1 = "hello"
s2 = copy.copy(s1)
print(s1 is s2)  # True (strings are immutable, no need to replicate)

# 5. List constructor shallow copy
a = [1, 2]
b = list(a)
print(a is b)  # False
```

### Medium Examples
```python
# 6. Deep copying a custom object containing list
class Person:
    def __init__(self, name, hobbies):
        self.name = name
        self.hobbies = hobbies

hobbies = ["running", "reading"]
p1 = Person("Bob", hobbies)

# Shallow copy of object
p_shallow = copy.copy(p1)
p_shallow.hobbies.append("coding")
print(p1.hobbies)  # ['running', 'reading', 'coding'] (shared list)

# Deep copy of object
p2 = Person("Bob", ["running", "reading"])
p_deep = copy.deepcopy(p2)
p_deep.hobbies.append("gaming")
print(p2.hobbies)  # ['running', 'reading'] (safe!)

# 7. Modifying nested dictionaries
original_dict = {"prod": {"price": 100}, "category": "books"}
copied_dict = copy.copy(original_dict)
copied_dict["prod"]["price"] = 200
print(original_dict["prod"]["price"])  # 200 (nested dict is shared!)

# 8. Shallow copying sets
set_a = {1, 2, (3, 4)} # contains tuple (immutable)
set_b = set_a.copy()
print(set_a is set_b) # False

# 9. Shallow copy of complex nesting of immutable items
# Since ints, floats, strings, tuples are immutable, shallow copies are perfectly safe for them
nested_tuple = (1, 2, (3, 4))
copied_tuple = copy.copy(nested_tuple)
print(nested_tuple is copied_tuple) # True (whole tuple is optimized)

# 10. Copying list of lists using list comprehension
# This is a faster alternative to deepcopy for simple 2D lists
original_matrix = [[1, 2], [3, 4]]
copied_matrix = [row.copy() for row in original_matrix]
copied_matrix[0][0] = 99
print(original_matrix[0][0])  # 1 (safe!)
```

### Advanced Examples
```python
# 11. Overriding custom copy logic with __copy__ and __deepcopy__
# Sometimes objects contain resources (db connections, open sockets) that cannot be copied.
class DatabaseRecord:
    def __init__(self, data, connection):
        self.data = data
        self.connection = connection  # cannot copy this connection reference

    def __copy__(self):
        # Return new record sharing the same connection
        return DatabaseRecord(self.data, self.connection)

    def __deepcopy__(self, memo):
        # Copy the data, but keep same database connection reference
        # 'memo' is a dict that keeps track of already copied objects to avoid infinite recursion
        return DatabaseRecord(copy.deepcopy(self.data, memo), self.connection)

# 12. Handling cyclic references in deepcopy
# deepcopy is smart enough to handle objects that reference each other without infinite loops
class Node:
    def __init__(self, val):
        self.val = val
        self.next = None

node1 = Node(1)
node2 = Node(2)
node1.next = node2
node2.next = node1  # Circular reference!

copied_nodes = copy.deepcopy(node1)
print(copied_nodes.next.next is copied_nodes)  # True (resolved cyclic references correctly!)

# 13. Deep copy vs pickle serialization
# pickling and unpickling is sometimes used as a faster alternative to deepcopy for massive structures
import pickle
def pickle_copy(obj):
    return pickle.loads(pickle.dumps(obj))

data = {"nested": [1, 2, 3]}
p_copy = pickle_copy(data)
print(p_copy == data and p_copy is not data) # True

# 14. Shallow copying objects with slots (__slots__)
# copy module natively supports classes using __slots__
class SlottedPoint:
    __slots__ = ['x', 'y']
    def __init__(self, x, y):
        self.x = x
        self.y = y

pt = SlottedPoint(5, 10)
copied_pt = copy.copy(pt)
print(copied_pt.x, copied_pt.y) # 5 10

# 15. The memo dictionary in deepcopy
# Demonstrating how memo prevents duplicating shared child nodes
class GraphNode:
    def __init__(self, name):
        self.name = name
        self.edges = []

shared_child = GraphNode("Child")
root = GraphNode("Root")
root.edges.append(shared_child)
root.edges.append(shared_child) # Root references shared_child twice

# Deep copy
copied_root = copy.deepcopy(root)
print(copied_root.edges[0] is copied_root.edges[1])  
# True! (deepcopy kept it as a single shared object instead of duplicating it, thanks to the memo dict)
```

---

## Common Mistakes & Edge Cases

### 1. Expecting `copy.copy` to be safe for nested data
This is the most common bug in Python code. Developers make a copy using `list.copy()` or `[:]` and assume the original list is protected. If the list contains sublists, dicts, or user-defined objects, the references are shared.

### 2. Copying thread locks or sockets
Attempting to deepcopy objects representing system resources, thread locks, network sockets, or open file streams will raise a `TypeError: cannot pickle` or `TypeError: cannot deepcopy`. You must implement custom `__copy__` or `__deepcopy__` to exclude or re-open these resources.

---

## Interview Questions (Top 5)

**Q1: What is the difference between reference assignment (`=`), shallow copy, and deep copy?**
> * **Assignment (`=`)** copies the reference to the object. Both variables point to the exact same location in memory; changes to either affect both.
> * **Shallow Copy (`copy.copy()`)** creates a new outer collection object, but populates it with references to the child objects of the original container. Nested elements remain shared.
> * **Deep Copy (`copy.deepcopy()`)** recursively duplicates both the outer container and all nested objects, creating a completely independent copy.

**Q2: How does the `is` operator differ from the `==` operator?**
> `==` checks for value equality. If two objects have the same contents, `==` returns `True`. `is` checks for reference identity. It returns `True` only if both variables point to the exact same object in memory (sharing the same address ID).

**Q3: How does `copy.deepcopy()` handle circular references (e.g. self-referencing objects)?**
> `copy.deepcopy()` maintains a `memo` dictionary. The `memo` dict tracks all objects that have already been copied in the current session. If it encounters an object it has already processed, it simply returns the reference to the already-copied version, preventing infinite recursion.

**Q4: Why does `copy.copy("hello")` return the original string instead of a new one?**
> Because strings are immutable in Python. Since they cannot be modified in-place, there is no risk of side effects from sharing the reference. Python optimizes memory by avoiding copying immutable objects (like strings, ints, floats, and tuples) during copy operations.

**Q5: How can you customize how a custom class is copied by the `copy` module?**
> By defining the magic methods `__copy__(self)` and `__deepcopy__(self, memo)` within your class. This allows you to override default cloning behaviors—for instance, to skip copying specific fields like system sockets or database connections.

---

*← [Previous: Chapter 13 (Context Managers)](13_Context_Managers.md) | [Next: Chapter 15 (Time Complexity & Memory Optimization) →](15_Time_Complexity_Memory.md)*
