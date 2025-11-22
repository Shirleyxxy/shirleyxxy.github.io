---
title: "Concurrency in Python"
date: 2025-08-30
---

## Concurrency Model

### `asyncio`
- Based on cooperative multitasking (single-threaded event loop).
- Tasks yield control explicitly when they hit an `await`.
- Best suited for **I/O-bound tasks** (e.g., network calls, file I/O, database queries).
- Runs all tasks on a single thread by default, avoiding thread overhead.


### `concurrent.futures`
- Runs tasks in either:
    - **ThreadPoolExecutor** → threads (good for I/O-bound when you don’t want to rewrite with async).
    - **ProcessPoolExecutor** → processes (good for CPU-bound tasks to bypass the GIL).
- Uses preemptive multitasking (OS schedules threads/processes).
- More natural for CPU-bound tasks or when dealing with blocking libraries that don’t support async.


## Programming Style

### `async/await`
- Requires you to write code in an asynchronous style (`async def`, `await`).
- **Libraries must provide async-compatible APIs.**
- Reads like synchronous code, but tasks run concurrently.

```python
import asyncio

async def fetch_data(x):
    await asyncio.sleep(1)  # Simulates async I/O
    return x * 2

async def main():
    results = await asyncio.gather(fetch_data(1), fetch_data(2))
    print(results)

asyncio.run(main())
```

### `concurrent.futures`
- Wraps existing synchronous/blocking functions.
- Doesn’t require changing function signatures.

```python
from concurrent.futures import ThreadPoolExecutor, as_completed
import time

def fetch_data(x):
    time.sleep(1)  # Blocking call
    return x * 2

with ThreadPoolExecutor() as executor:
    futures = [executor.submit(fetch_data, i) for i in [1, 2]]
    results = [f.result() for f in as_completed(futures)]
    print(results)
```


## Performance Characteristics

### `async/await`
- Lightweight: millions of coroutines can run in one thread.
- No thread-switching overhead.
- But can’t speed up CPU-bound tasks (stuck on the GIL).

### `concurrent.futures`
- Threads: good for blocking I/O in synchronous libraries. But thread context-switching has overhead.
- Processes: best for CPU-bound tasks since multiple processes bypass the GIL.
- Not as lightweight—limited by threads/process count.

## When to Use What

- Use `async/await` when:
    - You’re working with async-capable libraries (HTTP clients, DB drivers, etc.).
    - The workload is I/O-bound and you want efficient concurrency without threads.
    - You’re building scalable async servers (e.g., FastAPI, aiohttp).

- Use `concurrent.futures` when:
    - You need to parallelize CPU-heavy computations (use `ProcessPoolExecutor`).
    - You’re stuck with blocking libraries that don’t support async (use `ThreadPoolExecutor`).
    - You want simpler migration of existing synchronous code to run concurrently.


## Examples

### I/O-Bound Example (API calls)
Imagine we want to fetch data from multiple APIs, where the bottleneck is waiting for the response.

#### Using `async/await (asyncio)`

```python
import asyncio
import random

async def fetch_data(name):
    delay = random.uniform(0.5, 2.0)
    await asyncio.sleep(delay)   # non-blocking
    print(f"{name} finished after {delay:.2f}s")
    return name

async def main():
    tasks = [fetch_data(f"task{i}") for i in range(5)]
    results = await asyncio.gather(*tasks)
    print("Results:", results)

asyncio.run(main())
```

**Behavior:**
- All tasks start nearly at the same time.
- The event loop pauses each coroutine at `await asyncio.sleep` and switches to others.
- Total runtime ≈ longest single task (not sum of delays).


#### Using `concurrent.futures.ThreadPoolExecutor`

```python
import time
import random
from concurrent.futures import ThreadPoolExecutor, as_completed

def fetch_data(name):
    delay = random.uniform(0.5, 2.0)
    time.sleep(delay)  # blocking
    print(f"{name} finished after {delay:.2f}s")
    return name

with ThreadPoolExecutor() as executor:
    futures = [executor.submit(fetch_data, f"task{i}") for i in range(5)]
    results = [f.result() for f in as_completed(futures)]
    print("Results:", results)
```

**Behavior:**
- Threads allow blocking functions (`time.sleep`) to run in parallel.
- Total runtime ≈ longest single task (like `asyncio`).
- Overhead: more memory and thread context switching compared to async coroutines.


### CPU-Bound Example (Fibonacci)
Now imagine computing Fibonacci numbers (heavy CPU work), `async/await` is a bad fit.

#### Using `concurrent.futures.ProcessPoolExecutor` (best fit)

```python
from concurrent.futures import ProcessPoolExecutor, as_completed

def fib(n):
    if n < 2:
        return n
    return fib(n-1) + fib(n-2)

with ProcessPoolExecutor() as executor:
    futures = [executor.submit(fib, n) for n in (35, 36, 37)]
    results = [f.result() for f in as_completed(futures)]
    print("Results:", results)
```

**Behavior:**
- Each Fibonacci computation runs in its own process.
- Processes bypass the GIL, so Python can truly use multiple cores.
- Runtime ≈ longest task, not sum.

### 🔑 Takeaways
- **I/O-bound tasks** → `asyncio` is very efficient if libraries support async.
- **CPU-bound tasks** → `ProcessPoolExecutor` is the correct tool.
- **Blocking libraries** → `ThreadPoolExecutor` works if you can’t use async.