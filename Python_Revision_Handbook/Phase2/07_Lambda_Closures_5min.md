# Chapter 7: Lambda, Closures & Functional Programming

---

## Topic Overview

| Aspect | Details |
|--------|---------|
| **Lambda** | A tiny, anonymous, one-line function. |
| **Closure** | A function that remembers variables from the outer function that created it. |
| **Functional Programming** | Using `map`, `filter`, `reduce` to transform data without loops. |

---

## Part 1: Lambda Functions

### Verbal Explanation

A lambda is basically: "I need a function, but it's so small I don't want to give it a name."

```python
# Normal function
def double(x):
    return x * 2

# Lambda — same thing, one line
double = lambda x: x * 2

print(double(5))   # 10
```

**Rules:** One expression only. No `if:` blocks, no `for` loops, no multiple lines. Just one expression that gets returned automatically.

### Where Lambdas Actually Shine — Sort Keys

```python
students = [("Alice", 85), ("Bob", 92), ("Charlie", 78)]

# Sort by grade (second element)
sorted_students = sorted(students, key=lambda s: s[1])
print(sorted_students)   # [('Charlie', 78), ('Alice', 85), ('Bob', 92)]

# Sort by name length
words = ["banana", "kiwi", "apple", "cherry"]
print(sorted(words, key=lambda w: len(w)))   # ['kiwi', 'apple', 'banana', 'cherry']

# Sort dictionaries
people = [{"name": "Alice", "age": 30}, {"name": "Bob", "age": 25}]
print(sorted(people, key=lambda p: p["age"]))
```

---

## Part 2: map, filter, reduce

### `map()` — Transform Every Element

```python
nums = [1, 2, 3, 4, 5]

# Double each number
doubled = list(map(lambda x: x * 2, nums))
print(doubled)   # [2, 4, 6, 8, 10]

# Convert strings to integers
str_nums = ["10", "20", "30"]
int_nums = list(map(int, str_nums))   # [10, 20, 30]

# Pythonic alternative (usually better):
doubled = [x * 2 for x in nums]
```

### `filter()` — Keep Only Matching Elements

```python
nums = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

# Keep only even numbers
evens = list(filter(lambda x: x % 2 == 0, nums))
print(evens)   # [2, 4, 6, 8, 10]

# Remove falsy values
data = [0, 1, "", "hello", None, 42, [], [1]]
truthy = list(filter(None, data))   # [1, 'hello', 42, [1]]

# Pythonic alternative:
evens = [x for x in nums if x % 2 == 0]
```

### `reduce()` — Combine Everything Into One Value

```python
from functools import reduce

nums = [1, 2, 3, 4, 5]

# Sum: ((((1+2)+3)+4)+5) = 15
total = reduce(lambda a, b: a + b, nums)
print(total)   # 15

# Max
biggest = reduce(lambda a, b: a if a > b else b, nums)
print(biggest)   # 5

# Usually better to use built-ins: sum(), max(), etc.
```

### When to Use What?

| Tool | Pythonic Alternative | When to Use Lambda/Functional |
|------|---------------------|-------------------------------|
| `map(func, lst)` | `[func(x) for x in lst]` | When passing an existing function like `map(int, strings)` |
| `filter(func, lst)` | `[x for x in lst if cond]` | Rarely — comprehensions are clearer |
| `reduce(func, lst)` | `sum()`, `max()`, loop | Rarely — built-ins are clearer |

---

## Part 3: Closures

### Verbal Explanation

A closure is a function that "remembers" variables from the outer function that created it — even after that outer function is done running.

```python
def make_greeter(greeting):
    # greeting is a local variable of make_greeter

    def greet(name):
        # This inner function USES greeting from the outer function
        return f"{greeting}, {name}!"

    return greet

hello = make_greeter("Hello")
hey = make_greeter("Hey")

print(hello("Alice"))    # Hello, Alice!
print(hey("Bob"))        # Hey, Bob!
```

`make_greeter("Hello")` finished running. Its local variable `greeting = "Hello"` should be dead. But `hello` still uses it! The inner function `greet` **closed over** the variable `greeting`.

### Practical Example — Counter

```python
def make_counter():
    count = 0

    def increment():
        nonlocal count     # "I mean the OUTER count"
        count += 1
        return count

    return increment

counter = make_counter()
print(counter())   # 1
print(counter())   # 2
print(counter())   # 3

# Each counter is independent
counter2 = make_counter()
print(counter2())   # 1 (separate count)
```

### The Classic Closure Trap ⭐

```python
functions = []
for i in range(5):
    functions.append(lambda: i)

print([f() for f in functions])
# [4, 4, 4, 4, 4] — ALL return 4! NOT [0, 1, 2, 3, 4]!
```

**Why?** All lambdas share the SAME variable `i`. By the time you call them, `i` is 4.

```python
# Fix: Capture the value with a default argument
functions = [lambda i=i: i for i in range(5)]
print([f() for f in functions])   # [0, 1, 2, 3, 4] ✅
```

---

## Code Examples

### Easy
```python
# Example 1: Lambda as sort key
pairs = [(1, "one"), (3, "three"), (2, "two")]
pairs.sort(key=lambda p: p[0])
print(pairs)   # [(1, 'one'), (2, 'two'), (3, 'three')]

# Example 2: map with existing function
print(list(map(str.upper, ["hello", "world"])))  # ['HELLO', 'WORLD']

# Example 3: Simple closure
def multiplier(factor):
    return lambda x: x * factor

double = multiplier(2)
triple = multiplier(3)
print(double(5), triple(5))   # 10 15
```

### Medium
```python
# Example 4: Chaining map and filter
nums = range(1, 21)
result = list(map(lambda x: x**2, filter(lambda x: x % 2 == 0, nums)))
print(result)   # [4, 16, 36, 64, 100, 144, 196, 256, 324, 400]

# More readable version:
result = [x**2 for x in range(1, 21) if x % 2 == 0]

# Example 5: functools.partial — pre-fill arguments
from functools import partial

def power(base, exp):
    return base ** exp

square = partial(power, exp=2)
cube = partial(power, exp=3)
print(square(5))   # 25
print(cube(3))     # 27
```

### Advanced
```python
# Example 6: Closure-based configuration
def create_logger(level):
    def log(message):
        print(f"[{level}] {message}")
    return log

info = create_logger("INFO")
error = create_logger("ERROR")
debug = create_logger("DEBUG")

info("Server started")     # [INFO] Server started
error("Connection failed") # [ERROR] Connection failed
debug("x = 42")            # [DEBUG] x = 42
```

---

## Interview Questions (Top 5)

**Q1: What is a lambda function?**
> An anonymous, single-expression function. `lambda x: x * 2` is equivalent to `def f(x): return x * 2`. Best for short inline uses like sort keys.

**Q2: What is a closure?**
> A nested function that remembers variables from its enclosing scope, even after the outer function returns. It "closes over" those variables.

**Q3: What is the closure loop trap?**
> All closures in a loop share the same variable. By the time they execute, the variable has its final value. Fix with default arguments: `lambda i=i: i`.

**Q4: What is `functools.partial`?**
> Creates a new function with some arguments pre-filled. `partial(multiply, 2)` creates a doubler. Cleaner than lambdas for simple cases.

**Q5: When should you NOT use lambda?**
> When the function needs multiple lines, has complex logic, or is reused in multiple places. PEP 8: don't assign lambdas to variables — use `def` instead.

---

*← [Previous: Chapter 6 (Iterators & Generators)](06_Iterators_Generators.md) | [Next: Chapter 8 (Recursion) →](08_Recursion.md)*
