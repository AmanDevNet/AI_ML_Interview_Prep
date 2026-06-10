# Chapter 12: Loops

---

## Topic Overview

| Aspect | Details |
|--------|---------|
| **Definition** | Loops repeatedly execute a block of code until a condition is met (or an iterable is exhausted). |
| **Why It Exists** | To automate repetitive tasks, process collections, and implement algorithms that require iteration. |
| **Where It Is Used** | Data processing, searching/sorting, API pagination, training ML models, ETL pipelines. |
| **Real-World Analogy** | Like a **conveyor belt** in a factory — items keep moving through until the last one passes. |

---

## 12.1 for Loop

### Definition
`for` iterates over items of any iterable (list, tuple, string, range, dict, file, generator...).

### How It Works Internally
1. Python calls `iter()` on the iterable to get an **iterator**.
2. Repeatedly calls `next()` on the iterator to get each element.
3. When `StopIteration` is raised, the loop ends.

```python
# What Python does behind the scenes:
lst = [10, 20, 30]

# Your code:
for x in lst:
    print(x)

# Internally equivalent to:
iterator = iter(lst)
while True:
    try:
        x = next(iterator)
        print(x)
    except StopIteration:
        break
```

### Syntax and Examples
```python
# Iterate over list
for fruit in ["apple", "banana", "cherry"]:
    print(fruit)

# Iterate over string
for char in "Python":
    print(char, end=" ")  # P y t h o n

# Iterate over range
for i in range(5):
    print(i, end=" ")  # 0 1 2 3 4

# Iterate over dict
d = {"a": 1, "b": 2, "c": 3}
for key in d:           # Keys
    print(key, d[key])
for key, val in d.items():  # Key-value pairs
    print(f"{key}: {val}")

# Iterate with index
for i, fruit in enumerate(["apple", "banana", "cherry"]):
    print(f"{i}: {fruit}")

# Iterate multiple sequences
for name, age in zip(["Alice", "Bob"], [25, 30]):
    print(f"{name} is {age}")

# for-else (little-known feature!)
for n in [2, 4, 6, 8]:
    if n % 2 != 0:
        print("Found odd!")
        break
else:
    # Executes if loop completes WITHOUT break
    print("All even!")  # This prints
```

---

## 12.2 while Loop

### Definition
`while` repeats as long as a condition is `True`.

```python
# Basic
count = 0
while count < 5:
    print(count, end=" ")  # 0 1 2 3 4
    count += 1

# With user input
while True:
    response = input("Continue? (y/n): ")
    if response.lower() == 'n':
        break

# while-else
n = 10
while n > 0:
    if n == 5:
        break
    n -= 1
else:
    # Only executes if while condition becomes False (no break)
    print("Counted down without break")

# Sentinel pattern
total = 0
while (num := int(input("Enter number (0 to stop): "))) != 0:
    total += num
print(f"Total: {total}")
```

### Important: Avoid Infinite Loops
```python
# DANGER: infinite loop
# while True:
#     pass  # Ctrl+C to stop

# Always ensure the condition can become False:
i = 0
while i < 10:
    print(i)
    i += 1  # Without this → infinite loop!
```

---

## 12.3 Nested Loops

```python
# Multiplication table
for i in range(1, 6):
    for j in range(1, 6):
        print(f"{i*j:4}", end="")
    print()

# Pattern printing
n = 5
for i in range(1, n+1):
    print("*" * i)
# *
# **
# ***
# ****
# *****

# 2D list processing
matrix = [[1,2,3], [4,5,6], [7,8,9]]
for row in matrix:
    for val in row:
        print(f"{val:3}", end="")
    print()

# Nested loop time complexity: O(n × m)
```

---

## 12.4 break, continue, pass

### break
```python
# Exit the INNERMOST loop immediately
for i in range(10):
    if i == 5:
        break
    print(i, end=" ")  # 0 1 2 3 4

# break in nested loops — only breaks inner loop
for i in range(3):
    for j in range(3):
        if j == 2:
            break
        print(f"({i},{j})", end=" ")
    print()
# (0,0) (0,1)
# (1,0) (1,1)
# (2,0) (2,1)
```

### continue
```python
# Skip current iteration, go to next
for i in range(10):
    if i % 2 == 0:
        continue
    print(i, end=" ")  # 1 3 5 7 9
```

### pass
```python
# Placeholder — does nothing
for i in range(5):
    pass  # TODO: implement later

# Useful in:
# - Empty functions: def placeholder(): pass
# - Empty classes: class MyClass: pass
# - Empty loops for timing: for _ in range(1000): pass
```

---

## for-else and while-else

```python
# else block runs only if loop completes WITHOUT break

# Common use: search pattern
def find_prime(limit):
    for n in range(2, limit):
        for i in range(2, int(n**0.5) + 1):
            if n % i == 0:
                break
        else:
            # No divisor found → n is prime
            print(n, end=" ")

find_prime(30)  # 2 3 5 7 11 13 17 19 23 29

# Searching for an element
items = [10, 20, 30, 40]
target = 25
for item in items:
    if item == target:
        print(f"Found {target}!")
        break
else:
    print(f"{target} not found")
```

---

## Looping Techniques

```python
# enumerate — index + value
for i, val in enumerate(["a", "b", "c"], start=1):
    print(f"{i}. {val}")

# zip — parallel iteration
names = ["Alice", "Bob"]
scores = [95, 87]
for name, score in zip(names, scores):
    print(f"{name}: {score}")

# zip_longest — handle unequal lengths
from itertools import zip_longest
for a, b in zip_longest([1,2,3], [10,20], fillvalue=0):
    print(a, b)  # (1,10) (2,20) (3,0)

# reversed
for i in reversed(range(5)):
    print(i, end=" ")  # 4 3 2 1 0

# sorted
for ch in sorted("python"):
    print(ch, end="")  # hnopty

# itertools.product — nested loop alternative
from itertools import product
for i, j in product(range(3), range(3)):
    print(f"({i},{j})", end=" ")

# itertools.chain — iterate multiple sequences
from itertools import chain
for x in chain([1,2], [3,4], [5,6]):
    print(x, end=" ")  # 1 2 3 4 5 6

# Unpacking in loops
data = [(1, "a", True), (2, "b", False)]
for num, letter, flag in data:
    print(num, letter, flag)
```

---

## Common Mistakes

| Mistake | Example | Fix |
|---------|---------|-----|
| Modifying list during iteration | `for x in lst: lst.remove(x)` | `for x in lst[:]:` or use comprehension |
| Off-by-one | `for i in range(len(lst)):` accessing `lst[i+1]` | Check bounds or use `range(len(lst)-1)` |
| Infinite while loop | `while True:` without `break` condition | Ensure condition eventually becomes `False` |
| Unused loop variable | `for i in range(n):` but `i` unused | Use `_`: `for _ in range(n):` |
| Using `==` with float in while | `while x == 1.0:` | Use tolerance: `while abs(x - 1.0) > 1e-9:` |

---

## Interview Questions

**Q1: What is the difference between `for` and `while`?**
> `for` iterates over an iterable (known number of iterations). `while` repeats as long as a condition is True (unknown number of iterations). `for` is preferred when iterating over sequences.

**Q2: What does `for-else` do?**
> The `else` block after a `for` loop executes only if the loop completed without `break`. It's useful for search patterns.

**Q3: How do you break out of nested loops?**
> `break` only exits the innermost loop. To break out of multiple loops, use: (1) a flag variable, (2) put loops in a function and use `return`, (3) raise an exception, or (4) use `itertools.product`.

**Q4: What is the time complexity of a nested loop?**
> Two nested loops over ranges of size n and m give O(n × m). Three nested loops give O(n × m × k).

---

## Code Examples (15 total across difficulty levels)

### Easy (5)
```python
# 1: Sum of numbers 1 to 100
print(sum(range(1, 101)))  # 5050

# 2: Print even numbers
for i in range(2, 21, 2):
    print(i, end=" ")

# 3: Count vowels
text = "Hello World"
vowels = sum(1 for c in text.lower() if c in "aeiou")
print(f"\nVowels: {vowels}")

# 4: Reverse a string with loop
s = "Python"
reversed_s = ""
for c in s:
    reversed_s = c + reversed_s
print(reversed_s)  # nohtyP

# 5: Find maximum in list
nums = [3, 7, 2, 9, 1, 5]
max_val = nums[0]
for n in nums[1:]:
    if n > max_val:
        max_val = n
print(f"Max: {max_val}")
```

### Medium (5)
```python
# 6: Prime number checker
def is_prime(n):
    if n < 2: return False
    for i in range(2, int(n**0.5) + 1):
        if n % i == 0:
            return False
    return True
primes = [n for n in range(2, 50) if is_prime(n)]
print(primes)

# 7: Fibonacci sequence
def fibonacci(n):
    a, b = 0, 1
    result = []
    while len(result) < n:
        result.append(a)
        a, b = b, a + b
    return result
print(fibonacci(10))

# 8: Matrix transpose with loops
matrix = [[1,2,3], [4,5,6]]
transposed = []
for j in range(len(matrix[0])):
    row = []
    for i in range(len(matrix)):
        row.append(matrix[i][j])
    transposed.append(row)
print(transposed)

# 9: Remove consecutive duplicates
def remove_consecutive_dupes(lst):
    if not lst: return []
    result = [lst[0]]
    for item in lst[1:]:
        if item != result[-1]:
            result.append(item)
    return result
print(remove_consecutive_dupes([1,1,2,2,2,3,1,1]))  # [1,2,3,1]

# 10: Number guessing game
import random
target = random.randint(1, 100)
attempts = 0
# while True:
#     guess = int(input("Guess (1-100): "))
#     attempts += 1
#     if guess < target: print("Higher!")
#     elif guess > target: print("Lower!")
#     else:
#         print(f"Correct in {attempts} attempts!")
#         break
```

### Advanced (5)
```python
# 11: Sieve of Eratosthenes
def sieve(limit):
    is_prime = [True] * (limit + 1)
    is_prime[0] = is_prime[1] = False
    for i in range(2, int(limit**0.5) + 1):
        if is_prime[i]:
            for j in range(i*i, limit + 1, i):
                is_prime[j] = False
    return [i for i in range(limit + 1) if is_prime[i]]
print(sieve(50))

# 12: Iterator protocol demonstration
class Countdown:
    def __init__(self, start):
        self.start = start
    def __iter__(self):
        self.current = self.start
        return self
    def __next__(self):
        if self.current < 0:
            raise StopIteration
        val = self.current
        self.current -= 1
        return val

for n in Countdown(5):
    print(n, end=" ")  # 5 4 3 2 1 0

# 13: Binary search with while
def binary_search(lst, target):
    left, right = 0, len(lst) - 1
    while left <= right:
        mid = (left + right) // 2
        if lst[mid] == target: return mid
        elif lst[mid] < target: left = mid + 1
        else: right = mid - 1
    return -1
sorted_list = list(range(0, 100, 2))
print(f"\nIndex of 42: {binary_search(sorted_list, 42)}")

# 14: Collatz conjecture
def collatz_steps(n):
    steps = 0
    while n != 1:
        n = n // 2 if n % 2 == 0 else 3 * n + 1
        steps += 1
    return steps
for n in [1, 5, 27, 100]:
    print(f"Collatz({n}): {collatz_steps(n)} steps")

# 15: Breaking nested loops with function
def find_in_matrix(matrix, target):
    for i, row in enumerate(matrix):
        for j, val in enumerate(row):
            if val == target:
                return (i, j)
    return None

matrix = [[1,2,3], [4,5,6], [7,8,9]]
print(find_in_matrix(matrix, 5))  # (1, 1)
```

---

## Practice Questions

### Easy
**Q1:** Print all numbers from 1 to 50 that are divisible by both 3 and 5.
```python
for n in range(1, 51):
    if n % 3 == 0 and n % 5 == 0:
        print(n, end=" ")  # 15 30 45
```

**Q2:** Calculate the factorial of a number using a while loop.
```python
n = 5
factorial = 1
while n > 1:
    factorial *= n
    n -= 1
print(f"5! = {factorial}")  # 120
```

### Medium
**Q3:** Write a program to print the first 20 numbers in the Fibonacci sequence.
```python
a, b = 0, 1
for _ in range(20):
    print(a, end=" ")
    a, b = b, a + b
```

**Q4:** Write a program that finds all pairs in a list that sum to a target value.
```python
def find_pairs(lst, target):
    seen = set()
    pairs = []
    for num in lst:
        complement = target - num
        if complement in seen:
            pairs.append((complement, num))
        seen.add(num)
    return pairs
print(find_pairs([1,2,3,4,5,6], 7))  # [(3,4), (2,5), (1,6)]
```

### Advanced
**Q5:** Implement bubble sort with optimization (early termination).
```python
def bubble_sort(lst):
    lst = lst.copy()
    n = len(lst)
    for i in range(n):
        swapped = False
        for j in range(0, n-i-1):
            if lst[j] > lst[j+1]:
                lst[j], lst[j+1] = lst[j+1], lst[j]
                swapped = True
        if not swapped:
            break
    return lst
print(bubble_sort([64, 34, 25, 12, 22, 11, 90]))
```

**Q6:** Write a generator function for an infinite Fibonacci sequence, then use it to find the first Fibonacci number above 1 million.
```python
def fib_generator():
    a, b = 0, 1
    while True:
        yield a
        a, b = b, a + b

for n in fib_generator():
    if n > 1_000_000:
        print(f"First Fibonacci > 1M: {n}")
        break
```

---

*← [Previous: Conditional Statements](11_Conditional_Statements.md) | [Next: Range Function →](13_Range_Function.md)*
