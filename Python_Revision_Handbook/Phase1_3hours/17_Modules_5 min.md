# Chapter 17: Modules

---

## Topic Overview

| Aspect | Details |
|--------|---------|
| **Definition** | A module is a **Python file (.py)** containing definitions (functions, classes, variables) that can be imported into other files. |
| **Why It Exists** | Code organization, reusability, namespace management, encapsulation. |
| **Where It Is Used** | Every non-trivial Python program uses modules — standard library, third-party packages, project organization. |

---

## 17.1 import Statement

```python
# Import entire module
import math
print(math.sqrt(16))    # 4.0
print(math.pi)          # 3.14159...

# Import with alias
import numpy as np
import pandas as pd

# Import specific items
from math import sqrt, pi
print(sqrt(16))          # 4.0 (no prefix needed)

# Import all (avoid in production!)
from math import *       # Imports everything — pollutes namespace

# Import with alias for specific item
from collections import OrderedDict as OD
```

## 17.2 How Imports Work Internally

```python
# When you 'import math':
# 1. Python checks sys.modules cache (already imported?)
# 2. If not cached, Python searches for the module using sys.path
# 3. Module is found → loaded, compiled to bytecode (.pyc), executed
# 4. Module object is stored in sys.modules
# 5. Name is bound in current namespace

import sys

# Check search path
print(sys.path)  # List of directories Python searches

# Check cached modules
print('math' in sys.modules)  # True (after import math)

# Module is executed only ONCE — subsequent imports use cache
```

## 17.3 Creating Your Own Module

```python
# File: myutils.py
"""My utility functions."""

PI = 3.14159

def greet(name):
    return f"Hello, {name}!"

def add(a, b):
    return a + b

class Calculator:
    def multiply(self, a, b):
        return a * b

# Usage in another file:
# import myutils
# print(myutils.greet("Alice"))
# print(myutils.PI)
```

## 17.4 The `__name__` Variable

```python
# Every module has a __name__ attribute
# When run directly: __name__ == "__main__"
# When imported: __name__ == module_name

# File: mymodule.py
def main():
    print("Running as main program")

if __name__ == "__main__":
    # Only runs when file is executed directly, not when imported
    main()
```

## 17.5 Module Search Path

```python
import sys

# Python searches in this order:
# 1. Current directory
# 2. PYTHONPATH environment variable directories
# 3. Installation-dependent defaults (site-packages)

# Add to path at runtime
sys.path.append("/path/to/my/modules")

# Standard library location
import os
print(os.__file__)  # Shows where os module lives
```

---

## Important Standard Library Modules

| Module | Purpose |
|--------|---------|
| `os` | Operating system interface |
| `sys` | System-specific parameters |
| `math` | Mathematical functions |
| `json` | JSON encoding/decoding |
| `datetime` | Date and time |
| `re` | Regular expressions |
| `collections` | Specialized containers |
| `itertools` | Iterator building blocks |
| `functools` | Higher-order functions |
| `pathlib` | Object-oriented paths |
| `typing` | Type hints |
| `logging` | Logging facility |
| `unittest` | Testing framework |
| `copy` | Shallow/deep copying |

---

## Interview Questions

**Q1: What is the difference between `import module` and `from module import func`?**
> `import module` imports the module namespace — access via `module.func()`. `from module import func` imports directly — access via `func()`. The first keeps the namespace clean; the second is more convenient.

**Q2: What is `if __name__ == "__main__":`?**
> It checks if the file is being run directly (not imported). Code inside only executes when the file is the entry point, not when it's imported as a module.

**Q3: Are modules imported multiple times?**
> No. After the first import, the module is cached in `sys.modules`. Subsequent imports use the cache. Use `importlib.reload()` to force re-import.

---

## Practice Questions

### Easy
**Q1:** Import the `math` module and calculate the hypotenuse of a right triangle with sides 3 and 4.
```python
import math
hyp = math.hypot(3, 4)  # or math.sqrt(3**2 + 4**2)
print(f"Hypotenuse: {hyp}")  # 5.0
```

**Q2:** Import `random` and generate a list of 5 random numbers between 1 and 100.
```python
import random
nums = [random.randint(1, 100) for _ in range(5)]
print(nums)
```

### Medium
**Q3:** Write a module that provides a `is_prime` function and a `prime_range` function. Show how to import and use it.
```python
# File: primes.py
def is_prime(n):
    if n < 2: return False
    for i in range(2, int(n**0.5) + 1):
        if n % i == 0: return False
    return True

def prime_range(start, end):
    return [n for n in range(start, end) if is_prime(n)]

if __name__ == "__main__":
    print(prime_range(1, 50))

# Usage: from primes import is_prime, prime_range
```

**Q4:** Demonstrate `importlib.reload()` and explain when it's useful.
```python
import importlib
import mymodule  # First import

# After modifying mymodule.py:
importlib.reload(mymodule)  # Re-reads and re-executes the module
# Useful in interactive development (Jupyter, REPL)
```

### Advanced
**Q5:** Show how `sys.modules` caching works and demonstrate forced reimport.
```python
import sys

# Check if module is cached
import json
print(json.__name__ in sys.modules)  # True

# Remove from cache and reimport
del sys.modules['json']
import json  # Re-imported from scratch
```

**Q6:** Implement a lazy module import that only loads when accessed.
```python
class LazyImport:
    def __init__(self, module_name):
        self._module_name = module_name
        self._module = None
    def __getattr__(self, name):
        if self._module is None:
            import importlib
            self._module = importlib.import_module(self._module_name)
            print(f"Loaded {self._module_name}")
        return getattr(self._module, name)

np = LazyImport("math")  # Not loaded yet!
print(np.sqrt(16))  # "Loaded math" then 4.0
print(np.pi)        # 3.14... (already loaded)
```

---

*← [Previous: Scope](16_Scope.md) | [Next: Packages →](18_Packages.md)*
