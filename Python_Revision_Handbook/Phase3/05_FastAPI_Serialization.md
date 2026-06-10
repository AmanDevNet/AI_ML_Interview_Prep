# Chapter 5: FastAPI & Serialization Internals — "High-Throughput APIs & Data Formats"

---

## Topic Overview

| Aspect | Details |
|--------|---------|
| **Definition** | **FastAPI** is a modern async web framework built on Pydantic. **Serialization** is the process of converting complex objects in memory into a format that can be stored or transmitted (like JSON or byte streams). |
| **Why It Exists** | Traditional sync frameworks block requests during database queries. Standard serialization methods (like native `json`) have significant CPU overhead. Custom pickling is needed for saving complex configurations. |
| **Where It Is Used** | REST APIs, microservices, loading trained ML model checkpoints, cache storage (Redis), and message brokers. |
| **Real-World Analogy** | **FastAPI Depends:** A rental car service where fuel, insurance, and GPS are prepared and loaded into the car before you drive away. **Pickle / Deserialization:** Disassembling an engine (Serialization), shipping the parts, and reassembling them on arrival (Deserialization). If you buy parts from a suspicious vendor, they might add a bomb (arbitrary code execution) during reassembly. |

---

## Think of It This Way

### 1. FastAPI: Cooperative DI (Dependency Injection)
In FastAPI, you don't write manual database connection setup code inside every endpoint. You use **`Depends`**. 
FastAPI reads your dependency list, runs the setup code, injects the connection into your route function, and runs cleanup (using generators with `yield`) after the route returns.

### 2. The Danger of Pickle
The standard `pickle` library in Python is incredibly powerful because it can serialize almost any custom object structure, including its state and functions. 
However, **Pickle is highly insecure**. 
When you load a pickle file (`pickle.load()`), Python doesn't just read data variables. It executes instructions. 
If an attacker modifies the byte stream to include a malicious `__reduce__` method, Python will run arbitrary shell commands on your server the instant you call `load()`. **Never load unverified pickle files.**

---

## Step 1: FastAPI Dependency Injection & Lifespans

Modern FastAPI uses the `lifespan` context manager to manage application startup and shutdown tasks (like database connections or loading ML models into GPU memory):

```python
from fastapi import FastAPI, Depends
from contextlib import asynccontextmanager

# Mock database helper
class Database:
    def connect(self): print("Connected to Database")
    def close(self): print("Closed Database")

db_client = Database()

# 1. Lifespan event manager
@asynccontextmanager
async def lifespan(app: FastAPI):
    # Startup: Load resources
    db_client.connect()
    yield
    # Shutdown: Clean up resources
    db_client.close()

app = FastAPI(lifespan=lifespan)

# 2. Dependency Injection generator
def get_db():
    # Setup connection
    db = db_client
    try:
        yield db # Injected into route
    finally:
        # Cleanup runs after response is sent
        pass

@app.get("/users")
def read_users(db: Database = Depends(get_db)):
    return {"status": "ok"}
```

---

## Step 2: Serialization Performance & Pickle Security

### 1. High Performance JSON (orjson)
Python's built-in `json` module is written in C but has overhead. `orjson` is a fast alternative written in Rust that supports direct serialization of dataclasses, datetimes, and numpy arrays:

```python
import orjson

data = {"username": "alice", "active": True}
# orjson returns bytes directly, which is faster than native string formatting
json_bytes = orjson.dumps(data)
print(json_bytes) # b'{"username":"alice","active":true}'
```

### 2. Custom Pickling & Exploitation Hazard
When pickling an object, you can customize how its state is saved by implementing `__getstate__` and `__setstate__`:

```python
import pickle

class SecureData:
    def __init__(self, key, secret_val):
        self.key = key
        self.secret_val = secret_val

    def __getstate__(self):
        # Prevent secret_val from being serialized
        state = self.__dict__.copy()
        del state["secret_val"]
        return state

    def __setstate__(self, state):
        self.__dict__.update(state)
        self.secret_val = "UNINITIALIZED" # Reinitialize fresh

s = SecureData("id-123", "password")
serialized = pickle.dumps(s)
loaded = pickle.loads(serialized)
print(loaded.secret_val) # "UNINITIALIZED" (safe!)
```

---

## Code Examples

### Easy Examples
```python
# 1. Running async background tasks in FastAPI
from fastapi import BackgroundTasks

def log_email_status(email: str):
    print(f"Sent email receipt to {email}")

@app.post("/checkout")
def checkout(email: str, background_tasks: BackgroundTasks):
    # Sends response to client instantly, runs email logging in the background
    background_tasks.add_task(log_email_status, email)
    return {"message": "Order placed!"}
```

### Medium Examples
```python
# 2. Pickle Arbitrary Code Execution Exploit
class MaliciousExploit:
    def __reduce__(self):
        import os
        # Return a callable and its arguments. 
        # When unpickled, this executes os.system('echo HACKED')
        return (os.system, ("echo HACKED",))

# Attacker serializes this exploit
malicious_payload = pickle.dumps(MaliciousExploit())

# Victim runs this on their backend server (crashes or runs command!)
# pickle.loads(malicious_payload)
```

### Advanced Examples
```python
# 3. Custom JSON Encoder for unsupported structures
# Built-in json fails on bytes or decimal numbers.
from decimal import Decimal
import json

class CustomEncoder(json.JSONEncoder):
    def default(self, obj):
        if isinstance(obj, Decimal):
            return float(obj)
        return super().default(obj)

payload = {"price": Decimal("29.99")}
print(json.dumps(payload, cls=CustomEncoder)) # {"price": 29.99}
```

---

## Common Mistakes & Edge Cases

### 1. Blocking async endpoints with synchronous database drivers
If you write `async def read_users()` but call a blocking PostgreSQL driver (like standard `psycopg2`) inside it, you block the single-threaded event loop. All other API requests will hang waiting for that query to finish.
* **Fix:** Either use an async driver (like `asyncpg`) or define the endpoint using `def` instead of `async def` (FastAPI will run standard `def` routes inside a background threadpool automatically to prevent blocking the event loop).

### 2. Using pickle for long-term data storage
Pickled data is closely bound to your Python code structure. If you rename a class or structure inside your project, loading old pickled files will throw an `AttributeError` or `ModuleNotFoundError` because Pickle tries to import the old structure path. For long-term data, always prefer generic formats like JSON or Protocol Buffers.

---

## Interview Questions (Top 5)

**Q1: How does FastAPI's Dependency Injection (`Depends`) manage database connection lifespans?**
> FastAPI's dependency system supports generator functions (using `yield`). When an endpoint calls `Depends(get_db)`, FastAPI executes the setup code before the yield, passes the connection into the route, runs the route, and then executes the cleanup code immediately following the yield (in a `finally` block). This guarantees connections are closed even if the request crashes.

**Q2: What is the benefit of using the `lifespan` handler over legacy startup/shutdown events in FastAPI?**
> Legacy events run asynchronously as separate isolated hooks. The modern `lifespan` handler uses a single async context manager (`asynccontextmanager`). This allows sharing setup state (like connection pools) directly with the application, ensuring that the cleanup logic has access to the exact instantiated resources initialized during startup.

**Q3: Why is loading untrusted Pickle files dangerous? Explain the mechanism.**
> When `pickle.loads()` is called, Python parses the serialized byte stream to reconstruct the object. If the byte stream contains an object with a custom `__reduce__()` method, CPython automatically imports and executes the specified function (like `os.system()`) with the provided arguments to reconstruct the class state. This allows attackers to run arbitrary code on the host machine.

**Q4: How do high-performance JSON serializers like `orjson` or `ujson` achieve better speeds than Python's built-in `json` module?**
> * The built-in `json` module has validation loops and object-wrapping overhead in Python.
> * `orjson` is written in Rust and uses SIMD (Single Instruction, Multiple Data) CPU acceleration to parse strings faster. It also directly converts float numbers, UUIDs, datetimes, and numpy arrays into bytes without intermediate Python object instantiation, eliminating serialization loops.

**Q5: When should you define a FastAPI endpoint as `async def` versus standard `def`?**
> * **`async def`:** Use when all database and network calls within the route use non-blocking async libraries (like `httpx` or `motor`).
> * **`def`:** Use when the route relies on blocking libraries (like standard `requests` or `SQLAlchemy`'s sync driver). FastAPI runs standard `def` routes inside a threadpool, preventing blocking queries from locking the main event loop thread.

---

*← [Previous: Chapter 4 (Typing & Pydantic)](04_Typing_Protocols_Pydantic.md) | [Next: Chapter 6 (NumPy & Pandas Internals) →](06_Numpy_Pandas_Internals.md)*
