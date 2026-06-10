# Chapter 11: Conditional Statements

---

## Topic Overview

| Aspect | Details |
|--------|---------|
| **Definition** | Conditional statements execute different code blocks based on whether a condition evaluates to `True` or `False`. |
| **Why It Exists** | Programs need decision-making ability — executing different paths based on data, user input, or state. |
| **Where It Is Used** | Input validation, access control, branching logic, error handling, feature flags, ML model thresholds. |
| **Real-World Analogy** | Like a **traffic signal** — green means go (True path), red means stop (False path). |

### How It Works Internally
- Python evaluates the condition expression to a boolean (`True`/`False`).
- Any object can be evaluated: Python calls `__bool__()` or `__len__()` on the object.
- Code blocks are defined by **indentation** (typically 4 spaces).
- Python uses **short-circuit evaluation** for compound conditions.

---

## 11.1 if Statement

```python
age = 20

if age >= 18:
    print("You are an adult")
    print("You can vote")
# Both print statements execute if condition is True

# Single-line (for simple cases)
if age >= 18: print("Adult")
```

## 11.2 if-else Statement

```python
age = 15

if age >= 18:
    print("Adult")
else:
    print("Minor")
```

## 11.3 if-elif-else Statement

```python
score = 85

if score >= 90:
    grade = "A"
elif score >= 80:
    grade = "B"
elif score >= 70:
    grade = "C"
elif score >= 60:
    grade = "D"
else:
    grade = "F"

print(f"Grade: {grade}")  # B
```

## 11.4 Nested if

```python
age = 25
has_license = True

if age >= 18:
    if has_license:
        print("You can drive")
    else:
        print("Get a license first")
else:
    print("You're too young to drive")

# Flatten with 'and' (more Pythonic):
if age >= 18 and has_license:
    print("You can drive")
```

---

## Ternary (Conditional) Expression

```python
# Syntax: value_if_true if condition else value_if_false
age = 20
status = "adult" if age >= 18 else "minor"
print(status)  # "adult"

# Nested ternary (avoid for readability)
x = 15
result = "high" if x > 20 else "medium" if x > 10 else "low"
print(result)  # "medium"

# In function calls
print("even" if 10 % 2 == 0 else "odd")

# In list comprehensions
labels = ["even" if x % 2 == 0 else "odd" for x in range(6)]
print(labels)  # ['even', 'odd', 'even', 'odd', 'even', 'odd']
```

---

## Truthiness in Conditions

```python
# These are all FALSY:
if not 0:         print("0 is falsy")
if not 0.0:       print("0.0 is falsy")
if not "":        print("Empty string is falsy")
if not []:        print("Empty list is falsy")
if not {}:        print("Empty dict is falsy")
if not set():     print("Empty set is falsy")
if not None:      print("None is falsy")
if not False:     print("False is falsy")

# Pythonic patterns:
lst = [1, 2, 3]
if lst:  # Instead of: if len(lst) > 0:
    print("List is not empty")

name = ""
if not name:  # Instead of: if name == "":
    print("Name is empty")

result = None
if result is None:  # Preferred for None check
    print("No result")
```

---

## Pattern Matching (match-case) — Python 3.10+

```python
# Structural pattern matching
command = "quit"

match command:
    case "start":
        print("Starting...")
    case "stop":
        print("Stopping...")
    case "quit" | "exit":
        print("Goodbye!")
    case _:
        print(f"Unknown command: {command}")

# With value capture
point = (3, 4)
match point:
    case (0, 0):
        print("Origin")
    case (x, 0):
        print(f"On x-axis at {x}")
    case (0, y):
        print(f"On y-axis at {y}")
    case (x, y):
        print(f"Point at ({x}, {y})")

# With guard clauses
age = 25
match age:
    case n if n < 0:
        print("Invalid")
    case n if n < 18:
        print("Minor")
    case n if n < 65:
        print(f"Adult, age {n}")
    case n:
        print(f"Senior, age {n}")
```

---

## Common Mistakes

| Mistake | Example | Fix |
|---------|---------|-----|
| Using `=` instead of `==` | `if x = 5:` → SyntaxError | `if x == 5:` |
| Checking None with `==` | `if x == None:` | `if x is None:` |
| Redundant boolean comparison | `if flag == True:` | `if flag:` |
| Missing colon | `if x > 5` → SyntaxError | `if x > 5:` |
| Wrong indentation | Mixed tabs/spaces | Use 4 spaces consistently |
| Unnecessary nested ifs | `if a: if b:` | `if a and b:` |

---

## Interview Questions

**Q1: What is the difference between `if x` and `if x is not None`?**
> `if x` checks truthiness — fails for `0`, `""`, `[]`, etc. `if x is not None` specifically checks if `x` is not the `None` singleton. Use `is not None` when `0` or `""` are valid values.

**Q2: Does Python have a switch statement?**
> Python 3.10+ has `match-case` (structural pattern matching), which is more powerful than traditional switch. Before 3.10, use if-elif chains or dict dispatch.

**Q3: What is short-circuit evaluation in conditions?**
> In `if a and b`, if `a` is False, `b` is never evaluated. In `if a or b`, if `a` is True, `b` is never evaluated.

---

## Code Examples

### Easy Examples
```python
# 1: Check positive/negative/zero
num = -5
if num > 0: print("Positive")
elif num < 0: print("Negative")
else: print("Zero")

# 2: Max of two numbers
a, b = 10, 20
print(f"Max: {a if a > b else b}")

# 3: Check even/odd
n = 7
print(f"{n} is {'even' if n % 2 == 0 else 'odd'}")

# 4: Voting eligibility
age = 17
print("Can vote" if age >= 18 else "Cannot vote")

# 5: Grade assignment
score = 75
grade = "A" if score >= 90 else "B" if score >= 80 else "C" if score >= 70 else "F"
print(f"Grade: {grade}")
```

### Medium Examples
```python
# 6: Leap year check
def is_leap_year(year):
    return (year % 4 == 0 and year % 100 != 0) or (year % 400 == 0)
for y in [2000, 1900, 2024, 2023]:
    print(f"{y}: {'Leap' if is_leap_year(y) else 'Not leap'}")

# 7: FizzBuzz
for i in range(1, 21):
    if i % 15 == 0: print("FizzBuzz")
    elif i % 3 == 0: print("Fizz")
    elif i % 5 == 0: print("Buzz")
    else: print(i)

# 8: Triangle type
def triangle_type(a, b, c):
    if a + b <= c or b + c <= a or a + c <= b:
        return "Not a triangle"
    if a == b == c: return "Equilateral"
    if a == b or b == c or a == c: return "Isosceles"
    return "Scalene"

# 9: BMI calculator
def bmi_category(weight, height):
    bmi = weight / height**2
    if bmi < 18.5: return f"Underweight (BMI: {bmi:.1f})"
    elif bmi < 25: return f"Normal (BMI: {bmi:.1f})"
    elif bmi < 30: return f"Overweight (BMI: {bmi:.1f})"
    else: return f"Obese (BMI: {bmi:.1f})"

# 10: Safe division with validation
def safe_divide(a, b):
    if not isinstance(a, (int, float)) or not isinstance(b, (int, float)):
        return "Error: non-numeric input"
    if b == 0:
        return "Error: division by zero"
    return a / b
```

### Advanced Examples
```python
# 11: Dispatch table vs if-elif
operations = {
    "+": lambda a, b: a + b,
    "-": lambda a, b: a - b,
    "*": lambda a, b: a * b,
    "/": lambda a, b: a / b if b != 0 else "Error",
}
result = operations.get("*", lambda a, b: "Unknown")(6, 7)
print(result)  # 42

# 12: Nested ternary for comparison (interview question)
def compare(a, b):
    return 0 if a == b else (1 if a > b else -1)

# 13: Walrus operator in conditions
import re
text = "Contact: alice@example.com"
if (match := re.search(r'[\w.-]+@[\w.-]+', text)):
    print(f"Found email: {match.group()}")
else:
    print("No email found")

# 14: Pattern matching with class
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

# Python 3.10+:
# match point:
#     case Point(x=0, y=0): print("Origin")
#     case Point(x=x, y=0): print(f"X-axis: {x}")

# 15: Guard clause pattern (early return)
def process_user(user):
    if user is None:
        return "No user"
    if not user.get("active"):
        return "Inactive user"
    if user.get("age", 0) < 18:
        return "Underage"
    return f"Processing {user['name']}"
```

---

## Practice Questions

### Easy
**Q1:** Write a program to check if a number is positive, negative, or zero.
```python
num = float(input("Enter number: "))
if num > 0: print("Positive")
elif num < 0: print("Negative")
else: print("Zero")
```

**Q2:** Write a program to find the largest of three numbers.
```python
a, b, c = 10, 25, 15
largest = a if a >= b and a >= c else b if b >= c else c
print(f"Largest: {largest}")
```

### Medium
**Q3:** Write a function that validates a password (min 8 chars, at least 1 digit, 1 uppercase, 1 special char).
```python
def validate_password(pwd):
    if len(pwd) < 8: return False, "Too short"
    if not any(c.isdigit() for c in pwd): return False, "No digit"
    if not any(c.isupper() for c in pwd): return False, "No uppercase"
    if not any(c in "!@#$%^&*" for c in pwd): return False, "No special char"
    return True, "Valid"
print(validate_password("MyPass1!"))  # (True, 'Valid')
```

**Q4:** Implement a simple tax calculator with brackets.
```python
def calculate_tax(income):
    if income <= 10000: return income * 0.10
    elif income <= 40000: return 1000 + (income - 10000) * 0.20
    elif income <= 80000: return 7000 + (income - 40000) * 0.30
    else: return 19000 + (income - 80000) * 0.40
print(f"Tax on 50000: ${calculate_tax(50000):,.2f}")
```

### Advanced
**Q5:** Implement a number-to-words converter for 0-999.
```python
def num_to_words(n):
    ones = ["", "one", "two", "three", "four", "five", "six", "seven", "eight", "nine",
            "ten", "eleven", "twelve", "thirteen", "fourteen", "fifteen", "sixteen",
            "seventeen", "eighteen", "nineteen"]
    tens = ["", "", "twenty", "thirty", "forty", "fifty", "sixty", "seventy", "eighty", "ninety"]
    if n == 0: return "zero"
    if n < 20: return ones[n]
    if n < 100: return tens[n//10] + ("-" + ones[n%10] if n%10 else "")
    return ones[n//100] + " hundred" + (" and " + num_to_words(n%100) if n%100 else "")
for n in [0, 5, 15, 42, 100, 305, 999]:
    print(f"{n}: {num_to_words(n)}")
```

**Q6:** Implement a simple expression evaluator using match-case (Python 3.10+).
```python
# For Python 3.10+
def evaluate(expr):
    match expr:
        case ("+", a, b): return evaluate(a) + evaluate(b)
        case ("-", a, b): return evaluate(a) - evaluate(b)
        case ("*", a, b): return evaluate(a) * evaluate(b)
        case ("/", a, b) if evaluate(b) != 0: return evaluate(a) / evaluate(b)
        case int(n) | float(n): return n
        case _: raise ValueError(f"Invalid expression: {expr}")

# (+ (* 3 4) (- 10 5)) = 12 + 5 = 17
expr = ("+", ("*", 3, 4), ("-", 10, 5))
print(f"Result: {evaluate(expr)}")
```

---

*← [Previous: Dictionaries](10_Dictionaries.md) | [Next: Loops →](12_Loops.md)*
