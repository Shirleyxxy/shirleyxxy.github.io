---
title: "Concurrency in Python: asyncio vs. concurrent.futures"
---

## Concurrency Model

### `asyncio`
- Based on cooperative multitasking (single-threaded event loop).
- Tasks yield control explicitly when they hit an `await`.
- Best suited for I/O-bound tasks (e.g., network calls, file I/O, database queries).
- Runs all tasks on a single thread by default, avoiding thread overhead.