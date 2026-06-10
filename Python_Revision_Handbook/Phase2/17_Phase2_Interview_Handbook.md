# Chapter 17: Phase 2 Intermediate Python Interview Handbook
### *Total Questions: 75*
### *The Ultimate Conversational Guide for Technical Screens and System Design*

---

## 📂 Topic Directory
1. [OOP & Classes (Q1 - Q6)](#1-oop--classes-q1---q6)
2. [Inheritance & MRO (Q7 - Q11)](#2-inheritance--mro-q7---q11)
3. [Polymorphism & Operator Overloading (Q12 - Q16)](#3-polymorphism--operator-overloading-q12---q16)
4. [Abstraction & Encapsulation (Q17 - Q20)](#4-abstraction--encapsulation-q17---q20)
5. [Decorators (Q21 - Q25)](#5-decorators-q21---q25)
6. [Iterators & Generators (Q26 - Q30)](#6-iterators--generators-q26---q30)
7. [Lambdas, Closures & Functional Programming (Q31 - Q35)](#7-lambdas-closures--functional-programming-q31---q35)
8. [Recursion (Q36 - Q39)](#8-recursion-q36---q39)
9. [Collections Module (Q40 - Q44)](#9-collections-module-q40---q44)
10. [Dataclasses (Q45 - Q48)](#10-dataclasses-q45---q48)
11. [Regular Expressions (Regex) (Q49 - Q52)](#11-regular-expressions-regex-q49---q52)
12. [Logging (Q53 - Q56)](#12-logging-q53---q56)
13. [Virtual Environments & Requirements (Q57 - Q60)](#13-virtual-environments--requirements-q57---q60)
14. [Context Managers (Q61 - Q64)](#14-context-managers-q61---q64)
15. [Copy Module (Q65 - Q68)](#15-copy-module-q65---q68)
16. [Time & Memory Optimization (Q69 - Q72)](#16-time--memory-optimization-q69---q72)
17. [Multithreading & Multiprocessing (Q73 - Q75)](#17-multithreading--multiprocessing-q73---q75)

---

## 1. OOP & Classes (Q1 - Q6)

### Q1: What is the real difference between class attributes and instance attributes?
> **Answer:** So, the easiest way to think about it is where the data is actually stored in memory. 
> An **instance attribute** belongs to a specific object. When you write `self.name = name` inside `__init__`, you're saying "this data is just for this one instance." If you have ten objects, you have ten independent copies in memory.
> A **class attribute** is defined directly inside the class body, outside any methods. It is shared by *every* single object you instantiate from that class. If one object changes a class attribute, that change is immediately visible to all the other objects because they all point to the exact same place in memory.

### Q2: When should I use `@classmethod` vs `@staticmethod` in a real project?
> **Answer:** I use `@classmethod` when the method needs to know about the class itself—for example, if you're building "factory methods" to create objects in different ways. If you have a `User` class, and you want to create a user from a JSON string or from a database row, you write a `@classmethod` that takes `cls` and returns `cls(data)`.
> On the other hand, `@staticmethod` doesn't need to know anything about the class or the object instances. It's just a regular utility function that logically belongs inside the class namespace. For instance, a method `is_valid_email(email)` inside a `User` class doesn't touch any class properties, it just does validation, so it's static.

### Q3: What is the `@property` decorator, and why not just write a normal method like `get_area()`?
> **Answer:** `@property` is great because it lets you access a method like a normal attribute, without using parentheses (so `rectangle.area` instead of `rectangle.area()`). 
> The biggest practical reason to use it is that it keeps your API clean. If you start with a simple attribute `self.price`, and later on you decide you need to validate that prices are never negative, you can turn `price` into a `@property` with a setter. The developers using your code don't have to change their code from `obj.price` to `obj.get_price()`—it just works seamlessly.

### Q4: [Verbal Code] What happens if I try to update a class attribute using `self.attribute = value`?
> **Answer:** This is a very common trap. If you have a class attribute like `count = 0`, and you write `self.count = 5` inside an instance method, you aren't updating the class attribute. 
> Instead, Python dynamically creates a brand new **instance attribute** on that specific object called `count`, and masks the class attribute. The class attribute remains `0`, and other instances will still see `0`, but the instance you modified will now return `5`. If you actually want to update a class attribute, you must reference it using the class name: `MyClass.count = 5`.

### Q5: [Error Debugging] Why does my code throw `TypeError: bark() takes 0 positional arguments but 1 was given`?
> **Answer:** This happens because you defined a method inside a class but forgot to pass `self` as the first parameter:
> ```python
> class Dog:
>     def bark():  # Missing self!
>         return "Woof"
> ```
> When you call `my_dog.bark()`, Python automatically passes the object instance (`my_dog`) as the first argument behind the scenes. Because your method wasn't set up to accept any arguments, Python complains that it was given one argument it wasn't expecting.

### Q6: [System Design] How would you design a class that tracks the total number of its active instances in memory?
> **Answer:** You can handle this by combining a class attribute with the `__init__` and `__del__` magic methods.
> ```python
> class InstanceTracker:
>     active_count = 0  # Class attribute to store the count
> 
>     def __init__(self):
>         InstanceTracker.active_count += 1
> 
>     def __del__(self):
>         InstanceTracker.active_count -= 1
> ```
> Every time a new instance is created, `__init__` increments the count. When an instance is garbage-collected or deleted using `del`, Python runs the destructor `__del__`, which decrements the count. 
> *Note:* I'd warn the interviewer that `__del__` is not 100% reliable in Python if there are circular references or if the program terminates abruptly, but for standard scenarios, this works.

---

## 2. Inheritance & MRO (Q7 - Q11)

### Q7: How does multiple inheritance work in Python, and what is the Method Resolution Order (MRO)?
> **Answer:** Multiple inheritance allows a class to inherit from more than one parent class. To resolve where methods are looked up (especially if multiple parents override the same method), Python uses the **MRO** (Method Resolution Order), which is calculated using the C3 Linearization algorithm.
> MRO creates a linear, predictable lookup order. It prioritizes subclasses before parents, and left-parent classes before right-parent classes. You can inspect this path for any class by calling `MyClass.__mro__` or `MyClass.mro()`.

### Q8: What does `super()` actually do, and is it just a way to call the parent class?
> **Answer:** That's a common misconception. In simple inheritance, yes, `super()` calls the parent. But in multiple inheritance, `super()` does not look for the parent class. Instead, it returns a proxy object that delegates method calls to the **next class in the MRO chain**. 
> This is crucial because it ensures that in a complex inheritance structure, every parent class gets called exactly once and in the correct order, avoiding redundant calls.

### Q9: [Verbal Code] What is the output of `super().__init__()` in a classic "Diamond" inheritance pattern?
> **Answer:** Let's trace it. Suppose Class `D` inherits from `B` and `C`, and both `B` and `C` inherit from `A`. If you call `super().__init__()` inside `D`, the MRO lookup sequence is `D -> B -> C -> A`. 
> When `D` calls `super().__init__()`, it goes to `B`'s `__init__`. Inside `B`, when it calls `super().__init__()`, it doesn't call `A`—instead, it calls **`C`'s `__init__`**, because `C` is next in `D`'s MRO. Finally, `C`'s call to `super().__init__()` routes to `A`. This ensures every initializer runs once without duplicate calls to `A`.

### Q10: [Error Debugging] What does `TypeError: Cannot create a consistent method resolution order (MRO)` mean?
> **Answer:** This error means you've created an impossible inheritance structure that violates the rules of C3 Linearization. Usually, this happens when you try to define a subclass that contradicts the lookup hierarchy of its parents.
> For example, if you define `class A(B, C)` but `B` itself inherits from `C`, Python gets confused because the local precedence says `B` should come before `C`, but the inheritance rule says `C` is a parent of `B`. To fix this, you have to reorder the parents to respect the hierarchy.

### Q11: [System Design] How would you design a mixin class to add JSON serialization to any model?
> **Answer:** I would create a clean mixin class that utilizes Python's dynamic `__dict__` attribute to convert instance data into a dictionary, which can then be serialized.
> ```python
> import json
> 
> class JSONSerializerMixin:
>     def to_json(self):
>         # We filter out private/protected variables to keep the JSON clean
>         public_data = {k: v for k, v in self.__dict__.items() if not k.startswith('_')}
>         return json.dumps(public_data)
> 
> class User(JSONSerializerMixin):
>     def __init__(self, name, email):
>         self.name = name
>         self.email = email
> ```
> Now, any model that inherits from `JSONSerializerMixin` automatically gets the `.to_json()` method without needing to write custom serialization code for each class.

---

## 3. Polymorphism & Operator Overloading (Q12 - Q16)

### Q12: What is Duck Typing, and how does it make Python different from languages like Java?
> **Answer:** In languages like Java, if a function expects an object of type `DatabaseReader`, you *must* pass an object that explicitly implements that class or interface. 
> In Python, we use **Duck Typing**: *"If it walks like a duck and quacks like a duck, it's a duck."* Python doesn't care about the type or class hierarchy; it only checks if the object has the method or behavior we need. If my function reads data from an object, I can pass anything that has a `.read()` method—whether it's an actual file, an in-memory string buffer, or a mock database connection.

### Q13: How do you implement operator overloading, like making two custom objects addable with `+`?
> **Answer:** Python maps standard operators to special "dunder" (double underscore) methods. To make objects of a class addable, you implement the `__add__(self, other)` method inside your class. 
> ```python
> class Point:
>     def __init__(self, x, y):
>         self.x = x
>         self.y = y
>     def __add__(self, other):
>         return Point(self.x + other.x, self.y + other.y)
> ```
> When you run `p1 + p2`, Python runs `p1.__add__(p2)` under the hood and returns a new `Point` object.

### Q14: What is the difference between `__str__` and `__repr__`?
> **Answer:** The simplest way to remember is who they are for:
> * `__str__` is for **users**. It should return a clean, friendly, readable string representation (like when you use `print()` or `str()`).
> * `__repr__` is for **developers**. It should return an unambiguous representation, ideally looking like the code you would write to recreate the object (e.g., `Point(x=1, y=2)`). It is called inside lists, collections, and the interactive REPL.
> If you only implement one, implement `__repr__`, because Python will fallback to it if `__str__` is missing.

### Q15: [Verbal Code] What does the `__call__` method do, and how does it make an object act like a function?
> **Answer:** The `__call__` method allows you to call an instance of a class exactly like a regular function. You just write the method inside your class, and it runs when you write `instance()`.
> ```python
> class Multiplier:
>     def __init__(self, factor):
>         self.factor = factor
>     def __call__(self, x):
>         return x * self.factor
> 
> double = Multiplier(2)
> print(double(5))  # Output: 10
> ```
> This is super useful when you need an object to keep some state between calls, like a counter or a memoization cache, but still want it to look like a clean function call to the rest of the application.

### Q16: [Error Debugging] Why does printing my custom object show `<__main__.Car object at 0x...>` instead of its name?
> **Answer:** This happens because you haven't defined the `__str__` or `__repr__` magic methods in your class. When you call `print(my_car)`, Python fallback to the default implementation inherited from the base `object` class, which prints the class name and its memory address. To fix this, just define `__str__(self)` to return the string you want.

---

## 4. Abstraction & Encapsulation (Q17 - Q20)

### Q17: Does Python actually have private variables, or is it just a gentleman's agreement?
> **Answer:** To be honest, it is mostly a gentleman's agreement. Python doesn't have strict access modifiers like `private` in C++ or Java. 
> * A single underscore (like `_value`) is a visual hint to other developers: "This is private, don't modify it directly." But Python doesn't restrict anyone from doing `obj._value = 10`.
> * A double underscore (like `__value`) triggers **name mangling**, where Python renames the attribute in memory to prevent accidental access. However, you can still access it if you try hard enough.

### Q18: What is name mangling, and why does Python do it?
> **Answer:** Name mangling is what happens when you prefix a class attribute with a double underscore (like `__my_var`). Python automatically renames this attribute in memory to `_ClassName__my_var`. 
> Python does this to prevent name collisions in inheritance hierarchies. If a subclass has an attribute with the same name as a private attribute in the parent class, name mangling ensures they don't overwrite each other in memory.

### Q19: How do Abstract Base Classes (ABCs) work, and how do you enforce them?
> **Answer:** An Abstract Base Class is a class that serves as a blueprint. You cannot instantiate it directly; it forces its subclasses to implement specific methods. 
> You implement it by importing `ABC` and `abstractmethod` from the `abc` module, inheriting from `ABC`, and putting the `@abstractmethod` decorator above any method that the child classes *must* implement.

### Q20: [Error Debugging] Why does my code raise `TypeError: Can't instantiate abstract class with abstract methods`?
> **Answer:** This error means you defined a subclass of an Abstract Base Class, but you forgot to implement one of the methods marked with `@abstractmethod`. Python prevents you from instantiating the subclass until you implement all required abstract methods.

---

## 5. Decorators (Q21 - Q25)

### Q21: Can you explain how a decorator works under the hood without using the `@` symbol?
> **Answer:** A decorator is just a function that accepts another function as an argument, wraps it with some extra behavior, and returns the wrapped function. The `@` syntax is just syntactic sugar.
> Writing this:
> ```python
> @my_decorator
> def my_func():
>     pass
> ```
> Is exactly the same as writing:
> ```python
> def my_func():
>     pass
> my_func = my_decorator(my_func)
> ```

### Q22: Why is `@functools.wraps` practically mandatory when writing custom decorators?
> **Answer:** When you decorate a function, Python replaces it with the inner wrapper function defined inside the decorator. If you inspect the decorated function's name (`__name__`) or docstring (`__doc__`), it will show the wrapper's details instead of the original function's.
> Applying `@wraps(func)` to the inner wrapper copies the original function's metadata back, which prevents debugging tools, IDE autocompletes, and loggers from getting confused.

### Q23: How do you pass arguments to a decorator itself?
> **Answer:** You have to add an extra layer of nesting (making it a function that returns a decorator, which in turn returns a wrapper). The outer layer accepts the decorator arguments, the middle layer accepts the target function, and the inner wrapper executes the logic.
> ```python
> def repeat(times):
>     def decorator(func):
>         def wrapper(*args, **kwargs):
>             for _ in range(times):
>                 func(*args, **kwargs)
>         return wrapper
>     return decorator
> ```

### Q24: [Verbal Code] What is the execution order if you stack multiple decorators on a single function?
> **Answer:** They execute from the **inside out** (or bottom to top).
> If you write:
> ```python
> @dec_one
> @dec_two
> def my_func():
>     pass
> ```
> Python compiles this as: `my_func = dec_one(dec_two(my_func))`. So, when you call `my_func()`, `dec_one` runs first, passes control to `dec_two`, which then executes the actual function.

### Q25: [System Design] How would you build a decorator that limits how many times a function can be run per minute (rate limiter)?
> **Answer:** I would write a decorator that tracks call timestamps inside the wrapper using a list, and compares them to the current time:
> ```python
> import time
> 
> def rate_limit(max_per_minute):
>     def decorator(func):
>         calls = []  # Keeps track of call timestamps
>         def wrapper(*args, **kwargs):
>             now = time.time()
>             # Keep only calls made in the last 60 seconds
>             calls[:] = [t for t in calls if now - t < 60]
>             if len(calls) >= max_per_minute:
>                 raise Exception("Rate limit exceeded!")
>             calls.append(now)
>             return func(*args, **kwargs)
>         return wrapper
>     return decorator
> ```
> Since lists are mutable, the `calls` list acts as a persistent state inside the closure, allowing the decorator to track state across multiple calls.

---

## 6. Iterators & Generators (Q26 - Q30)

### Q26: Explain the difference between an Iterable and an Iterator.
> **Answer:** An **Iterable** is anything you can loop over (like a list, set, or string). It doesn't track its own traversal state. It just implements the `__iter__` method, which must return an Iterator.
> An **Iterator** is the object that actually does the work of traversing. It tracks where it is in the collection. It must implement `__next__` (which returns the next item or raises `StopIteration` when finished) and `__iter__` (which typically returns itself).

### Q27: How does the `yield` keyword work, and how does it save memory?
> **Answer:** When a function contains `yield`, it becomes a **generator**. When you call the function, Python doesn't execute the code. Instead, it returns a generator object (which is an iterator).
> When you call `next()` on it, the function runs until it hits `yield`, pauses, returns that value, and remembers its entire state. On the next call, it picks up exactly where it paused. This saves memory because instead of building a massive list in RAM all at once (`O(n)` memory), you generate one item at a time (`O(1)` memory).

### Q28: What is the difference between a generator expression and a list comprehension?
> **Answer:** Syntactically, list comprehensions use square brackets `[x for x in data]` and generator expressions use parentheses `(x for x in data)`.
> In execution, a list comprehension executes immediately and loads the entire resulting list into RAM. A generator expression executes lazily, returning a generator object immediately and calculating values on the fly only when you iterate over it.

### Q29: [Verbal Code] How do you pass data into a generator using `.send()`?
> **Answer:** You call `generator.send(value)`. When you do this, the generator resumes execution, and the value you sent becomes the result of the current `yield` expression.
> ```python
> def my_gen():
>     val = yield "Start"
>     yield f"Received: {val}"
> 
> g = my_gen()
> print(next(g))       # Output: "Start" (pauses at first yield)
> print(g.send("Hi"))  # Output: "Received: Hi" (val gets the value "Hi")
> ```
> *Note:* You must call `next(g)` or `g.send(None)` to "prime" the generator before sending any real data.

### Q30: [Error Debugging] Why does my code throw `StopIteration` inside a loop?
> **Answer:** You shouldn't see `StopIteration` inside a normal `for` loop because Python catches it and exits the loop automatically. 
> If you are seeing it, you are likely calling `next(my_iterator)` manually inside your code, and the iterator ran out of elements. To fix this, you should either wrap your `next()` calls in a `try-except` block to handle the end of iteration, or pass a default value to `next()`, like `next(my_iterator, None)`.

---

## 7. Lambdas, Closures & Functional Programming (Q31 - Q35)

### Q31: What is a closure, and how does it remember outer scope variables?
> **Answer:** A closure is a nested function that retains access to variables from its outer (enclosing) function, even after the outer function has finished executing. 
> Python does this by storing these variables in a special attribute on the nested function called `__closure__`. These referenced variables are called "free variables."

### Q32: What is the "late binding trap" in Python loops, and how do you fix it?
> **Answer:** This trap occurs because Python closures look up variables in the outer scope **when the function is called**, not when it is defined. 
> If you create a list of functions inside a loop:
> ```python
> funcs = [lambda: i for i in range(3)]
> ```
> Calling them will return `[2, 2, 2]` because the loop finishes before the functions are run, and the final value of `i` in memory is `2`.
> **Fix:** Bind the variable as a default argument at definition time: `lambda i=i: i`. Default arguments are evaluated immediately when the function is defined.

### Q33: When should you use a lambda function, and when should you avoid them?
> **Answer:** Lambdas are great for simple, one-line functions that you only need to use once—like passing a key function to `sorted()` (e.g., `sorted(data, key=lambda x: x['age'])`).
> You should avoid them if the logic is complex, requires multiple lines, or needs clear error tracebacks. Lambdas are anonymous, so if they crash, the stack trace just says `<lambda>`, making debugging much harder. In those cases, define a proper named function using `def`.

### Q34: [Verbal Code] Explain what `map()`, `filter()`, and `reduce()` do and what they return.
> **Answer:** 
> * `map(func, iterable)` applies `func` to every item in the collection. It returns an iterator.
> * `filter(func, iterable)` tests every item with `func(item)`. It returns an iterator yielding only items that returned a truthy value.
> * `reduce(func, iterable)` (from `functools`) repeatedly applies `func` to accumulate the elements into a single final value (e.g. summing a list). It returns a single value.

### Q35: [Error Debugging] Why am I getting `UnboundLocalError: local variable 'x' referenced before assignment` inside a nested function?
> **Answer:** This happens because you are trying to modify an outer variable (like `x += 1`) inside a nested function. 
> Python sees the assignment (`x = ...`) and automatically flags `x` as a local variable. But because you tried to read it before assigning it, it crashes. To fix this, declare `nonlocal x` at the top of the nested function to tell Python to use the outer scope's variable instead of creating a new local one.

---

## 8. Recursion (Q36 - Q39)

### Q36: What is the default recursion limit in Python, and why does it exist?
> **Answer:** The default limit is **1000**. It exists because CPython (the standard Python interpreter) does not optimize tail-call recursion. 
> Every recursive call adds a new stack frame to the computer's memory. If there were no limit, infinite recursion would consume all available stack memory and cause a segmentation fault, crashing the entire Python process.

### Q37: How do you write a recursive function without running into a `RecursionError`?
> **Answer:** First, make sure you have a clear **base case** (the condition that stops the recursion). Second, make sure your **recursive case** modifies the input so that it moves closer to the base case with every call. If the depth is legitimately deeper than 1000, you should convert the function into an iterative loop using an explicit list as a stack.

### Q38: What is memoization, and how does `@lru_cache` work?
> **Answer:** Memoization is a caching technique where you store the results of function calls so that if the function is called again with the same arguments, you return the cached result instead of recomputing it.
> In Python, you can apply this to recursive functions by adding the `@functools.lru_cache` decorator. It speeds up exponential recursive algorithms (like naive Fibonacci) to linear time.

### Q39: [Verbal Code] Walk me through the call stack changes during a simple recursive factorial calculation of `factorial(3)`.
> **Answer:** 
> 1. `factorial(3)` is called. A stack frame is created. It checks the base case (fails), then pauses to compute `3 * factorial(2)`.
> 2. `factorial(2)` is called. Another stack frame is added. It pauses to compute `2 * factorial(1)`.
> 3. `factorial(1)` is called. A third stack frame is added. The base case hits and returns `1`.
> 4. The stack starts unwinding. `factorial(1)` frame is popped, returning `1` to `factorial(2)`.
> 5. `factorial(2)` computes `2 * 1 = 2`, its frame is popped, and returns `2` to `factorial(3)`.
> 6. `factorial(3)` computes `3 * 2 = 6`, and returns `6` as the final result.

---

## 9. Collections Module (Q40 - Q44)

### Q40: Why would I use a `defaultdict` instead of a regular dictionary with `.get()` or `.setdefault()`?
> **Answer:** Using `.setdefault(key, [])` or `.get(key, [])` requires you to write boilerplate code every time you insert or access values, and it can be slow because you are instantiating default values even when they aren't needed.
> A `defaultdict` sets up a default factory (like `list` or `int`) once at creation time. If you reference a missing key (`d['missing']`), it automatically instantiates and inserts the default value under the hood, making your code much cleaner and faster.

### Q41: What is a `Counter`, and how does it handle missing elements?
> **Answer:** `Counter` is a dictionary subclass designed for counting elements. If you look up a key that doesn't exist, instead of raising a `KeyError`, it returns `0`. It also includes helper methods like `.most_common(n)` to quickly find the most frequent elements.

### Q42: What is a `deque`, and why is it faster than a list for queues?
> **Answer:** A `deque` (double-ended queue) is implemented as a doubly linked list under the hood. This means adding or removing elements from either the beginning or the end takes constant time (`O(1)`).
> A standard Python `list` is a dynamic array. Adding or removing from the beginning (`list.insert(0, val)` or `list.pop(0)`) takes linear time (`O(n)`) because all the other elements in the array have to be shifted in memory.

### Q43: How does `namedtuple` compare to a regular class or a dictionary?
> **Answer:** 
> * Compared to a **dict**: `namedtuple` is immutable, has a smaller memory footprint, and allows you to access values using dot notation (`point.x`) instead of brackets (`point['x']`), preventing typos.
> * Compared to a **class**: `namedtuple` requires zero boilerplate to write, is highly lightweight, and supports indexing and unpacking out of the box because it inherits from the built-in `tuple`.

### Q44: [Error Debugging] Why is my dictionary not maintaining insertion order in an older Python codebase?
> **Answer:** Prior to Python 3.7, standard dictionaries did not guarantee insertion order. If you need to write code that is backward-compatible with legacy Python versions, you must use `collections.OrderedDict` to guarantee that insertion order is preserved.

---

## 10. Dataclasses (Q45 - Q48)

### Q45: What is the benefit of using `@dataclass` over a traditional class?
> **Answer:** It eliminates boilerplate code. For a class designed primarily to store data, writing `__init__`, `__repr__`, and `__eq__` manually is tedious. 
> The `@dataclass` decorator reads your type annotations and writes these methods for you automatically, saving time and keeping your code DRY.

### Q46: How do you validate fields in a dataclass after it is initialized?
> **Answer:** You write a special method called `__post_init__(self)`. Python automatically runs this method immediately after the generated `__init__` finishes. It is the perfect place to run validation checks or calculate derived fields.

### Q47: How do you make a dataclass read-only (immutable)?
> **Answer:** You pass `frozen=True` to the decorator: `@dataclass(frozen=True)`. This auto-generates setter and deleter methods that raise a `FrozenInstanceError` if anyone tries to modify or delete attributes after the object is created. It also makes the class instances hashable so they can be stored in sets or used as dictionary keys.

### Q48: [Error Debugging] Why does writing `items: list = []` in a dataclass raise a `ValueError`?
> **Answer:** Python flags this because mutable default arguments (like lists or dicts) are shared across all class instances. If you modify the list in one object, it will change for all other objects. 
> **Fix:** You must use `default_factory`:
> ```python
> from dataclasses import dataclass, field
> @dataclass
> class User:
>     items: list = field(default_factory=list)
> ```

---

## 11. Regular Expressions (Regex) (Q49 - Q52)

### Q49: What is the difference between `re.match()` and `re.search()`?
> **Answer:** `re.match()` only checks for a pattern starting at the **very first character** of the string. If it's not there, it returns `None`. `re.search()` scans the **entire string** and returns the first match it finds anywhere.

### Q50: What does it mean when a regex is "greedy" vs "lazy"?
> **Answer:** Greedy quantifiers (`*`, `+`, `{}`) match as much text as possible. Lazy quantifiers match as little text as possible to satisfy the pattern.
> For example, matching `<.*>` against `<div>Hello</div>` returns the whole string (greedy). Adding a question mark to make it lazy (`<.*?>`) matches only `<div>`.

### Q51: How do capture groups and backreferences work?
> **Answer:** Capture groups are marked with parentheses `()`. They allow you to isolate and extract parts of a match. 
> **Backreferences** (like `\1` or `\2`) let you reference a previously matched group inside the pattern itself. For example, `r"(\w+)\s+\1"` matches duplicate words like "the the".

### Q52: [Error Debugging] Why is my regex pattern `\d+\b` matching unexpected things?
> **Answer:** You likely forgot to prefix your string with `r` (raw string). Without the `r` prefix, Python's string parser interprets `\b` as a backspace character before passing it to the regex engine, causing the pattern to fail. Always write regex patterns as raw strings: `r"\d+\b"`.

---

## 12. Logging (Q53 - Q56)

### Q53: Why should I use the `logging` module instead of just redirecting standard prints?
> **Answer:** Standard print statements have no concept of log levels (like info vs crash), no built-in timestamps, and cannot easily be routed to files or third-party monitoring services dynamically. 
> The `logging` library allows you to configure formatting, levels, and destinations globally without changing any application code.

### Q54: What are logging handlers, formatters, and propagation?
> **Answer:** 
> * **Handlers:** Determine where the log goes (e.g. `StreamHandler` writes to console, `FileHandler` writes to disk).
> * **Formatters:** Define the layout of the log string (e.g., adding timestamp, line number, and log level).
> * **Propagation:** When a child logger (like `app.db`) forwards its log records up to its parent loggers so parent handlers can handle them too.

### Q55: How does log level filtering work across loggers and handlers?
> **Answer:** The logger is the first gatekeeper. If the logger level is set to `WARNING`, it discards `INFO` and `DEBUG` logs immediately. 
> If the log passes, it is sent to the handlers. Each handler has its own level gatekeeper. For example, a `FileHandler` might be set to `ERROR`, meaning it will ignore any `WARNING` logs passed down by the logger.

### Q56: [System Design] How would you set up log rotation to prevent a production server's disk from filling up?
> **Answer:** I would configure the logger to use `RotatingFileHandler` or `TimedRotatingFileHandler` from `logging.handlers`. 
> You configure `maxBytes` (to cap file size, e.g., 5MB) and `backupCount` (to define how many archived files to keep). Once the log hits the cap, it rolls over to `app.log.1` and deletes the oldest archive, ensuring log storage never exceeds a fixed size.

---

## 13. Virtual Environments & Requirements (Q57 - Q60)

### Q57: What actually happens to your computer's state when you "activate" a virtual environment?
> **Answer:** It doesn't use virtual machines. The activation script simply alters your current terminal session's `PATH` environment variable. 
> It prepends your virtual environment's `bin` (or `Scripts`) folder to the front of `PATH`. When you type `python` or `pip`, the OS finds and runs the environment's isolated binary copy before finding the global system python.

### Q58: What is the difference between `pip freeze` and `pip list`?
> **Answer:** `pip list` shows a clean, human-readable list of all installed packages. `pip freeze` prints them formatted in `package==version` format, which is the exact syntax expected by `requirements.txt`.

### Q59: What do specifiers like `~=` and `==` mean in `requirements.txt`?
> **Answer:** 
> * `==` is an exact match (e.g., `requests==2.31.0` locks requests to that exact version).
> * `~=` is compatible release. `requests~=2.31.0` means "install any `2.31.x` version, but do not install `2.32.0` or higher." This is great for safely receiving minor security patches without introducing breaking changes.

### Q60: [System Design] How do you handle production lockfiles vs development requirements?
> **Answer:** I split them. I keep a `requirements.txt` containing only the direct libraries my app needs (like `django` or `pandas`), with flexible version markers. 
> For deployment, I use a tool like `pip-tools` or `Poetry` to compile a `requirements-lock.txt` that records the exact versions and hashes of every package and sub-dependency. This ensures developers and production run the exact same code environment.

---

## 14. Context Managers (Q61 - Q64)

### Q61: What are the two magic methods that power the `with` statement?
> **Answer:** 
> * `__enter__(self)` runs at the start of the `with` block. Whatever it returns is bound to the variable after the `as` keyword.
> * `__exit__(self, exc_type, exc_val, exc_tb)` runs at the end of the block, even if the code inside crashed. It handles cleaning up resources.

### Q62: How do you implement a context manager using a generator instead of a class?
> **Answer:** You write a generator function and decorate it with `@contextmanager` from `contextlib`. 
> Anything before the `yield` statement acts as `__enter__`. The value you `yield` goes to the `as` variable. Anything after the `yield` (written inside a `try...finally` block) acts as the `__exit__` cleanup code.

### Q63: [Verbal Code] How does a context manager handle or suppress exceptions?
> **Answer:** Inside `__exit__(self, exc_type, exc_val, exc_tb)`, if an exception occurred in the block, the details are passed in the parameters. 
> If you return `True` from `__exit__`, Python suppresses the exception and continues running the script. If you return `False` or `None`, the exception propagates and crashes the program.

### Q64: [Error Debugging] Why did my generator-based context manager fail to run its cleanup code after a crash?
> **Answer:** This is a classic error. If you write a generator-based context manager like this:
> ```python
> @contextmanager
> def my_manager():
>     setup()
>     yield resource
>     cleanup()  # ❌ Dangerous!
> ```
> If the `with` block crashes, the exception is raised *at the yield statement* inside your generator. Because there is no error handling around `yield`, the generator crashes immediately, and the `cleanup()` line is never reached. 
> **Fix:** Always wrap the `yield` in a `try...finally` block:
> ```python
> @contextmanager
> def my_manager():
>     setup()
>     try:
>         yield resource
>     finally:
>         cleanup()  # ✅ Guaranteed to run
> ```

---

## 15. Copy Module (Q65 - Q68)

### Q65: What is the difference between reference assignment (`=`), shallow copy, and deep copy?
> **Answer:** 
> * **Assignment (`=`)** doesn't copy anything. It just points a new variable to the same object in memory.
> * **Shallow Copy (`copy.copy()`)** creates a new container, but inserts references to the child objects. If the container has nested lists, the nested lists are shared between the original and the copy.
> * **Deep Copy (`copy.deepcopy()`)** recursively copies the container *and* all nested objects, creating a completely independent duplicate.

### Q66: How does a shallow copy handle nested lists or dictionaries?
> **Answer:** It copies the references to the nested lists, not the list objects themselves. If you change a top-level item in the copy, the original is unaffected. But if you modify a nested item (like appending to a sublist), both the original and copy will show the change because they both point to the same sublist in memory.

### Q67: What is the difference between `is` and `==`?
> **Answer:** 
> * `==` checks for **equality of value**. If two objects contain the same data, they are `==` equal.
> * `is` checks for **identity in memory**. It returns `True` only if both variables point to the exact same memory address.

### Q68: [Error Debugging] Why does copying an object with a database connection raise `TypeError: cannot pickle`?
> **Answer:** This happens because `copy.deepcopy()` recursively attempts to copy everything inside an object. System resources like file streams, sockets, database connection pools, and thread locks cannot be copied because they represent real connections bound to the OS.
> **Fix:** You must define `__deepcopy__(self, memo)` on your class to define custom copy logic, ensuring these resource pointers are skipped or initialized fresh in the copy.

---

## 16. Time & Memory Optimization (Q69 - Q72)

### Q69: What is the time complexity of searching a list vs a set?
> **Answer:** 
> * **List:** `O(n)` (Linear time). Python must check elements one by one from index 0 until it finds a match.
> * **Set:** `O(1)` (Constant time). Sets are implemented as hash tables, so Python calculates the memory address of the item instantly, regardless of the size of the set.

### Q70: How does declaring `__slots__` optimize class instances memory?
> **Answer:** Normally, Python objects store their attributes inside a dynamic dictionary (`__dict__`). Dictionaries consume a lot of memory because they keep extra buffers to prevent collisions.
> Declaring `__slots__` tells Python: *"This class will only ever have these specific attributes."* Python drops the dynamic dictionary and allocates a small, fixed-size array in memory instead, reducing memory footprint by up to 70%.

### Q71: What is the difference in memory footprint between list comprehensions and generator expressions?
> **Answer:** A list comprehension allocates memory and builds the entire list in RAM immediately. 
> A generator expression returns a generator object that computes elements lazily one-by-one only when requested, using a constant `O(1)` memory footprint regardless of list size.

### Q72: [System Design] How does Python's generational garbage collector clean up circular references?
> **Answer:** Python's primary memory management is Reference Counting. When reference count hits 0, the object is deleted. But circular references (e.g. A references B, B references A) will never reach 0 reference count, causing a memory leak.
> To solve this, Python runs a generational garbage collector in the background. It divides objects into three generations (0, 1, 2) based on how long they've survived. It periodically scans these objects, detects isolated cycles of objects that are unreachable from the root namespace, and cleans them up.

---

## 17. Multithreading & Multiprocessing (Q73 - Q75)

### Q73: What is the GIL, and why does it make Python multithreading useless for CPU-bound tasks?
> **Answer:** The GIL is the **Global Interpreter Lock**. It is a mutex lock in the CPython interpreter that ensures only one thread executes Python bytecode at any given time.
> For CPU-bound tasks (like image processing or calculations), multiple threads end up fighting for the lock, adding context-switching overhead and running slower than sequential code. 
> Threads are only useful for **I/O-bound tasks** (like web scraping), because Python releases the GIL while a thread is waiting on a response from a database or network.

### Q74: When should I use multithreading vs multiprocessing?
> **Answer:** 
> * Use **Multithreading** for **I/O-bound** tasks where the code spends most of its time waiting for external responses (e.g. calling APIs, downloading files, database reads).
> * Use **Multiprocessing** for **CPU-bound** tasks that need raw processing power (e.g. data science models, math equations). Since processes spawn separate interpreters in separate memory spaces, they bypass the GIL entirely.

### Q75: [Error Debugging] Why does my multiprocessing code crash on Windows if I don't use `if __name__ == '__main__':`?
> **Answer:** On Windows, the OS spawns child processes by importing your script file. If you do not wrap your process-spawning code in `if __name__ == '__main__':`, the child processes will execute the script from the beginning, attempt to spawn *their own* child processes, and crash in an infinite fork-bomb loop.

---

*← [Previous: Chapter 16 (Multithreading & Multiprocessing)](16_Multithreading_Multiprocessing.md) | [Home (README) →](../README.md)*
