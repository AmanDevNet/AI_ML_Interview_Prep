# Chapter 10: Regular Expressions (Regex) — "Text Searching with Superpowers"

---

## Topic Overview

| Aspect | Details |
|--------|---------|
| **Definition** | Regular Expressions (Regex) are special sequences of characters that form a search pattern. You use them to find, match, and manipulate text. |
| **Why It Exists** | Finding text using simple methods like `.find()` or `in` is too basic. Regex allows matching complex patterns (e.g., "any email address", "any phone number with or without dashes"). |
| **Where It Is Used** | Data validation (email, password strength), data cleaning (scraping text), log parsing, text refactoring, and NLP preprocessing. |
| **Real-World Analogy** | Using a stencil to paint only specific shapes on a wall, rather than trying to draw freehand. |

---

## Think of It This Way

Imagine you are looking for phone numbers in a 1,000-page book. 
A phone number might look like:
* `123-456-7890`
* `(123) 456-7890`
* `123.456.7890`

If you try to write standard Python code to check for every combination of dashes, parentheses, spaces, and periods, you'll end up with dozens of nested `if` statements.

With **Regex**, you write a single pattern:
`\(?\d{3}\)?[-.\s]?\d{3}[-.\s]?\d{4}`

This reads as: *"An optional opening parenthesis, 3 digits, an optional closing parenthesis, an optional separator (dash, dot, or space), 3 digits, an optional separator, and 4 digits."*

---

## Step 1: Python's `re` Module Functions

Python provides the `re` module for regular expressions. Here are the core functions you must know:

### 1. `re.search()` vs `re.match()`
* **`re.match()`** checks for a match **only at the beginning** of the string.
* **`re.search()`** searches the **entire string** for the first match.

```python
import re

text = "The price is $100"

# Match checks the very start (fails because 'The' is not a number)
print(re.match(r'\d+', text))  # None

# Search scans the whole string (finds '100')
match = re.search(r'\d+', text)
if match:
    print(match.group())  # '100'
```

### 2. `re.findall()` vs `re.finditer()`
* **`re.findall()`** returns a **list of strings** of all matches.
* **`re.finditer()`** returns an **iterator** yielding Match objects (saves memory, gives positions).

```python
text = "Contact us at 123-456 and 789-012"

# Findall
print(re.findall(r'\d+-\d+', text))  # ['123-456', '789-012']

# Finditer
for match in re.finditer(r'\d+-\d+', text):
    print(f"Found: {match.group()} at index {match.start()}-{match.end()}")
```

### 3. `re.sub()` (Substitute/Replace) and `re.split()`
```python
text = "apple, banana; orange  grape"

# Split by comma, semicolon, or spaces
fruits = re.split(r'[,;\s]+', text)
print(fruits)  # ['apple', 'banana', 'orange', 'grape']

# Replace all digits with 'X'
text_with_digits = "Order 4529 is complete"
clean_text = re.sub(r'\d', 'X', text_with_digits)
print(clean_text)  # "Order XXXX is complete"
```

---

## Step 2: Greediness vs Laziness

By default, Regex operators (`*`, `+`, `?`, `{}`) are **greedy** — they match as much text as possible. Adding a `?` after them makes them **lazy** (or non-greedy) — they match as little text as possible.

```python
text = "<div>Hello</div><div>World</div>"

# Greedy match: matches from the first '<' to the last '>'
greedy = re.search(r'<div>.*</div>', text)
print(greedy.group())  # "<div>Hello</div><div>World</div>"

# Lazy match (added '?'): matches up to the first closing sequence
lazy = re.search(r'<div>.*?</div>', text)
print(lazy.group())  # "<div>Hello</div>"
```

---

## Code Examples

### Easy Examples
```python
# 1. Matching digits
print(re.findall(r'\d', "abc123xyz"))  # ['1', '2', '3']

# 2. Matching non-digits
print(re.findall(r'\D+', "abc123xyz"))  # ['abc', 'xyz']

# 3. Simple email format check
email_pattern = r'^[\w\.-]+@[\w\.-]+\.\w+$'
print(bool(re.match(email_pattern, "test@example.com")))  # True
print(bool(re.match(email_pattern, "invalid-email")))      # False

# 4. Word boundary check (\b)
# Match 'cat' as a full word, not inside 'catalog'
text = "The catalog has a cat."
print(re.findall(r'\bcat\b', text))  # ['cat']

# 5. Case-insensitive matching
print(re.findall(r'python', "Python is cool, pyThOn!", re.IGNORECASE))
# ['Python', 'pyThOn']
```

### Medium Examples
```python
# 6. Extracting matching groups
text = "John Doe (Age: 32)"
pattern = r"([A-Za-z\s]+)\(Age:\s(\d+)\)"
match = re.search(pattern, text)
if match:
    print(match.group(1).strip()) # "John Doe" (Group 1)
    print(match.group(2))         # "32"       (Group 2)

# 7. Named Groups
pattern = r"(?P<name>[A-Za-z\s]+)\(Age:\s(?P<age>\d+)\)"
match = re.search(pattern, text)
if match:
    print(match.groupdict())  # {'name': 'John Doe ', 'age': '32'}

# 8. Formatting Phone Numbers
phone = "1234567890"
# Reformat to (123) 456-7890
formatted = re.sub(r'(\d{3})(\d{3})(\d{4})', r'(\1) \2-\3', phone)
print(formatted)  # (123) 456-7890

# 9. Matching hexadecimal colors
color_pattern = r'^#(?:[0-9a-fA-F]{3}){1,2}$'
print(bool(re.match(color_pattern, "#FFF")))   # True
print(bool(re.match(color_pattern, "#invalid"))) # False

# 10. Split text preserving delimiters
# Wrapping pattern in parenthesis keeps separators in the result
print(re.split(r'([,;])', "a,b;c"))  # ['a', ',', 'b', ';', 'c']
```

### Advanced Examples
```python
# 11. Custom Regex Replacement using a Function
# Convert Celsius temperatures to Fahrenheit in a text
def c_to_f(match):
    celsius = float(match.group(1))
    fahrenheit = (celsius * 9/5) + 32
    return f"{fahrenheit:.1f}°F"

text = "The room is 25°C and the processor is 65°C."
converted = re.sub(r'(\d+)°C', c_to_f, text)
print(converted)  # "The room is 77.0°F and the processor is 149.0°F."

# 12. Password validation (Lookahead assertions)
# Requirements: 1 uppercase, 1 lowercase, 1 digit, min length 8
password_pattern = r'^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)[A-Za-z\d]{8,}$'
print(bool(re.match(password_pattern, "Weakpass")))   # False (no digit)
print(bool(re.match(password_pattern, "Str0ngPass"))) # True

# 13. HTML tag content extractor (Lazy matching & groups)
html = "Paragraph: <p id='main'>Hello World</p> and <b>bold text</b>"
tags = re.findall(r'<(\w+).*?>(.*?)</\1>', html)
# \1 refers back to the matched tag name (e.g., 'p' or 'b')
print(tags)  # [('p', 'Hello World'), ('b', 'bold text')]

# 14. Compiling patterns for performance
# Compiling saves regex parse time if pattern is used repeatedly in a loop
compiled_pattern = re.compile(r'\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b')
emails = ["alice@example.com", "bob@org.co.uk", "invalid.email@"]
valid_emails = [email for email in emails if compiled_pattern.match(email)]
print(valid_emails)  # ['alice@example.com', 'bob@org.co.uk']

# 15. Finding duplicate words (Backreferences)
text = "The the cat sat on on the mat."
duplicates = re.findall(r'\b(\w+)\s+\1\b', text, re.IGNORECASE)
print(duplicates)  # ['The', 'on']
```

---

## Common Mistakes & Edge Cases

### 1. Forgetting the `r` Prefix (Raw String)
Python uses backslashes `\` for escape characters (like `\n` for newline). Regex *also* uses backslashes (like `\d` for digit). If you forget the `r` prefix before your pattern, Python might misinterpret your regex escape sequences.
* **Bad:** `"\d+"` (Python interprets this, might break or trigger warning)
* **Good:** `r"\d+"` (Raw string: backslashes are treated as literal characters and passed straight to the regex engine)

### 2. Match vs Search
* **`re.match()`** will return `None` if the pattern isn't at the *very first character* of the string. Beginners often write `re.match()` when they actually want `re.search()`.

### 3. Star `*` vs Plus `+`
`*` matches 0 or more times. This means it can match an empty string!
```python
print(re.findall(r'\d*', "abc"))  # ['', '', '', ''] — matches empty strings!
print(re.findall(r'\d+', "abc"))  # [] — correct behavior
```

---

## Interview Questions (Top 5)

**Q1: What is the difference between `re.match()` and `re.search()` in Python?**
> `re.match()` checks for a match only at the very beginning of the target string. `re.search()` searches throughout the entire string for the first match, regardless of its position.

**Q2: What are greedy and lazy quantifiers? How do you toggle between them?**
> Greedy quantifiers (`*`, `+`, `{}`) match as much text as possible. Lazy (non-greedy) quantifiers match the minimum amount of text required to satisfy the pattern. You convert a greedy quantifier to a lazy one by adding a question mark `?` immediately after it (e.g., `.*?` or `.+?`).

**Q3: What does the `r` prefix in `r'\d+'` do and why is it recommended for regex?**
> The `r` prefix stands for "raw string". It tells Python's parser not to escape backslashes. Without it, sequences like `\b` (regex word boundary) would be parsed by Python as an ASCII backspace, leading to invalid patterns or silent failures.

**Q4: How do backreferences work in Python regex?**
> Backreferences allow you to match the exact same text that was captured by a previous group. Inside a pattern, you reference groups by index using `\1`, `\2`, etc., or by name using `(?P=name)`. In replacement strings with `re.sub()`, you reference them as `\1` or `\g<name>`.

**Q5: How can you write comments inside a complex regex pattern to make it readable?**
> By using the `re.VERBOSE` (or `re.X`) flag. This tells the regex parser to ignore unescaped whitespace and comments starting with `#`, allowing you to write multi-line, self-documenting regex.
> ```python
> pattern = re.compile(r"""
>     ^\(?\d{3}\)? # Area code
>     [-.\s]?      # Optional separator
>     \d{3}        # First 3 digits
>     [-.\s]?      # Optional separator
>     \d{4}$       # Last 4 digits
> """, re.VERBOSE)
> ```

---

*← [Previous: Chapter 9 (Collections & Dataclasses)](09_Collections_Dataclasses.md) | [Next: Chapter 11 (Logging) →](11_Logging.md)*
