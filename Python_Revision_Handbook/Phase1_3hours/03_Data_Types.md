# Chapter 3: Data Types

---

## Topic Overview

| Aspect | Details |
|--------|---------|
| **Definition** | Data types define the **kind of data** an object can hold and the **operations** that can be performed on it. |
| **Why It Exists** | Types allow Python to know how to allocate memory, what operations are valid, and how to represent data internally. |
| **Where It Is Used** | Every value in Python has a type. Understanding types is fundamental to writing correct, efficient code. |
| **Real-World Analogy** | Data types are like different **containers** — a bottle holds liquid, a box holds solids, an envelope holds letters. Each container is designed for specific content. |

### Python's Type Hierarchy

```
object
├── int
├── float
├── complex
├── bool (subclass of int)
├── str
├── NoneType
├── list
├── tuple
├── set
├── frozenset
├── dict
├── bytes
├── bytearray
├── memoryview
├── range
├── type
└── function
```

---

## 3.1 int (Integer)

### Definition
The `int` type represents **whole numbers** of arbitrary precision — there is no overflow in Python.

### How It Works Internally
- CPython uses a **variable-length** array of digits (each "digit" is 30 bits on 64-bit systems).
- Small integers (-5 to 256) are **interned** (pre-created and cached).
- Integers are **immutable** — operations create new objects.
- Python seamlessly handles arbitrarily large integers (no `int` vs `long` distinction since Python 3).

### Memory Behavior
```python
import sys

print(sys.getsizeof(0))      # 28 bytes (base overhead)
print(sys.getsizeof(1))      # 28 bytes
print(sys.getsizeof(2**30))  # 32 bytes (needs more storage)
print(sys.getsizeof(2**60))  # 36 bytes
print(sys.getsizeof(2**90))  # 40 bytes
# Each additional "digit" adds 4 bytes
```

### Syntax
```python
# Decimal
x = 42

# Binary (prefix 0b or 0B)
b = 0b101010     # 42

# Octal (prefix 0o or 0O)
o = 0o52         # 42

# Hexadecimal (prefix 0x or 0X)
h = 0x2A         # 42

# Underscores for readability (Python 3.6+)
million = 1_000_000
binary = 0b_1010_0001

# Arbitrary precision
huge = 10 ** 1000  # 1 followed by 1000 zeros — no overflow!
```

### Important Rules
1. No size limit — Python handles bigints automatically.
2. Division of ints with `/` returns `float`: `5 / 2 = 2.5`.
3. Integer division with `//` returns `int`: `5 // 2 = 2`.
4. `bool` is a subclass of `int`: `True == 1`, `False == 0`.
5. Leading zeros are **not** allowed: `x = 042` → `SyntaxError`.

### Interview Questions

**Q1: Is there a maximum integer size in Python?**
> No. Python 3 integers have **arbitrary precision**. They grow as needed, limited only by available memory. This is different from C/Java where `int` is 32-bit.

**Q2: What is integer interning?**
> CPython caches integers from -5 to 256. When you write `x = 5`, Python reuses the pre-created object instead of creating a new one. This is an implementation detail, not a language guarantee.

---

## 3.2 float (Floating Point)

### Definition
The `float` type represents **real numbers** using double-precision (64-bit) IEEE 754 format.

### How It Works Internally
- 8 bytes (64 bits) per float: 1 sign bit, 11 exponent bits, 52 mantissa bits.
- Precision: approximately **15-17 significant decimal digits**.
- Range: approximately ±1.8 × 10^308.
- **Not** arbitrary precision — subject to **floating-point representation errors**.

### The Infamous Floating-Point Issue
```python
print(0.1 + 0.2)           # 0.30000000000000004
print(0.1 + 0.2 == 0.3)    # False!

# Why? 0.1 in binary is a repeating fraction (like 1/3 in decimal).
# The binary representation is approximately:
# 0.0001100110011001100110011001100110011001100110011010 (repeating)

# Solutions:
from decimal import Decimal
print(Decimal('0.1') + Decimal('0.2') == Decimal('0.3'))  # True

import math
print(math.isclose(0.1 + 0.2, 0.3))  # True
```

### Syntax
```python
# Standard notation
x = 3.14
y = -0.001

# Scientific notation
avogadro = 6.022e23    # 6.022 × 10²³
planck = 6.626e-34     # 6.626 × 10⁻³⁴

# From integer
z = float(42)          # 42.0

# Special values
pos_inf = float('inf')
neg_inf = float('-inf')
not_a_number = float('nan')

print(pos_inf > 10**308)  # True
print(float('nan') == float('nan'))  # False! NaN ≠ NaN
```

### Important Rules
1. `float` has **limited precision** — don't use for money/financial calculations.
2. Use `decimal.Decimal` for exact decimal arithmetic.
3. `float('nan') != float('nan')` — NaN is never equal to anything, including itself.
4. `float('inf') + 1 == float('inf')` — infinity absorbs finite values.
5. `int / int` returns `float`: `4 / 2 = 2.0` (not `2`).

### Interview Questions

**Q1: Why does `0.1 + 0.2 != 0.3`?**
> Because `0.1` and `0.2` cannot be represented exactly in binary (IEEE 754). The closest binary approximations are used, and their sum has a tiny rounding error. Use `math.isclose()` or `decimal.Decimal` for exact comparisons.

**Q2: How would you handle monetary calculations in Python?**
> Use `decimal.Decimal` with string initialization: `Decimal('19.99')`. Never use `float` for money. The `decimal` module provides exact decimal arithmetic with configurable precision.

---

## 3.3 complex (Complex Numbers)

### Definition
The `complex` type represents numbers with a **real part** and an **imaginary part**, written as `a + bj`.

### How It Works Internally
- Stored as two `float` values (real and imaginary parts).
- Uses `j` (not `i`) as the imaginary unit, following engineering convention.
- Both parts are IEEE 754 double-precision floats.

### Syntax
```python
z = 3 + 4j
print(z.real)    # 3.0
print(z.imag)    # 4.0
print(abs(z))    # 5.0 (magnitude = sqrt(3² + 4²))
print(z.conjugate())  # (3-4j)

# Complex from function
z2 = complex(2, 3)   # (2+3j)
z3 = complex("5+2j") # (5+2j) — no spaces around +!

# Arithmetic
z1 = 1 + 2j
z2 = 3 + 4j
print(z1 + z2)   # (4+6j)
print(z1 * z2)   # (-5+10j)  → (1·3 - 2·4) + (1·4 + 2·3)j
```

### Where It Is Used
- Signal processing, electrical engineering, quantum computing
- FFT (Fast Fourier Transform) in NumPy
- Control systems analysis

---

## 3.4 bool (Boolean)

### Definition
The `bool` type has exactly two values: `True` and `False`. It is a **subclass of `int`**.

### How It Works Internally
- `True` is equivalent to `1`, `False` to `0`.
- `bool` inherits from `int`: `isinstance(True, int)` → `True`.
- `True` and `False` are **singletons** — there's only one instance of each.
- Any object can be evaluated in a boolean context via its `__bool__()` or `__len__()` method.

### Truthiness and Falsiness

**Falsy values** (evaluate to `False`):
```python
bool(False)     # False
bool(0)         # False
bool(0.0)       # False
bool(0j)        # False
bool("")        # False (empty string)
bool([])        # False (empty list)
bool(())        # False (empty tuple)
bool({})        # False (empty dict)
bool(set())     # False (empty set)
bool(None)      # False
bool(range(0))  # False (empty range)
```

**Truthy values** (evaluate to `True`):
```python
bool(1)         # True (any non-zero number)
bool(-1)        # True
bool("hello")   # True (non-empty string)
bool([0])       # True (non-empty list, even if contents are falsy!)
bool(" ")       # True (space is a character)
```

### Important Interview Trap
```python
print(True + True)     # 2
print(True * 10)       # 10
print(False + 5)       # 5
print(sum([True, False, True, True]))  # 3 (counts True values!)
```

### Interview Questions

**Q1: Is `bool` a separate type or a subclass?**
> `bool` is a subclass of `int`. `True` equals `1` and `False` equals `0` in arithmetic. `isinstance(True, int)` returns `True`.

**Q2: What is the difference between `[]` and `[0]` in boolean context?**
> `bool([])` is `False` (empty container). `bool([0])` is `True` (non-empty container). It's the **emptiness** that matters, not the contents.

**Q3: What is the output of `True + True + False`?**
> `2`. Since `True` is `1` and `False` is `0`, this is `1 + 1 + 0 = 2`.

---

## 3.5 str (String)

### Definition
The `str` type represents **immutable sequences of Unicode characters**.

### How It Works Internally
- Python 3 strings are **Unicode** by default (UTF-8 in source code).
- Internally, CPython uses **compact representation** with one of three encodings:
  - Latin-1 (1 byte per char) if all chars are ASCII
  - UCS-2 (2 bytes per char) if any char needs 2 bytes
  - UCS-4 (4 bytes per char) if any char needs 4 bytes
- Strings are **immutable** — every modification creates a new string object.
- String **interning**: Python may cache small/common strings for performance.

### Syntax
```python
# Single quotes
s1 = 'hello'

# Double quotes
s2 = "hello"

# Triple quotes (multiline)
s3 = """This is a
multiline string"""

s4 = '''Another
multiline'''

# Raw strings (no escape processing)
path = r"C:\Users\name\folder"

# F-strings (formatted string literals)
name = "World"
greeting = f"Hello, {name}!"

# Byte strings
b = b"hello"  # bytes, not str
```

### Memory Behavior
```python
import sys

print(sys.getsizeof(""))       # 49 bytes (empty string overhead)
print(sys.getsizeof("a"))      # 50 bytes (1 ASCII char)
print(sys.getsizeof("hello"))  # 54 bytes (5 ASCII chars)
print(sys.getsizeof("é"))      # 76 bytes (needs UCS-2)
print(sys.getsizeof("🐍"))    # 80 bytes (needs UCS-4)
```

> Detailed string operations are covered in **Chapter 6: Strings**.

---

## 3.6 NoneType

### Definition
`None` is Python's **null value** — it represents the **absence of a value**. It is a **singleton** (only one `None` object exists).

### How It Works Internally
- `None` is an object of type `NoneType`.
- There is exactly **one** `None` object in memory (singleton).
- Functions that don't explicitly return a value return `None`.
- `None` is **falsy** in boolean context.

### Syntax and Usage
```python
x = None
print(type(x))      # <class 'NoneType'>
print(x is None)    # True — always use 'is' to check for None
print(x == None)    # True — but 'is' is preferred (more reliable)

# Functions return None by default
def greet():
    print("Hello")

result = greet()
print(result)        # None
print(result is None)  # True

# Common patterns
def find_item(lst, target):
    for item in lst:
        if item == target:
            return item
    return None  # Explicitly returning None (optional but clear)
```

### Important Rules
1. **Always use `is None`** instead of `== None` for checking.
2. `None` is **falsy**: `if not None:` is `True`.
3. `None` is a **singleton**: `id(None)` is always the same.
4. `None` is **not** the same as `0`, `""`, `[]`, or `False`.

### Interview Questions

**Q1: Why use `is None` instead of `== None`?**
> `is` checks identity (same object). Since `None` is a singleton, `is` is faster and more correct. A custom class could override `__eq__` to return `True` when compared with `None`, making `== None` unreliable:
```python
class Tricky:
    def __eq__(self, other):
        return True

t = Tricky()
print(t == None)   # True (misleading!)
print(t is None)   # False (correct!)
```

---

## 3.7 Mutable vs Immutable

### Definition
- **Immutable**: Object's state **cannot** be changed after creation. Any "modification" creates a new object.
- **Mutable**: Object's state **can** be changed in-place without creating a new object.

### Classification

| Type | Mutable? | Example |
|------|----------|---------|
| `int` | ❌ Immutable | `x = 5; x += 1` creates new int |
| `float` | ❌ Immutable | Same as int |
| `complex` | ❌ Immutable | Same |
| `bool` | ❌ Immutable | Same |
| `str` | ❌ Immutable | `s = "hi"; s[0] = "H"` → Error |
| `tuple` | ❌ Immutable | Cannot add/remove elements |
| `frozenset` | ❌ Immutable | Immutable version of set |
| `bytes` | ❌ Immutable | Immutable sequence of bytes |
| `NoneType` | ❌ Immutable | Singleton |
| `list` | ✅ Mutable | Can append, insert, delete |
| `dict` | ✅ Mutable | Can add/remove key-value pairs |
| `set` | ✅ Mutable | Can add/remove elements |
| `bytearray` | ✅ Mutable | Mutable sequence of bytes |

### How Immutability Works

```python
# String immutability
s = "hello"
# s[0] = "H"  # TypeError: 'str' object does not support item assignment

# But you can create a new string:
s = "H" + s[1:]  # "Hello" — new object
print(s)

# Proof that a new object is created:
x = "hello"
original_id = id(x)
x = x + " world"
print(id(x) == original_id)  # False — different object!
```

### Why Immutability Matters
1. **Hashability**: Only immutable objects can be dictionary keys or set elements.
2. **Thread safety**: Immutable objects are inherently thread-safe.
3. **Caching**: Python can safely cache/intern immutable objects.
4. **Function arguments**: Immutable arguments prevent unintended side effects.

### Interview Trap: Mutable Default Arguments
```python
# DANGEROUS — mutable default argument is shared across calls!
def append_to(item, lst=[]):
    lst.append(item)
    return lst

print(append_to(1))  # [1]
print(append_to(2))  # [1, 2] — NOT [2]! Same list object!
print(append_to(3))  # [1, 2, 3]

# CORRECT way:
def append_to_safe(item, lst=None):
    if lst is None:
        lst = []
    lst.append(item)
    return lst
```

---

## 3.8 Type Conversion & Type Casting

### Definition
- **Implicit conversion** (coercion): Python automatically converts types when safe.
- **Explicit conversion** (casting): You manually convert using type constructors.

### Implicit Conversion
```python
# int → float (safe promotion)
result = 5 + 3.14
print(result, type(result))  # 8.14 <class 'float'>

# bool → int (bool is subclass of int)
result = True + 5
print(result, type(result))  # 6 <class 'int'>

# int → complex
result = 5 + 2j
print(result, type(result))  # (5+2j) <class 'complex'>
```

### Explicit Conversion (Type Casting)
```python
# str → int
x = int("42")          # 42
x = int("101", 2)      # 5 (binary to decimal)
x = int("0xFF", 16)    # 255 (hex to decimal)

# str → float
y = float("3.14")      # 3.14
y = float("1e3")       # 1000.0

# int/float → str
s = str(42)             # "42"
s = str(3.14)           # "3.14"

# to bool
print(bool(0))          # False
print(bool(""))         # False
print(bool([]))         # False
print(bool(42))         # True
print(bool("hello"))    # True

# float → int (truncation, not rounding!)
print(int(3.7))         # 3 (not 4!)
print(int(-3.7))        # -3 (towards zero)

# For proper rounding:
print(round(3.7))       # 4
print(round(3.5))       # 4 (banker's rounding!)
print(round(4.5))       # 4 (banker's rounding!)
```

### Type Checking
```python
x = 42

# type() — exact type
print(type(x))           # <class 'int'>
print(type(x) == int)    # True
print(type(x) is int)    # True

# isinstance() — includes inheritance (preferred)
print(isinstance(x, int))       # True
print(isinstance(True, int))    # True (bool IS int)
print(isinstance(True, bool))   # True

# type() vs isinstance()
print(type(True) == int)         # False (type is bool, not int)
print(isinstance(True, int))    # True (bool inherits from int)

# Check multiple types
print(isinstance(x, (int, float)))  # True
```

### Important Interview Point
```python
# type() checks EXACT type
# isinstance() checks type hierarchy

print(type(True) is bool)    # True
print(type(True) is int)     # False

print(isinstance(True, bool))  # True
print(isinstance(True, int))   # True  ← includes parent class
```

---

## Code Examples

### Easy Examples

**Example 1: Identify types of different literals**
```python
values = [42, 3.14, 2+3j, True, "hello", None, [1,2], (1,2), {1,2}, {"a":1}]

for v in values:
    print(f"{str(v):15} → {type(v).__name__}")

# Output:
# 42              → int
# 3.14            → float
# (2+3j)          → complex
# True            → bool
# hello           → str
# None            → NoneType
# [1, 2]          → list
# (1, 2)          → tuple
# {1, 2}          → set
# {'a': 1}        → dict
```

**Example 2: Type conversion basics**
```python
# String to number
age = int("25")
price = float("19.99")
print(age + 5)       # 30
print(price * 2)     # 39.98
```

**Example 3: Check truthiness**
```python
test_values = [0, 1, -1, "", "hello", [], [0], {}, None, True, False]

for val in test_values:
    print(f"{str(val):10} → {'Truthy' if val else 'Falsy'}")
```

**Example 4: Integer representations**
```python
num = 255
print(f"Decimal: {num}")
print(f"Binary:  {bin(num)}")   # 0b11111111
print(f"Octal:   {oct(num)}")   # 0o377
print(f"Hex:     {hex(num)}")   # 0xff
```

**Example 5: NoneType checks**
```python
def divide(a, b):
    if b == 0:
        return None
    return a / b

result = divide(10, 0)
if result is None:
    print("Cannot divide by zero!")
else:
    print(f"Result: {result}")
```

### Medium Examples

**Example 6: Floating-point precision**
```python
# The classic issue
print(0.1 + 0.2)             # 0.30000000000000004
print(0.1 + 0.2 == 0.3)      # False

# Solution 1: math.isclose
import math
print(math.isclose(0.1 + 0.2, 0.3))  # True

# Solution 2: decimal.Decimal
from decimal import Decimal
print(Decimal('0.1') + Decimal('0.2') == Decimal('0.3'))  # True

# Solution 3: Round
print(round(0.1 + 0.2, 1) == 0.3)  # True

# Banker's rounding (round half to even)
print(round(0.5))  # 0
print(round(1.5))  # 2
print(round(2.5))  # 2
print(round(3.5))  # 4
```

**Example 7: Mutability demonstration**
```python
# Immutable (int)
a = 10
print(f"Before: id={id(a)}, value={a}")
a += 1  # Creates new object
print(f"After:  id={id(a)}, value={a}")  # Different id!

print()

# Mutable (list)
b = [1, 2, 3]
print(f"Before: id={id(b)}, value={b}")
b.append(4)  # Modifies in-place
print(f"After:  id={id(b)}, value={b}")  # Same id!
```

**Example 8: Type hierarchy exploration**
```python
# bool is int
print(issubclass(bool, int))    # True
print(isinstance(True, int))    # True
print(True == 1)                # True
print(True is 1)                # False (different objects)

# Method Resolution Order
print(bool.__mro__)
# (<class 'bool'>, <class 'int'>, <class 'object'>)
```

**Example 9: Complex number operations**
```python
z1 = 3 + 4j
z2 = 1 - 2j

print(f"Addition: {z1 + z2}")        # (4+2j)
print(f"Multiplication: {z1 * z2}")  # (11-2j)
print(f"Magnitude of z1: {abs(z1)}") # 5.0
print(f"Conjugate of z1: {z1.conjugate()}")  # (3-4j)

# Polar form
import cmath
r, phi = cmath.polar(z1)
print(f"Polar: r={r:.2f}, φ={phi:.2f} rad")
```

**Example 10: Comprehensive type casting**
```python
# All valid conversions
conversions = {
    "int('42')": int('42'),
    "int(3.9)": int(3.9),        # Truncates to 3
    "int(True)": int(True),      # 1
    "float('3.14')": float('3.14'),
    "float(42)": float(42),      # 42.0
    "str(42)": str(42),          # '42'
    "str(3.14)": str(3.14),      # '3.14'
    "bool(0)": bool(0),          # False
    "bool('')": bool(''),        # False
    "bool('0')": bool('0'),      # True! (non-empty string)
    "list('abc')": list('abc'),  # ['a', 'b', 'c']
    "tuple([1,2])": tuple([1,2]),  # (1, 2)
    "set([1,1,2])": set([1,1,2]),  # {1, 2}
}

for expr, result in conversions.items():
    print(f"{expr:25} → {result!r}")
```

### Advanced Examples

**Example 11: Memory size comparison across types**
```python
import sys

types_and_values = {
    'int(0)': 0,
    'int(1)': 1,
    'int(2**30)': 2**30,
    'int(2**100)': 2**100,
    'float(0.0)': 0.0,
    'complex(0,0)': 0+0j,
    'bool(True)': True,
    'str("")': "",
    'str("hello")': "hello",
    'None': None,
    'list()': [],
    'tuple()': (),
    'dict()': {},
    'set()': set(),
}

print(f"{'Type/Value':<20} {'Size (bytes)':>12}")
print("=" * 35)
for name, val in types_and_values.items():
    print(f"{name:<20} {sys.getsizeof(val):>12}")
```

**Example 12: Demonstrating hashability and immutability connection**
```python
# Immutable types are hashable (can be dict keys / set elements)
hashable_types = [42, 3.14, "hello", True, None, (1, 2, 3), frozenset([1, 2])]
for val in hashable_types:
    print(f"{str(val):20} hash={hash(val)}")

# Mutable types are NOT hashable
unhashable = [[1,2], {"a":1}, {1,2}]
for val in unhashable:
    try:
        hash(val)
    except TypeError as e:
        print(f"{type(val).__name__:20} → {e}")

# Exception: tuple containing mutable element
try:
    hash((1, [2, 3]))  # Tuple contains a list!
except TypeError as e:
    print(f"Tuple with list → {e}")
```

**Example 13: String interning behavior**
```python
# Short strings with only letters/digits/underscores may be interned
a = "hello"
b = "hello"
print(f"'hello': a is b → {a is b}")  # Usually True (interned)

# Strings with spaces typically aren't interned
a = "hello world"
b = "hello world"
print(f"'hello world': a is b → {a is b}")  # Usually False

# Force interning
import sys
a = sys.intern("hello world")
b = sys.intern("hello world")
print(f"interned 'hello world': a is b → {a is b}")  # True

# Compile-time vs runtime
a = "hello" + "world"    # Optimized at compile time
b = "helloworld"
print(f"Compile-time concat: a is b → {a is b}")  # True

c = "hello"
d = c + "world"          # Runtime concatenation
e = "helloworld"
print(f"Runtime concat: d is e → {d is e}")  # Usually False
```

**Example 14: Custom class truthiness**
```python
class Wallet:
    def __init__(self, balance):
        self.balance = balance
    
    def __bool__(self):
        """A wallet is truthy if it has money."""
        return self.balance > 0
    
    def __repr__(self):
        return f"Wallet({self.balance})"

empty_wallet = Wallet(0)
full_wallet = Wallet(100)

print(f"{empty_wallet}: bool → {bool(empty_wallet)}")  # False
print(f"{full_wallet}: bool → {bool(full_wallet)}")    # True

if full_wallet:
    print("You have money!")
else:
    print("Wallet is empty!")

# If __bool__ is not defined, __len__ is used
class Inventory:
    def __init__(self, items):
        self.items = items
    
    def __len__(self):
        return len(self.items)

inv = Inventory([])
print(f"Empty inventory: bool → {bool(inv)}")  # False (len=0)

inv = Inventory(["sword", "shield"])
print(f"Full inventory: bool → {bool(inv)}")   # True (len=2)
```

**Example 15: Numeric tower and type promotion**
```python
# Python's numeric tower: bool → int → float → complex

# bool + int → int
print(type(True + 1))       # <class 'int'>

# int + float → float
print(type(1 + 1.0))        # <class 'float'>

# float + complex → complex
print(type(1.0 + 1j))       # <class 'complex'>

# bool + float → float
print(type(True + 1.0))     # <class 'float'>

# bool + complex → complex
print(type(True + 1j))      # <class 'complex'>

# Demonstrating the promotion chain
import numbers
print(issubclass(bool, numbers.Integral))  # True
print(issubclass(int, numbers.Real))       # True
print(issubclass(float, numbers.Complex))  # True
```

---

## Edge Cases

1. **`bool('False')` is `True`**: Any non-empty string is truthy, even `"False"` or `"0"`.
2. **`int(3.9)` is `3`**: `int()` truncates towards zero, doesn't round.
3. **`int(-3.9)` is `-3`**: Truncation towards zero, not towards negative infinity.
4. **`round(0.5)` is `0`**: Python uses banker's rounding (round half to even).
5. **`float('nan') != float('nan')`**: NaN is never equal to itself (IEEE 754 rule).
6. **`type(True) is int`** is `False`: The exact type is `bool`, not `int`.
7. **`isinstance(True, int)`** is `True`: `bool` is a subclass of `int`.
8. **`hash(-1) == hash(-2)`**: In CPython, `-1` is reserved as an error indicator for hash, so `hash(-1)` returns `-2`.

---

## Comparison with Similar Concepts

| Feature | Python | Java | C | JavaScript |
|---------|--------|------|---|------------|
| Integer overflow | Never | Yes (32/64 bit) | Yes | No (BigInt) / Yes (Number) |
| Float precision | IEEE 754 double | IEEE 754 double | IEEE 754 double | IEEE 754 double |
| String encoding | Unicode (UTF-8 source) | UTF-16 | ASCII (char) | UTF-16 |
| Null value | `None` | `null` | `NULL` (pointer) | `null` / `undefined` |
| Boolean type | `bool` (int subclass) | `boolean` (primitive) | `_Bool` / `int` | `boolean` |
| Type checking | `isinstance()` | `instanceof` | N/A | `typeof` / `instanceof` |

---

## Memory Tips

- **"Everything is an object"** — Even `42` is an `int` object with methods.
- **"Immutable = Safe, Mutable = Careful"** — Immutables as dict keys, mutables need copying.
- **"Falsy Five: 0, 0.0, '', [], None"** — Zero, empty, or None = False.
- **"bool('False') is True"** — Only empty string is falsy. Content doesn't matter!
- **"int() truncates, round() rounds"** — `int(3.9)=3`, `round(3.9)=4`.
- **"-5 to 256"** — CPython's integer cache range.

---

## Practice Questions

### Easy

**Q1:** Write a program that takes a number as string input and converts it to int, float, and complex. Print each with its type.
```python
# Answer:
num_str = "42"

as_int = int(num_str)
as_float = float(num_str)
as_complex = complex(num_str)

print(f"int:     {as_int} ({type(as_int).__name__})")
print(f"float:   {as_float} ({type(as_float).__name__})")
print(f"complex: {as_complex} ({type(as_complex).__name__})")
# Output:
# int:     42 (int)
# float:   42.0 (float)
# complex: (42+0j) (complex)
```

**Q2:** Write a program that checks which of the following values are truthy and which are falsy: `0, 1, "", "0", [], [0], {}, None, True, False`.
```python
# Answer:
values = [0, 1, "", "0", [], [0], {}, None, True, False]

for val in values:
    status = "Truthy" if val else "Falsy"
    print(f"{str(val):10} → {status}")
# Output:
# 0          → Falsy
# 1          → Truthy
#            → Falsy
# 0          → Truthy    ← "0" is non-empty string!
# []         → Falsy
# [0]        → Truthy    ← non-empty list!
# {}         → Falsy
# None       → Falsy
# True       → Truthy
# False      → Falsy
```

### Medium

**Q3:** Demonstrate that `float` has precision issues and show three different ways to handle exact decimal arithmetic.
```python
# Answer:
# The problem
print(f"0.1 + 0.2 = {0.1 + 0.2}")  # 0.30000000000000004
print(f"0.1 + 0.2 == 0.3? {0.1 + 0.2 == 0.3}")  # False

# Solution 1: math.isclose()
import math
print(f"\nmath.isclose: {math.isclose(0.1 + 0.2, 0.3)}")  # True

# Solution 2: decimal.Decimal
from decimal import Decimal
result = Decimal('0.1') + Decimal('0.2')
print(f"Decimal: {result} == 0.3? {result == Decimal('0.3')}")  # True

# Solution 3: fractions.Fraction
from fractions import Fraction
result = Fraction(1, 10) + Fraction(2, 10)
print(f"Fraction: {result} == 3/10? {result == Fraction(3, 10)}")  # True
```

**Q4:** Write a function that determines whether a given value can be used as a dictionary key. Test it with various types.
```python
# Answer:
def can_be_dict_key(value):
    """Check if a value is hashable (required for dict keys)."""
    try:
        hash(value)
        return True
    except TypeError:
        return False

test_values = [
    42, 3.14, "hello", True, None,
    (1, 2, 3), frozenset([1, 2]),
    [1, 2], {"a": 1}, {1, 2},
    (1, [2, 3])  # tuple containing mutable!
]

for val in test_values:
    result = can_be_dict_key(val)
    print(f"{str(val):25} → {'✅ Hashable' if result else '❌ Not hashable'}")
# ints, floats, strings, bools, None, tuples (of immutables), frozensets → Hashable
# lists, dicts, sets, tuples containing lists → Not hashable
```

### Advanced

**Q5:** Write a program that explores integer interning by finding the exact boundary where CPython stops caching integers. Also demonstrate string interning behavior.
```python
# Answer:
# Integer interning boundary
print("=== Integer Interning ===")
for i in range(254, 260):
    a = i
    b = i
    # Using a trick to avoid compile-time optimization
    a = int(str(i))
    b = int(str(i))
    print(f"{i}: same object? {a is b}")

# In CPython: -5 to 256 are cached
# 254: True, 255: True, 256: True, 257: False, 258: False, 259: False

print("\n=== Negative boundary ===")
for i in range(-7, -3):
    a = int(str(i))
    b = int(str(i))
    print(f"{i}: same object? {a is b}")

# String interning
print("\n=== String Interning ===")
import sys

# Simple identifiers are interned
a = "hello"
b = "hello"
print(f"'hello': {a is b}")  # True

# Strings with special chars usually aren't
a = "hello world"
b = "hello world"
print(f"'hello world': {a is b}")  # May be False

# Manual interning
a = sys.intern("hello world!")
b = sys.intern("hello world!")
print(f"interned 'hello world!': {a is b}")  # True
```

**Q6:** Create a comprehensive type conversion matrix showing which conversions are valid, which raise errors, and what the results are. Include at least 20 conversions.
```python
# Answer:
conversions = [
    ("int('42')", lambda: int('42')),
    ("int('3.14')", lambda: int('3.14')),          # ValueError!
    ("int(3.14)", lambda: int(3.14)),              # 3 (truncation)
    ("int('')", lambda: int('')),                   # ValueError!
    ("int(True)", lambda: int(True)),              # 1
    ("int(None)", lambda: int(None)),              # TypeError!
    ("float('inf')", lambda: float('inf')),        # inf
    ("float('nan')", lambda: float('nan')),        # nan
    ("float(True)", lambda: float(True)),          # 1.0
    ("float(None)", lambda: float(None)),          # TypeError!
    ("str(None)", lambda: str(None)),              # 'None'
    ("str([1,2])", lambda: str([1,2])),            # '[1, 2]'
    ("bool('')", lambda: bool('')),                # False
    ("bool('False')", lambda: bool('False')),      # True!
    ("bool([])", lambda: bool([])),                # False
    ("bool([False])", lambda: bool([False])),      # True!
    ("list('abc')", lambda: list('abc')),           # ['a','b','c']
    ("list(123)", lambda: list(123)),               # TypeError!
    ("tuple({1:2})", lambda: tuple({1:2})),         # (1,) keys only
    ("int('0xFF', 16)", lambda: int('0xFF', 16)),   # 255
]

print(f"{'Expression':<25} {'Result':<20} {'Type':<15}")
print("=" * 60)
for expr, func in conversions:
    try:
        result = func()
        print(f"{expr:<25} {str(result):<20} {type(result).__name__:<15}")
    except (ValueError, TypeError) as e:
        print(f"{expr:<25} {'ERROR':<20} {type(e).__name__}: {e}")
```

---

*← [Previous: Variables](02_Variables.md) | [Next: Operators →](04_Operators.md)*
