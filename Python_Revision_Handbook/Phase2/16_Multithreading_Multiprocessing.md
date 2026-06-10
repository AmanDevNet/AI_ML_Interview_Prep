# Chapter 16: Multithreading & Multiprocessing — "Conquering Concurrency"

---

## Topic Overview

| Aspect | Details |
|--------|---------|
| **Definition** | **Multithreading** runs multiple tasks concurrently within a single process. **Multiprocessing** runs multiple tasks in parallel across separate processes. |
| **Why It Exists** | To write non-blocking applications (like web servers) and to utilize modern multi-core CPU architectures. |
| **Where It Is Used** | Downloading files concurrently (I/O-bound), computing large mathematical models (CPU-bound), backend web workers (Celery, Gunicorn), UI responsiveness. |
| **Real-World Analogy** | **Multithreading:** A single chef working on multiple dishes at once (e.g. while water is boiling, they chop vegetables). **Multiprocessing:** Hiring 4 independent chefs in 4 separate kitchens to cook 4 different meals. |

---

## Think of It This Way

### The Elephant in the Room: The GIL
Python (specifically CPython) has a lock called the **Global Interpreter Lock (GIL)**.
The GIL ensures that **only one thread executes Python bytecode at any given time**.

Why does the GIL exist? It protects CPython's internal memory management from race conditions, making reference counting thread-safe.

**The Golden Rule of Python Concurrency:**
* **I/O-Bound Tasks (Waiting on Network/Disk):** Use **Multithreading**. When a thread waits for data to download or write to disk, CPython releases the GIL, allowing other threads to run.
* **CPU-Bound Tasks (Heavy calculations, data compression):** Use **Multiprocessing**. Processes bypass the GIL entirely because each process runs its own independent Python interpreter and has its own virtual memory space.

---

## Step 1: Multithreading & Thread Safety

Because threads share the same memory, they can read and write the same variables. This leads to **race conditions** if multiple threads modify the same variable concurrently. 

To prevent race conditions, we use a **Lock**.

```python
import threading
import time

shared_counter = 0
counter_lock = threading.Lock()

def increment():
    global shared_counter
    for _ in range(100_000):
        # 1. Acquire the lock before modifying shared state
        with counter_lock:
            # Under the hood, this compiles to read -> increment -> write
            shared_counter += 1
        # 2. Lock is automatically released at the end of 'with' block

threads = []
for _ in range(2):
    t = threading.Thread(target=increment)
    threads.append(t)
    t.start()

for t in threads:
    t.join()  # Wait for threads to finish

print(f"Final Counter: {shared_counter}")  # Guaranteed to be 200,000!
```
*Note: If you remove the `with counter_lock:`, the final counter will likely be less than 200,000 because threads overwrite each other's increments.*

---

## Step 2: Multiprocessing (Bypassing the GIL)

Processes do not share memory. Each process gets its own copy of the script's variables.

```python
import multiprocessing
import os

def check_cpu():
    print(f"Running task on Process ID (PID): {os.getpid()}")

if __name__ == "__main__":
    # We must wrap process creation in 'if __name__ == "__main__"' on Windows
    # to prevent infinite recursion during child process spawning.
    
    processes = []
    for _ in range(4):
        p = multiprocessing.Process(target=check_cpu)
        processes.append(p)
        p.start()

    for p in processes:
        p.join()
```

---

## Step 3: Modern API — `concurrent.futures`

The `concurrent.futures` module provides a high-level API for managing pools of threads or processes. It is much cleaner than manually starting and joining threads/processes.

```python
from concurrent.futures import ThreadPoolExecutor, ProcessPoolExecutor
import urllib.request

urls = [
    "https://www.google.com",
    "https://www.python.org",
    "https://www.github.com"
]

def fetch_url(url):
    with urllib.request.urlopen(url, timeout=5) as response:
        return f"{url}: {response.status}"

# I/O Bound task -> ThreadPoolExecutor
with ThreadPoolExecutor(max_workers=3) as executor:
    # 'map' automatically schedules tasks and returns results in order
    results = executor.map(fetch_url, urls)
    for result in results:
        print(result)
```

---

## Code Examples

### Easy Examples
```python
# 1. Basic Thread Spawning
def print_numbers():
    for i in range(5):
        print(i)
        time.sleep(0.1)

t = threading.Thread(target=print_numbers)
t.start()
t.join() # Wait for thread t to complete before running next lines
print("Thread finished!")

# 2. Daemon Threads
# A daemon thread runs in the background and closes automatically when the main script ends.
t_daemon = threading.Thread(target=lambda: time.sleep(100), daemon=True)
t_daemon.start()
# Main program exits instantly, kills t_daemon

# 3. Simple Process Spawning
def compute_square(n):
    return n * n

p = multiprocessing.Process(target=compute_square, args=(10,))
p.start()
p.join()

# 4. Checking CPU Core Count
print(f"Total CPU Cores: {multiprocessing.cpu_count()}")

# 5. Lock vs RLock
# RLock (Reentrant Lock) can be acquired multiple times by the SAME thread without causing a deadlock.
rlock = threading.RLock()
with rlock:
    with rlock: # Normal Lock would hang here forever!
        print("Acquired reentrant lock!")
```

### Medium Examples
```python
# 6. Worker Queue Threading (Producer-Consumer)
from queue import Queue

task_queue = Queue()

def worker():
    while True:
        item = task_queue.get()
        if item is None: # Exit sentinel
            break
        print(f"Processing item: {item}")
        task_queue.task_done()

# Start worker thread
t = threading.Thread(target=worker)
t.start()

# Add items
for item in range(5):
    task_queue.put(item)

# Block main thread until all items in queue are processed
task_queue.join()
# Stop worker
task_queue.put(None)
t.join()

# 7. Sharing state in Multiprocessing using Value and Array
# Processes don't share memory, so we must use special shared memory objects
def add_points(score, arr):
    score.value += 10
    for i in range(len(arr)):
        arr[i] += 1

# Shared float Value 'd', shared array of ints 'i'
s_score = multiprocessing.Value('d', 0.0)
s_arr = multiprocessing.Array('i', [1, 2, 3])

p = multiprocessing.Process(target=add_points, args=(s_score, s_arr))
p.start()
p.join()
print(s_score.value) # 10.0
print(s_arr[:])      # [2, 3, 4]

# 8. Thread-Local Data (Threading.local)
# Variables that look global but are completely isolated inside each thread
thread_local = threading.local()
def run_thread(name):
    thread_local.user = name
    time.sleep(0.1)
    print(f"Thread local user value: {thread_local.user}")

t1 = threading.Thread(target=run_thread, args=("Alice",))
t2 = threading.Thread(target=run_thread, args=("Bob",))
t1.start(); t2.start()
t1.join(); t2.join()

# 9. ThreadPoolExecutor returning Future objects (as_completed)
from concurrent.futures import as_completed

def task(n):
    return n * 2

with ThreadPoolExecutor() as executor:
    # submit returns a Future object representing a pending calculation
    futures = [executor.submit(task, i) for i in range(5)]
    # Process them as soon as they finish, regardless of order
    for future in as_completed(futures):
        print(future.result())

# 10. Multiprocessing Queue for inter-process communication (IPC)
def sender(q):
    q.put("Message from Child Process")

q = multiprocessing.Queue()
p = multiprocessing.Process(target=sender, args=(q,))
p.start()
print(q.get()) # "Message from Child Process"
p.join()
```

### Advanced Examples
```python
# 11. CPU-bound Parallel processing comparison
# Finding primes (CPU-bound)
def count_primes(limit):
    count = 0
    for num in range(2, limit):
        for i in range(2, int(num**0.5) + 1):
            if num % i == 0:
                break
        else:
            count += 1
    return count

# Bypassing the GIL using ProcessPoolExecutor
if __name__ == "__main__":
    import time
    limits = [3_000_000, 3_000_000, 3_000_000, 3_000_000]
    
    # Run sequentially / with threads (Slow due to GIL)
    start = time.perf_counter()
    with ThreadPoolExecutor() as ex:
        list(ex.map(count_primes, limits))
    print(f"Threads time (Slow): {time.perf_counter() - start:.2f} seconds")

    # Run with processes (Fast, utilizes multiple CPU cores)
    start = time.perf_counter()
    with ProcessPoolExecutor() as ex:
        list(ex.map(count_primes, limits))
    print(f"Processes time (Fast): {time.perf_counter() - start:.2f} seconds")

# 12. Deadlock Demonstration
# A deadlock occurs when two threads are blocked forever, waiting for each other to release locks.
lock_a = threading.Lock()
lock_b = threading.Lock()

def thread_one():
    with lock_a:
        time.sleep(0.1)
        with lock_b:
            print("Thread 1 completed")

def thread_two():
    with lock_b:
        time.sleep(0.1)
        with lock_a: # Hangs here forever because thread_one holds lock_a!
            print("Thread 2 completed")

# 13. Event objects for thread synchronization
# One thread signals an event, another thread blocks waiting for it.
event = threading.Event()

def task_waiter():
    print("Waiter: waiting for event signal...")
    event.wait() # Blocks here
    print("Waiter: event received! Moving forward.")

def task_trigger():
    time.sleep(0.5)
    print("Trigger: task complete, signaling waiter.")
    event.set() # Unblocks waiter

# 14. Multiprocessing Pool for massive mapping tasks
# Pool manages workers automatically and chunks data for process performance
def square(x):
    return x * x

if __name__ == "__main__":
    with multiprocessing.Pool(processes=4) as pool:
        # map handles chunking and passes elements efficiently to processes
        results = pool.map(square, range(10000))

# 15. Semaphore to limit resource access
# Allowing max 3 concurrent threads to hit an endpoint
semaphore = threading.Semaphore(3)

def access_resource(thread_id):
    with semaphore:
        print(f"Thread {thread_id} active")
        time.sleep(0.2)

threads = [threading.Thread(target=access_resource, args=(i,)) for i in range(6)]
for t in threads: t.start()
for t in threads: t.join()
```

---

## Common Mistakes & Edge Cases

### 1. The `if __name__ == "__main__"` Requirement
In Windows, Python spawns new processes by importing the main script file. If you don't wrap your process-starting code in `if __name__ == "__main__":`, the child processes will run the script from the start, attempt to spawn *their own* child processes, and crash your machine in a fork-bomb loops.

### 2. Assuming threads speed up CPU-bound tasks
Due to the Global Interpreter Lock (GIL), running CPU-bound calculations in threads offers **no speedup**. In fact, they run *slower* than sequential execution because of thread context-switching overhead.

### 3. Thread Deadlocks
Always acquire locks in the **exact same order** across all threads. If Thread A acquires Lock 1 then Lock 2, and Thread B acquires Lock 2 then Lock 1, they will deadlock if they execute concurrently.

---

## Interview Questions (Top 5)

**Q1: What is Python's Global Interpreter Lock (GIL), and why does it exist?**
> The GIL is a mutex lock in the standard CPython interpreter that ensures only one native thread executes Python bytecode at any given time. It was implemented to make CPython's memory management (which uses reference counting) thread-safe. Without the GIL, concurrent modifications of reference counts could lead to memory leaks or segmentation faults.

**Q2: When should you use Multithreading versus Multiprocessing in Python?**
> * **Multithreading** should be used for **I/O-bound tasks** (e.g. downloading files, database queries, scraping pages). When a thread blocks waiting for network or disk output, Python releases the GIL, allowing other threads to execute concurrently.
> * **Multiprocessing** should be used for **CPU-bound tasks** (e.g. video rendering, machine learning modeling, heavy mathematical equations). Since processes run in separate memory spaces with their own Python interpreters, they bypass the GIL, enabling true parallel execution across multiple CPU cores.

**Q3: What is a Race Condition, and how do you prevent it in a multithreaded application?**
> A race condition occurs when two or more threads access and modify shared memory concurrently, and the final state depends on the non-deterministic order of execution. It is prevented by using a synchronization primitive like a Lock (`threading.Lock`). A thread must acquire the lock before accessing the shared resource and release it immediately after completing the operation, ensuring mutual exclusion.

**Q4: Why does a child process have independent variables from the parent process?**
> When a new process is spawned (using `fork` in Linux or `spawn` in Windows), the OS creates a copy of the parent process's address space. The child process has its own virtual memory. Modifying a variable in a child process will not change its value in the parent process. To share state between processes, you must use inter-process communication (IPC) tools like `multiprocessing.Queue`, `Pipe`, or shared memory variables (`Value`, `Array`).

**Q5: What is the difference between `threading.Lock` and `threading.RLock`?**
> A standard `Lock` can only be acquired once. If the thread holding the lock tries to acquire it again, it will block itself forever (self-deadlock). An `RLock` (Reentrant Lock) keeps track of which thread currently holds the lock and the recursion level. The thread that holds the lock can acquire it multiple times without blocking, but it must release it the same number of times to fully unlock it.

---

*← [Previous: Chapter 15 (Time Complexity)](15_Time_Complexity_Memory.md) | [Next: Chapter 17 (Phase 2 Interview Handbook) →](17_Phase2_Interview_Handbook.md)*
