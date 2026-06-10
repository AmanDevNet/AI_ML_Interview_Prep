# Chapter 13: Context Managers — "Safe Resource Management"

---

## Topic Overview

| Aspect | Details |
|--------|---------|
| **Definition** | A context manager is an object that defines the runtime context to be established when executing a `with` statement. |
| **Why It Exists** | Code that manages resources (like file streams, database connections, locks) must clean up after itself (e.g. close the file), even if an error/exception occurs during execution. |
| **Where It Is Used** | File operations (`open()`), acquiring/releasing locks (`threading.Lock`), database connections, API client sessions (`aiohttp.ClientSession`), redirecting standard output. |
| **Real-World Analogy** | Borrowing a book from a library. You get the book (setup), read it inside the library, and then return it when you are done (cleanup), regardless of whether you enjoyed reading it or got bored (exceptions). |

---

## Think of It This Way

If you open a file manually without a context manager:
```python
f = open("data.txt", "w")
f.write("Hello")
# If something goes wrong here...
x = 1 / 0  # CRASH!
f.close()  # This line is NEVER reached! The file remains open.
```
Having too many files open at once will eventually cause the operating system to throw an error like `OSError: [Errno 24] Too many open files`.

To prevent this without context managers, you'd write:
```python
f = open("data.txt", "w")
try:
    f.write("Hello")
    x = 1 / 0
finally:
    f.close()  # Guarantees the file is closed no matter what
```
This is correct, but verbose and repetitive. 

**Context Managers** wrap this setup/cleanup logic cleanly using the `with` statement:
```python
with open("data.txt", "w") as f:
    f.write("Hello")
    x = 1 / 0
# The file is closed automatically the instant we exit the block, even during crashes.
```

---

## Step 1: Class-Based Context Managers

You can make any class a context manager by implementing two magic methods:
1. **`__enter__(self)`**: Sets up the resource. The value it returns is assigned to the variable after the `as` keyword.
2. **`__exit__(self, exc_type, exc_val, exc_tb)`**: Runs cleanup code. It receives information about any exception that occurred inside the block.

```python
class ManagedFile:
    def __init__(self, filename, mode):
        self.filename = filename
        self.mode = mode

    def __enter__(self):
        self.file = open(self.filename, self.mode)
        return self.file  # Returned to the 'as' variable

    def __exit__(self, exc_type, exc_val, exc_tb):
        if self.file:
            self.file.close()
        # Returning True suppresses any exception that happened inside the block.
        # Returning False (or None) propagates the exception normally.
        return False 

with ManagedFile("hello.txt", "w") as f:
    f.write("Hello, World!")
```

### Exception Suppression in `__exit__`
```python
class IgnoreZeroDivision:
    def __enter__(self):
        print("Entering...")
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        if exc_type is ZeroDivisionError:
            print("Suppressed division by zero!")
            return True  # Suppress!
        return False     # Let all other exceptions crash normally

with IgnoreZeroDivision():
    x = 1 / 0

print("Code continues running normally!")
```

---

## Step 2: Generator-Based Context Managers (`contextlib`)

Instead of writing a whole class with `__enter__` and `__exit__`, you can write a generator function decorated with `@contextmanager` from the standard `contextlib` library:

```python
from contextlib import contextmanager

@contextmanager
def managed_file(filename, mode):
    # 1. Everything before 'yield' is the __enter__ logic
    f = open(filename, mode)
    try:
        yield f  # 2. Yield the resource to the 'as' variable
    finally:
        # 3. Everything after 'yield' (in finally) is the __exit__ logic
        f.close()

with managed_file("hello.txt", "w") as f:
    f.write("Short code!")
```

---

## Code Examples

### Easy Examples
```python
# 1. Standard open context manager
with open("test.txt", "w") as f:
    f.write("Simple write")

# 2. Multiple context managers in one line (Python 3.10+)
with open("source.txt", "r") as src, open("dest.txt", "w") as dest:
    dest.write(src.read())

# 3. Suppressing expected exceptions with contextlib.suppress
from contextlib import suppress
with suppress(FileNotFoundError):
    # If file doesn't exist, code won't crash
    open("ghost_file.txt", "r")

# 4. Timer using Class-based manager
import time
class Timer:
    def __enter__(self):
        self.start = time.perf_counter()
        return self
    def __exit__(self, *args):
        elapsed = time.perf_counter() - self.start
        print(f"Elapsed time: {elapsed:.4f} seconds")

with Timer():
    time.sleep(0.5)

# 5. Database Connection Mock
class DBConnection:
    def __enter__(self):
        print("Open Connection")
        return self
    def execute(self, query):
        print(f"Executing: {query}")
    def __exit__(self, exc_type, exc_val, exc_tb):
        print("Close Connection")

with DBConnection() as db:
    db.execute("SELECT * FROM users")
```

### Medium Examples
```python
# 6. Redirecting stdout to a file
import sys
@contextmanager
def redirect_stdout(file_object):
    original_stdout = sys.stdout
    sys.stdout = file_object  # Redirect
    try:
        yield
    finally:
        sys.stdout = original_stdout  # Restore

with open("output.log", "w") as f:
    with redirect_stdout(f):
        print("This goes into the file, not the terminal!")

# 7. Auto-releasing a Lock
from threading import Lock
lock = Lock()

with lock:
    # Safely perform thread-sensitive write
    pass # lock is automatically released here

# 8. Temporary Directory Manager
import shutil
import tempfile

class TempDir:
    def __enter__(self):
        self.path = tempfile.mkdtemp()
        return self.path
    def __exit__(self, *args):
        shutil.rmtree(self.path)

with TempDir() as temp_path:
    print(f"Working in: {temp_path}")

# 9. Context Manager for modifying settings/state
class TemporarySetting:
    def __init__(self, obj, attribute, temp_value):
        self.obj = obj
        self.attribute = attribute
        self.temp_value = temp_value

    def __enter__(self):
        self.original = getattr(self.obj, self.attribute)
        setattr(self.obj, self.attribute, self.temp_value)

    def __exit__(self, *args):
        setattr(self.obj, self.attribute, self.original)

class Server:
    debug = False

server = Server()
with TemporarySetting(server, "debug", True):
    print(f"Server debug status inside block: {server.debug}") # True
print(f"Server debug status outside: {server.debug}")          # False

# 10. contextlib.closing helper
# Converts any class with a close() method into a context manager
from contextlib import closing
class SimpleResource:
    def close(self):
        print("Resource closed!")

with closing(SimpleResource()) as res:
    print("Using resource...")
```

### Advanced Examples
```python
# 11. Custom Database Transaction rollback manager
class Transaction:
    def __init__(self, db_client):
        self.db = db_client

    def __enter__(self):
        self.db.begin()
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        if exc_type is not None:
            print(f"Error detected: {exc_val}. Rolling back transaction!")
            self.db.rollback()
            return False  # Propagate the error to the caller
        else:
            print("No errors. Committing transaction!")
            self.db.commit()
            return True

# Mock DB Client
class DummyDB:
    def begin(self): print("DB: Begin")
    def commit(self): print("DB: Commit")
    def rollback(self): print("DB: Rollback")

db = DummyDB()
try:
    with Transaction(db):
        print("Inserting records...")
        raise ValueError("Database write failed due to validation!")
except ValueError:
    pass # Rollback was already called automatically!

# 12. Nested context managers using ExitStack
# Dynamically open list of files without nesting with statements
from contextlib import ExitStack
filenames = ["a.txt", "b.txt", "c.txt"]

# Open all files dynamically and close them at the end safely
with ExitStack() as stack:
    files = [stack.enter_context(open(name, "w")) for name in filenames]
    for i, f in enumerate(files):
        f.write(f"File contents {i}")

# 13. Context-manager decorated functions
# Using context manager directly as decorator on function
class TagWrapper:
    def __enter__(self):
        print("<html>")
    def __exit__(self, *args):
        print("</html>")

@TagWrapper()
def render():
    print("<body>Hello</body>")

render()
# Output:
# <html>
# <body>Hello</body>
# </html>

# 14. Remote connection tunnel context manager (Mock)
class SSHTunnel:
    def __init__(self, host):
        self.host = host
    def __enter__(self):
        print(f"Opening SSH tunnel to {self.host}")
        return self
    def send_data(self, data):
        print(f"Sending: {data}")
    def __exit__(self, exc_type, exc_val, exc_tb):
        print("Closing SSH tunnel connection.")

with SSHTunnel("192.168.1.1") as tunnel:
    tunnel.send_data("API payload")

# 15. Context manager returning a callable (e.g. Profiler)
class Profiler:
    def __enter__(self):
        self.start = time.perf_counter()
        return lambda: time.perf_counter() - self.start  # Returns a getter function

    def __exit__(self, *args):
        pass

with Profiler() as get_elapsed:
    time.sleep(0.1)
    print(f"Checkpoint 1: {get_elapsed():.4f}")
    time.sleep(0.2)
    print(f"Checkpoint 2: {get_elapsed():.4f}")
```

---

## Common Mistakes & Edge Cases

### 1. Forgetting to return the resource in `__enter__`
If you write `__enter__` but don't explicitly return `self` or a resource, Python returns `None` implicitly. 
```python
# ❌ WRONG
def __enter__(self):
    self.file = open("test.txt", "w")
    # Forgot to return self.file!

with ManagedFile("test.txt", "w") as f:
    f.write("hi")  # AttributeError: 'NoneType' object has no attribute 'write'
```

### 2. Not catching Exceptions inside generator-based managers
In generator context managers (`@contextmanager`), if an exception occurs in the `with` block, it is raised *at the yield line*. If you do not wrap your `yield` in a `try...finally` block, the cleanup code below the yield will **never** run.
```python
# ❌ WRONG
@contextmanager
def file_manager(path):
    f = open(path)
    yield f
    f.close() # Will NOT run if an exception occurs in the with block!
```

### 3. Returning `True` in `__exit__` by mistake
Returning `True` from `__exit__` suppresses *all* exceptions. This can lead to silent errors that are extremely hard to debug because the code doesn't crash on syntax errors or invalid logic inside the `with` block.

---

## Interview Questions (Top 5)

**Q1: What are the three parameters received by the `__exit__` method and what do they represent?**
> The three parameters are `exc_type` (the class of the exception, e.g. `ZeroDivisionError`), `exc_val` (the instance/message of the exception, e.g. `division by zero`), and `exc_tb` (the traceback object containing call stack details). If no exception occurred inside the `with` block, all three parameters are passed as `None`.

**Q2: How do you suppress exceptions inside a custom context manager's `__exit__` method?**
> By returning a truthy value (typically `True`) from the `__exit__` method. If `__exit__` returns `False` or `None`, the exception will propagate up and crash the application unless caught by an outer `try-except` block.

**Q3: What is the purpose of `contextlib.ExitStack`?**
> `ExitStack` allows you to enter a dynamic number of context managers (like opening a dynamic list of files based on user input) in a single block. It acts as a stack, opening the resources as they are added and guaranteeing that all opened resources are closed in reverse order at the end of the block.

**Q4: How do you implement a context manager using a generator instead of a class?**
> By using the `contextlib.contextmanager` decorator on a generator function. The setup phase goes before the `yield` statement, the resource is yielded via `yield resource`, and the cleanup phase is written inside a `finally` block following the `yield`.

**Q5: What happens if an exception is raised inside a `@contextmanager` generator's setup phase (before the `yield`)?**
> The exception propagates immediately. The `yield` is never reached, the code inside the `with` block does not run, and the cleanup code in the `finally` block of the generator does not execute because the generator was never successfully started.

---

*← [Previous: Chapter 12 (Virtual Environments & Requirements)](12_Virtual_Environments_Requirements.md) | [Next: Chapter 14 (Copy Module) →](14_Copy_Module.md)*
