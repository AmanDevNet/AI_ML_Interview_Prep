# Chapter 5: Input & Output

---

## Topic Overview

| Aspect | Details |
|--------|---------|
| **Definition** | I/O (Input/Output) refers to reading data from the user and displaying results. `input()` reads from stdin, `print()` writes to stdout. |
| **Why It Exists** | Programs need to interact with users — accepting data and presenting results. |
| **Where It Is Used** | CLI tools, scripts, prototyping, competitive programming, interview coding rounds. |
| **Real-World Analogy** | `input()` is like a **microphone** (receives data), `print()` is like a **speaker** (outputs data). |

---

## 5.1 input()

### Definition
`input()` reads a line of text from standard input (keyboard) and returns it as a **string**.

### How It Works Internally
1. Displays the prompt string (if provided).
2. Reads characters until the user presses Enter.
3. Strips the trailing newline.
4. Returns the result as a `str` — **always a string**, never a number.

### Syntax
```python
name = input("Enter your name: ")
print(f"Hello, {name}!")

# input() ALWAYS returns a string
age = input("Enter your age: ")
print(type(age))  # <class 'str'> — NOT int!

# Convert to int/float explicitly
age = int(input("Enter your age: "))
price = float(input("Enter price: "))
```

### Important Rules
1. `input()` **always returns a string**.
2. Pass prompt text as an argument: `input("Enter value: ")`.
3. To get numbers, wrap with `int()` or `float()`.
4. Empty input returns an empty string `""`.
5. `input()` blocks execution until Enter is pressed.

### Common Patterns
```python
# Reading multiple values in one line
a, b = input("Enter two numbers: ").split()
a, b = int(a), int(b)

# Or more Pythonically:
a, b = map(int, input("Enter two numbers: ").split())

# Reading a list of numbers
nums = list(map(int, input("Enter numbers: ").split()))

# Yes/No input
response = input("Continue? (y/n): ").strip().lower()
if response in ('y', 'yes'):
    print("Continuing...")
```

---

## 5.2 print()

### Definition
`print()` outputs data to the console (stdout). It converts objects to strings and writes them.

### Function Signature
```python
print(*objects, sep=' ', end='\n', file=sys.stdout, flush=False)
```

### Parameters

| Parameter | Default | Description |
|-----------|---------|-------------|
| `*objects` | — | Zero or more objects to print |
| `sep` | `' '` | Separator between objects |
| `end` | `'\n'` | String appended after the last object |
| `file` | `sys.stdout` | Output stream |
| `flush` | `False` | Whether to flush the stream |

### How It Works Internally
1. Each object is converted to a string using `str()`.
2. Objects are joined with the `sep` string.
3. The `end` string is appended.
4. The result is written to `file`.
5. If `flush=True`, the output buffer is flushed immediately.

### Examples
```python
# Basic
print("Hello, World!")

# Multiple arguments
print("Name:", "Alice", "Age:", 25)   # Name: Alice Age: 25

# Custom separator
print("2024", "01", "15", sep="-")    # 2024-01-15
print(1, 2, 3, 4, 5, sep=" → ")      # 1 → 2 → 3 → 4 → 5

# Custom end
print("Loading", end="...")
print("Done!")
# Output: Loading...Done!

# Print to file
with open("output.txt", "w") as f:
    print("Hello, File!", file=f)

# Flush (useful for progress indicators)
import time
for i in range(5):
    print(f"\rProgress: {i+1}/5", end="", flush=True)
    time.sleep(0.5)
print()  # Final newline

# Print nothing (just a newline)
print()

# Print with repr (shows quotes, escape chars)
print(repr("hello\tworld"))  # 'hello\tworld'
print(ascii("café"))         # 'caf\xe9'
```

---

## 5.3 String Formatting

### Method 1: f-strings (Formatted String Literals) — Python 3.6+ ⭐ RECOMMENDED

```python
name = "Alice"
age = 25
pi = 3.14159265

# Basic interpolation
print(f"Name: {name}, Age: {age}")

# Expressions inside braces
print(f"In 5 years: {age + 5}")
print(f"Name upper: {name.upper()}")
print(f"2 + 3 = {2 + 3}")

# Format specifiers
print(f"Pi: {pi:.2f}")           # Pi: 3.14
print(f"Pi: {pi:.4f}")           # Pi: 3.1416
print(f"Percentage: {0.856:.1%}") # Percentage: 85.6%

# Width and alignment
print(f"{'left':<20}")           # left                (left-aligned)
print(f"{'right':>20}")          # right               (right-aligned)
print(f"{'center':^20}")         #        center        (centered)
print(f"{'padded':*^20}")        # *******padded******* (centered, padded with *)

# Number formatting
num = 1234567
print(f"{num:,}")                # 1,234,567 (thousands separator)
print(f"{num:_}")                # 1_234_567 (underscore separator)

# Binary, octal, hex
n = 255
print(f"Binary: {n:b}")         # 11111111
print(f"Octal:  {n:o}")         # 377
print(f"Hex:    {n:x}")         # ff
print(f"Hex:    {n:X}")         # FF
print(f"Hex:    {n:#x}")        # 0xff

# Debugging (Python 3.8+) — self-documenting expressions
x = 42
print(f"{x = }")                # x = 42
print(f"{x + 1 = }")            # x + 1 = 43
print(f"{name = !r}")           # name = 'Alice'

# Multiline f-strings
message = (
    f"Name: {name}\n"
    f"Age:  {age}\n"
    f"Pi:   {pi:.2f}"
)
print(message)

# Nested f-strings (Python 3.12+)
width = 10
print(f"{'hello':>{width}}")    # Dynamic width

# Escaping braces
print(f"Use {{braces}} in f-strings")  # Use {braces} in f-strings
```

### Method 2: str.format()

```python
# Positional arguments
print("{} is {} years old".format("Alice", 25))

# Indexed arguments
print("{0} likes {1}. {0} is awesome.".format("Alice", "Python"))

# Named arguments
print("{name} is {age} years old".format(name="Alice", age=25))

# Format specifiers
print("{:.2f}".format(3.14159))         # 3.14
print("{:>20}".format("right"))          # right
print("{:,}".format(1000000))            # 1,000,000

# Accessing attributes and items
person = {"name": "Alice", "age": 25}
print("{0[name]} is {0[age]}".format(person))

# Unpacking dictionary
print("{name} is {age}".format(**person))

# Nested formatting
for i, name in enumerate(["Alice", "Bob", "Charlie"]):
    print("{:<3} {:<10}".format(i+1, name))
```

### Method 3: % Formatting (Old-style, C-style)

```python
# Basic
print("Name: %s, Age: %d" % ("Alice", 25))

# Format specifiers
print("Pi: %.2f" % 3.14159)           # Pi: 3.14
print("Hex: %x" % 255)                # Hex: ff
print("Octal: %o" % 255)              # Octal: 377
print("Padded: %10s" % "hello")       # Padded:      hello
print("Left: %-10s|" % "hello")       # Left: hello     |
print("Zero-pad: %05d" % 42)          # Zero-pad: 00042

# Multiple values (must use tuple)
print("(%d, %d)" % (3, 4))            # (3, 4)

# Dictionary-based
print("%(name)s is %(age)d" % {"name": "Alice", "age": 25})
```

### Comparison of Methods

| Feature | f-string | .format() | % formatting |
|---------|----------|-----------|--------------|
| Python version | 3.6+ | 2.6+ | All |
| Readability | ⭐⭐⭐ | ⭐⭐ | ⭐ |
| Performance | Fastest | Medium | Slowest |
| Expressions | Yes | No | No |
| Self-documenting | Yes (3.8+) | No | No |
| Recommendation | ✅ Preferred | OK | Legacy only |

### Format Specifiers Cheat Sheet

```
{value:fill align sign # 0 width grouping .precision type}

fill:      Any character (default: space)
align:     < (left), > (right), ^ (center), = (pad after sign)
sign:      + (always), - (negative only), space (space for positive)
#:         Prefix for binary (0b), octal (0o), hex (0x)
0:         Zero-padding
width:     Minimum width
grouping:  , or _ (thousands separator)
.precision: Decimal places for float, max length for string
type:      d (int), f (float), e (scientific), % (percentage), 
           b (binary), o (octal), x (hex), s (string)
```

---

## Common Mistakes

| Mistake | Example | Fix |
|---------|---------|-----|
| Forgetting input() returns string | `age = input(); age + 1` → TypeError | `age = int(input())` |
| Using f-string with single `{` | `f"dict: {key: value}"` | `f"dict: {{key: value}}"` |
| % with multiple values without tuple | `"%s %s" % "a", "b"` | `"%s %s" % ("a", "b")` |
| Mixing format styles | `f"{name} is %d" % age` | Use one style consistently |
| Forgetting `flush=True` for progress | `print("...", end="")` | `print("...", end="", flush=True)` |

---

## Interview Questions

**Q1: What does `input()` return?**
> Always a `str`, regardless of what the user types. You must explicitly convert with `int()`, `float()`, etc.

**Q2: How is `f-string` different from `.format()`?**
> f-strings allow inline expressions (e.g., `f"{x+1}"`), are evaluated at runtime, and are faster. `.format()` uses placeholders and is more suitable for reusable template strings.

**Q3: What is the difference between `str()` and `repr()`?**
> `str()` returns a human-readable string (calls `__str__`). `repr()` returns an unambiguous, developer-facing string (calls `__repr__`). Example: `str("hello") = hello`, `repr("hello") = 'hello'` (with quotes).

**Q4: How do you print without a newline?**
> `print("text", end="")` — set the `end` parameter to an empty string.

**Q5: What does `print(f"{x = }")` do?**
> Self-documenting expression (Python 3.8+). If `x = 42`, it prints `x = 42`. Shows both the expression and its value.

---

## Code Examples

### Easy Examples

```python
# Example 1: Basic I/O
name = input("What is your name? ")
print(f"Hello, {name}!")

# Example 2: Number input
num = int(input("Enter a number: "))
print(f"Double: {num * 2}")

# Example 3: Multiple outputs with sep
print("Python", "Java", "C++", sep=" | ")  # Python | Java | C++

# Example 4: Formatted numbers
price = 49.99
print(f"Price: ${price:.2f}")  # Price: $49.99

# Example 5: Simple alignment
items = [("Apple", 1.50), ("Banana", 0.75), ("Cherry", 3.00)]
for name, price in items:
    print(f"{name:<10} ${price:.2f}")
# Apple      $1.50
# Banana     $0.75
# Cherry     $3.00
```

### Medium Examples

```python
# Example 6: Reading multiple values
a, b = map(int, input("Enter two numbers separated by space: ").split())
print(f"Sum: {a+b}, Product: {a*b}")

# Example 7: Table formatting
data = [("Alice", 95, "A"), ("Bob", 82, "B"), ("Charlie", 91, "A")]
print(f"{'Name':<12} {'Score':>6} {'Grade':>6}")
print("-" * 26)
for name, score, grade in data:
    print(f"{name:<12} {score:>6} {grade:>6}")

# Example 8: Number formatting showcase
num = 1234567.89
print(f"Default:    {num}")
print(f"Comma:      {num:,.2f}")
print(f"Width 20:   {num:>20,.2f}")
print(f"Scientific: {num:.2e}")
print(f"Percentage: {0.4567:.1%}")

# Example 9: Dynamic formatting
def format_table(headers, rows, widths):
    header_line = " | ".join(f"{h:^{w}}" for h, w in zip(headers, widths))
    print(header_line)
    print("-" * len(header_line))
    for row in rows:
        print(" | ".join(f"{str(v):^{w}}" for v, w in zip(row, widths)))

format_table(
    ["ID", "Name", "Score"],
    [(1, "Alice", 95), (2, "Bob", 82), (3, "Charlie", 91)],
    [5, 12, 8]
)

# Example 10: Input validation loop
while True:
    try:
        age = int(input("Enter your age (1-120): "))
        if 1 <= age <= 120:
            break
        print("Age must be between 1 and 120!")
    except ValueError:
        print("Please enter a valid number!")
print(f"Your age is {age}")
```

### Advanced Examples

```python
# Example 11: Progress bar using print
import time

def progress_bar(total, current, width=40):
    filled = int(width * current / total)
    bar = "█" * filled + "░" * (width - filled)
    percent = current / total * 100
    print(f"\r[{bar}] {percent:.1f}% ({current}/{total})", end="", flush=True)

for i in range(101):
    progress_bar(100, i)
    time.sleep(0.02)
print()  # Final newline

# Example 12: Custom print function with logging
import sys
from datetime import datetime

def log_print(*args, level="INFO", **kwargs):
    timestamp = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
    prefix = f"[{timestamp}] [{level}]"
    print(prefix, *args, **kwargs)

log_print("Server started", level="INFO")
log_print("Connection lost!", level="ERROR")
log_print("Retrying...", level="WARN")

# Example 13: format_map with defaultdict
from collections import defaultdict

template = "Hello {name}, welcome to {city}!"
# format_map handles missing keys gracefully with defaultdict
data = defaultdict(lambda: "UNKNOWN", name="Alice")
print(template.format_map(data))  # Hello Alice, welcome to UNKNOWN!

# Example 14: String formatting for debugging
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y
    
    def __str__(self):
        return f"({self.x}, {self.y})"
    
    def __repr__(self):
        return f"Point(x={self.x}, y={self.y})"
    
    def __format__(self, spec):
        if spec == "polar":
            import math
            r = math.sqrt(self.x**2 + self.y**2)
            theta = math.atan2(self.y, self.x)
            return f"(r={r:.2f}, θ={math.degrees(theta):.1f}°)"
        return str(self)

p = Point(3, 4)
print(f"str:   {p}")          # (3, 4)
print(f"repr:  {p!r}")        # Point(x=3, y=4)
print(f"polar: {p:polar}")    # (r=5.00, θ=53.1°)

# Example 15: Redirect print output
from io import StringIO

# Capture print output
buffer = StringIO()
print("Captured output!", file=buffer)
print("More output!", file=buffer)

captured = buffer.getvalue()
print(f"Captured: {captured!r}")
# Captured: 'Captured output!\nMore output!\n'

# Print to stderr
import sys
print("Error message!", file=sys.stderr)
```

---

## Edge Cases

1. **Empty input**: `input()` with just Enter returns `""` (empty string).
2. **`print()` with no args**: Prints just a newline.
3. **`print(None)`**: Prints `None` as a string, not nothing.
4. **f-string with `=`**: `f"{x=}"` is Python 3.8+; older versions raise `SyntaxError`.
5. **`input()` returns string "None"**: If user types `None`, it's the string `"None"`, not `None`.
6. **EOFError**: `input()` raises `EOFError` if stdin is closed (e.g., piped input ends).

---

## Practice Questions

### Easy

**Q1:** Write a program that asks for the user's name and age, then prints a greeting in the format: "Hello Alice! You will be 26 next year."
```python
# Answer:
name = input("Enter your name: ")
age = int(input("Enter your age: "))
print(f"Hello {name}! You will be {age + 1} next year.")
```

**Q2:** Print the following table using f-string formatting:
```
Item         Price    Qty
----------   -----   ---
Apple        $1.50     5
Banana       $0.75    12
Cherry       $3.00     3
```
```python
# Answer:
items = [("Apple", 1.50, 5), ("Banana", 0.75, 12), ("Cherry", 3.00, 3)]
print(f"{'Item':<13}{'Price':>6}{'Qty':>6}")
print(f"{'-'*12}   {'─'*5}   {'─'*3}")
for name, price, qty in items:
    print(f"{name:<13}${price:>4.2f}{qty:>6}")
```

### Medium

**Q3:** Write a program that reads a list of integers from one line of input, and prints their sum, average, min, and max with proper formatting.
```python
# Answer:
nums = list(map(int, input("Enter numbers separated by spaces: ").split()))
print(f"Count:   {len(nums)}")
print(f"Sum:     {sum(nums):,}")
print(f"Average: {sum(nums)/len(nums):.2f}")
print(f"Min:     {min(nums)}")
print(f"Max:     {max(nums)}")
```

**Q4:** Create a receipt formatter that aligns item names on the left and prices on the right within a 40-character wide format.
```python
# Answer:
def print_receipt(items):
    width = 40
    print("=" * width)
    print(f"{'RECEIPT':^{width}}")
    print("=" * width)
    total = 0
    for name, price in items:
        dots = "." * (width - len(name) - len(f"${price:.2f}"))
        print(f"{name}{dots}${price:.2f}")
        total += price
    print("-" * width)
    print(f"{'TOTAL':>{width-8}}${total:>7.2f}")
    print("=" * width)

print_receipt([
    ("Coffee", 4.99),
    ("Sandwich", 8.50),
    ("Cookie", 2.25),
    ("Water Bottle", 1.50),
])
```

### Advanced

**Q5:** Implement a `printf`-style function that supports `%s`, `%d`, `%f`, and `%x` format specifiers without using Python's built-in `%` formatting.
```python
# Answer:
import re

def my_printf(template, *args):
    """Simple printf implementation without using % formatting."""
    specifiers = {'s': str, 'd': lambda x: str(int(x)), 
                  'f': lambda x: f"{float(x):.6f}", 'x': lambda x: hex(int(x))[2:]}
    
    result = []
    arg_index = 0
    i = 0
    while i < len(template):
        if template[i] == '%' and i + 1 < len(template):
            spec = template[i + 1]
            if spec in specifiers:
                result.append(specifiers[spec](args[arg_index]))
                arg_index += 1
                i += 2
                continue
            elif spec == '%':
                result.append('%')
                i += 2
                continue
        result.append(template[i])
        i += 1
    
    print(''.join(result))

my_printf("Name: %s, Age: %d, Score: %f, ID: %x", "Alice", 25, 95.5, 255)
# Name: Alice, Age: 25, Score: 95.500000, ID: ff
```

**Q6:** Write a function that pretty-prints any nested dictionary with proper indentation.
```python
# Answer:
def pretty_print(obj, indent=0, indent_str="  "):
    """Recursively pretty-print nested structures."""
    prefix = indent_str * indent
    
    if isinstance(obj, dict):
        print("{")
        for i, (key, value) in enumerate(obj.items()):
            print(f"{prefix}{indent_str}{key!r}: ", end="")
            pretty_print(value, indent + 1, indent_str)
            if i < len(obj) - 1:
                print(",")
            else:
                print()
        print(f"{prefix}}}", end="")
    elif isinstance(obj, (list, tuple)):
        bracket = "[]" if isinstance(obj, list) else "()"
        print(bracket[0])
        for i, item in enumerate(obj):
            print(f"{prefix}{indent_str}", end="")
            pretty_print(item, indent + 1, indent_str)
            if i < len(obj) - 1:
                print(",")
            else:
                print()
        print(f"{prefix}{bracket[1]}", end="")
    else:
        print(repr(obj), end="")

data = {"user": {"name": "Alice", "scores": [95, 87, 92], "active": True}}
pretty_print(data)
print()
```

---

*← [Previous: Operators](04_Operators.md) | [Next: Strings →](06_Strings.md)*
