# Chapter 20: File Handling

---

## Topic Overview

| Aspect | Details |
|--------|---------|
| **Definition** | File handling allows Python programs to **create, read, write, and manipulate files** on the filesystem. |
| **Why It Exists** | Programs need to persist data, read configurations, process data files, generate reports. |
| **Where It Is Used** | Data pipelines, logging, configuration, CSV/JSON processing, ML datasets, web scraping output. |

---

## 20.1 open() Function

```python
# Syntax: open(file, mode='r', encoding=None, ...)
# Returns a file object

# Always use 'with' statement (context manager) — auto-closes file
with open("data.txt", "r") as f:
    content = f.read()
```

## File Modes

| Mode | Description | Creates file? | Overwrites? |
|------|-------------|---------------|-------------|
| `'r'` | Read (default) | ❌ | — |
| `'w'` | Write | ✅ | ✅ Truncates! |
| `'a'` | Append | ✅ | ❌ Adds to end |
| `'x'` | Exclusive create | ✅ | Error if exists |
| `'r+'` | Read + Write | ❌ | No truncation |
| `'w+'` | Write + Read | ✅ | ✅ Truncates! |
| `'a+'` | Append + Read | ✅ | ❌ Adds to end |
| `'b'` | Binary mode (add to above) | — | — |
| `'t'` | Text mode (default) | — | — |

```python
# Common combinations:
# 'rb'  — Read binary (images, PDFs)
# 'wb'  — Write binary
# 'rt'  — Read text (default, same as 'r')
```

---

## 20.2 Reading Files

```python
# read() — entire file as string
with open("data.txt", "r") as f:
    content = f.read()
    print(content)

# read(n) — read n characters
with open("data.txt", "r") as f:
    first_10 = f.read(10)

# readline() — one line at a time
with open("data.txt", "r") as f:
    line1 = f.readline()   # First line (includes \n)
    line2 = f.readline()   # Second line

# readlines() — all lines as list
with open("data.txt", "r") as f:
    lines = f.readlines()  # ['line1\n', 'line2\n', ...]

# Iterate line by line (BEST — memory efficient)
with open("data.txt", "r") as f:
    for line in f:
        print(line.strip())  # strip() removes trailing \n

# Read with encoding
with open("data.txt", "r", encoding="utf-8") as f:
    content = f.read()
```

---

## 20.3 Writing Files

```python
# write() — write string
with open("output.txt", "w") as f:
    f.write("Hello, World!\n")
    f.write("Second line\n")

# writelines() — write list of strings (no automatic newlines!)
lines = ["Line 1\n", "Line 2\n", "Line 3\n"]
with open("output.txt", "w") as f:
    f.writelines(lines)

# print() to file
with open("output.txt", "w") as f:
    print("Hello!", file=f)
    print("World!", file=f)
```

---

## 20.4 Appending

```python
with open("log.txt", "a") as f:
    f.write(f"[{datetime.now()}] Event occurred\n")
# Adds to end of file, creates if doesn't exist
```

---

## 20.5 The with Statement (Context Manager)

```python
# WITH (recommended — auto-closes even on exception)
with open("file.txt", "r") as f:
    data = f.read()
# f is automatically closed here

# WITHOUT (manual — error-prone)
f = open("file.txt", "r")
try:
    data = f.read()
finally:
    f.close()  # Must remember to close!

# Multiple files
with open("input.txt", "r") as fin, open("output.txt", "w") as fout:
    for line in fin:
        fout.write(line.upper())
```

---

## 20.6 File Object Methods

```python
with open("data.txt", "r+") as f:
    print(f.name)       # 'data.txt'
    print(f.mode)       # 'r+'
    print(f.closed)     # False
    print(f.readable()) # True
    print(f.writable()) # True
    print(f.tell())     # Current position (0 initially)
    
    f.read(5)           # Read 5 chars
    print(f.tell())     # Now at position 5
    
    f.seek(0)           # Move back to beginning
    print(f.tell())     # 0

print(f.closed)  # True (after with block)
```

---

## Working with Different File Types

### CSV
```python
import csv

# Write CSV
with open("data.csv", "w", newline="") as f:
    writer = csv.writer(f)
    writer.writerow(["Name", "Age", "City"])
    writer.writerow(["Alice", 25, "NYC"])
    writer.writerow(["Bob", 30, "LA"])

# Read CSV
with open("data.csv", "r") as f:
    reader = csv.reader(f)
    header = next(reader)
    for row in reader:
        print(row)

# DictReader/DictWriter
with open("data.csv", "r") as f:
    reader = csv.DictReader(f)
    for row in reader:
        print(f"{row['Name']} is {row['Age']}")
```

### JSON
```python
import json

# Write JSON
data = {"name": "Alice", "age": 25, "scores": [90, 85, 92]}
with open("data.json", "w") as f:
    json.dump(data, f, indent=2)

# Read JSON
with open("data.json", "r") as f:
    loaded = json.load(f)
    print(loaded["name"])
```

### Binary Files
```python
# Copy binary file (image, PDF, etc.)
with open("source.jpg", "rb") as src, open("copy.jpg", "wb") as dst:
    dst.write(src.read())

# Read in chunks (for large files)
with open("large_file.bin", "rb") as f:
    while chunk := f.read(8192):  # 8KB chunks
        process(chunk)
```

---

## pathlib (Modern File Path Handling)

```python
from pathlib import Path

# Create path objects
p = Path("data") / "output" / "results.txt"
print(p)  # data/output/results.txt

# Check existence
print(p.exists())
print(p.is_file())
print(p.is_dir())

# Read/write (convenience methods)
p.write_text("Hello, World!")
content = p.read_text()
data = p.read_bytes()

# Directory operations
p.parent.mkdir(parents=True, exist_ok=True)

# List directory
for file in Path(".").glob("*.py"):
    print(file.name)

# Recursive glob
for file in Path(".").rglob("*.txt"):
    print(file)

# File info
print(p.stem)      # 'results' (name without extension)
print(p.suffix)    # '.txt'
print(p.name)      # 'results.txt'
print(p.parent)    # data/output
```

---

## Common Mistakes

| Mistake | Why It's Wrong | Fix |
|---------|---------------|-----|
| Not using `with` | File may not close on error | Always use `with` |
| `'w'` instead of `'a'` | Overwrites entire file | Use `'a'` to append |
| Forgetting `newline=""` in CSV | Extra blank lines on Windows | `open(f, "w", newline="")` |
| Reading large file with `read()` | Loads entire file into memory | Iterate line by line |
| Not specifying encoding | Platform-dependent default | Use `encoding="utf-8"` |

---

## Interview Questions

**Q1: What is the difference between `read()`, `readline()`, and `readlines()`?**
> `read()` returns entire file as one string. `readline()` returns one line. `readlines()` returns a list of all lines. For large files, iterate directly: `for line in f:`.

**Q2: Why use `with` statement for files?**
> It guarantees the file is closed even if an exception occurs. It calls `__enter__` and `__exit__` methods automatically.

**Q3: What is the difference between text mode and binary mode?**
> Text mode (`'t'`) handles encoding/decoding and line ending translation. Binary mode (`'b'`) reads/writes raw bytes without any processing.

---

## Practice Questions

### Easy
**Q1:** Write a program to create a file, write 5 lines, then read and display them.
```python
with open("test.txt", "w") as f:
    for i in range(1, 6):
        f.write(f"Line {i}\n")

with open("test.txt", "r") as f:
    print(f.read())
```

**Q2:** Count the number of lines, words, and characters in a file.
```python
with open("test.txt", "r") as f:
    content = f.read()
    lines = content.count("\n")
    words = len(content.split())
    chars = len(content)
    print(f"Lines: {lines}, Words: {words}, Chars: {chars}")
```

### Medium
**Q3:** Copy a file line by line, converting each line to uppercase.
```python
with open("input.txt", "r") as fin, open("output.txt", "w") as fout:
    for line in fin:
        fout.write(line.upper())
```

**Q4:** Implement a simple log function that appends timestamped messages.
```python
from datetime import datetime

def log(message, filename="app.log"):
    timestamp = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
    with open(filename, "a") as f:
        f.write(f"[{timestamp}] {message}\n")

log("Application started")
log("User logged in")
```

### Advanced
**Q5:** Write a function that reads a large file in chunks and counts word frequency.
```python
from collections import Counter

def word_freq_large_file(filepath, chunk_size=8192):
    counter = Counter()
    with open(filepath, "r", encoding="utf-8") as f:
        leftover = ""
        while True:
            chunk = f.read(chunk_size)
            if not chunk:
                if leftover:
                    counter.update(leftover.lower().split())
                break
            text = leftover + chunk
            words = text.split()
            leftover = words[-1] if not chunk.endswith((" ", "\n")) else ""
            if leftover:
                words = words[:-1]
            counter.update(w.lower() for w in words)
    return counter.most_common(10)
```

**Q6:** Implement a file-based key-value store.
```python
import json, os

class FileKVStore:
    def __init__(self, filepath):
        self.filepath = filepath
        self._load()
    
    def _load(self):
        if os.path.exists(self.filepath):
            with open(self.filepath, "r") as f:
                self._data = json.load(f)
        else:
            self._data = {}
    
    def _save(self):
        with open(self.filepath, "w") as f:
            json.dump(self._data, f, indent=2)
    
    def get(self, key, default=None):
        return self._data.get(key, default)
    
    def set(self, key, value):
        self._data[key] = value
        self._save()
    
    def delete(self, key):
        self._data.pop(key, None)
        self._save()

# store = FileKVStore("config.json")
# store.set("theme", "dark")
# print(store.get("theme"))
```

---

*← [Previous: Exception Handling](19_Exception_Handling.md) | [Next: Built-in Functions →](21_Builtin_Functions.md)*
