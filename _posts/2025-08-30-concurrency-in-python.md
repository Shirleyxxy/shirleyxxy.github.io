---
title: "Concurrency in Python"
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
