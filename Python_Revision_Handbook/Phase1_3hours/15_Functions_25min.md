# Chapter 15: Functions

---

## Topic Overview

| Aspect | Details |
|--------|---------|
| **Definition** | A function is a **reusable block of code** that performs a specific task. Defined with `def`, called by name. |
| **Why It Exists** | DRY principle, modularity, abstraction, testability, code organization. |
| **Where It Is Used** | Every Python program. Functions are the primary unit of code reuse and abstraction. |
| **Real-World Analogy** | A function is like a **vending machine** — you insert inputs (arguments), it processes them, and returns output. |

### How It Works Internally
1. `def` creates a **function object** and binds it to the name.
2. The function body is **not** executed at definition time.
3. When called, Python creates a new **frame** on the call stack.
4. Arguments are passed by **assignment** (pass-by-object-reference).
5. The frame is destroyed when the function returns.

---

## 15.1 Definition & Calling

```python
# Definition
def greet(name):
    """Greet someone by name."""  # Docstring
    return f"Hello, {name}!"

# Calling
message = greet("Alice")
print(message)  # Hello, Alice!

# Functions are objects
print(type(greet))     # <class 'function'>
print(greet.__name__)  # greet
print(greet.__doc__)   # Greet someone by name.

# Functions can be assigned to variables
say_hello = greet
print(say_hello("Bob"))  # Hello, Bob!

# Functions return None by default
def no_return():
    print("Hello")

result = no_return()
print(result)  # None
```

---

## 15.2 Parameters & Arguments

### Positional Arguments
```python
def add(a, b):
    return a + b

print(add(3, 5))      # 8 — positional
print(add(b=5, a=3))  # 8 — keyword (order doesn't matter)
```

### Keyword Arguments
```python
def profile(name, age, city="Unknown"):
    return f"{name}, {age}, from {city}"

print(profile("Alice", 25, city="NYC"))
print(profile(age=25, name="Alice"))
```

### Default Arguments
```python
def greet(name, greeting="Hello"):
    return f"{greeting}, {name}!"

print(greet("Alice"))            # Hello, Alice!
print(greet("Alice", "Hi"))      # Hi, Alice!
print(greet("Alice", greeting="Hey"))  # Hey, Alice!

# ⚠️ DANGEROUS: Mutable default argument
def bad_append(item, lst=[]):
    lst.append(item)
    return lst

print(bad_append(1))  # [1]
print(bad_append(2))  # [1, 2] — NOT [2]! Same list object!

# ✅ CORRECT:
def good_append(item, lst=None):
    if lst is None:
        lst = []
    lst.append(item)
    return lst
```

### Variable-Length Arguments: *args
```python
def sum_all(*args):
    """Accept any number of positional arguments."""
    print(type(args))  # <class 'tuple'>
    return sum(args)

print(sum_all(1, 2, 3))        # 6
print(sum_all(1, 2, 3, 4, 5))  # 15
print(sum_all())                # 0

# *args captures remaining positional arguments
def first_and_rest(first, *rest):
    print(f"First: {first}")
    print(f"Rest: {rest}")

first_and_rest(1, 2, 3, 4)
# First: 1
# Rest: (2, 3, 4)
```

### Variable-Length Arguments: **kwargs
```python
def print_info(**kwargs):
    """Accept any number of keyword arguments."""
    print(type(kwargs))  # <class 'dict'>
    for key, value in kwargs.items():
        print(f"{key}: {value}")

print_info(name="Alice", age=25, city="NYC")
# name: Alice
# age: 25
# city: NYC
```

### Combining All Parameter Types
```python
# Order: positional, *args, keyword-only, **kwargs
def complex_func(a, b, *args, keyword_only="default", **kwargs):
    print(f"a={a}, b={b}")
    print(f"args={args}")
    print(f"keyword_only={keyword_only}")
    print(f"kwargs={kwargs}")

complex_func(1, 2, 3, 4, keyword_only="custom", x=5, y=6)
# a=1, b=2
# args=(3, 4)
# keyword_only=custom
# kwargs={'x': 5, 'y': 6}

# Keyword-only arguments (after *)
def keyword_only_func(*, name, age):
    """name and age MUST be passed as keywords."""
    return f"{name}, {age}"

# keyword_only_func("Alice", 25)  # TypeError!
print(keyword_only_func(name="Alice", age=25))  # Alice, 25

# Positional-only arguments (Python 3.8+, before /)
def positional_only(a, b, /, c, d):
    """a and b MUST be passed positionally."""
    return a + b + c + d

print(positional_only(1, 2, 3, d=4))  # 10
# positional_only(a=1, b=2, c=3, d=4)  # TypeError for a and b!
```

---

## 15.3 Return Values

```python
# Single return
def square(x):
    return x ** 2

# Multiple returns (tuple packing)
def stats(numbers):
    return min(numbers), max(numbers), sum(numbers) / len(numbers)

mn, mx, avg = stats([10, 20, 30])

# Early return (guard clause)
def divide(a, b):
    if b == 0:
        return None  # Early return
    return a / b

# Return function (higher-order)
def make_multiplier(n):
    def multiplier(x):
        return x * n
    return multiplier

double = make_multiplier(2)
print(double(5))  # 10
```

---

## 15.4 Pass by Assignment (Object Reference)

```python
# Python passes arguments by ASSIGNMENT (pass-by-object-reference)
# - Immutable objects (int, str, tuple): changes create new objects, original unaffected
# - Mutable objects (list, dict, set): changes modify the original

# Immutable: no side effect
def increment(n):
    n += 1  # Creates new int, local reference
    return n

x = 10
increment(x)
print(x)  # 10 — unchanged

# Mutable: side effect!
def add_item(lst):
    lst.append("new")  # Modifies the same list

my_list = [1, 2, 3]
add_item(my_list)
print(my_list)  # [1, 2, 3, 'new'] — modified!

# Mutable: rebinding doesn't affect caller
def replace_list(lst):
    lst = [10, 20, 30]  # Creates new local reference

my_list = [1, 2, 3]
replace_list(my_list)
print(my_list)  # [1, 2, 3] — unchanged (local rebinding)
```

---

## Common Mistakes

| Mistake | Example | Fix |
|---------|---------|-----|
| Mutable default argument | `def f(lst=[]):` | `def f(lst=None):` |
| Forgetting return | `def add(a,b): a+b` | `return a + b` |
| Modifying mutable argument | Side effects on lists/dicts | Pass copies or document intent |
| Wrong argument order | `def f(a, *args, b):` b is keyword-only | Understand parameter ordering |
| Confusing `return` and `print` | Function prints but doesn't return | Use `return` for values |

---

## Interview Questions

**Q1: How are arguments passed in Python?**
> By assignment (pass-by-object-reference). The function receives a reference to the same object. For immutable objects, modifications create new objects. For mutable objects, in-place modifications affect the original.

**Q2: What is the difference between `*args` and `**kwargs`?**
> `*args` collects extra positional arguments into a tuple. `**kwargs` collects extra keyword arguments into a dictionary.

**Q3: Why is mutable default argument dangerous?**
> The default is evaluated ONCE at function definition time. If it's mutable (like a list), all calls share the same object. Use `None` as default and create inside the function.

**Q4: What is a first-class function?**
> Functions in Python are first-class objects — they can be assigned to variables, passed as arguments, returned from other functions, and stored in data structures.

**Q5: What is the difference between parameters and arguments?**
> Parameters are the variables in the function definition. Arguments are the actual values passed when calling the function.

---

## Code Examples

### Easy (5)
```python
# 1: Simple function
def celsius_to_fahrenheit(c):
    return c * 9/5 + 32
print(celsius_to_fahrenheit(100))  # 212.0

# 2: Default parameter
def power(base, exp=2):
    return base ** exp
print(power(3))     # 9
print(power(3, 3))  # 27

# 3: Multiple return values
def min_max(lst):
    return min(lst), max(lst)
lo, hi = min_max([3, 7, 1, 9, 4])
print(f"Min: {lo}, Max: {hi}")

# 4: *args sum
def total(*args):
    return sum(args)
print(total(1, 2, 3, 4, 5))  # 15

# 5: **kwargs profile
def create_profile(**kwargs):
    return kwargs
print(create_profile(name="Alice", age=25, city="NYC"))
```

### Medium (5)
```python
# 6: Function as argument
def apply_operation(func, a, b):
    return func(a, b)
print(apply_operation(lambda a, b: a + b, 3, 5))  # 8

# 7: Unpacking arguments
def add(a, b, c):
    return a + b + c
args = [1, 2, 3]
kwargs = {"a": 1, "b": 2, "c": 3}
print(add(*args))     # 6
print(add(**kwargs))   # 6

# 8: Decorator pattern (simplified)
def log_calls(func):
    def wrapper(*args, **kwargs):
        print(f"Calling {func.__name__} with {args}, {kwargs}")
        result = func(*args, **kwargs)
        print(f"Result: {result}")
        return result
    return wrapper

@log_calls
def multiply(a, b):
    return a * b
multiply(3, 4)

# 9: Type hints
def greet(name: str, times: int = 1) -> str:
    return (f"Hello, {name}! " * times).strip()
print(greet("Alice", 3))

# 10: Recursive function
def factorial(n: int) -> int:
    if n <= 1: return 1
    return n * factorial(n - 1)
print(factorial(5))  # 120
```

### Advanced (5)
```python
# 11: Closure
def make_counter():
    count = 0
    def counter():
        nonlocal count
        count += 1
        return count
    return counter
c = make_counter()
print(c(), c(), c())  # 1 2 3

# 12: Function introspection
import inspect
def example(a: int, b: str = "hello", *args, **kwargs) -> bool:
    """Example function."""
    pass

sig = inspect.signature(example)
print(f"Parameters: {sig}")
print(f"Docstring: {example.__doc__}")
print(f"Annotations: {example.__annotations__}")
print(f"Defaults: {example.__defaults__}")

# 13: Memoization
def memoize(func):
    cache = {}
    def wrapper(*args):
        if args not in cache:
            cache[args] = func(*args)
        return cache[args]
    return wrapper

@memoize
def fib(n):
    if n < 2: return n
    return fib(n-1) + fib(n-2)
print(fib(100))  # Instant! Without memo: takes forever

# 14: Keyword-only and positional-only combined
def api_call(endpoint, /, *, method="GET", headers=None, body=None):
    """endpoint is positional-only; rest are keyword-only."""
    print(f"{method} {endpoint}")
    if headers: print(f"Headers: {headers}")
    if body: print(f"Body: {body}")

api_call("/users", method="POST", body={"name": "Alice"})

# 15: Function factory with configuration
def make_validator(min_len=0, max_len=100, required_chars=""):
    def validate(s):
        if len(s) < min_len: return False, f"Too short (min {min_len})"
        if len(s) > max_len: return False, f"Too long (max {max_len})"
        for c in required_chars:
            if c not in s: return False, f"Missing required char: {c}"
        return True, "Valid"
    return validate

password_validator = make_validator(min_len=8, required_chars="!@#$")
print(password_validator("MyP@ss!23"))
print(password_validator("short"))
```

---

## Practice Questions

### Easy
**Q1:** Write a function that takes a list and returns a new list with duplicates removed.
```python
def remove_dupes(lst):
    return list(dict.fromkeys(lst))
print(remove_dupes([1,2,2,3,3,3,4]))  # [1, 2, 3, 4]
```

**Q2:** Write a function that checks if a string is a palindrome.
```python
def is_palindrome(s):
    s = s.lower().replace(" ", "")
    return s == s[::-1]
print(is_palindrome("Race Car"))  # True
```

### Medium
**Q3:** Write a function that accepts any number of lists and returns their intersection.
```python
def intersect(*lists):
    if not lists: return []
    result = set(lists[0])
    for lst in lists[1:]:
        result &= set(lst)
    return sorted(result)
print(intersect([1,2,3,4], [2,3,4,5], [3,4,5,6]))  # [3, 4]
```

**Q4:** Write a higher-order function that takes a function and returns a new function that caches results.
```python
def cache(func):
    memo = {}
    def wrapper(*args):
        if args not in memo:
            memo[args] = func(*args)
        return memo[args]
    wrapper.cache = memo
    return wrapper

@cache
def expensive(n):
    return sum(range(n))
print(expensive(1000000))
print(expensive(1000000))  # Instant — cached
print(expensive.cache)
```

### Advanced
**Q5:** Implement a `retry` decorator that retries a function n times on failure.
```python
import time
def retry(max_attempts=3, delay=1):
    def decorator(func):
        def wrapper(*args, **kwargs):
            for attempt in range(1, max_attempts + 1):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    print(f"Attempt {attempt} failed: {e}")
                    if attempt < max_attempts:
                        time.sleep(delay)
            raise Exception(f"All {max_attempts} attempts failed")
        return wrapper
    return decorator

@retry(max_attempts=3, delay=0.1)
def unstable_api():
    import random
    if random.random() < 0.7:
        raise ConnectionError("Server down")
    return "Success!"
```

**Q6:** Explain with code examples the difference between modifying a mutable argument in-place vs rebinding it.
```python
def modify_in_place(lst):
    lst.append(4)  # Modifies the SAME object
    lst[0] = 99    # Modifies the SAME object

def rebind(lst):
    lst = [10, 20, 30]  # Creates NEW local object

original = [1, 2, 3]
modify_in_place(original)
print(f"After modify_in_place: {original}")  # [99, 2, 3, 4]

original = [1, 2, 3]
rebind(original)
print(f"After rebind: {original}")  # [1, 2, 3] — unchanged
```

---

*← [Previous: List Comprehension](14_List_Comprehension.md) | [Next: Scope →](16_Scope.md)*
