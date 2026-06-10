# Chapter 4: Operators

---

## Topic Overview

| Aspect | Details |
|--------|---------|
| **Definition** | Operators are **special symbols or keywords** that perform operations on operands (values or variables). |
| **Why It Exists** | Operators enable computation, comparison, logical reasoning, and data manipulation — the core building blocks of any program. |
| **Where It Is Used** | Every expression in Python uses operators. From simple math to complex boolean logic in ML pipelines. |
| **Real-World Analogy** | Operators are like **verbs** in a sentence — they describe what action to perform on the **nouns** (operands). |

### Internal Working
Every operator in Python is backed by a **dunder (double underscore) method** on the object. When you write `a + b`, Python actually calls `a.__add__(b)`. This is why operators work differently on different types.

```python
# These are equivalent:
print(5 + 3)           # 8
print(int.__add__(5, 3))  # 8
print((5).__add__(3))     # 8

# String "addition" uses the same operator but different implementation
print("hello" + " world")           # "hello world"
print(str.__add__("hello", " world"))  # "hello world"
```

---

## 4.1 Arithmetic Operators

| Operator | Name | Example | Result | Dunder Method |
|----------|------|---------|--------|---------------|
| `+` | Addition | `5 + 3` | `8` | `__add__` |
| `-` | Subtraction | `5 - 3` | `2` | `__sub__` |
| `*` | Multiplication | `5 * 3` | `15` | `__mul__` |
| `/` | True Division | `7 / 2` | `3.5` | `__truediv__` |
| `//` | Floor Division | `7 // 2` | `3` | `__floordiv__` |
| `%` | Modulo | `7 % 2` | `1` | `__mod__` |
| `**` | Exponentiation | `2 ** 3` | `8` | `__pow__` |

### Important Rules & Edge Cases

```python
# True division ALWAYS returns float
print(6 / 3)       # 2.0 (not 2!)
print(type(6 / 3)) # <class 'float'>

# Floor division truncates TOWARDS NEGATIVE INFINITY
print(7 // 2)      # 3
print(-7 // 2)     # -4 (not -3!)
print(7 // -2)     # -4 (not -3!)

# Modulo follows: a = (a // b) * b + (a % b)
print(-7 % 2)      # 1 (not -1! Python ensures result has sign of divisor)
print(7 % -2)      # -1

# Exponentiation
print(2 ** -1)      # 0.5 (returns float for negative exponents)
print((-1) ** 0.5)  # Error concept: use cmath for complex results

# Operator overloading for strings and lists
print("ha" * 3)     # "hahaha"
print([1, 2] * 2)   # [1, 2, 1, 2]
print("hello" + " " + "world")  # "hello world"
print([1] + [2])    # [1, 2]
```

### Time Complexity
| Operation | Time Complexity |
|-----------|----------------|
| `int + int` | O(n) where n = number of digits for big ints; O(1) for small ints |
| `str + str` | O(n + m) where n, m are string lengths |
| `list + list` | O(n + m) |
| `int * int` | O(n²) for very large ints (Karatsuba for huge numbers) |
| `int ** int` | O(n²·log(exp)) via fast exponentiation |

---

## 4.2 Comparison (Relational) Operators

| Operator | Name | Example | Result | Dunder Method |
|----------|------|---------|--------|---------------|
| `==` | Equal | `5 == 5` | `True` | `__eq__` |
| `!=` | Not Equal | `5 != 3` | `True` | `__ne__` |
| `>` | Greater Than | `5 > 3` | `True` | `__gt__` |
| `<` | Less Than | `5 < 3` | `False` | `__lt__` |
| `>=` | Greater or Equal | `5 >= 5` | `True` | `__ge__` |
| `<=` | Less or Equal | `5 <= 3` | `False` | `__le__` |

### Chained Comparisons (Python Special Feature)
```python
# Python supports chained comparisons!
x = 5
print(1 < x < 10)     # True (same as: 1 < x and x < 10)
print(1 < x < 3)      # False
print(1 <= x <= 5)    # True

# Each value is evaluated ONCE
print(1 < 2 < 3 < 4)  # True (all conditions met)
print(1 < 3 > 2)      # True (1 < 3 and 3 > 2)

# Comparison between different types
print(5 == 5.0)        # True (numeric comparison)
print(5 is 5.0)        # False (different types!)
print("a" < "b")       # True (lexicographic/Unicode comparison)
print([1,2] < [1,3])   # True (element-by-element comparison)
```

### Important Rules
1. `==` compares **values**, `is` compares **identity** (memory address).
2. Different numeric types can be compared: `5 == 5.0` is `True`.
3. Strings are compared **lexicographically** (by Unicode code points).
4. Lists/tuples are compared **element by element**.
5. Comparing incompatible types with `<`, `>` raises `TypeError` (Python 3).

---

## 4.3 Logical Operators

| Operator | Description | Example | Result |
|----------|-------------|---------|--------|
| `and` | True if both True | `True and False` | `False` |
| `or` | True if either True | `True or False` | `True` |
| `not` | Inverts boolean | `not True` | `False` |

### How It Works Internally: Short-Circuit Evaluation

Python's `and` and `or` are **short-circuit operators** — they stop evaluating as soon as the result is determined. They also return the **actual value**, not necessarily `True`/`False`.

```python
# 'and' returns the first falsy value, or the last value if all truthy
print(1 and 2 and 3)     # 3 (all truthy, returns last)
print(1 and 0 and 3)     # 0 (first falsy value)
print(1 and "" and 3)    # "" (first falsy value)

# 'or' returns the first truthy value, or the last value if all falsy
print(0 or "" or 3)      # 3 (first truthy value)
print(0 or "" or None)   # None (all falsy, returns last)
print(1 or 2 or 3)       # 1 (first truthy value)

# Short-circuit in action
x = 5
print(x > 0 and x < 10)  # True (both evaluated)
print(x > 10 and x < 20) # False (x < 20 never evaluated!)

# Common pattern: default values
name = "" or "Unknown"
print(name)  # "Unknown"

# Common pattern: safe attribute access
# result = obj and obj.method()  # obj.method() only called if obj is truthy
```

### Truth Table

| A | B | A and B | A or B | not A |
|---|---|---------|--------|-------|
| True | True | True | True | False |
| True | False | False | True | False |
| False | True | False | True | True |
| False | False | False | False | True |

### Operator Precedence (for logical)
`not` > `and` > `or`

```python
# Without parentheses:
print(True or False and False)  # True (and binds tighter)
# Equivalent to: True or (False and False) → True or False → True

# Explicit is better:
print((True or False) and False)  # False
print(True or (False and False))  # True
```

---

## 4.4 Assignment Operators

| Operator | Example | Equivalent To |
|----------|---------|---------------|
| `=` | `x = 5` | Assign |
| `+=` | `x += 3` | `x = x + 3` |
| `-=` | `x -= 3` | `x = x - 3` |
| `*=` | `x *= 3` | `x = x * 3` |
| `/=` | `x /= 3` | `x = x / 3` |
| `//=` | `x //= 3` | `x = x // 3` |
| `%=` | `x %= 3` | `x = x % 3` |
| `**=` | `x **= 3` | `x = x ** 3` |
| `&=` | `x &= 3` | `x = x & 3` |
| `\|=` | `x \|= 3` | `x = x \| 3` |
| `^=` | `x ^= 3` | `x = x ^ 3` |
| `>>=` | `x >>= 3` | `x = x >> 3` |
| `<<=` | `x <<= 3` | `x = x << 3` |
| `:=` | `(x := 5)` | Walrus operator (assign + return) |

### Critical Difference: `+=` with Mutable vs Immutable
```python
# Immutable (int): += creates NEW object
x = 5
old_id = id(x)
x += 1
print(id(x) == old_id)  # False — new object!

# Mutable (list): += modifies IN-PLACE
lst = [1, 2]
old_id = id(lst)
lst += [3]  # Calls __iadd__, modifies in-place
print(id(lst) == old_id)  # True — same object!

# But: list = list + [...] creates NEW object
lst = [1, 2]
old_id = id(lst)
lst = lst + [3]  # Calls __add__, creates new list
print(id(lst) == old_id)  # False — new object!
```

---

## 4.5 Membership Operators

| Operator | Description | Example |
|----------|-------------|---------|
| `in` | True if value found in sequence | `"a" in "apple"` → `True` |
| `not in` | True if value NOT found | `"x" not in "apple"` → `True` |

### How It Works Internally
- For lists/tuples: **O(n)** linear search.
- For sets/frozensets: **O(1)** hash lookup.
- For dicts: checks **keys** only, O(1) hash lookup.
- For strings: substring search using optimized algorithm.
- Custom classes: implement `__contains__()`.

```python
# List (O(n))
print(3 in [1, 2, 3, 4, 5])    # True

# Set (O(1) — much faster for large collections!)
print(3 in {1, 2, 3, 4, 5})    # True

# Dict (checks keys only)
d = {"name": "Alice", "age": 25}
print("name" in d)              # True (key exists)
print("Alice" in d)             # False (values aren't checked!)
print("Alice" in d.values())   # True (explicitly check values)

# String (substring search)
print("Py" in "Python")        # True
print("py" in "Python")        # False (case-sensitive!)

# Membership with generator (lazy evaluation)
print(5 in (x**2 for x in range(10)))  # True (stops at 5, doesn't generate all)
```

### Time Complexity

| Container | `in` Time Complexity |
|-----------|---------------------|
| `list` | O(n) |
| `tuple` | O(n) |
| `set` | O(1) average |
| `frozenset` | O(1) average |
| `dict` | O(1) average (keys) |
| `str` | O(n·m) worst case (but highly optimized) |
| `range` | O(1)! (arithmetic check, not iteration) |

### Interview Trap: `range` membership is O(1)!
```python
# This does NOT iterate through the range!
print(999_999_999 in range(1_000_000_000))  # True — instant!
# range uses arithmetic: start <= value < stop and (value - start) % step == 0
```

---

## 4.6 Identity Operators

| Operator | Description | Example |
|----------|-------------|---------|
| `is` | True if same object | `a is b` |
| `is not` | True if different objects | `a is not b` |

### How It Works
`is` compares `id()` values — it checks if two names refer to the **exact same object in memory**.

```python
a = [1, 2, 3]
b = [1, 2, 3]
c = a

print(a == b)      # True (same value)
print(a is b)      # False (different objects!)
print(a is c)      # True (same object)
print(id(a), id(b), id(c))  # a and c share id

# Always use 'is' for None, True, False (they're singletons)
x = None
print(x is None)      # ✅ Correct
print(x == None)      # ⚠️ Works but not recommended
```

### When to Use `is` vs `==`
| Use Case | Use `is` | Use `==` |
|----------|----------|----------|
| Check for `None` | ✅ `x is None` | ❌ |
| Check for `True`/`False` | ✅ `x is True` | ❌ |
| Compare values | ❌ | ✅ `a == b` |
| Check same object | ✅ | ❌ |
| Singleton patterns | ✅ | ❌ |

---

## 4.7 Bitwise Operators

| Operator | Name | Example | Result | Description |
|----------|------|---------|--------|-------------|
| `&` | AND | `5 & 3` | `1` | Bits: `101 & 011 = 001` |
| `\|` | OR | `5 \| 3` | `7` | Bits: `101 \| 011 = 111` |
| `^` | XOR | `5 ^ 3` | `6` | Bits: `101 ^ 011 = 110` |
| `~` | NOT | `~5` | `-6` | Inverts all bits (two's complement) |
| `<<` | Left Shift | `5 << 1` | `10` | Bits: `101 → 1010` |
| `>>` | Right Shift | `5 >> 1` | `2` | Bits: `101 → 10` |

### How Bitwise Operators Work

```python
# Visual representation
a = 5   # Binary: 0101
b = 3   # Binary: 0011

print(f"a = {a:4d} = {a:08b}")
print(f"b = {b:4d} = {b:08b}")
print(f"a & b  = {a & b:4d} = {a & b:08b}")   # AND:  0001 = 1
print(f"a | b  = {a | b:4d} = {a | b:08b}")   # OR:   0111 = 7
print(f"a ^ b  = {a ^ b:4d} = {a ^ b:08b}")   # XOR:  0110 = 6
print(f"~a     = {~a:4d}")                      # NOT: -6 (two's complement)
print(f"a << 1 = {a << 1:4d} = {a << 1:08b}")  # Left shift: 1010 = 10
print(f"a >> 1 = {a >> 1:4d} = {a >> 1:08b}")  # Right shift: 0010 = 2

# Left shift = multiply by 2^n
print(5 << 3)   # 40 (5 × 2³ = 5 × 8 = 40)

# Right shift = integer divide by 2^n
print(40 >> 3)  # 5 (40 ÷ 2³ = 40 ÷ 8 = 5)
```

### Practical Uses
```python
# Check if number is even/odd
n = 7
print("Odd" if n & 1 else "Even")  # Odd (last bit is 1)

# Swap without temp variable (interview classic)
a, b = 5, 3
a ^= b    # a = 5 ^ 3 = 6
b ^= a    # b = 3 ^ 6 = 5
a ^= b    # a = 6 ^ 5 = 3
print(a, b)  # 3 5

# Set/clear/toggle specific bits
flags = 0b0000
flags |= 0b0001    # Set bit 0
flags |= 0b0100    # Set bit 2
print(f"Flags: {flags:04b}")  # 0101

flags &= ~0b0001   # Clear bit 0
print(f"Flags: {flags:04b}")  # 0100

flags ^= 0b0100    # Toggle bit 2
print(f"Flags: {flags:04b}")  # 0000

# Check if number is power of 2
def is_power_of_2(n):
    return n > 0 and (n & (n - 1)) == 0

print(is_power_of_2(16))   # True
print(is_power_of_2(18))   # False
```

---

## Operator Precedence (Complete)

| Priority | Operator | Description |
|----------|----------|-------------|
| 1 (highest) | `()` | Parentheses |
| 2 | `**` | Exponentiation |
| 3 | `~`, `+x`, `-x` | Unary NOT, unary plus/minus |
| 4 | `*`, `/`, `//`, `%` | Multiplication, division, modulo |
| 5 | `+`, `-` | Addition, subtraction |
| 6 | `<<`, `>>` | Bitwise shifts |
| 7 | `&` | Bitwise AND |
| 8 | `^` | Bitwise XOR |
| 9 | `\|` | Bitwise OR |
| 10 | `==`, `!=`, `>`, `<`, `>=`, `<=`, `is`, `in` | Comparisons |
| 11 | `not` | Logical NOT |
| 12 | `and` | Logical AND |
| 13 (lowest) | `or` | Logical OR |

### Memory Tip: **PEMDAS extended**
P → Parentheses → `()`
E → Exponentiation → `**`
U → Unary → `~, +, -`
M → Multiply/Divide → `*, /, //, %`
A → Add/Subtract → `+, -`
S → Shifts → `<<, >>`
B → Bitwise → `&, ^, |`
C → Comparisons → `==, !=, <, >, in, is`
L → Logical → `not, and, or`

---

## Common Mistakes

| Mistake | Why It's Wrong | Correct |
|---------|---------------|---------|
| `if x = 5:` | Assignment, not comparison | `if x == 5:` |
| `if x == None:` | Works but unreliable | `if x is None:` |
| `-7 // 2 = -3` | Floor div goes towards -∞ | Answer is `-4` |
| `-7 % 2 = -1` | Python mod follows divisor's sign | Answer is `1` |
| `2 ** 3 ** 2 = 64` | `**` is right-associative | Answer is `512` (2^9) |
| `True + True = True` | Bool arithmetic returns int | Answer is `2` |
| `not 1 == 2` | May misread precedence | `not (1 == 2)` → `True` |

---

## Interview Questions

**Q1: What is the difference between `/` and `//` in Python 3?**
> `/` is true division (always returns float): `7/2 = 3.5`. `//` is floor division (rounds towards negative infinity): `7//2 = 3`, `-7//2 = -4`.

**Q2: What is short-circuit evaluation?**
> `and` stops at the first falsy value; `or` stops at the first truthy value. They return the actual value, not just True/False. Example: `0 and print("never")` — `print` is never called.

**Q3: What does `a or b or c` return?**
> It returns the first truthy value, or the last value if all are falsy. Example: `0 or "" or 5` returns `5`. `0 or "" or None` returns `None`.

**Q4: Why is `**` right-associative?**
> `2 ** 3 ** 2` is evaluated as `2 ** (3 ** 2) = 2 ** 9 = 512`, not `(2 ** 3) ** 2 = 64`. This follows mathematical convention for exponentiation.

**Q5: How do bitwise operators work on negative numbers?**
> Python uses **two's complement** representation. `~x` equals `-(x+1)`. So `~5 = -6`. For negative numbers, Python treats them as having an infinite number of leading 1 bits.

---

## Code Examples

### Easy Examples

```python
# Example 1: All arithmetic operators
a, b = 17, 5
print(f"a + b  = {a + b}")    # 22
print(f"a - b  = {a - b}")    # 12
print(f"a * b  = {a * b}")    # 85
print(f"a / b  = {a / b}")    # 3.4
print(f"a // b = {a // b}")   # 3
print(f"a % b  = {a % b}")    # 2
print(f"a ** b = {a ** b}")   # 1419857

# Example 2: Comparison operators
x, y = 10, 20
print(f"x == y: {x == y}")    # False
print(f"x != y: {x != y}")    # True
print(f"x > y:  {x > y}")     # False
print(f"x < y:  {x < y}")     # True
print(f"x >= y: {x >= y}")    # False
print(f"x <= y: {x <= y}")    # True

# Example 3: Logical operators
print(True and True)     # True
print(True and False)    # False
print(True or False)     # True
print(not True)          # False

# Example 4: Membership operators
fruits = ["apple", "banana", "cherry"]
print("apple" in fruits)     # True
print("mango" not in fruits) # True

# Example 5: Identity operators
a = [1, 2]
b = [1, 2]
c = a
print(a is c)      # True
print(a is b)      # False
print(a is not b)  # True
```

### Medium Examples

```python
# Example 6: Short-circuit evaluation
def expensive():
    print("Called!")
    return True

# 'or' short-circuits — expensive() never called
result = True or expensive()  
# No "Called!" printed

# 'and' short-circuits — expensive() never called
result = False and expensive()
# No "Called!" printed

# Example 7: Chained comparisons
age = 25
print(18 <= age <= 65)  # True (working age)
print(0 < age < 120)    # True (valid age)

score = 85
grade = (
    "A" if 90 <= score <= 100 else
    "B" if 80 <= score < 90 else
    "C" if 70 <= score < 80 else
    "F"
)
print(f"Grade: {grade}")  # B

# Example 8: Bitwise flags
READ  = 0b100   # 4
WRITE = 0b010   # 2
EXEC  = 0b001   # 1

# Set permissions
permissions = READ | WRITE  # 6 (110)
print(f"Permissions: {permissions:03b}")  # 110

# Check permission
has_read = bool(permissions & READ)
has_exec = bool(permissions & EXEC)
print(f"Has read: {has_read}")   # True
print(f"Has exec: {has_exec}")   # False

# Add permission
permissions |= EXEC  # 7 (111)
print(f"After adding exec: {permissions:03b}")  # 111

# Remove permission
permissions &= ~WRITE  # 5 (101)
print(f"After removing write: {permissions:03b}")  # 101

# Example 9: Walrus operator usage
data = [1, 5, 3, 8, 2, 9, 4, 7, 6]

# Find first value > 5 and its square
if (val := next((x for x in data if x > 5), None)) is not None:
    print(f"First > 5: {val}, squared: {val**2}")  # 8, 64

# Filter and transform in one pass
results = [(y := x**2) for x in range(10) if y > 20]
# Actually, walrus in comprehension filter:
results = [y for x in range(10) if (y := x**2) > 20]
print(results)  # [25, 36, 49, 64, 81]

# Example 10: Modulo applications
# Check even/odd
for n in range(6):
    print(f"{n} is {'even' if n % 2 == 0 else 'odd'}")

# Wrap-around (circular index)
days = ["Mon", "Tue", "Wed", "Thu", "Fri", "Sat", "Sun"]
for offset in [0, 3, 7, 10, -1]:
    print(f"Day at offset {offset:+d}: {days[offset % 7]}")
```

### Advanced Examples

```python
# Example 11: Operator overloading
class Vector:
    def __init__(self, x, y):
        self.x = x
        self.y = y
    
    def __add__(self, other):
        return Vector(self.x + other.x, self.y + other.y)
    
    def __mul__(self, scalar):
        return Vector(self.x * scalar, self.y * scalar)
    
    def __eq__(self, other):
        return self.x == other.x and self.y == other.y
    
    def __repr__(self):
        return f"Vector({self.x}, {self.y})"

v1 = Vector(1, 2)
v2 = Vector(3, 4)
print(v1 + v2)     # Vector(4, 6)
print(v1 * 3)      # Vector(3, 6)
print(v1 == Vector(1, 2))  # True

# Example 12: Floor division edge cases
test_cases = [
    (7, 2), (-7, 2), (7, -2), (-7, -2),
    (7.5, 2), (-7.5, 2), (7, 0.5)
]

for a, b in test_cases:
    print(f"{a:>6} // {b:>4} = {a // b:>6}  |  {a:>6} % {b:>4} = {a % b:>6}")
    # Verify: a == (a // b) * b + (a % b)
    assert a == (a // b) * b + (a % b), "Invariant broken!"

# Example 13: Bitwise tricks for interviews
# Count set bits (population count)
def count_bits(n):
    count = 0
    while n:
        count += n & 1
        n >>= 1
    return count

print(f"Bits in 255: {count_bits(255)}")  # 8
print(f"Bits in 10:  {count_bits(10)}")   # 2 (1010)

# Find the only non-duplicate in a list (XOR trick)
def find_unique(nums):
    result = 0
    for n in nums:
        result ^= n
    return result

print(find_unique([1, 2, 3, 2, 1]))  # 3

# Example 14: Comparison operator chaining internals
# Python's chained comparison x < y < z is equivalent to:
# (x < y) and (y < z) BUT y is only evaluated ONCE

call_count = 0
def get_value():
    global call_count
    call_count += 1
    return 5

# In chained comparison, get_value() is called ONCE
call_count = 0
result = 1 < get_value() < 10
print(f"Calls: {call_count}")  # 1 (not 2!)

# Without chaining, it would be called twice:
call_count = 0
result = 1 < get_value() and get_value() < 10
print(f"Calls: {call_count}")  # 2

# Example 15: De Morgan's Laws in Python
a, b = True, False

# De Morgan's Law 1: not (A and B) == (not A) or (not B)
print(not (a and b) == (not a) or (not b))  # Need parentheses!
print((not (a and b)) == ((not a) or (not b)))  # True

# De Morgan's Law 2: not (A or B) == (not A) and (not B)
print((not (a or b)) == ((not a) and (not b)))   # True

# Practical: simplify conditions
items = [1, 2, 3]
# These are equivalent:
print(not (len(items) == 0))   # True
print(len(items) != 0)         # True
print(bool(items))             # True
print(not not items)           # True
```

---

## Edge Cases

1. **`2 ** 3 ** 2 = 512`** not `64`: `**` is right-associative.
2. **`-1 ** 2 = -1`** not `1`: Unary minus has lower precedence than `**`. Use `(-1) ** 2 = 1`.
3. **`True + True + True = 3`**: Bool is int subclass.
4. **`not 0 == 0`** is `True`: parsed as `not (0 == 0)` → `not True` → `False`. Wait — actually `not 0 == 0` → `not (0 == 0)` → `not True` → `False`. Tricky!
5. **`[] == []` is `True`** but **`[] is []` is `False`**: Equal but not identical.
6. **`range` membership is O(1)**: `999999 in range(1000000)` is instant.
7. **`//` with floats**: `7.5 // 2` = `3.0` (returns float if either operand is float).

---

## Practice Questions

### Easy

**Q1:** Calculate the area and perimeter of a rectangle with length 15 and width 8 using arithmetic operators.
```python
# Answer:
length, width = 15, 8
area = length * width
perimeter = 2 * (length + width)
print(f"Area: {area}")         # 120
print(f"Perimeter: {perimeter}")  # 46
```

**Q2:** Check if a number is even or odd using the modulo operator AND using bitwise AND.
```python
# Answer:
num = 17

# Using modulo
print(f"{num} is {'even' if num % 2 == 0 else 'odd'}")  # odd

# Using bitwise AND (faster!)
print(f"{num} is {'even' if not (num & 1) else 'odd'}")  # odd
```

### Medium

**Q3:** Write a program that finds all values in a list greater than the average using chained comparisons and the walrus operator.
```python
# Answer:
data = [45, 67, 23, 89, 12, 56, 78, 34, 91, 5]

avg = sum(data) / len(data)
above_avg = [x for x in data if x > avg]

print(f"Average: {avg:.1f}")
print(f"Above average: {above_avg}")

# With walrus (calculating avg inline):
above_avg2 = [x for x in data if x > (avg := sum(data) / len(data))]
print(f"Above average (walrus): {above_avg2}")
```

**Q4:** Implement a function that uses only bitwise operators to multiply a number by 7.
```python
# Answer:
def multiply_by_7(n):
    """n * 7 = n * 8 - n = (n << 3) - n"""
    return (n << 3) - n

# Test
for i in range(1, 11):
    result = multiply_by_7(i)
    assert result == i * 7, f"Failed for {i}"
    print(f"{i} × 7 = {result}")
```

### Advanced

**Q5:** Write a function that evaluates a simple mathematical expression string (e.g., "2 + 3 * 4") respecting operator precedence, without using `eval()`.
```python
# Answer (simplified — handles +, -, *, / with correct precedence):
def simple_eval(expression):
    """Evaluate expression with +, -, *, / respecting precedence."""
    tokens = expression.replace('+', ' + ').replace('-', ' - ').replace('*', ' * ').replace('/', ' / ').split()
    
    # First pass: handle * and /
    i = 0
    nums = [float(tokens[0])]
    ops = []
    
    i = 1
    while i < len(tokens):
        op = tokens[i]
        num = float(tokens[i + 1])
        
        if op in ('*', '/'):
            if op == '*':
                nums[-1] *= num
            else:
                nums[-1] /= num
        else:
            ops.append(op)
            nums.append(num)
        i += 2
    
    # Second pass: handle + and -
    result = nums[0]
    for i, op in enumerate(ops):
        if op == '+':
            result += nums[i + 1]
        else:
            result -= nums[i + 1]
    
    return result

# Test
print(simple_eval("2 + 3 * 4"))      # 14.0 (not 20)
print(simple_eval("10 - 2 * 3 + 1")) # 5.0
print(simple_eval("100 / 4 * 2"))    # 50.0
```

**Q6:** Explain why `[] == []` is `True` but `[] is []` is `False`. Then demonstrate a case where `is` gives unexpected results with small integers vs large integers.
```python
# Answer:
# == compares values, 'is' compares identity (memory address)
a = []
b = []
print(f"[] == []: {a == b}")   # True — same value (both empty)
print(f"[] is []: {a is b}")   # False — different objects in memory
print(f"id(a): {id(a)}, id(b): {id(b)}")  # Different IDs

# Small integer caching
x = 100
y = 100
print(f"\n100 is 100: {x is y}")  # True — cached (interned)

# Large integer (outside cache range)
x = 1000
y = 1000
# In the REPL: may be False (different objects)
# In a script: may be True (CPython optimization)
print(f"1000 is 1000: {x is y}")

# Demonstrating the boundary
for val in [256, 257]:
    a = val
    b = val
    # Force separate creation:
    a = int(f"{val}")
    b = int(f"{val}")
    print(f"{val}: a is b → {a is b}")
# 256: True (cached), 257: False (not cached)

# Lesson: NEVER use 'is' to compare values. Use '==' for values.
# Use 'is' ONLY for None, True, False (singletons).
```

---

*← [Previous: Data Types](03_Data_Types.md) | [Next: Input & Output →](05_Input_Output.md)*
