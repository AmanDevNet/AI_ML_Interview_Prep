# Chapter 18: Packages

---

## Topic Overview

| Aspect | Details |
|--------|---------|
| **Definition** | A package is a **directory containing Python modules** and an `__init__.py` file, enabling hierarchical module organization. |
| **Why It Exists** | Organizes large codebases into logical namespaces, prevents name conflicts, enables distribution. |
| **Where It Is Used** | Every Python project beyond a single file — Django apps, ML frameworks, utility libraries. |

---

## Package Structure

```
my_project/
├── __init__.py          # Makes this a package
├── module_a.py
├── module_b.py
├── subpackage/
│   ├── __init__.py      # Makes this a sub-package
│   ├── module_c.py
│   └── module_d.py
└── tests/
    ├── __init__.py
    └── test_module_a.py
```

## __init__.py

```python
# __init__.py can be:
# 1. Empty — just marks directory as a package
# 2. Contains initialization code
# 3. Controls what's exported with __all__

# my_package/__init__.py
from .module_a import func_a
from .module_b import ClassB

__all__ = ["func_a", "ClassB"]  # Controls 'from package import *'
__version__ = "1.0.0"
```

## Importing from Packages

```python
# Absolute imports
import my_package.module_a
from my_package import module_a
from my_package.module_a import some_function
from my_package.subpackage.module_c import ClassC

# Relative imports (inside a package)
# From module_a.py:
from . import module_b          # Same package
from .module_b import func      # Same package, specific item
from ..subpackage import module_c  # Parent package, then into subpackage
```

## Namespace Packages (Python 3.3+)

```python
# Packages WITHOUT __init__.py
# Allow splitting a package across multiple directories
# Used by some large frameworks for plugin systems
# Regular packages (with __init__.py) are preferred for most use cases
```

---

## Interview Questions

**Q1: What is `__init__.py`?**
> It marks a directory as a Python package. It's executed when the package is imported. Can be empty or contain initialization code, exports (`__all__`), and version info.

**Q2: What is the difference between a module and a package?**
> A module is a single `.py` file. A package is a directory containing modules and an `__init__.py` file. A package can contain sub-packages (nested directories).

**Q3: What are relative imports?**
> Imports using dots: `.` means current package, `..` means parent package. Only work inside packages, not in scripts run directly.

---

## Practice Questions

### Easy
**Q1:** Create a package structure for a calculator app with `add`, `subtract` modules.
```
calculator/
├── __init__.py     # from .add import add_numbers
├── add.py          # def add_numbers(a, b): return a + b
└── subtract.py     # def subtract_numbers(a, b): return a - b
```

**Q2:** What does `from package import *` import?
```python
# It imports names listed in __all__ (if defined in __init__.py)
# If __all__ is not defined, it imports all public names (not starting with _)
```

### Medium
**Q3:** Write a package with proper `__init__.py` that exposes a clean API.
```python
# mylib/__init__.py
from .core import process
from .utils import validate
from .models import DataModel

__all__ = ["process", "validate", "DataModel"]
__version__ = "2.0.0"

# Usage: from mylib import process, validate
```

**Q4:** Explain absolute vs relative imports with examples.
```python
# Absolute (always works, more explicit):
from mypackage.utils import helper

# Relative (only in packages, shorter):
from .utils import helper      # same directory
from ..core import engine       # parent directory

# Rule: Use absolute for top-level scripts, relative within packages
```

### Advanced
**Q5:** Create a plugin system using packages.
```python
import importlib
import os

def load_plugins(plugin_dir):
    plugins = {}
    for filename in os.listdir(plugin_dir):
        if filename.endswith('.py') and not filename.startswith('_'):
            module_name = filename[:-3]
            module = importlib.import_module(f"plugins.{module_name}")
            if hasattr(module, 'register'):
                plugins[module_name] = module.register()
    return plugins
```

**Q6:** Explain how `sys.path` affects package imports and demonstrate adding custom paths.
```python
import sys
# sys.path is a list of directories Python searches for modules
# sys.path[0] is usually the script's directory or '' for interactive
print(sys.path)

# Add custom directory
sys.path.insert(0, '/path/to/my/packages')

# Better: Use .pth files in site-packages
# Or: Set PYTHONPATH environment variable
# Best: Install packages properly with pip
```

---

*← [Previous: Modules](17_Modules.md) | [Next: Exception Handling →](19_Exception_Handling.md)*
