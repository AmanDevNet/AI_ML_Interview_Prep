# Chapter 6: Strings

---

## Topic Overview

| Aspect | Details |
|--------|---------|
| **Definition** | A string (`str`) is an **immutable sequence of Unicode characters**. |
| **Why It Exists** | Text processing is fundamental to virtually all programs — web, data, AI, scripting. |
| **Where It Is Used** | Everywhere: user input, file I/O, APIs, database queries, NLP, prompt engineering. |
| **Real-World Analogy** | A string is like a **necklace of beads** — each bead (character) has a fixed position, and you can't change a bead without making a new necklace. |

### How It Works Internally
- Strings are **immutable sequences** — every modification creates a new string object.
- CPython uses **compact representation**: 1, 2, or 4 bytes per character depending on the widest character.
- Short/common strings may be **interned** (cached) for performance.
- String concatenation with `+` in a loop is O(n²) — use `"".join()` for O(n).

---

## 6.1 Creation

```python
# Single quotes
s1 = 'hello'

# Double quotes
s2 = "hello"

# Triple quotes (multiline)
s3 = '''line 1
line 2
line 3'''

s4 = """also
multiline"""

# Empty string
empty = ""

# From other types
s5 = str(42)        # "42"
s6 = str(3.14)      # "3.14"
s7 = str([1, 2])    # "[1, 2]"
s8 = str(True)      # "True"

# String with special characters
s9 = "He said \"hello\""
s10 = 'It\'s Python'
s11 = "It's Python"     # No escape needed with double quotes

# Unicode
emoji = "🐍"
chinese = "你好"
print(len(emoji))    # 1 (one character, even though 4 bytes in UTF-8)
```

---

## 6.2 Indexing

```python
s = "Python"
#    P  y  t  h  o  n
#    0  1  2  3  4  5   (positive index)
#   -6 -5 -4 -3 -2 -1   (negative index)

print(s[0])    # P (first character)
print(s[5])    # n (last character)
print(s[-1])   # n (last character)
print(s[-6])   # P (first character)

# IndexError
# print(s[6])   # IndexError: string index out of range
# print(s[-7])  # IndexError
```

---

## 6.3 Slicing

```python
s = "Python Programming"

# Syntax: s[start:stop:step]
# start = inclusive, stop = exclusive, step = increment

print(s[0:6])      # "Python"
print(s[7:])       # "Programming"
print(s[:6])       # "Python"
print(s[:])        # "Python Programming" (full copy)
print(s[::2])      # "Pto rgamn" (every 2nd character)
print(s[::-1])     # "gnimmargorP nohtyP" (reversed!)

# Negative indexing in slices
print(s[-11:])     # "Programming"
print(s[:-12])     # "Python"

# Step with direction
print(s[5::-1])    # "nohtyP" (from index 5 to 0, reversed)

# Slicing NEVER raises IndexError (graceful handling)
print(s[0:100])    # "Python Programming" (no error, goes to end)
print(s[100:200])  # "" (empty string, no error)
```

### Slicing Memory Behavior
```python
# Slicing creates a NEW string object (strings are immutable)
s = "hello"
t = s[:]
print(s is t)   # True in CPython (optimization for full slice)
                 # But semantically, treat as a copy

t = s[1:]
print(s is t)   # False (different string)
```

---

## 6.4 Traversing

```python
s = "Python"

# Method 1: Direct iteration (Pythonic)
for char in s:
    print(char, end=" ")   # P y t h o n

# Method 2: Index-based
for i in range(len(s)):
    print(f"s[{i}] = {s[i]}")

# Method 3: enumerate (best for index + value)
for i, char in enumerate(s):
    print(f"Index {i}: {char}")

# Method 4: While loop
i = 0
while i < len(s):
    print(s[i], end="")
    i += 1
```

---

## 6.5 String Methods

### Case Methods
```python
s = "hello World"

print(s.upper())        # "HELLO WORLD"
print(s.lower())        # "hello world"
print(s.title())        # "Hello World"
print(s.capitalize())   # "Hello world" (only first char)
print(s.swapcase())     # "HELLO wORLD"
print(s.casefold())     # "hello world" (aggressive lowering for comparisons)

# casefold vs lower
print("straße".lower())     # "straße"
print("straße".casefold())  # "strasse" (German ß → ss)
```

### Search Methods
```python
s = "hello world hello"

print(s.find("hello"))       # 0 (first occurrence)
print(s.find("hello", 1))    # 12 (search from index 1)
print(s.find("xyz"))         # -1 (not found)
print(s.rfind("hello"))      # 12 (last occurrence)

print(s.index("hello"))      # 0
# print(s.index("xyz"))      # ValueError! (unlike find())

print(s.count("hello"))      # 2
print(s.count("l"))          # 4

print(s.startswith("hello")) # True
print(s.endswith("hello"))   # True
print(s.startswith(("he", "Hi")))  # True (tuple of prefixes)
```

### Modification Methods (Return New String)
```python
s = "  hello world  "

print(s.strip())        # "hello world" (both sides)
print(s.lstrip())       # "hello world  " (left only)
print(s.rstrip())       # "  hello world" (right only)
print(s.strip(" hd"))   # "ello worl" (strip specific chars)

s = "hello world"
print(s.replace("world", "Python"))   # "hello Python"
print(s.replace("l", "L", 1))         # "heLlo world" (max 1 replacement)

# Split and Join
s = "a,b,c,d,e"
print(s.split(","))      # ['a', 'b', 'c', 'd', 'e']
print(s.split(",", 2))   # ['a', 'b', 'c,d,e'] (max 2 splits)

s = "hello   world"
print(s.split())          # ['hello', 'world'] (splits on any whitespace)
print(s.split(" "))       # ['hello', '', '', 'world'] (splits on single space)

# Join (opposite of split)
words = ['Python', 'is', 'awesome']
print(" ".join(words))    # "Python is awesome"
print("-".join(words))    # "Python-is-awesome"
print("".join(words))     # "Pythonisawesome"

# Partition
s = "hello=world"
print(s.partition("="))    # ('hello', '=', 'world')
print(s.rpartition("="))   # ('hello', '=', 'world')
print(s.partition("x"))    # ('hello=world', '', '')

# Center, ljust, rjust, zfill
print("hello".center(20))        # "       hello        "
print("hello".center(20, "*"))   # "*******hello********"
print("hello".ljust(20, "."))    # "hello..............."
print("hello".rjust(20, "."))    # "...............hello"
print("42".zfill(5))             # "00042"
print("-42".zfill(5))            # "-0042" (sign-aware!)
```

### Validation Methods
```python
print("hello".isalpha())      # True (all alphabetic)
print("hello123".isalnum())   # True (all alphanumeric)
print("12345".isdigit())      # True (all digits)
print("12345".isnumeric())    # True
print("12345".isdecimal())    # True
print("hello".islower())      # True
print("HELLO".isupper())      # True
print("Hello World".istitle()) # True
print("   ".isspace())        # True
print("hello".isascii())      # True (Python 3.7+)
print("hello123".isidentifier())  # True (valid variable name)
print("".isalpha())           # False (empty string returns False for all is* methods)

# isdigit vs isnumeric vs isdecimal
print("²".isdigit())     # True (superscript 2)
print("²".isnumeric())   # True
print("²".isdecimal())   # False (not a decimal digit)
print("½".isnumeric())   # True (fraction)
print("½".isdigit())     # False
print("½".isdecimal())   # False
```

### Encoding Methods
```python
# Encode (str → bytes)
s = "hello 🐍"
b = s.encode("utf-8")
print(b)            # b'hello \xf0\x9f\x90\x8d'
print(len(b))       # 10 (4 bytes for emoji)

# Decode (bytes → str)
print(b.decode("utf-8"))  # "hello 🐍"

# Handle encoding errors
s = "hello 🐍"
print(s.encode("ascii", errors="replace"))   # b'hello ?'
print(s.encode("ascii", errors="ignore"))    # b'hello '
print(s.encode("ascii", errors="xmlcharrefreplace"))  # b'hello &#128013;'
```

---

## 6.6 Escape Characters

| Escape | Meaning |
|--------|---------|
| `\n` | Newline |
| `\t` | Tab |
| `\\` | Backslash |
| `\'` | Single quote |
| `\"` | Double quote |
| `\r` | Carriage return |
| `\b` | Backspace |
| `\0` | Null character |
| `\a` | Bell/alert |
| `\f` | Form feed |
| `\ooo` | Octal value |
| `\xhh` | Hex value |
| `\uxxxx` | Unicode (16-bit) |
| `\Uxxxxxxxx` | Unicode (32-bit) |
| `\N{name}` | Unicode by name |

```python
print("Line1\nLine2")           # Newline
print("Col1\tCol2")             # Tab
print("Path: C:\\Users\\name")  # Backslash
print("He said \"hi\"")         # Quotes
print("\u0041")                  # A (Unicode)
print("\N{SNAKE}")               # 🐍 (Unicode by name)
print("\x41\x42\x43")           # ABC (hex)
```

---

## 6.7 Raw Strings

```python
# Raw strings don't process escape sequences
raw = r"C:\Users\name\folder\file.txt"
print(raw)   # C:\Users\name\folder\file.txt (backslashes preserved)

# Useful for regex patterns
import re
pattern = r"\d+\.\d+"  # Without raw: "\\d+\\.\\d+"

# Raw strings CANNOT end with odd number of backslashes
# r"hello\"  → SyntaxError (the backslash escapes the closing quote)
# Workaround: r"hello" + "\\"
```

---

## 6.8 String Formatting

> Covered in detail in Chapter 5. Quick reference:

```python
name = "Alice"
age = 25

# f-string (recommended)
print(f"{name} is {age}")

# .format()
print("{} is {}".format(name, age))

# % formatting
print("%s is %d" % (name, age))
```

---

## Performance: String Concatenation

```python
import time

# BAD: O(n²) — creates new string each iteration
start = time.time()
result = ""
for i in range(100000):
    result += str(i)
print(f"Concatenation: {time.time() - start:.3f}s")

# GOOD: O(n) — join at the end
start = time.time()
parts = []
for i in range(100000):
    parts.append(str(i))
result = "".join(parts)
print(f"Join: {time.time() - start:.3f}s")

# BEST: O(n) — list comprehension + join
start = time.time()
result = "".join(str(i) for i in range(100000))
print(f"GenExpr Join: {time.time() - start:.3f}s")
```

---

## Common Mistakes

| Mistake | Example | Fix |
|---------|---------|-----|
| Strings are immutable | `s[0] = "H"` → TypeError | `s = "H" + s[1:]` |
| Loop concatenation | `s += char` in loop → O(n²) | Use `"".join(list)` |
| `split()` vs `split(" ")` | `"a  b".split(" ")` → `['a', '', 'b']` | `"a  b".split()` → `['a', 'b']` |
| `find()` returns -1, not error | Using result as index without checking | Check `if s.find(x) != -1` |
| Confusing `is` and `==` | `"hello" is "hello"` → may be True (interning) | Always use `==` for string comparison |

---

## Interview Questions

**Q1: Are strings mutable in Python?**
> No. Strings are immutable. Every operation that appears to modify a string actually creates a new string object.

**Q2: What is string interning?**
> CPython automatically caches small/simple strings (typically those that look like identifiers). This means `a = "hello"; b = "hello"; a is b` may be `True`. Use `sys.intern()` to manually intern strings.

**Q3: Why is `"".join(list)` better than `+` in a loop?**
> `+` creates a new string each iteration → O(n²) total. `"".join()` pre-calculates the total length and copies once → O(n).

**Q4: What is the difference between `find()` and `index()`?**
> `find()` returns `-1` if substring not found. `index()` raises `ValueError`.

**Q5: How does Python store strings internally?**
> CPython uses compact representation: 1 byte per char (Latin-1) if all ASCII, 2 bytes (UCS-2) if any char needs it, or 4 bytes (UCS-4) for characters like emojis.

---

## Code Examples

### Easy Examples

```python
# Example 1: String basics
s = "Python"
print(f"Length: {len(s)}")
print(f"First: {s[0]}, Last: {s[-1]}")
print(f"Upper: {s.upper()}")
print(f"Lower: {s.lower()}")
print(f"Reversed: {s[::-1]}")

# Example 2: Check palindrome
word = "racecar"
print(f"'{word}' is palindrome: {word == word[::-1]}")

# Example 3: Count vowels
text = "Hello World"
vowels = sum(1 for c in text.lower() if c in "aeiou")
print(f"Vowels in '{text}': {vowels}")

# Example 4: String repetition
print("=" * 40)
print("Python".center(40))
print("=" * 40)

# Example 5: Split and rejoin
csv_line = "Alice,25,New York,Engineer"
fields = csv_line.split(",")
print(fields)  # ['Alice', '25', 'New York', 'Engineer']
print(" | ".join(fields))  # Alice | 25 | New York | Engineer
```

### Medium Examples

```python
# Example 6: Title case with exceptions
def smart_title(s, exceptions={"a", "an", "the", "in", "on", "at", "to", "for", "of"}):
    words = s.lower().split()
    result = [words[0].capitalize()]
    for word in words[1:]:
        result.append(word if word in exceptions else word.capitalize())
    return " ".join(result)

print(smart_title("the lord of the rings"))
# The Lord of the Rings

# Example 7: Extract all numbers from string
import re
text = "I have 3 cats, 2 dogs, and 15 fish."
numbers = re.findall(r'\d+', text)
print(numbers)  # ['3', '2', '15']
print(sum(int(n) for n in numbers))  # 20

# Example 8: Caeser cipher
def caesar_encrypt(text, shift):
    result = []
    for char in text:
        if char.isalpha():
            base = ord('A') if char.isupper() else ord('a')
            result.append(chr((ord(char) - base + shift) % 26 + base))
        else:
            result.append(char)
    return "".join(result)

encrypted = caesar_encrypt("Hello World", 3)
print(f"Encrypted: {encrypted}")  # Khoor Zruog
print(f"Decrypted: {caesar_encrypt(encrypted, -3)}")  # Hello World

# Example 9: String alignment table
data = [
    ("Product", "Price", "Stock"),
    ("Laptop", "$999.99", "15"),
    ("Mouse", "$29.99", "150"),
    ("Keyboard", "$79.99", "45"),
]

widths = [max(len(row[i]) for row in data) for i in range(3)]
for i, row in enumerate(data):
    line = " | ".join(val.ljust(widths[j]) for j, val in enumerate(row))
    print(line)
    if i == 0:
        print("-+-".join("-" * w for w in widths))

# Example 10: Remove duplicates preserving order
s = "programming"
result = "".join(dict.fromkeys(s))
print(f"'{s}' → '{result}'")  # 'programming' → 'progamin'
```

### Advanced Examples

```python
# Example 11: String interning investigation
import sys

# Identifier-like strings are interned
a = "hello"
b = "hello"
print(f"Identifier: {a is b}")  # True

# Strings with spaces aren't automatically interned
a = "hello world"
b = "hello world"
print(f"With space: {a is b}")  # Varies by context

# Manual interning
a = sys.intern("hello world!")
b = sys.intern("hello world!")
print(f"Interned: {a is b}")  # True

# Example 12: String memory optimization
import sys

# ASCII-only string
ascii_str = "hello"
print(f"ASCII '{ascii_str}': {sys.getsizeof(ascii_str)} bytes")

# With Latin character
latin_str = "hëllo"
print(f"Latin '{latin_str}': {sys.getsizeof(latin_str)} bytes")

# With CJK character
cjk_str = "h你llo"
print(f"CJK '{cjk_str}': {sys.getsizeof(cjk_str)} bytes")

# With emoji
emoji_str = "h🐍llo"
print(f"Emoji '{emoji_str}': {sys.getsizeof(emoji_str)} bytes")

# Example 13: Efficient string builder pattern
class StringBuilder:
    def __init__(self):
        self._parts = []
    
    def append(self, s):
        self._parts.append(str(s))
        return self  # Enable chaining
    
    def __str__(self):
        return "".join(self._parts)
    
    def __len__(self):
        return sum(len(p) for p in self._parts)

sb = StringBuilder()
sb.append("Hello").append(" ").append("World").append("!")
print(str(sb))  # Hello World!
print(f"Length: {len(sb)}")  # 12

# Example 14: Unicode normalization
import unicodedata

# Two ways to represent "é"
e_composed = "\u00e9"        # Single code point (é)
e_decomposed = "e\u0301"     # e + combining accent

print(f"Composed: {e_composed}, len={len(e_composed)}")    # é, len=1
print(f"Decomposed: {e_decomposed}, len={len(e_decomposed)}")  # é, len=2
print(f"Visual equal: {e_composed} == {e_decomposed}")  # True visually
print(f"Code equal: {e_composed == e_decomposed}")       # False!

# Normalize to fix
nfc = unicodedata.normalize("NFC", e_decomposed)
print(f"After NFC: {nfc == e_composed}")  # True

# Example 15: String translation table (fastest character replacement)
# Create translation table
table = str.maketrans(
    "aeiou",       # Characters to replace
    "12345",       # Replacement characters
    " "            # Characters to delete
)

text = "hello world python"
print(text.translate(table))  # h2ll4w4rldpyth4n

# Useful for removing punctuation
import string
remove_punct = str.maketrans("", "", string.punctuation)
messy = "Hello, World! How's it going?"
clean = messy.translate(remove_punct)
print(clean)  # Hello World Hows it going
```

---

## Edge Cases

1. **Empty string**: `""` has `len() = 0`, is falsy, `"".split()` returns `[]`.
2. **Single character**: In Python, there's no separate `char` type — it's a string of length 1.
3. **`"".join([])` returns `""`**: Joining an empty list gives an empty string.
4. **`s * 0` returns `""`**: Multiplying any string by 0 gives empty string.
5. **Negative step with wrong bounds**: `s[0:5:-1]` returns `""` (can't go backwards from 0 to 5).
6. **`str(None)` returns `"None"`**: The string `"None"`, not an empty string.
7. **Unicode length**: `len("🐍")` is 1, but it's 4 bytes in UTF-8.
8. **`split()` vs `split(' ')` with multiple spaces**: `"a  b".split()` → `['a', 'b']` but `"a  b".split(' ')` → `['a', '', 'b']`.

---

## Comparison with Similar Concepts

| Feature | Python `str` | Java `String` | C `char[]` | JavaScript `string` |
|---------|-------------|---------------|------------|---------------------|
| Mutable | ❌ | ❌ | ✅ | ❌ |
| Unicode | ✅ (default) | ✅ (UTF-16) | ❌ (ASCII) | ✅ (UTF-16) |
| Indexing | `s[i]` | `s.charAt(i)` | `s[i]` | `s[i]` or `s.charAt(i)` |
| Null-terminated | ❌ | ❌ | ✅ | ❌ |
| Length | `len(s)` | `s.length()` | `strlen(s)` | `s.length` |
| Comparison | `==` | `.equals()` | `strcmp()` | `===` |

---

## Memory Tips

- **"Strings are immutable necklaces"** — can't change beads, must make new necklace.
- **"join() beats + in loops"** — O(n) vs O(n²).
- **"find returns -1, index raises ValueError"** — find is forgiving, index is strict.
- **"split() is smart, split(' ') is literal"** — no-arg split handles multiple spaces.
- **"All is* methods return False for empty string"** — `"".isdigit()` is `False`.

---

## Practice Questions

### Easy

**Q1:** Write a function that checks if a string is a palindrome (ignore case and spaces).
```python
# Answer:
def is_palindrome(s):
    cleaned = s.lower().replace(" ", "")
    return cleaned == cleaned[::-1]

print(is_palindrome("racecar"))        # True
print(is_palindrome("Race Car"))       # True
print(is_palindrome("hello"))          # False
print(is_palindrome("Was it a car or a cat I saw"))  # True
```

**Q2:** Count the frequency of each character in a string (ignore spaces).
```python
# Answer:
def char_frequency(s):
    freq = {}
    for char in s.lower():
        if char != " ":
            freq[char] = freq.get(char, 0) + 1
    return freq

result = char_frequency("Hello World")
for char, count in sorted(result.items()):
    print(f"'{char}': {count}")
```

### Medium

**Q3:** Write a function that compresses a string: `"aaabbbcc"` → `"a3b3c2"`. If compressed is not shorter, return original.
```python
# Answer:
def compress(s):
    if not s:
        return s
    compressed = []
    count = 1
    for i in range(1, len(s)):
        if s[i] == s[i-1]:
            count += 1
        else:
            compressed.append(f"{s[i-1]}{count}")
            count = 1
    compressed.append(f"{s[-1]}{count}")
    result = "".join(compressed)
    return result if len(result) < len(s) else s

print(compress("aaabbbcc"))     # a3b3c2
print(compress("aabbaabb"))     # aabbaabb (compressed = "a2b2a2b2", same length)
print(compress("abcde"))        # abcde (compressed would be longer)
```

**Q4:** Write a function that finds the first non-repeating character in a string.
```python
# Answer:
def first_unique_char(s):
    freq = {}
    for char in s:
        freq[char] = freq.get(char, 0) + 1
    for char in s:
        if freq[char] == 1:
            return char
    return None

print(first_unique_char("aabbcde"))  # c
print(first_unique_char("aabb"))     # None
print(first_unique_char("abcab"))    # c
```

### Advanced

**Q5:** Implement a function that checks if two strings are anagrams using only O(1) extra space (assuming ASCII characters).
```python
# Answer:
def are_anagrams(s1, s2):
    if len(s1) != len(s2):
        return False
    # Use a fixed-size array (256 for ASCII)
    counts = [0] * 256
    for c1, c2 in zip(s1.lower(), s2.lower()):
        counts[ord(c1)] += 1
        counts[ord(c2)] -= 1
    return all(c == 0 for c in counts)

print(are_anagrams("listen", "silent"))   # True
print(are_anagrams("hello", "world"))     # False
print(are_anagrams("Astronomer", "Moon starer"))  # False (spaces count)

# Alternative: using sorted (O(n log n) time, O(n) space)
def are_anagrams_v2(s1, s2):
    return sorted(s1.lower()) == sorted(s2.lower())
```

**Q6:** Implement an efficient `str.replace()` equivalent that handles overlapping patterns correctly.
```python
# Answer:
def replace_all(s, old, new):
    """Replace all non-overlapping occurrences of old with new."""
    if not old:
        # Empty pattern: insert new between every character
        return new + new.join(s) + new
    
    result = []
    i = 0
    while i < len(s):
        if s[i:i+len(old)] == old:
            result.append(new)
            i += len(old)
        else:
            result.append(s[i])
            i += 1
    return "".join(result)

print(replace_all("hello world", "world", "Python"))  # hello Python
print(replace_all("aaa", "aa", "b"))                   # ba (non-overlapping)
print(replace_all("hello", "", "-"))                    # -h-e-l-l-o-
```

---

*← [Previous: Input & Output](05_Input_Output.md) | [Next: Lists →](07_Lists.md)*
