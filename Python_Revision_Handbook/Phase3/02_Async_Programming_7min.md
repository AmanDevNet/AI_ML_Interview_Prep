# Chapter 2: Async Programming & Concurrency — "Single-Threaded Speed"

---

## Topic Overview

| Aspect | Details |
|--------|---------|
| **Definition** | **Async Programming** is a concurrent execution model where a single thread cooperative yields execution control back to an **Event Loop** during waiting periods (like network requests). |
| **Why It Exists** | Spawning threads or processes consumes significant OS resources. Async programming allows handling tens of thousands of concurrent network connections within a single thread. |
| **Where It Is Used** | High-performance API servers (FastAPI), web scrapers, websocket connections, chat applications, and databases clients. |
| **Real-World Analogy** | A waiter taking order tables. Instead of standing at Table 1 waiting for the chef to cook the food, the waiter takes Table 1's order, passes it to the kitchen, and immediately goes to take Table 2's order. |

---

## Think of It This Way

Normally, Python code is synchronous and blocking:
```python
import time

def fetch_data():
    time.sleep(2) # CPU hangs here doing NOTHING for 2 seconds
    return "data"
```

In asynchronous programming, we write non-blocking code using `async` and `await`:
```python
import asyncio

async def fetch_data():
    await asyncio.sleep(2) # The thread yields control back to the event loop!
    return "data"
```

During those 2 seconds of sleep, the single Python thread isn't idle. 
The event loop runs other tasks. When the 2 seconds are up, the event loop resumes our `fetch_data` function exactly where it paused.

**Crucial point:** Async is NOT running things in parallel. It is multitasking on a single thread. If you run a CPU-heavy loop inside an async function, you will block the event loop and freeze your entire application.

---

## Step 1: Coroutines, Tasks, and Futures

* **Coroutine:** A function defined with `async def`. Calling it returns a coroutine object. It doesn't run the code until you `await` it or schedule it on the event loop.
* **Future:** A low-level object representing an event/result that hasn't happened yet.
* **Task:** A high-level wrapper around a Coroutine. When you create a task, you register the coroutine to run on the event loop immediately.

```python
import asyncio

async def my_coroutine():
    print("Running...")

# 1. Calling it just creates a coroutine object (does not run code)
coro = my_coroutine() 
print(type(coro)) # <class 'coroutine'>

# 2. To execute it, run it in the event loop
asyncio.run(coro)
```

---

## Step 2: running Tasks Concurrently

Use `asyncio.gather` to execute multiple tasks concurrently:

```python
async def worker(worker_id, sleep_time):
    print(f"Worker {worker_id} started")
    await asyncio.sleep(sleep_time)
    print(f"Worker {worker_id} finished")
    return f"Result {worker_id}"

async def main():
    # Schedules all three to run concurrently
    results = await asyncio.gather(
        worker(1, 2),
        worker(2, 1),
        worker(3, 1.5)
    )
    print(results) # ['Result 1', 'Result 2', 'Result 3']

asyncio.run(main())
```

---

## Step 3: Mixing Blocking Code with Async (`run_in_executor`)

If you must run blocking code (like standard file read/write, legacy SDKs, or math-heavy tasks) inside an async application, use `asyncio.to_thread()` or `loop.run_in_executor()` to run it on a separate background thread without blocking the event loop:

```python
import time

def blocking_io():
    time.sleep(2) # Blocks the thread!
    return "Done"

async def main():
    # Runs the blocking function in a background threadpool safely
    result = await asyncio.to_thread(blocking_io)
    print(result)

asyncio.run(main())
```

---

## Code Examples

### Easy Examples
```python
# 1. Simple task execution
async def say_hello():
    await asyncio.sleep(0.1)
    print("Hello")

asyncio.run(say_hello())

# 2. Timeout handling
async def slow_task():
    await asyncio.sleep(5)

async def main():
    try:
        await asyncio.wait_for(slow_task(), timeout=1.0)
    except asyncio.TimeoutError:
        print("Task took too long!")

asyncio.run(main())
```

### Medium Examples
```python
# 3. Fire-and-forget Tasks with create_task
# create_task schedules the coroutine to run in the background instantly
async def background_sync():
    await asyncio.sleep(1)
    print("Sync complete")

async def main():
    print("Main start")
    # Starts background_sync in background
    task = asyncio.create_task(background_sync())
    print("Main continues...")
    await task # Wait for it to finish before exiting

asyncio.run(main())

# 4. Async Context Managers
class AsyncResource:
    async def __aenter__(self):
        print("Connecting async...")
        await asyncio.sleep(0.1)
        return self

    async def __aexit__(self, exc_type, exc_val, exc_tb):
        print("Disconnecting async...")
        await asyncio.sleep(0.1)

async def run_resource():
    async with AsyncResource() as res:
        print("Using resource")

asyncio.run(run_resource())
```

### Advanced Examples
```python
# 5. Async Generator (Streaming data)
async def async_generator():
    for i in range(3):
        await asyncio.sleep(0.1)
        yield i

async def consume():
    async for val in async_generator():
        print(val)

asyncio.run(consume())

# 6. Implementing an Async Queue (Producer-Consumer)
async def producer(q):
    for i in range(5):
        await asyncio.sleep(0.1)
        await q.put(f"item {i}")
        print(f"Produced: item {i}")

async def consumer(q):
    while True:
        item = await q.get()
        print(f"Consumed: {item}")
        q.task_done()

async def pipeline():
    q = asyncio.Queue()
    prod = asyncio.create_task(producer(q))
    cons = asyncio.create_task(consumer(q))
    
    await prod
    await q.join() # Wait for all items to be processed
    cons.cancel() # Stop the infinite consumer task

asyncio.run(pipeline())
```

---

## Common Mistakes & Edge Cases

### 1. Blocking the Event Loop with standard time.sleep()
Writing `time.sleep(5)` inside an `async def` function halts the entire thread. The event loop cannot run any other task during this time. Always use `await asyncio.sleep(5)`.

### 2. Forgetting to `await` a Coroutine
If you call an async function without `await` (e.g. `x = fetch_data()`), Python does **not** execute the function. It just returns a coroutine object. It will trigger a warning: `RuntimeWarning: coroutine 'fetch_data' was never awaited`.

---

## Interview Questions (Top 5)

**Q1: What is the core difference between Multithreading and Asynchronous Programming?**
> * **Multithreading:** The operating system manages threads. It context-switches between them preemptively. This uses more memory (each thread allocates its own stack space) and introduces race condition risks on shared variables.
> * **Async Programming:** A single thread is shared. Context-switching is cooperative (tasks yield control voluntarily using `await`). It uses significantly less memory and does not have race conditions on shared memory (since only one block of code runs at a time).

**Q2: What happens under the hood when a coroutine calls `await`?**
> When a coroutine calls `await`, it pauses its execution state and yields control back to the **Event Loop**. The event loop registers the paused coroutine's future and checks its list of pending tasks to find which task is ready to run next. Once the awaited future completes (e.g. socket has data ready), the event loop schedules the paused coroutine to resume.

**Q3: Can async code run faster than multithreading for CPU-bound tasks?**
> No. Async code runs on a **single thread**. If a task is CPU-bound (doing heavy math or rendering), it will block the thread completely and freeze the event loop. For CPU-bound tasks, you must use **Multiprocessing** to scale across multiple cores, or run the CPU task in a separate process pool executor using `loop.run_in_executor`.

**Q4: What is the difference between `asyncio.gather()` and `asyncio.wait()`?**
> * `asyncio.gather()` takes multiple awaitables, runs them concurrently, and returns their results in a structured list matching the order of input. It throws exceptions immediately unless `return_exceptions=True` is set.
> * `asyncio.wait()` takes a collection of Task objects and returns two sets: `(done, pending)`. It allows fine-grained control, like exiting as soon as the first task finishes (`return_when=FIRST_COMPLETED`).

**Q5: Why does calling `asyncio.run()` inside an already running event loop raise a `RuntimeError`?**
> Python does not allow nesting event loops on a single thread. `asyncio.run()` creates a brand new event loop and attempts to close it at the end. If an event loop is already running (for example, inside a Jupyter notebook or a FastAPI application), this raises `RuntimeError: asyncio.run() cannot be called from a running event loop`. In these environments, you should schedule your coroutines using `asyncio.create_task()` or use libraries like `nest_asyncio`.

---

*← [Previous: Chapter 1 (Memory Management)](01_Advanced_Memory_GIL.md) | [Next: Chapter 3 (Metaclasses & Descriptors) →](03_Metaclasses_Descriptors.md)*
