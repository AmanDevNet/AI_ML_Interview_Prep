# Chapter 5: Decorators — "Wrapping Functions with Superpowers"

---

## Topic Overview

| Aspect | Details |
|--------|---------|
| **Definition** | A decorator is a function that takes another function, **adds some behavior** to it, and returns it — **without changing the original function's code**. |
| **Why It Exists** | You want to add logging, timing, caching, authentication, etc. to multiple functions WITHOUT modifying each one. |
| **Where It Is Used** | Flask routes (`@app.route`), Django views, authentication (`@login_required`), caching (`@lru_cache`), testing (`@pytest.fixture`). |
| **Real-World Analogy** | A phone case. Your phone works fine without it. The case wraps around it and adds protection (extra behavior) without changing how the phone works inside. |

---

## Think of It This Way

You have 50 functions in your app. Your boss says "I need every function to print how long it takes to run."

**Without decorators:** You add timing code inside ALL 50 functions. Copy-paste nightmare.

**With decorators:** You write the timing code ONCE in a decorator, and just put `@timer` above each function. Done.

---

## Step 1: Functions Are Objects (This Is the Foundation)

Before decorators make sense, you need to know that functions are objects in Python. You can pass them around like variables.

```python
# 1. Assign a function to a variable
def greet(name):
    return f"Hello, {name}!"

say_hi = greet              # say_hi now points to the same function
print(say_hi("Alice"))      # Hello, Alice!
print(type(greet))          # <class 'function'>

# 2. Pass a function as an argument
def call_twice(func, arg):
    print(func(arg))
    print(func(arg))

call_twice(greet, "Bob")    # Prints "Hello, Bob!" twice

# 3. Return a function from another function
def make_greeter(greeting):
    def greeter(name):
        return f"{greeting}, {name}!"
    return greeter

hi = make_greeter("Hi")
hey = make_greeter("Hey")
print(hi("Charlie"))    # Hi, Charlie!
print(hey("Diana"))     # Hey, Diana!
```

---

## Step 2: Building Your First Decorator (Step by Step)

**The goal:** Print "START" before any function runs and "END" after.

```python
# Step A: The decorator function
def my_decorator(func):
    def wrapper(*args, **kwargs):     # wrapper replaces the original function
        print("START")
        result = func(*args, **kwargs)  # call the ORIGINAL function
        print("END")
        return result
    return wrapper

# Step B: Use it manually
def say_hello(name):
    print(f"Hello, {name}!")

say_hello = my_decorator(say_hello)   # Wrap it!
say_hello("Alice")
# START
# Hello, Alice!
# END

# Step C: Use @ syntax (same thing, cleaner)
@my_decorator
def say_goodbye(name):
    print(f"Goodbye, {name}!")

say_goodbye("Bob")
# START
# Goodbye, Bob!
# END
```

**`@my_decorator` is literally just shorthand for `say_goodbye = my_decorator(say_goodbye)`.**

That's it! That's a decorator!

---

## Step 3: Real-World Decorator — Timer

```python
import time

def timer(func):
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        elapsed = time.time() - start
        print(f"⏱️ {func.__name__} took {elapsed:.4f} seconds")
        return result
    return wrapper

@timer
def slow_function():
    time.sleep(1)
    return "done"

@timer
def fast_function():
    return sum(range(1000))

slow_function()   # ⏱️ slow_function took 1.0012 seconds
fast_function()   # ⏱️ fast_function took 0.0001 seconds
```

You wrote the timing logic ONCE. You can add `@timer` to any function you want to time.

---

## Step 4: Real-World Decorator — Logging

```python
def log(func):
    def wrapper(*args, **kwargs):
        print(f"📋 Calling {func.__name__}(args={args}, kwargs={kwargs})")
        result = func(*args, **kwargs)
        print(f"📋 {func.__name__} returned {result}")
        return result
    return wrapper

@log
def add(a, b):
    return a + b

@log
def greet(name, greeting="Hello"):
    return f"{greeting}, {name}!"

add(3, 5)
# 📋 Calling add(args=(3, 5), kwargs={})
# 📋 add returned 8

greet("Alice", greeting="Hey")
# 📋 Calling greet(args=('Alice',), kwargs={'greeting': 'Hey'})
# 📋 greet returned Hey, Alice!
```

---

## Step 5: The `functools.wraps` Problem (IMPORTANT!)

Decorators break the function's identity:

```python
def my_decorator(func):
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper

@my_decorator
def greet(name):
    """Greet someone by name."""
    return f"Hello, {name}!"

print(greet.__name__)   # "wrapper" — WRONG! Should be "greet"
print(greet.__doc__)    # None — WRONG! Should be "Greet someone by name."
```

**Fix: Always use `@wraps(func)`**

```python
from functools import wraps

def my_decorator(func):
    @wraps(func)              # ← ADD THIS LINE
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper

@my_decorator
def greet(name):
    """Greet someone by name."""
    return f"Hello, {name}!"

print(greet.__name__)   # "greet" ✅
print(greet.__doc__)    # "Greet someone by name." ✅
```

**Rule: ALWAYS use `@wraps(func)` in your decorators. No exceptions.**

---

## Step 6: Decorators with Arguments

What if you want `@retry(3)` instead of just `@retry`?

You need THREE layers: factory → decorator → wrapper.

```python
from functools import wraps
import time

def retry(max_attempts, delay=1):
    """This is the FACTORY — it takes arguments and returns the actual decorator."""
    def decorator(func):
        """This is the DECORATOR — it takes the function and returns wrapper."""
        @wraps(func)
        def wrapper(*args, **kwargs):
            """This is the WRAPPER — it does the actual work."""
            for attempt in range(1, max_attempts + 1):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    print(f"  Attempt {attempt}/{max_attempts} failed: {e}")
                    if attempt < max_attempts:
                        time.sleep(delay)
            raise Exception(f"All {max_attempts} attempts failed!")
        return wrapper
    return decorator

@retry(3, delay=0.5)        # retry(3, delay=0.5) returns the decorator
def unreliable_api():
    import random
    if random.random() < 0.7:
        raise ConnectionError("Server down!")
    return "Success!"

print(unreliable_api())
```

**Think of it this way:**
1. `retry(3, delay=0.5)` → returns `decorator`
2. `decorator(unreliable_api)` → returns `wrapper`
3. When you call `unreliable_api()`, you're actually calling `wrapper()`

---

## Step 7: Stacking Multiple Decorators

```python
@log
@timer
def process_data():
    time.sleep(0.5)
    return "processed"

# This is equivalent to: process_data = log(timer(process_data))
# The INNER decorator (timer) wraps first, then the OUTER (log) wraps that.
```

---

## Step 8: Class-Based Decorators

You can also write decorators as classes:

```python
class CountCalls:
    """Counts how many times a function has been called."""
    def __init__(self, func):
        self.func = func
        self.count = 0

    def __call__(self, *args, **kwargs):
        self.count += 1
        print(f"Call #{self.count} to {self.func.__name__}")
        return self.func(*args, **kwargs)

@CountCalls
def greet(name):
    return f"Hello, {name}!"

greet("Alice")    # Call #1 to greet
greet("Bob")      # Call #2 to greet
greet("Charlie")  # Call #3 to greet
print(f"Total calls: {greet.count}")  # Total calls: 3
```

---

## Step 9: Built-in Decorators You MUST Know

```python
# @property — covered in OOP chapter (makes method act like attribute)

# @staticmethod — no self, no cls
# @classmethod — takes cls instead of self

# @lru_cache — automatic memoization (SUPER useful!)
from functools import lru_cache

@lru_cache(maxsize=128)
def fibonacci(n):
    if n < 2:
        return n
    return fibonacci(n-1) + fibonacci(n-2)

print(fibonacci(100))  # 354224848179261915075 — INSTANT!
# Without @lru_cache, this would take billions of years for n=100

# Check cache stats
print(fibonacci.cache_info())
# CacheInfo(hits=98, misses=101, maxsize=128, currsize=101)
```

---

## Common Mistakes

| Mistake | Why It's Wrong | Fix |
|---------|---------------|-----|
| Forgetting `@wraps(func)` | Function loses its name and docstring | Always add `@wraps(func)` |
| Forgetting `*args, **kwargs` in wrapper | Decorator breaks if function has arguments | Always use `*args, **kwargs` |
| Forgetting to `return result` in wrapper | Decorated function always returns `None` | Return the original function's result |
| Confusing `@decorator` with `@decorator()` | Without `()` passes function, with `()` passes return value | Know if your decorator takes arguments |

---

## Code Examples

### Easy
```python
# Example 1: Simplest decorator
from functools import wraps

def shout(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        result = func(*args, **kwargs)
        return result.upper()
    return wrapper

@shout
def greet(name):
    return f"hello, {name}"

print(greet("alice"))  # HELLO, ALICE


# Example 2: Decorator that doesn't modify the result
def debug(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        print(f"Calling {func.__name__}...")
        return func(*args, **kwargs)
    return wrapper

@debug
def add(a, b):
    return a + b

print(add(3, 5))  # Calling add... \n 8
```

### Medium
```python
# Example 3: Authentication decorator
def require_auth(func):
    @wraps(func)
    def wrapper(user, *args, **kwargs):
        if not user.get("authenticated", False):
            return "❌ Access denied! Please log in."
        return func(user, *args, **kwargs)
    return wrapper

@require_auth
def view_dashboard(user):
    return f"Welcome to the dashboard, {user['name']}!"

guest = {"name": "Guest", "authenticated": False}
admin = {"name": "Admin", "authenticated": True}
print(view_dashboard(guest))  # ❌ Access denied!
print(view_dashboard(admin))  # Welcome to the dashboard, Admin!


# Example 4: Cache decorator (simplified version of lru_cache)
def cache(func):
    memo = {}
    @wraps(func)
    def wrapper(*args):
        if args not in memo:
            memo[args] = func(*args)
        return memo[args]
    wrapper.cache = memo
    return wrapper

@cache
def expensive_calc(n):
    print(f"  Computing for {n}...")
    return sum(range(n))

print(expensive_calc(1000))    # Computing... then result
print(expensive_calc(1000))    # No computing — cached!
print(expensive_calc(2000))    # Computing... (new input)
```

### Advanced
```python
# Example 5: Decorator with arguments — validate types
def validate_types(*types):
    def decorator(func):
        @wraps(func)
        def wrapper(*args):
            for arg, expected_type in zip(args, types):
                if not isinstance(arg, expected_type):
                    raise TypeError(
                        f"Expected {expected_type.__name__}, got {type(arg).__name__}"
                    )
            return func(*args)
        return wrapper
    return decorator

@validate_types(str, int)
def repeat(text, times):
    return text * times

print(repeat("ha", 3))     # hahaha
# repeat(123, 3)            # TypeError: Expected str, got int
```

---

## Interview Questions (Top 10)

**Q1: What is a decorator in Python?**
> A function that takes another function, adds behavior, and returns a modified version — without changing the original's code. `@decorator` is shorthand for `func = decorator(func)`.

**Q2: What is `functools.wraps` and why use it?**
> It preserves the original function's `__name__`, `__doc__`, and other metadata after wrapping. Without it, debugging is painful because the function appears as "wrapper."

**Q3: How do decorators with arguments work?**
> Three layers: factory → decorator → wrapper. `@retry(3)` calls `retry(3)` which returns the decorator, which wraps the function.

**Q4: Can you stack multiple decorators?**
> Yes. `@A @B def func` equals `func = A(B(func))`. Bottom decorator wraps first, top wraps last.

**Q5: What is `@lru_cache`?**
> Built-in memoization decorator from `functools`. Caches function results by arguments. Perfect for recursive functions like Fibonacci. Turns O(2^n) into O(n).

**Q6: What is `@property`?**
> Makes a method behave like an attribute. `obj.name` calls the getter. Add `@name.setter` for write access with validation.

**Q7: What's the difference between `@staticmethod` and `@classmethod`?**
> `@staticmethod` takes nothing — just a utility function in the class. `@classmethod` takes `cls` — for factory methods and class-level operations.

**Q8: Can you write a decorator as a class?**
> Yes. `__init__` receives the function, `__call__` runs when the decorated function is called. Useful for decorators that need state (like counting calls).

**Q9: Write a simple timer decorator.**
```python
import time
from functools import wraps

def timer(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        print(f"{func.__name__}: {time.time()-start:.4f}s")
        return result
    return wrapper
```

**Q10: What is the difference between a decorator and a higher-order function?**
> A decorator IS a higher-order function. The difference is the usage pattern: decorators use `@` syntax and modify functions at definition time.

---

*← [Previous: Chapter 4 (Abstraction & Encapsulation)](04_Abstraction_Encapsulation.md) | [Next: Chapter 6 (Iterators & Generators) →](06_Iterators_Generators.md)*
