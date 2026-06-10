# Chapter 1: Python Introduction

---

## 1.1 What is Python?

### Definition
Python is a **high-level, interpreted, general-purpose programming language** created by **Guido van Rossum** and first released in **1991**. It emphasizes code readability with its use of significant indentation and supports multiple programming paradigms including procedural, object-oriented, and functional programming.

### Why It Exists
Python was created to bridge the gap between shell scripting (too simple for complex tasks) and C/C++ (too complex for rapid development). Guido van Rossum wanted a language that was:
- Easy to read (like English)
- Powerful enough for real applications
- Fun to use (named after *Monty Python's Flying Circus*, not the snake)

### Where It Is Used
- **Web Development** — Django, Flask, FastAPI
- **Data Science & Analytics** — Pandas, NumPy, Matplotlib
- **Machine Learning / AI** — TensorFlow, PyTorch, scikit-learn
- **Automation & Scripting** — DevOps, system administration
- **Scientific Computing** — SciPy, SymPy
- **Game Development** — Pygame
- **Desktop Applications** — Tkinter, PyQt
- **API Development** — FastAPI, Flask
- **AI Agents & LLM Apps** — LangChain, LlamaIndex

### How It Works Internally
1. You write `.py` source code.
2. The CPython interpreter compiles it to **bytecode** (`.pyc` files in `__pycache__`).
3. The **Python Virtual Machine (PVM)** executes the bytecode line by line.
4. This happens transparently — you never manually compile.

### Real-World Analogy
Think of Python as a **universal translator**. You speak in natural-language-like syntax, and the Python interpreter translates your instructions into machine operations, just like a human translator converts English to another language in real time.

### Memory Behavior
- Python manages memory automatically via **reference counting** and **garbage collection**.
- Objects are allocated on the **private heap**.
- The programmer has no direct access to the memory allocator; Python's memory manager handles it.

### Important Rules
1. Python is **case-sensitive**: `Name` and `name` are different.
2. **Indentation** is mandatory — it defines code blocks (no curly braces).
3. Python uses **dynamic typing** — no need to declare variable types.
4. Python files have the `.py` extension.
5. The standard interpreter is **CPython** (written in C).

### Common Mistakes
| Mistake | Why It's Wrong |
|---------|---------------|
| Thinking Python is "just a scripting language" | Python powers Instagram, YouTube, Dropbox at massive scale |
| Confusing Python 2 and Python 3 | Python 2 reached EOL on Jan 1, 2020. Always use Python 3 |
| Thinking interpreted = slow for everything | NumPy, TensorFlow use C/C++ under the hood |
| Ignoring indentation | `IndentationError` is the most common beginner error |

### Interview Questions

**Q1: What is Python and why is it popular?**
> Python is a high-level, interpreted, dynamically-typed language known for its readability and vast ecosystem. It's popular because of its simplicity, extensive standard library, strong community, and dominance in AI/ML, web development, and automation.

**Q2: Is Python compiled or interpreted?**
> Both. Python source code is first **compiled** to bytecode (`.pyc`), then this bytecode is **interpreted** by the Python Virtual Machine (PVM). So technically, Python is a "compiled-then-interpreted" language.

**Q3: What is the difference between CPython, Jython, PyPy, and IronPython?**
> - **CPython**: Reference implementation written in C (most common)
> - **Jython**: Python implemented in Java, runs on JVM
> - **PyPy**: Python implemented in Python, uses JIT compilation for speed
> - **IronPython**: Python implemented in C#, runs on .NET CLR

**Q4: Why is Python called a "glue language"?**
> Because Python can easily integrate with C, C++, Java, and other languages. Libraries like `ctypes`, `cffi`, and tools like SWIG allow Python to "glue" different systems together.

**Q5: Name 5 companies that use Python in production.**
> Google, Instagram (Meta), Netflix, Spotify, Dropbox, NASA, Reddit, Stripe, Bloomberg, JPMorgan.

### Memory Tips
- **"Python = Simplicity + Power"** — Easy syntax, industrial-strength applications.
- **"GIL = Global Interpreter Lock"** — Remember Python has a GIL (more in Phase 3).
- **".py → .pyc → PVM"** — The execution pipeline in three steps.

---

## 1.2 Features of Python

### Definition
Python's features are the design characteristics that make it suitable for rapid development and diverse applications.

### Complete Feature List

| Feature | Description |
|---------|-------------|
| **Easy to Learn** | Clean, readable syntax close to English |
| **Interpreted** | Executed line by line (after bytecode compilation) |
| **Dynamically Typed** | Variable types determined at runtime |
| **Strongly Typed** | No implicit type coercion between incompatible types |
| **Object-Oriented** | Everything in Python is an object |
| **Functional** | Supports lambda, map, filter, reduce |
| **Cross-Platform** | Runs on Windows, macOS, Linux without modification |
| **Extensive Standard Library** | "Batteries included" — 200+ modules |
| **Open Source** | Free to use, modify, and distribute (PSF License) |
| **Embeddable** | Can be embedded in C/C++ programs |
| **Extensible** | Can call C/C++ code for performance-critical sections |
| **Large Community** | Massive ecosystem of third-party packages (PyPI) |
| **Garbage Collected** | Automatic memory management |
| **Supports Multiple Paradigms** | Procedural, OOP, Functional |

### Why These Features Matter for Interviews
Interviewers often ask "Why Python?" to test if you understand language trade-offs:
- **Dynamic typing** → Faster development, but potential runtime errors
- **Interpreted** → No compilation step, but generally slower than C++
- **GIL** → Thread-safe reference counting, but limits true parallelism
- **Strongly typed** → `"5" + 5` raises `TypeError`, unlike JavaScript

### Interview Questions

**Q1: What does "Python is strongly typed" mean?**
> Python doesn't perform implicit type conversions between incompatible types. `"hello" + 5` raises a `TypeError`. Compare with JavaScript where `"5" + 3` gives `"53"`.

**Q2: What does "batteries included" mean?**
> Python's standard library includes modules for file I/O, networking, JSON parsing, regular expressions, databases, HTTP, and more — without needing to install external packages.

**Q3: Is Python suitable for large-scale applications?**
> Yes. Instagram serves 2+ billion users with a Python/Django backend. YouTube, Dropbox, and Reddit all use Python extensively. Performance-critical paths can be optimized with C extensions.

---

## 1.3 Applications of Python

### Where Python Dominates

```
┌─────────────────────────────────────────────────────────┐
│                   Python Applications                    │
├──────────────────┬──────────────────────────────────────┤
│ Web Development  │ Django, Flask, FastAPI                │
│ Data Science     │ Pandas, NumPy, Matplotlib, Seaborn   │
│ Machine Learning │ scikit-learn, TensorFlow, PyTorch     │
│ Deep Learning    │ Keras, PyTorch, JAX                   │
│ NLP              │ spaCy, NLTK, Hugging Face             │
│ Computer Vision  │ OpenCV, PIL/Pillow                    │
│ Automation       │ Selenium, Beautiful Soup, Scrapy      │
│ DevOps           │ Ansible, Fabric, SaltStack            │
│ API Development  │ FastAPI, Flask-RESTful, DRF           │
│ Desktop GUI      │ Tkinter, PyQt, Kivy                   │
│ Game Development │ Pygame, Panda3D                       │
│ Finance          │ QuantLib, Zipline, Backtrader          │
│ IoT              │ MicroPython, CircuitPython             │
│ AI Agents        │ LangChain, AutoGen, CrewAI             │
│ Cybersecurity    │ Scapy, Nmap scripting                  │
└──────────────────┴──────────────────────────────────────┘
```

---

## 1.4 Python Execution Process

### Definition
The Python execution process describes how source code is transformed into machine-executable instructions.

### How It Works Internally (Step by Step)

```
Source Code (.py)
       │
       ▼
┌──────────────┐
│   Lexer      │  → Breaks code into tokens
│  (Tokenizer) │
└──────┬───────┘
       │
       ▼
┌──────────────┐
│   Parser     │  → Builds Abstract Syntax Tree (AST)
└──────┬───────┘
       │
       ▼
┌──────────────┐
│  Compiler    │  → Generates bytecode (.pyc)
└──────┬───────┘
       │
       ▼
┌──────────────┐
│    PVM       │  → Executes bytecode instruction by instruction
│ (Interpreter)│
└──────────────┘
```

### Detailed Steps:

1. **Lexical Analysis (Tokenization):**
   The source code is broken into tokens — keywords, identifiers, operators, literals.
   ```python
   x = 5 + 3
   # Tokens: NAME('x'), OP('='), NUMBER('5'), OP('+'), NUMBER('3')
   ```

2. **Parsing (AST Generation):**
   Tokens are organized into an Abstract Syntax Tree representing the program's structure.
   ```python
   import ast
   tree = ast.parse("x = 5 + 3")
   print(ast.dump(tree, indent=2))
   ```

3. **Compilation to Bytecode:**
   The AST is compiled into bytecode — a low-level, platform-independent representation.
   ```python
   import dis
   def example():
       x = 5 + 3
   dis.dis(example)
   # Output:
   #   LOAD_CONST     8 (5+3 is optimized at compile time!)
   #   STORE_FAST     0 (x)
   ```

4. **Execution by PVM:**
   The Python Virtual Machine reads each bytecode instruction and executes it using a **stack-based** virtual machine.

### Important Rules
- `.pyc` files are cached in `__pycache__/` for faster subsequent runs.
- If the source `.py` file changes, Python automatically recompiles.
- The bytecode is **not** machine code — it's an intermediate representation.
- You can view bytecode using the `dis` module.

### Interview Questions

**Q1: What happens when you run `python script.py`?**
> The CPython interpreter tokenizes the source code, parses it into an AST, compiles the AST to bytecode, and then the PVM executes the bytecode. The bytecode may be cached as `.pyc` in `__pycache__/`.

**Q2: Where are .pyc files stored?**
> In the `__pycache__/` directory, named as `module.cpython-{version}.pyc` (e.g., `module.cpython-311.pyc`).

**Q3: Can you distribute .pyc files without source code?**
> Yes, but it's not true compilation. The bytecode can be decompiled. For actual protection, consider Cython or other tools.

---

## 1.5 Interpreter vs Compiler

### Definition
- **Compiler**: Translates the entire source code into machine code **before** execution (C, C++, Rust, Go).
- **Interpreter**: Translates and executes source code **line by line** at runtime (Python, Ruby, JavaScript).

### Comparison Table

| Feature | Compiler | Interpreter |
|---------|----------|-------------|
| Translation | Entire program at once | Line by line |
| Output | Machine code / executable | No separate executable |
| Speed of Execution | Faster (pre-compiled) | Slower (runtime translation) |
| Error Detection | All errors at compile time | Stops at first error |
| Debugging | Harder (need to recompile) | Easier (immediate feedback) |
| Memory | Generates executable file | No intermediate file (usually) |
| Examples | C, C++, Go, Rust | Python, Ruby, JavaScript |

### Where Python Fits
Python is **hybrid**: it **compiles** to bytecode (like Java), then **interprets** the bytecode via PVM. This gives:
- Development speed of interpreted languages
- Some optimization benefits of compilation (constant folding, etc.)

### Real-World Analogy
- **Compiler** = Translating an entire book before reading it
- **Interpreter** = Having a translator read each sentence aloud in real-time

### Interview Questions

**Q1: Is Python interpreted or compiled?**
> Python is both. Source code is compiled to bytecode (`.pyc`), which is then interpreted by the PVM. The compilation step is implicit and happens automatically.

**Q2: Why is Python slower than C?**
> 1. Dynamic typing requires runtime type checks.
> 2. Bytecode interpretation adds overhead vs. native machine code.
> 3. The GIL limits true multi-threaded parallelism.
> 4. Automatic memory management (garbage collection) has overhead.

---

## 1.6 Python Virtual Machine (PVM)

### Definition
The PVM is the **runtime engine** of Python. It is the last step of the Python execution process — it reads bytecode instructions and executes them one by one.

### How It Works Internally
- The PVM is a **stack-based virtual machine**.
- It maintains a **call stack** for function calls.
- Each bytecode instruction pushes/pops values from the **evaluation stack**.
- It manages **namespaces** (local, global, built-in) for variable lookup.

### Key Components:
1. **Evaluation Stack** — Where operations happen
2. **Frame Objects** — One per function call, holds local variables
3. **Code Objects** — Contains the bytecode, constants, variable names
4. **Block Stack** — Manages loops, try/except blocks

### Interview Questions

**Q1: What is the PVM?**
> The Python Virtual Machine is the runtime component that executes Python bytecode. It's a stack-based interpreter that reads bytecode instructions, manipulates an evaluation stack, and manages execution frames.

**Q2: Is the PVM the same as the JVM?**
> Conceptually similar (both execute bytecode), but the JVM is more optimized with JIT compilation. CPython's PVM is a simple bytecode interpreter without JIT (PyPy adds JIT to Python).

---

## 1.7 Bytecode

### Definition
Bytecode is the **intermediate, low-level, platform-independent** representation of Python source code that the PVM can execute.

### How It Works
```python
# View bytecode with the dis module
import dis

def greet(name):
    return "Hello, " + name

dis.dis(greet)
```

**Output:**
```
  2           0 LOAD_CONST               1 ('Hello, ')
              2 LOAD_FAST                0 (name)
              4 BINARY_ADD
              6 RETURN_VALUE
```

### Key Bytecode Instructions
| Instruction | Meaning |
|-------------|---------|
| `LOAD_CONST` | Push a constant onto the stack |
| `LOAD_FAST` | Push a local variable onto the stack |
| `STORE_FAST` | Pop stack top and store in local variable |
| `BINARY_ADD` | Pop two values, add them, push result |
| `CALL_FUNCTION` | Call a function |
| `RETURN_VALUE` | Return the top of the stack |
| `COMPARE_OP` | Compare two values |
| `POP_JUMP_IF_FALSE` | Conditional jump |

### Why Understanding Bytecode Matters
1. **Performance**: Knowing what bytecode Python generates helps you write faster code.
2. **Debugging**: Understanding execution at the bytecode level helps debug tricky issues.
3. **Interviews**: Advanced interviews may ask about Python internals.

### Interview Questions

**Q1: What is Python bytecode?**
> Bytecode is the compiled, platform-independent intermediate code that Python generates from source code. It's stored in `.pyc` files and executed by the PVM. Each instruction is a single byte opcode with optional arguments.

**Q2: How can you view bytecode?**
> Using the `dis` module: `import dis; dis.dis(function_name)`. You can also use `compile()` to get code objects and inspect their `co_code` attribute.

---

## Code Examples

### Easy Examples

**Example 1: Check Python version**
```python
import sys
print(sys.version)
# Output: 3.11.5 (or your installed version)
```

**Example 2: Hello World**
```python
print("Hello, World!")
# Output: Hello, World!
```

**Example 3: View Python's path**
```python
import sys
print(sys.executable)
# Output: /usr/bin/python3 (or your Python path)
```

**Example 4: Check if Python is 64-bit**
```python
import struct
print(struct.calcsize("P") * 8, "bit")
# Output: 64 bit
```

**Example 5: View Python's Zen**
```python
import this
# Output: The Zen of Python, by Tim Peters
# Beautiful is better than ugly...
```

### Medium Examples

**Example 6: Inspect bytecode**
```python
import dis

def add(a, b):
    return a + b

dis.dis(add)
# Shows bytecode instructions for the add function
```

**Example 7: View the Abstract Syntax Tree**
```python
import ast

code = "x = [1, 2, 3]"
tree = ast.parse(code)
print(ast.dump(tree, indent=2))
```

**Example 8: Check platform information**
```python
import platform

print(f"System: {platform.system()}")
print(f"Release: {platform.release()}")
print(f"Python Implementation: {platform.python_implementation()}")
print(f"Python Version: {platform.python_version()}")
```

**Example 9: Compile code to code object**
```python
code_string = "x = 5 + 3; print(x)"
code_object = compile(code_string, "<string>", "exec")
print(code_object.co_consts)  # Constants used in the code
exec(code_object)  # Output: 8
```

**Example 10: View imported module's file location**
```python
import os
print(os.__file__)
# Shows where the os module is located on disk
```

### Advanced Examples

**Example 11: Explore code object attributes**
```python
def calculate(a, b, c=10):
    result = a + b + c
    return result

code = calculate.__code__
print(f"Name: {code.co_name}")
print(f"Arg count: {code.co_argcount}")
print(f"Local variables: {code.co_varnames}")
print(f"Constants: {code.co_consts}")
print(f"Bytecode: {code.co_code}")
print(f"Stack size: {code.co_stacksize}")
```

**Example 12: Tokenize Python source code**
```python
import tokenize
import io

code = "x = 5 + 3"
tokens = tokenize.generate_tokens(io.StringIO(code).readline)
for tok in tokens:
    print(tok)
# Shows each token: type, string, start, end, line
```

**Example 13: Measure bytecode instruction count**
```python
import dis

def loop_example():
    total = 0
    for i in range(100):
        total += i
    return total

instructions = list(dis.get_instructions(loop_example))
print(f"Number of bytecode instructions: {len(instructions)}")
for instr in instructions:
    print(f"  {instr.opname:20} {instr.argrepr}")
```

**Example 14: Compare bytecode of two equivalent functions**
```python
import dis

def sum_loop(n):
    total = 0
    for i in range(n):
        total += i
    return total

def sum_builtin(n):
    return sum(range(n))

print("=== Loop version ===")
dis.dis(sum_loop)
print("\n=== Built-in version ===")
dis.dis(sum_builtin)
# The built-in version has fewer bytecode instructions
```

**Example 15: Inspect __pycache__ mechanism**
```python
import py_compile
import os

# Manually compile a Python file to bytecode
# py_compile.compile('example.py')

# Check importlib for cache details
import importlib.util
print(f"Cache tag: {importlib.util.MAGIC_NUMBER.hex()}")
print(f"Source suffixes: {importlib.machinery.SOURCE_SUFFIXES}")
print(f"Bytecode suffixes: {importlib.machinery.BYTECODE_SUFFIXES}")
```

---

## Edge Cases

1. **Running `.pyc` without `.py`**: Python can execute `.pyc` files directly, but they are version-specific.
2. **Syntax errors prevent bytecode generation**: If your code has a `SyntaxError`, no `.pyc` file is created.
3. **`__pycache__` permissions**: On shared systems, you might not have write permissions to create `.pyc` files. Python will still run but won't cache.
4. **Frozen modules**: Some standard library modules are "frozen" — their bytecode is embedded directly in the Python interpreter binary.
5. **Optimized bytecode**: Running `python -O` removes `assert` statements; `python -OO` also removes docstrings.

---

## Comparison with Similar Concepts

| Aspect | Python | Java | JavaScript | C |
|--------|--------|------|------------|---|
| Type System | Dynamic, Strong | Static, Strong | Dynamic, Weak | Static, Weak |
| Compilation | To bytecode | To bytecode | JIT compiled | To machine code |
| VM | PVM (CPython) | JVM | V8/SpiderMonkey | None (native) |
| Memory Management | Automatic (GC) | Automatic (GC) | Automatic (GC) | Manual |
| Speed | Moderate | Fast | Fast (JIT) | Very Fast |
| Typing | Duck typing | Nominal typing | Duck typing | Nominal typing |

---

## Practice Questions

### Easy

**Q1:** Write a program to print "Python is amazing!" and then print the Python version.
```python
# Answer:
import sys
print("Python is amazing!")
print(f"Version: {sys.version}")
```

**Q2:** Write a program that prints the platform (operating system) you're running Python on.
```python
# Answer:
import platform
print(f"Operating System: {platform.system()}")
print(f"Architecture: {platform.architecture()[0]}")
```

### Medium

**Q3:** Write a program that uses the `dis` module to display the bytecode of a simple function that multiplies two numbers.
```python
# Answer:
import dis

def multiply(a, b):
    return a * b

dis.dis(multiply)
# Expected output shows LOAD_FAST, BINARY_MULTIPLY, RETURN_VALUE instructions
```

**Q4:** Write a program that uses the `ast` module to count the number of assignment statements in a code string.
```python
# Answer:
import ast

code = """
x = 5
y = 10
z = x + y
name = "Python"
"""

tree = ast.parse(code)
assignments = [node for node in ast.walk(tree) if isinstance(node, ast.Assign)]
print(f"Number of assignments: {len(assignments)}")
# Output: Number of assignments: 4
```

### Advanced

**Q5:** Write a program that compares the number of bytecode instructions between a list comprehension and an equivalent for loop. Which is more efficient?
```python
# Answer:
import dis

def using_loop():
    result = []
    for i in range(10):
        result.append(i * 2)
    return result

def using_comprehension():
    return [i * 2 for i in range(10)]

loop_instructions = list(dis.get_instructions(using_loop))
comp_instructions = list(dis.get_instructions(using_comprehension))

print(f"Loop: {len(loop_instructions)} instructions")
print(f"Comprehension: {len(comp_instructions)} instructions")
# Comprehension typically has fewer instructions and runs faster
# because it avoids the LOAD_ATTR + CALL_FUNCTION for .append()
```

**Q6:** Explain what happens internally when you type `python -c "print(1 + 2)"` and press Enter. Write code to demonstrate each stage.
```python
# Answer:
# Stage 1: Tokenization
import tokenize, io
code = "print(1 + 2)"
tokens = list(tokenize.generate_tokens(io.StringIO(code).readline))
print("TOKENS:")
for t in tokens:
    print(f"  {t}")

# Stage 2: AST
import ast
tree = ast.parse(code)
print("\nAST:")
print(ast.dump(tree, indent=2))

# Stage 3: Bytecode
code_obj = compile(code, "<string>", "exec")
import dis
print("\nBYTECODE:")
dis.dis(code_obj)

# Stage 4: Execution
print("\nEXECUTION:")
exec(code_obj)  # Output: 3

# Explanation:
# 1. Lexer breaks "print(1 + 2)" into tokens: NAME, OP, NUMBER, OP, NUMBER, OP
# 2. Parser creates an AST with a Call node (func=print, args=[BinOp(1+2)])
# 3. Compiler generates bytecode: LOAD_NAME(print), LOAD_CONST(3), CALL_FUNCTION
#    Note: 1+2 is constant-folded to 3 at compile time!
# 4. PVM executes the bytecode, calling print(3)
```

---

*Next Chapter: [Variables →](02_Variables.md)*
