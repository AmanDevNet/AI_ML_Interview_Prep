# Chapter 16: Scope — Where Can Python See Your Variables?

---

## The One-Line Summary

> **Scope = Where a variable lives and who can see it.**

That's it. The entire topic of "scope" is just answering this question:
**"When I write `print(x)`, which `x` does Python pick?"**

---

## Step 1: Variables Inside a Function Stay Inside

```python
def greet():
    message = "hello"
    print(message)   # ✅ works

greet()
print(message)       # ❌ NameError — message doesn't exist out here
```

**Why?** `message` was born inside `greet()`. When `greet()` finishes, `message` dies.
Think of it like a hotel room — what's inside the room stays in the room.

---

## Step 2: Functions CAN Read Outside Variables

```python
name = "Aman"

def greet():
    print(name)   # ✅ prints "Aman"

greet()
```

**Why?** Python says: *"I don't see `name` inside `greet()`, let me look outside."*
It finds `name = "Aman"` at the file level. Done.

**Rule: Reading outside variables = FREE. Always works.**

---

## Step 3: But You Can't CHANGE Them (This Is Where Everyone Gets Confused)

```python
count = 10

def add_one():
    count = count + 1   # ❌ CRASHES — UnboundLocalError
    print(count)

add_one()
```

**Wait, why? We could READ `name` in Step 2, why can't we change `count` here?**

Because Python sees `count = ...` (an assignment) inside the function.
The moment Python sees you assigning to `count`, it thinks:
*"Oh, `count` is a LOCAL variable of this function."*

But then `count + 1` tries to read `count` before you assigned it. Crash.

**Think of it this way:**
- Step 2 was just LOOKING at an outside variable (reading) ✅
- Step 3 is trying to WRITE to it — Python gets confused ❌

---

## Step 4: The `global` Keyword — "I Mean the Outside One"

```python
count = 10

def add_one():
    global count          # "Hey Python, I mean the OUTSIDE count"
    count = count + 1     # ✅ now this works
    print(count)

add_one()
print(count)   # 11 — the outside count was changed too
```

`global` tells Python: *"Don't create a local variable. Use the one from outside."*

---

## Step 5: Nested Functions & `nonlocal`

Sometimes you have a function inside a function:

```python
def outer():
    x = 10

    def inner():
        print(x)    # ✅ can READ outer's x (just like Step 2)

    inner()

outer()   # prints 10
```

But what if `inner()` wants to CHANGE `outer()`'s variable?

```python
def outer():
    x = 10

    def inner():
        x = x + 1     # ❌ Same crash as Step 3!

    inner()

outer()
```

Same problem. Fix it with `nonlocal` (it's like `global` but for the parent function):

```python
def outer():
    x = 10

    def inner():
        nonlocal x     # "I mean outer's x"
        x = x + 1      # ✅ works now

    inner()
    print(x)           # 11

outer()
```

---

## Step 6: The LEGB Rule (Just a Fancy Name for the Search Order)

When Python sees a variable name, it searches in this order:

```
1. L — Local        (inside the current function)
2. E — Enclosing    (inside any parent functions)
3. G — Global       (at the top level of the file)
4. B — Built-in     (things like print, len, int)
```

It stops at the FIRST match. That's it. That's the whole LEGB rule.

```python
x = "global"

def outer():
    x = "enclosing"

    def inner():
        x = "local"
        print(x)       # "local" — found at L, stops looking

    inner()

outer()
```

Remove `x = "local"` from inner → prints `"enclosing"` (found at E).
Remove that too → prints `"global"` (found at G).
Remove that too → `NameError` (not found anywhere).

---

## Quick Cheat Sheet

| I want to... | Use |
|---|---|
| READ an outside variable | Just use it. Nothing special needed. |
| CHANGE a file-level variable | `global varname` |
| CHANGE a parent function's variable | `nonlocal varname` |
| CHANGE a list/dict without `global` | Just mutate it: `my_list.append(x)` works fine. Only reassignment `my_list = [...]` needs `global`. |

---

## Interview Traps — "What Does This Print?"

### Trap 1
```python
x = 5
def f():
    print(x)
    x = 10
f()
```
> **Answer:** `UnboundLocalError` — Python sees `x = 10` and marks `x` as local for the entire function. Then `print(x)` tries to read it before `x = 10` runs.

### Trap 2
```python
def outer():
    x = 10
    def change_x():
        x = 99          # no nonlocal, so this creates a NEW local x
    change_x()
    print(x)
```
> **Answer:** `10` — `change_x()` created its own local `x = 99` which died when `change_x()` ended. `outer()`'s `x` was never touched.

### Trap 3
```python
x = 1
def f():
    global x
    x = 2
    def g():
        global x
        x = 3
    g()
f()
print(x)
```
> **Answer:** `3` — both `f()` and `g()` point to the same global `x`. `f()` sets it to 2, then `g()` sets it to 3.

### Trap 4
```python
data = [1, 2, 3]
def add():
    data.append(4)     # mutating, not reassigning — works without global
def reset():
    data = []           # reassigning — creates a LOCAL data
add()
reset()
print(data)
```
> **Answer:** `[1, 2, 3, 4]` — `add()` mutated the global list. `reset()` only created a local variable and threw it away.

### Trap 5
```python
def outer():
    msg = "hello"
    def middle():
        msg = "world"
        def inner():
            nonlocal msg
            msg = "python"
        inner()
        print("middle:", msg)
    middle()
    print("outer:", msg)
outer()
```
> **Answer:** `middle: python` then `outer: hello`
> `nonlocal` targets the NEAREST parent that has `msg` — that's `middle()`. So `middle()`'s `msg` becomes `"python"`, but `outer()`'s `msg` stays `"hello"`.

### Trap 6
```python
x = 100
def f(x):
    x = x + 1
    print(x)
f(x)
print(x)
```
> **Answer:** `101` then `100` — function parameters are local variables. `f(x)` copies the value 100 into a local `x`. Adding 1 changes only the local copy.

### Trap 7
```python
val = "A"
class MyClass:
    val = "B"
    def show(self):
        print(val)
MyClass().show()
```
> **Answer:** `"A"` — Class body is NOT part of the LEGB chain for methods. `show()` skips the class and finds the global `val`. To get `"B"`, use `self.val`.

---

## Summary — The Entire Topic in 5 Lines

1. **Variables inside a function = local.** They die when the function ends.
2. **Reading** outside variables works automatically (Python searches outward).
3. **Writing** to outside variables needs `global` (for file-level) or `nonlocal` (for parent function).
4. **LEGB** = the search order: Local → Enclosing → Global → Built-in.
5. **The big trap:** Any assignment inside a function makes the variable local for the ENTIRE function — even lines before the assignment.

---

*← [Previous: Chapter 15 (Functions)](15_Functions.md) | [Next: Chapter 17 (Modules) →](17_Modules.md)*
