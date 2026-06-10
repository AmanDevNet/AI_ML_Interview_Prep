# Phase 1 — Interview Questions (150)

> All answers are written in **natural interview language** — the way you'd actually say it to a person sitting across the table. Read them out loud and practice.

---

## SECTION 1: Python Basics & Variables (Q1–Q15)

**Q1: What is Python? Why do companies use it?**
> Python is a high-level, interpreted language. Companies use it because it's readable, has a massive ecosystem of libraries, and you can build things fast — whether it's a web app, ML model, or automation script. It's the go-to language for data science and AI right now.

**Q2: What does "interpreted" mean?**
> It means Python runs your code line by line. It doesn't compile the whole program into machine code first like C or Java. So you can test things quickly, but it's generally slower at runtime.

**Q3: What is PEP 8?**
> It's Python's official style guide — rules for how to format your code so everyone writes it the same way. Stuff like use 4 spaces for indentation, keep lines under 79 characters, use snake_case for variable names. It's not enforced by the language, but every team expects you to follow it.

**Q4: Is Python dynamically typed or statically typed? What does that mean?**
> Dynamically typed. It means you don't declare variable types — Python figures it out at runtime. So `x = 5` makes `x` an int, and then `x = "hello"` makes it a string. In Java, that would throw an error because you'd have to declare the type upfront.

**Q5: What is the difference between `=`, `==`, and `is`?**
> `=` is assignment — it stores a value. `==` checks if two values are equal. `is` checks if two variables point to the exact same object in memory. So `[1,2] == [1,2]` is `True` because values match, but `[1,2] is [1,2]` is `False` because they're two different list objects.

**Q6: What happens if you do `x = 10` then `y = x` then `x = 20`? What is `y`?**
> `y` is still `10`. When you did `y = x`, Python copied the reference to the integer `10`. When you reassigned `x = 20`, you pointed `x` to a new object. `y` still points to the old `10`. Integers are immutable, so this is straightforward.

**Q7: What are Python's naming conventions?**
> Variables and functions use `snake_case`. Classes use `PascalCase`. Constants use `ALL_CAPS`. A single underscore prefix `_var` means "private by convention". Double underscore `__var` triggers name mangling in classes.

**Q8: Can a variable name start with a number?**
> No. Variable names must start with a letter or underscore. `2name` is invalid but `name2` or `_name` are fine.

**Q9: What's the difference between `None`, `0`, `False`, and `""`?**
> They're all "falsy" — they evaluate to `False` in a boolean context. But they're different types. `None` is NoneType (means "nothing here"), `0` is an integer, `False` is a boolean, and `""` is an empty string. `None is not 0` and `None is not False`.

**Q10: What is multiple assignment in Python?**
> You can assign multiple variables in one line: `a, b, c = 1, 2, 3`. It's actually tuple unpacking under the hood. You can even swap: `a, b = b, a` — no temp variable needed. Python evaluates the right side first, then assigns.

**Q11: 🗣️ VERBAL — "What does this print?"**
```python
a = [1, 2, 3]
b = a
b.append(4)
print(a)
```
> `[1, 2, 3, 4]`. Because `b = a` doesn't copy the list — both `a` and `b` point to the same list object. So when you append through `b`, you're modifying the same list that `a` sees.

**Q12: 🗣️ VERBAL — "What does this print?"**
```python
x = 5
y = 5
print(x is y)
```
> `True`. Python caches small integers from -5 to 256. So `x` and `y` are literally the same object in memory. But if you try `x = 500; y = 500`, then `x is y` might be `False` because they could be different objects.

**Q13: What is the `type()` function?**
> It returns the type of an object. `type(42)` gives `<class 'int'>`. You can use it for debugging or type checking, but in interviews, the preferred way to check types is `isinstance(x, int)` because it also handles inheritance.

**Q14: What is type casting?**
> Converting one type to another — `int("42")` converts string to int, `str(42)` converts int to string, `list("abc")` gives `['a', 'b', 'c']`. If the conversion doesn't make sense, like `int("hello")`, you get a `ValueError`.

**Q15: 🐛 DEBUG — "Why does this crash?"**
```python
age = input("Enter age: ")
if age > 18:
    print("Adult")
```
> `input()` always returns a string. You're comparing a string with an integer using `>`, which throws a `TypeError` in Python 3. Fix: `if int(age) > 18:`.

---

## SECTION 2: Data Types & Operators (Q16–Q30)

**Q16: What are Python's main data types?**
> Numeric: `int`, `float`, `complex`. Sequence: `str`, `list`, `tuple`. Set: `set`, `frozenset`. Mapping: `dict`. Boolean: `bool`. And `None` (NoneType). That covers the built-in ones.

**Q17: What is the difference between mutable and immutable types?**
> Mutable means you can change the object after creating it — lists, dicts, sets. Immutable means once created, it can't be changed — int, float, str, tuple, frozenset. When you do `x = "hello"; x = "world"`, you didn't change the string — you created a new one and pointed `x` to it.

**Q18: Why does immutability matter?**
> Three reasons: immutable objects can be dictionary keys (lists can't), they're thread-safe, and Python can optimize them (like caching small integers and interning strings). In interviews, the most common follow-up is "can a tuple be a dict key?" — yes, because it's immutable.

**Q19: What is `//` in Python?**
> Floor division — it divides and rounds DOWN to the nearest integer. `7 // 2` gives `3`. The trap is with negatives: `-7 // 2` gives `-4`, not `-3`, because it rounds toward negative infinity, not toward zero.

**Q20: What is `%` (modulo) with negative numbers?**
> `-7 % 2` gives `1`, not `-1`. Python's modulo always returns a result with the same sign as the divisor. This is different from C/Java where it keeps the sign of the dividend. It's a common trap.

**Q21: What does `**` do?**
> Exponentiation. `2 ** 3` is 8. `2 ** 0.5` is the square root of 2 (~1.414). It also works in assignments: `x **= 2` squares x.

**Q22: What's the difference between `and`/`or` and `&`/`|`?**
> `and`/`or` are logical operators for boolean logic with short-circuiting. `&`/`|` are bitwise operators that work on individual bits of integers. For booleans they give the same result, but they behave differently — `and`/`or` short-circuit (stop early), `&`/`|` always evaluate both sides.

**Q23: What is short-circuit evaluation?**
> With `and`, if the first value is falsy, Python doesn't even check the second — it already knows the result is `False`. With `or`, if the first is truthy, Python skips the second. So `True or expensive_function()` never calls the function.

**Q24: 🗣️ VERBAL — "What does `bool('False')` return?"**
> `True`. The string `"False"` is a non-empty string, and any non-empty string is truthy. Only `bool("")` returns `False`. This is one of the most common interview traps.

**Q25: 🗣️ VERBAL — "What does `0.1 + 0.2 == 0.3` return?"**
> `False`. This is a floating-point precision issue. `0.1 + 0.2` is actually `0.30000000000000004` because floats can't represent these decimals exactly in binary. Use `math.isclose(0.1 + 0.2, 0.3)` or the `decimal` module for precise comparison.

**Q26: What is the walrus operator `:=`?**
> It assigns a value AND returns it in the same expression. Introduced in Python 3.8. Example: `if (n := len(data)) > 10: print(n)`. Without it, you'd need two lines — one to assign, one to check. Useful in while loops and list comprehensions.

**Q27: What is the ternary operator in Python?**
> `value = x if condition else y`. It's a one-line if-else. Like: `status = "adult" if age >= 18 else "minor"`. It's not `condition ? x : y` like in C/Java.

**Q28: 🗣️ VERBAL — "What does `3 * 'abc'` give you?"**
> `'abcabcabc'`. The `*` operator repeats strings (and lists). `[1, 2] * 3` gives `[1, 2, 1, 2, 1, 2]`.

**Q29: What are truthy and falsy values?**
> Falsy: `False`, `0`, `0.0`, `""`, `[]`, `()`, `{}`, `set()`, `None`. Everything else is truthy. That means `if my_list:` is the Pythonic way to check if a list is non-empty — you don't need `if len(my_list) > 0:`.

**Q30: 🐛 DEBUG — "Why doesn't this work?"**
```python
x = 10
y = "The value is: " + x
```
> You can't concatenate a string with an integer using `+`. Fix: `"The value is: " + str(x)` or better, use f-strings: `f"The value is: {x}"`.

---

## SECTION 3: Strings (Q31–Q45)

**Q31: Are strings mutable or immutable?**
> Immutable. You cannot change a string in place. `s = "hello"; s[0] = "H"` throws a `TypeError`. You have to create a new string: `s = "H" + s[1:]`.

**Q32: What is the difference between single, double, and triple quotes?**
> Single and double quotes are the same — `'hello'` equals `"hello"`. Triple quotes `'''` or `"""` let you write multi-line strings and are also used for docstrings.

**Q33: What are f-strings?**
> Formatted string literals, added in Python 3.6. You put `f` before the string and use `{}` to embed expressions: `f"My name is {name} and I'm {age} years old"`. They're the cleanest way to format strings and they're faster than `.format()` or `%` formatting.

**Q34: What does `strip()` do?**
> Removes whitespace from both ends of a string. `"  hello  ".strip()` gives `"hello"`. `lstrip()` removes from left only, `rstrip()` from right only. You can also pass characters to strip: `"##hello##".strip("#")` gives `"hello"`.

**Q35: Difference between `find()` and `index()`?**
> Both find a substring's position. The difference: `find()` returns `-1` if not found, `index()` raises a `ValueError`. Use `find()` when you're not sure if the substring exists and don't want to deal with exceptions.

**Q36: What does `split()` and `join()` do?**
> `split()` breaks a string into a list: `"a,b,c".split(",")` gives `["a", "b", "c"]`. `join()` does the opposite: `",".join(["a", "b", "c"])` gives `"a,b,c"`. Notice `join()` is called on the separator, not the list.

**Q37: How do you reverse a string?**
> `s[::-1]`. This uses slicing with a step of -1. There's no `.reverse()` method on strings because strings are immutable.

**Q38: How do you check if a string contains a substring?**
> Use `in`: `"py" in "python"` returns `True`. You can also use `find()` or `index()`, but `in` is the most Pythonic and readable way.

**Q39: 🗣️ VERBAL — "What does `'hello'[1:4]` return?"**
> `'ell'`. Slicing is `[start:stop]` where start is inclusive and stop is exclusive. So index 1, 2, 3 → `'e'`, `'l'`, `'l'`.

**Q40: 🗣️ VERBAL — "What does `'hello'[-1]` return?"**
> `'o'`. Negative indexing counts from the end. `-1` is the last character, `-2` is second-to-last, etc.

**Q41: What is string interning?**
> Python automatically caches certain strings (like short strings and identifiers) so identical strings share the same memory. That's why `"hello" is "hello"` is usually `True`. But don't rely on this — always use `==` for string comparison, not `is`.

**Q42: 🗣️ VERBAL — "What does `'python'.replace('p', 'P')` return? Does it change the original?"**
> Returns `'Python'`. Does NOT change the original because strings are immutable. Every string method returns a NEW string.

**Q43: How do you count occurrences of a character in a string?**
> `"banana".count("a")` returns `3`. It counts non-overlapping occurrences of the substring.

**Q44: 🐛 DEBUG — "This code doesn't capitalize properly. Why?"**
```python
name = "john"
name.upper()
print(name)
```
> It prints `"john"` because `upper()` returns a NEW string — it doesn't modify the original. Strings are immutable. Fix: `name = name.upper()`.

**Q45: What is the difference between `encode()` and `decode()`?**
> `encode()` converts a string to bytes (str → bytes): `"hello".encode("utf-8")` gives `b'hello'`. `decode()` converts bytes back to string: `b'hello'.decode("utf-8")` gives `"hello"`. You need this when dealing with files, network data, or APIs.

---

## SECTION 4: Lists & Tuples (Q46–Q65)

**Q46: What is a list in Python?**
> An ordered, mutable collection that can hold any type of element. You create it with square brackets: `[1, "hello", 3.14, True]`. Lists are the workhorse of Python — you'll use them everywhere.

**Q47: What is the difference between `append()` and `extend()`?**
> `append()` adds ONE item to the end: `[1,2].append([3,4])` gives `[1, 2, [3, 4]]` — the list is added as a single element. `extend()` adds each item individually: `[1,2].extend([3,4])` gives `[1, 2, 3, 4]`. People mix these up all the time.

**Q48: Difference between `remove()`, `pop()`, and `del`?**
> `remove(value)` removes the first occurrence of that value. `pop(index)` removes by index and returns the removed item. `del list[index]` removes by index but doesn't return anything. `pop()` with no argument removes the last item.

**Q49: How do you copy a list?**
> `new = old.copy()`, or `new = old[:]`, or `new = list(old)`. These create a SHALLOW copy. If your list has nested lists, the inner lists are still shared. For a full independent copy, use `import copy; new = copy.deepcopy(old)`.

**Q50: 🗣️ VERBAL — "What's the output?"**
```python
a = [[]] * 3
a[0].append(1)
print(a)
```
> `[[1], [1], [1]]`. This is a classic trap. `[[]] * 3` creates three references to the SAME inner list — not three separate lists. When you append to one, all three show the change. Fix: `a = [[] for _ in range(3)]`.

**Q51: What does `sorted()` vs `.sort()` do?**
> `sorted()` returns a NEW sorted list and works on any iterable. `.sort()` sorts the list IN PLACE and returns `None`. So `x = [3,1,2].sort()` makes `x` equal to `None` — that's a common bug.

**Q52: 🗣️ VERBAL — "What does this print?"**
```python
lst = [1, 2, 3, 4, 5]
print(lst[1:4])
print(lst[::2])
print(lst[::-1])
```
> `[2, 3, 4]` — indices 1,2,3 (stop is exclusive). `[1, 3, 5]` — every 2nd element. `[5, 4, 3, 2, 1]` — reversed list.

**Q53: What is a tuple? Why use it instead of a list?**
> A tuple is an ordered, IMMUTABLE collection. Use `()` or just commas: `t = (1, 2, 3)` or `t = 1, 2, 3`. Use tuples when you want data that shouldn't change — like coordinates `(x, y)`, database records, or dictionary keys. Tuples are also slightly faster and use less memory than lists.

**Q54: How do you create a single-element tuple?**
> `t = (1,)` — the comma is mandatory. Without the comma, `t = (1)` is just the integer `1` in parentheses. This is a very common interview trap.

**Q55: Can a tuple contain mutable objects?**
> Yes. `t = ([1, 2], [3, 4])` is valid. You can even modify the list inside: `t[0].append(5)` works. But you can't reassign `t[0] = [9, 9]` because that changes the tuple itself. The tuple's immutability means you can't change which objects it points to, but the objects themselves can be mutable.

**Q56: What is tuple unpacking?**
> Assigning tuple elements to variables in one line: `a, b, c = (1, 2, 3)`. You can also use `*` for the rest: `first, *rest = [1, 2, 3, 4, 5]` gives `first = 1`, `rest = [2, 3, 4, 5]`. Works with any iterable, not just tuples.

**Q57: What is `namedtuple`?**
> A tuple subclass with named fields. `from collections import namedtuple; Point = namedtuple('Point', ['x', 'y']); p = Point(1, 2)`. Now you can do `p.x` and `p.y` instead of `p[0]` and `p[1]`. It's like a lightweight class without methods.

**Q58: 🗣️ VERBAL — "Can you use a list as a dictionary key?"**
> No, because lists are mutable and dict keys must be hashable (immutable). Use a tuple instead: `{(1, 2): "value"}` works fine. The rule is: if it can change, it can't be a key.

**Q59: 🗣️ VERBAL — "What does this print?"**
```python
a = [1, 2, 3]
b = a
a = a + [4, 5]
print(b)
```
> `[1, 2, 3]`. The trap: `a + [4, 5]` creates a NEW list and assigns it to `a`. It doesn't modify the original list that `b` points to. Compare this with `a += [4, 5]` which DOES modify in place — that would make `b` become `[1, 2, 3, 4, 5]`.

**Q60: What is `list()` vs `[]`?**
> Both create lists. `[]` is a literal and slightly faster. `list()` is a constructor that can convert other iterables: `list("abc")` gives `['a', 'b', 'c']`, `list(range(5))` gives `[0, 1, 2, 3, 4]`.

**Q61: 🗣️ VERBAL — "What does `[1, 2, 3] + [4, 5]` give?"**
> `[1, 2, 3, 4, 5]`. The `+` operator concatenates lists and returns a new list. Neither original list is modified.

**Q62: What is the `in` keyword for lists?**
> Membership testing. `3 in [1, 2, 3]` returns `True`. For lists, it does a linear search (O(n)). For sets and dicts, it's O(1). So if you're checking membership a lot, use a set.

**Q63: 🐛 DEBUG — "Why does this give unexpected results?"**
```python
lst = [1, 2, 3, 4, 5]
for i in range(len(lst)):
    if lst[i] % 2 == 0:
        lst.remove(lst[i])
print(lst)
```
> You're modifying a list while iterating over it. When you remove element `2` at index 1, everything shifts left. Now index 2 points to `4` instead of `3`, so `3` is skipped. Fix: iterate over a copy `lst[:]`, or use a list comprehension: `lst = [x for x in lst if x % 2 != 0]`.

**Q64: What is the difference between `+` and `+=` for lists?**
> `+` creates a NEW list. `+=` modifies the list IN PLACE (it calls `extend()` internally). This matters when other variables reference the same list. `a = [1]; b = a; a += [2]` — now `b` is `[1, 2]` because `+=` modified the same object. But `a = [1]; b = a; a = a + [2]` — now `b` is still `[1]`.

**Q65: How do you flatten a nested list `[[1,2],[3,4],[5]]`?**
> List comprehension: `[item for sublist in nested for item in sublist]` gives `[1, 2, 3, 4, 5]`. Or `sum(nested, [])` but that's slow for large lists. For deeply nested lists, use recursion or `itertools.chain.from_iterable()`.

---

## SECTION 5: Sets & Dictionaries (Q66–Q80)

**Q66: What is a set? When would you use it?**
> An unordered collection of unique elements. Use it when you need to remove duplicates (`set([1,1,2,3])` → `{1,2,3}`), or when you need fast membership testing (`x in my_set` is O(1) vs O(n) for lists), or when you need set operations like union, intersection, difference.

**Q67: How do you create an empty set?**
> `s = set()`, NOT `s = {}`. The `{}` creates an empty dictionary. This is a common trap.

**Q68: What is a frozenset?**
> An immutable version of a set. You can't add or remove elements after creation. Because it's immutable, it can be used as a dictionary key or placed inside another set. `fs = frozenset([1, 2, 3])`.

**Q69: What are the main dict methods you should know?**
> `get(key, default)` — returns default if key missing instead of crashing. `keys()`, `values()`, `items()` — views of the dict. `update()` — merge another dict. `setdefault(key, default)` — get value, but set it first if key missing. `pop(key)` — remove and return value.

**Q70: 🗣️ VERBAL — "What's the difference between `dict[key]` and `dict.get(key)`?"**
> `dict[key]` raises `KeyError` if the key doesn't exist. `dict.get(key)` returns `None` (or a default you specify). Use `get()` when the key might not exist and you don't want your program to crash.

**Q71: What is a `defaultdict`?**
> A dict subclass from `collections` that provides a default value for missing keys. `from collections import defaultdict; d = defaultdict(list); d["key"].append(1)`. Without `defaultdict`, you'd first need to check if the key exists. Very useful for grouping data.

**Q72: What are dictionary comprehensions?**
> Creating dicts in one line: `{x: x**2 for x in range(5)}` gives `{0: 0, 1: 1, 2: 4, 3: 9, 4: 16}`. Just like list comprehensions but with `key: value` syntax.

**Q73: Are dicts ordered in Python?**
> Yes, since Python 3.7 dicts maintain insertion order — it's guaranteed by the language spec. Before that (3.6), it was an implementation detail in CPython but not officially guaranteed.

**Q74: 🗣️ VERBAL — "What does this print?"**
```python
d = {"a": 1, "b": 2}
d["c"] = 3
d["a"] = 99
print(d)
```
> `{'a': 99, 'b': 2, 'c': 3}`. Assigning to an existing key updates it. Assigning to a new key adds it. Order is maintained from first insertion.

**Q75: How do you merge two dictionaries?**
> Python 3.9+: `merged = d1 | d2`. Python 3.5+: `merged = {**d1, **d2}`. Or `d1.update(d2)` but that modifies `d1` in place. In all cases, if keys overlap, the second dict's values win.

**Q76: 🐛 DEBUG — "Why does this crash?"**
```python
d = {"a": 1, "b": 2, "c": 3}
for key in d:
    if d[key] % 2 == 0:
        del d[key]
```
> `RuntimeError: dictionary changed size during iteration`. You can't modify a dict while iterating over it. Fix: iterate over a copy of the keys: `for key in list(d):` or build a new dict with a comprehension: `d = {k: v for k, v in d.items() if v % 2 != 0}`.

**Q77: What is `Counter` from collections?**
> It counts occurrences of elements. `from collections import Counter; Counter("banana")` gives `Counter({'a': 3, 'n': 2, 'b': 1})`. Super useful for frequency counting. `most_common(n)` returns the top n elements.

**Q78: 🗣️ VERBAL — "Can you put a list as a value in a dict? As a key?"**
> As a value — yes, absolutely: `{"nums": [1, 2, 3]}` is fine. As a key — no, because lists are mutable and dict keys must be hashable. Use a tuple instead.

**Q79: What is `items()` and how do you use it?**
> `d.items()` returns key-value pairs as tuples. Use it to loop through both: `for key, value in d.items(): print(key, value)`. It returns a view object, not a list, so it's memory efficient.

**Q80: How do you sort a dictionary by values?**
> `sorted(d.items(), key=lambda x: x[1])` returns a sorted list of tuples. To get a dict: `dict(sorted(d.items(), key=lambda x: x[1]))`. The `key` parameter tells `sorted()` what to sort by.

---

## SECTION 6: Control Flow & Loops (Q81–Q95)

**Q81: What is the difference between `if-elif-else` and multiple `if` statements?**
> `if-elif-else` stops at the first True condition. Multiple `if`s check ALL conditions independently. So if both conditions could be true, `elif` runs only the first match, while separate `if`s run both.

**Q82: Can `if` work without `else`?**
> Yes, absolutely. `else` is optional. If the condition is False and there's no else, Python just skips ahead.

**Q83: What are `for` loops used for vs `while` loops?**
> `for` loops are for iterating over a known sequence — a list, string, range, etc. `while` loops are for repeating until a condition changes — useful when you don't know how many iterations you'll need, like reading user input until it's valid.

**Q84: What does `break`, `continue`, and `pass` do?**
> `break` exits the loop entirely. `continue` skips to the next iteration. `pass` does literally nothing — it's a placeholder when you need a statement syntactically but don't want to do anything yet.

**Q85: What is `else` on a loop?**
> `for...else` — the `else` block runs only if the loop completed WITHOUT hitting a `break`. If `break` was triggered, `else` is skipped. It's rare but useful for search loops: "if we found it, break; else, it wasn't there."

**Q86: 🗣️ VERBAL — "What does this print?"**
```python
for i in range(5):
    if i == 3:
        break
else:
    print("Loop completed")
print("Done")
```
> Just `"Done"`. The `else` doesn't print because the loop hit `break` at `i == 3`. If we removed the `break`, it would print both `"Loop completed"` and `"Done"`.

**Q87: What is `range()`? Is it a list?**
> No, `range()` is a lazy sequence. `range(1000000)` doesn't create a million numbers in memory — it generates them one at a time as you iterate. That's why it's memory efficient. To get an actual list, use `list(range(5))`.

**Q88: 🗣️ VERBAL — "What does `range(2, 10, 3)` give?"**
> `2, 5, 8`. Start at 2, step by 3, stop before 10. The stop value is always exclusive.

**Q89: What is `enumerate()`?**
> Gives you both the index and value while looping: `for i, val in enumerate(["a", "b", "c"]):`. Without it, people use `range(len(list))` which is ugly and un-Pythonic. You can set a starting index: `enumerate(lst, start=1)`.

**Q90: What is `zip()`?**
> Combines multiple iterables element by element: `zip([1,2,3], ["a","b","c"])` gives `[(1,"a"), (2,"b"), (3,"c")]`. Stops at the shortest iterable. Use `zip_longest` from `itertools` if you want to handle unequal lengths.

**Q91: What is a list comprehension?**
> A one-line way to create lists: `[x**2 for x in range(5)]` gives `[0, 1, 4, 9, 16]`. Can include conditions: `[x for x in range(10) if x % 2 == 0]` gives even numbers. They're faster than for-loops for creating lists and more Pythonic.

**Q92: 🗣️ VERBAL — "What does this list comprehension produce?"**
```python
result = [i*j for i in range(1,4) for j in range(1,4)]
```
> `[1, 2, 3, 2, 4, 6, 3, 6, 9]`. It's like a nested loop: outer loop is `i`, inner loop is `j`. For each `i`, it goes through all `j` values.

**Q93: What is a generator expression?**
> Same as list comprehension but with `()` instead of `[]`: `(x**2 for x in range(1000000))`. It doesn't create the whole list in memory — generates values one at a time. Use when you just need to iterate, not store everything.

**Q94: 🐛 DEBUG — "Why is this an infinite loop?"**
```python
x = 10
while x > 0:
    print(x)
    x = x + 1
```
> Because `x` starts at 10 and keeps increasing. The condition `x > 0` will never be False. Fix: should be `x = x - 1` (or `x -= 1`).

**Q95: What's the difference between `while True` with `break` vs a regular while condition?**
> Both work. `while True` with `break` is useful when the exit condition is in the middle of the loop body, not at the top. Like: get input, validate it, break if valid. Sometimes it reads cleaner than putting a complex condition in the `while` header.

---

## SECTION 7: Functions & Scope (Q96–Q115)

**Q96: What is the difference between a parameter and an argument?**
> Parameters are the names in the function definition: `def greet(name):` — `name` is a parameter. Arguments are the values you pass when calling: `greet("Aman")` — `"Aman"` is an argument. People use these interchangeably but interviewers love asking the difference.

**Q97: What are `*args` and `**kwargs`?**
> `*args` collects extra positional arguments into a tuple. `**kwargs` collects extra keyword arguments into a dictionary. Together, they let a function accept any number of arguments. `def f(*args, **kwargs): print(args, kwargs)`. Call with `f(1, 2, name="Aman")` → `args = (1, 2)`, `kwargs = {"name": "Aman"}`.

**Q98: 🗣️ VERBAL — "What does this print?"**
```python
def greet(name, greeting="Hello"):
    print(f"{greeting}, {name}!")

greet("Aman")
greet("Aman", "Hey")
greet(greeting="Yo", name="Aman")
```
> `Hello, Aman!` then `Hey, Aman!` then `Yo, Aman!`. Shows default arguments and keyword arguments. The third call shows you can pass arguments in any order using keyword syntax.

**Q99: ⚠️ THE MUTABLE DEFAULT ARGUMENT TRAP — What does this print?**
```python
def add_item(item, lst=[]):
    lst.append(item)
    return lst

print(add_item(1))
print(add_item(2))
print(add_item(3))
```
> `[1]`, `[1, 2]`, `[1, 2, 3]`. The default list `[]` is created ONCE when the function is defined, not each time it's called. Every call shares the same list. Fix: use `lst=None`, then `if lst is None: lst = []` inside the function. This is probably the #1 most asked Python trap.

**Q100: What is a lambda function?**
> An anonymous, one-line function: `square = lambda x: x ** 2`. Same as `def square(x): return x ** 2`. Used mainly as arguments to `sorted()`, `map()`, `filter()`: `sorted(names, key=lambda x: len(x))`.

**Q101: What is the difference between `return` and `print`?**
> `return` sends a value back to the caller — you can store it, use it, pass it. `print` just displays text on the screen and returns `None`. A function without `return` (or with bare `return`) returns `None`.

**Q102: What is scope in Python?**
> Scope is where a variable lives and who can see it. Python follows the LEGB rule — it searches Local → Enclosing → Global → Built-in. A variable created inside a function is local and dies when the function ends.

**Q103: What is the LEGB rule?**
> The order Python searches for variable names: **L**ocal (current function) → **E**nclosing (parent function if nested) → **G**lobal (file level) → **B**uilt-in (like `print`, `len`). It stops at the first match.

**Q104: 🗣️ VERBAL — "What does this print?"**
```python
x = 10
def foo():
    print(x)
    x = 20
foo()
```
> `UnboundLocalError`. Python sees `x = 20` and marks `x` as local for the ENTIRE function at compile time. Then `print(x)` tries to read local `x` before it's been assigned. Fix: add `global x` at the top of the function, or rearrange the code.

**Q105: What is `global` vs `nonlocal`?**
> `global` lets you modify a file-level variable from inside a function. `nonlocal` lets you modify a variable from the nearest enclosing (parent) function. Without these keywords, assigning inside a function creates a new local variable.

**Q106: 🗣️ VERBAL — "What does this print?"**
```python
def outer():
    x = "outer"
    def inner():
        x = "inner"
    inner()
    print(x)
outer()
```
> `"outer"`. Inside `inner()`, `x = "inner"` creates a LOCAL variable that dies when `inner()` ends. It doesn't touch `outer()`'s `x`. To change it, you'd need `nonlocal x` inside `inner()`.

**Q107: 🗣️ VERBAL — "What does this print?"**
```python
x = 100
def f(x):
    x = x + 1
    return x
result = f(x)
print(x, result)
```
> `100 101`. Function parameters are local variables. The `x` inside `f` is a separate local copy. The global `x` is never modified.

**Q108: What is a closure?**
> A closure is a function that remembers variables from its enclosing scope even after the enclosing function has finished. Example: a function that generates counters or multipliers. The inner function "closes over" the outer variable.

```python
def make_multiplier(n):
    def multiply(x):
        return x * n     # 'n' is remembered from outer scope
    return multiply

double = make_multiplier(2)
print(double(5))   # 10
```

**Q109: What is a decorator?**
> A function that wraps another function to add behavior — like logging, timing, or authentication. It uses the `@` syntax. Under the hood, `@decorator` above a function is the same as `func = decorator(func)`.

```python
def timer(func):
    def wrapper(*args, **kwargs):
        import time
        start = time.time()
        result = func(*args, **kwargs)
        print(f"{func.__name__} took {time.time() - start:.2f}s")
        return result
    return wrapper

@timer
def slow_function():
    import time
    time.sleep(1)
```

**Q110: What is the difference between `map()`, `filter()`, and `reduce()`?**
> `map(func, iterable)` applies the function to every item. `filter(func, iterable)` keeps only items where the function returns True. `reduce(func, iterable)` from `functools` combines all items into one value (like summing). In modern Python, list comprehensions often replace `map` and `filter`.

**Q111: 🗣️ VERBAL — "What does this print?"**
```python
nums = [1, 2, 3, 4, 5]
result = list(filter(lambda x: x % 2 == 0, nums))
print(result)
```
> `[2, 4]`. `filter` keeps elements where the lambda returns `True` — the even numbers.

**Q112: What is recursion? Give a simple example.**
> A function calling itself. Every recursive function needs a base case to stop. Example: factorial.
```python
def factorial(n):
    if n <= 1:
        return 1          # base case
    return n * factorial(n - 1)   # recursive case
```
> Python has a default recursion limit of 1000. You can change it with `sys.setrecursionlimit()` but that's usually a sign you should use a loop instead.

**Q113: What is the difference between `is` and `==` for function returns?**
> Always use `==` to check values. The only time you use `is` is to check for `None`: `if result is None:`. That's a PEP 8 recommendation. Don't use `is` with strings or numbers because caching behavior is an implementation detail.

**Q114: Can a function return multiple values?**
> Yes, technically it returns a tuple: `def f(): return 1, 2, 3`. The caller can unpack: `a, b, c = f()`. It's just syntactic sugar for returning a tuple.

**Q115: What is a docstring?**
> A string at the start of a function/class/module that describes what it does. Triple-quoted. Accessible via `func.__doc__` or `help(func)`. Good practice: describe what the function does, its parameters, and what it returns.

---

## SECTION 8: Modules, Packages & Imports (Q116–Q122)

**Q116: What is a module? What is a package?**
> A module is just a `.py` file. A package is a folder containing modules and an `__init__.py` file (though in Python 3.3+, the init file is optional for "namespace packages"). You use `import` to bring in modules.

**Q117: What is `if __name__ == "__main__":`?**
> When you run a file directly, `__name__` is set to `"__main__"`. When you import it, `__name__` is set to the module's name. So this guard lets you write code that only runs when the file is executed directly — not when imported. Use it for test code, demo code, or a script's entry point.

**Q118: What is the difference between `import module` and `from module import func`?**
> `import module` brings in the whole module — you access things with `module.func()`. `from module import func` brings in just that function — you use `func()` directly. The second is convenient but can cause name collisions. Never use `from module import *` in production code.

**Q119: What is `sys.path`?**
> A list of directories where Python looks for modules when you `import`. It includes the current directory, installed packages, and standard library. If your import fails, it's usually because the module isn't in any of these paths.

**Q120: What is circular import?**
> When module A imports module B and module B imports module A. Python can handle it sometimes, but it often leads to `ImportError` or `AttributeError`. Fix: restructure your code, move the import inside a function, or create a third module for shared code.

**Q121: What are `__init__.py` files for?**
> They mark a directory as a Python package and run when the package is imported. You can use them to set up package-level variables, import submodules for convenience, or leave them empty. In Python 3.3+, they're optional for namespace packages.

**Q122: What is the `importlib` module?**
> It lets you import modules dynamically at runtime using strings: `importlib.import_module("module_name")`. Useful for plugin systems or when the module name is determined at runtime.

---

## SECTION 9: Exception Handling (Q123–Q132)

**Q123: What is exception handling?**
> It's a way to deal with runtime errors without your program crashing. Wrap risky code in `try`, catch errors with `except`, and your program keeps running. It's mandatory for anything that can fail — file operations, network calls, user input, etc.

**Q124: What is the flow of `try/except/else/finally`?**
> `try` runs the risky code. If it crashes, `except` handles the specific error. `else` runs ONLY if `try` succeeded (no error). `finally` ALWAYS runs — error or not — for cleanup. Memory trick: T-E-E-F.

**Q125: What is the difference between `except Exception` and bare `except:`?**
> Bare `except:` catches EVERYTHING, including `SystemExit` and `KeyboardInterrupt`, which means you can't even Ctrl+C to stop your program. `except Exception:` catches normal errors but lets system-level exceptions pass through. Always use `except Exception:` at minimum.

**Q126: 🗣️ VERBAL — "What does this print?"**
```python
try:
    print("A")
    x = 1 / 0
    print("B")
except ZeroDivisionError:
    print("C")
else:
    print("D")
finally:
    print("E")
```
> `A C E`. Line `1/0` crashes, so `B` is skipped. `except` catches it → `C`. `else` is skipped because an error occurred. `finally` always runs → `E`.

**Q127: 🗣️ VERBAL — "What does this return?"**
```python
def f():
    try:
        return 1
    except:
        return 2
    finally:
        return 3
print(f())
```
> `3`. `finally` overrides the return from `try`. This is a trap — never put `return` inside `finally`.

**Q128: What is `raise`?**
> It deliberately throws an exception. Use it for input validation or enforcing business rules: `if age < 0: raise ValueError("Age can't be negative")`. You can also `raise` inside an `except` block to re-throw the error after logging it.

**Q129: How do you create custom exceptions?**
> Inherit from `Exception`: `class MyError(Exception): pass`. You can add custom attributes like error codes or field names. Create a hierarchy for related errors: `class DatabaseError(AppError): pass`.

**Q130: What does `raise ... from e` mean?**
> Exception chaining. It links a new exception to the original cause. `raise RuntimeError("Failed") from original_error` shows both errors in the traceback with "The above exception was the direct cause..." message. Use it to wrap low-level errors with higher-level context.

**Q131: What is EAFP vs LBYL?**
> EAFP = "Easier to Ask Forgiveness than Permission" — just try it and catch the error. LBYL = "Look Before You Leap" — check conditions first. Python strongly prefers EAFP. Instead of `if key in dict:` then `dict[key]`, just do `try: dict[key] except KeyError:`. It's faster when errors are rare.

**Q132: 🐛 DEBUG — "Why does this silently fail?"**
```python
try:
    data = process(input_data)
except:
    pass
```
> Bare `except: pass` catches ALL errors and does nothing. If `process()` has a bug, you'll never know. Fix: catch specific exceptions and at least log them: `except ValueError as e: print(f"Error: {e}")`.

---

## SECTION 10: File Handling & Built-in Functions (Q133–Q150)

**Q133: What is the best way to open files in Python?**
> Use the `with` statement: `with open("file.txt") as f: data = f.read()`. It automatically closes the file when the block ends, even if an error occurs. Without `with`, you'd need a `try/finally` with `f.close()`.

**Q134: What are the file modes?**
> `"r"` read (default, file must exist), `"w"` write (creates or overwrites), `"a"` append (creates or adds to end), `"x"` exclusive create (fails if exists). Add `"b"` for binary: `"rb"`, `"wb"`. Add `"+"` for both read and write: `"r+"`.

**Q135: Difference between `read()`, `readline()`, and `readlines()`?**
> `read()` returns the entire file as one string. `readline()` returns one line at a time. `readlines()` returns a list of all lines. For large files, iterate directly: `for line in f:` — it reads one line at a time without loading everything into memory.

**Q136: How do you read a CSV file without pandas?**
> Use the `csv` module: `import csv; with open("data.csv") as f: reader = csv.reader(f); for row in reader: print(row)`. Each `row` is a list of strings. Use `csv.DictReader` for rows as dictionaries with column headers as keys.

**Q137: What does `json.dump()` vs `json.dumps()` do?**
> `dumps()` converts a Python object to a JSON string (s = string). `dump()` writes it directly to a file. Similarly, `loads()` reads from a string, `load()` reads from a file. The 's' versions work with strings, the others with file objects.

**Q138: What is `map()`?**
> Applies a function to every item in an iterable: `list(map(str, [1, 2, 3]))` gives `["1", "2", "3"]`. Returns a lazy iterator, so wrap in `list()` to see results. Often replaced by list comprehensions: `[str(x) for x in [1, 2, 3]]`.

**Q139: What is `zip()`? What if iterables are different lengths?**
> Pairs up elements from multiple iterables: `list(zip([1,2], ["a","b"]))` gives `[(1,"a"), (2,"b")]`. Stops at the SHORTEST iterable. Use `itertools.zip_longest` to pad shorter ones with a fill value.

**Q140: What is `enumerate()` and why is it better than `range(len())`?**
> `enumerate(iterable)` gives you `(index, value)` pairs: `for i, val in enumerate(["a","b","c"])`. It's cleaner and more Pythonic than `for i in range(len(lst)): val = lst[i]`. You can set start index: `enumerate(lst, start=1)`.

**Q141: What is `any()` and `all()`?**
> `any()` returns `True` if ANY element is truthy. `all()` returns `True` if ALL elements are truthy. `any([False, False, True])` → `True`. `all([True, True, False])` → `False`. Both short-circuit — `any()` stops at the first True, `all()` stops at the first False.

**Q142: What is `sorted()` vs `.sort()`?**
> `sorted()` returns a NEW sorted list and works on any iterable. `.sort()` sorts a list IN PLACE and returns `None`. Common trap: `x = [3,1,2].sort()` — `x` is `None`, not `[1,2,3]`. Use `sorted()` when you need the original unchanged.

**Q143: 🗣️ VERBAL — "What does `list(zip(*matrix))` do?"**
```python
matrix = [[1, 2, 3],
          [4, 5, 6],
          [7, 8, 9]]
print(list(zip(*matrix)))
```
> `[(1, 4, 7), (2, 5, 8), (3, 6, 9)]`. It transposes the matrix. `*matrix` unpacks the rows as separate arguments to `zip`, which then pairs up corresponding elements.

**Q144: What is `isinstance()` and why prefer it over `type()`?**
> `isinstance(x, int)` checks if x is an int or a subclass of int. `type(x) == int` only matches the exact type, not subclasses. `isinstance` also takes a tuple of types: `isinstance(x, (int, float))`.

**Q145: 🗣️ VERBAL — "What does `sorted("hello")` return?"**
> `['e', 'h', 'l', 'l', 'o']`. `sorted()` works on any iterable including strings, and returns a list of characters in alphabetical order.

**Q146: What does `abs()`, `round()`, `min()`, `max()`, `sum()` do?**
> `abs(-5)` → `5` (absolute value). `round(3.14159, 2)` → `3.14` (round to n decimals). `min/max` return smallest/largest. `sum([1,2,3])` → `6`. All basic but you should know `round(2.5)` gives `2` not `3` — Python uses banker's rounding (round half to even).

**Q147: 🗣️ VERBAL — "What does `bool([])`, `bool([0])`, `bool("")`, `bool(" ")` return?"**
> `False`, `True`, `False`, `True`. Empty list is falsy, but a list containing zero is truthy (it's not empty). Empty string is falsy, but a string with a space is truthy (it has a character).

**Q148: What is `id()` used for?**
> Returns the memory address (unique identity) of an object. `id(x) == id(y)` is the same as `x is y`. Useful for debugging when you want to know if two variables point to the same object.

**Q149: What is `dir()` used for?**
> Lists all attributes and methods of an object. `dir([])` shows you everything you can do with a list. Great for exploration and debugging in the REPL. Without arguments, `dir()` shows names in the current scope.

**Q150: 🧠 FINAL TRAP — "What does this print?"**
```python
a = [1, 2, 3]
b = a[:]
b.append(4)
print(a)
print(a is b)
```
> `[1, 2, 3]` and `False`. `a[:]` creates a SHALLOW COPY — a new list object. So appending to `b` doesn't affect `a`. And `is` returns `False` because they're separate objects. Compare this with Q11 where `b = a` (no copy) made them the same object.

---

## 🔥 Top 15 Traps — Quick Revision

| # | Trap | Answer |
|---|------|--------|
| 1 | `bool("False")` | `True` — non-empty string |
| 2 | `[] == []` vs `[] is []` | `True` vs `False` |
| 3 | `-7 // 2` | `-4` not -3 (floors toward -∞) |
| 4 | `-7 % 2` | `1` not -1 |
| 5 | `[[]] * 3` then modify one | All three change (shared ref) |
| 6 | `0.1 + 0.2 == 0.3` | `False` (floating point) |
| 7 | `(1)` vs `(1,)` | `int` vs `tuple` |
| 8 | `{}` type | `dict`, not `set`. Use `set()` |
| 9 | Mutable default arg `def f(lst=[])` | Shared across calls |
| 10 | `x = [].sort()` | `x` is `None` |
| 11 | `print(x); x = 10` inside function | `UnboundLocalError` |
| 12 | `round(2.5)` | `2` not 3 (banker's rounding) |
| 13 | `except: pass` | Swallows ALL errors including Ctrl+C |
| 14 | `return` in `finally` | Overrides `try`'s return |
| 15 | Deleting from list/dict while iterating | `RuntimeError` / skips elements |

---

*This covers all 21 chapters of Phase 1. Practice by reading each Q&A out loud as if you're in an interview.*
