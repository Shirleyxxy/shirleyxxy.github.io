---
title: "Batch Processing with Retry"
---

## Retry-per-batch Approach

The chunker [`itertools.batched`](https://docs.python.org/3/library/itertools.html#itertools.batched) pairs very well with Tenacity’s [`retry`](https://tenacity.readthedocs.io/en/latest/) for batch processing.


### Core Arguments in `@retry`

Tenacity’s decorator is very flexible, but these are the ones you’ll use most often:

#### 1. **stop** (when to give up)

* Controls how many times to retry, or for how long.
* Common helpers:

  * `stop_after_attempt(n)` → stop after *n* attempts.
  * `stop_after_delay(seconds)` → stop after total elapsed time.
* Example:

  ```python
  stop=stop_after_attempt(3)
  ```

#### 2. **wait** (how long to sleep between attempts)

* Defines the wait strategy between retries.
* Common helpers:

  * `wait_fixed(1)` → always wait 1 second.
  * `wait_random(min=1, max=3)` → wait random between 1 and 3 seconds.
  * `wait_exponential(multiplier=1, max=60)` → exponential backoff.
* Example:

  ```python
  wait=wait_fixed(1)  # simple and predictable
  ```

#### 3. **retry** (when to retry)

* Predicate to decide whether to retry given an exception or result.
* Common helpers:

  * `retry_if_exception_type(SomeException)` → retry on that error.
  * `retry_if_result(lambda r: r is None)` → retry on “bad” results.
* Example:

  ```python
  # will retry only when the function raises ConnectionError
  retry=retry_if_exception_type(ConnectionError)
  
  # will retry on multiple exceptions:
  retry=retry_if_exception_type((TimeoutError, ConnectionError)) 
  ```

#### 4. **reraise**

This controls **what exception you see after retries are exhausted**.

* **`reraise=False` (default):**

  * Tenacity raises a `RetryError`.
  * The `RetryError` wraps the last attempt’s exception.
  * You can inspect details via `.last_attempt`.

  ```python
  try:
      flaky()
  except RetryError as e:
      print("wrapped:", e.last_attempt.exception())
  ```

* **`reraise=True`:**

  * Tenacity directly re-raises the *last real exception* (e.g., `ConnectionError`).
  * Makes the retry logic transparent to callers.

  ```python
  try:
      flaky2()
  except ConnectionError:
      print("saw real error after retries")
  ```

- `reraise=True` is usually nicer for **library-style code** where the caller shouldn’t care that Tenacity was used.
- `reraise=False` is useful if you want **retry metadata** (e.g., logging or metrics on how many attempts were made).


#### 5. **before / after / before\_sleep**

* Hooks for logging, metrics, or side effects.
* Example:

  ```python
  from tenacity import before_sleep_log
  import logging
  
  log = logging.getLogger(__name__)

  before_sleep=before_sleep_log(log, logging.WARNING)
  ```



### Putting It Together (Batch Example with `itertools.batched`)



```python
import itertools
import logging
from tenacity import (
    retry, stop_after_attempt, wait_fixed,
    retry_if_exception_type, before_sleep_log
)

# --- set up built-in logger ---
logging.basicConfig(
    level=logging.INFO,  # or DEBUG if you want more detail
    format="%(asctime)s [%(levelname)s] %(name)s: %(message)s"
)
log = logging.getLogger(__name__)

# retry policy: up to 3 times, 1s apart, on ConnectionError
@retry(
    stop=stop_after_attempt(3),
    wait=wait_fixed(1),
    retry=retry_if_exception_type(ConnectionError),
    reraise=True,
    before_sleep=before_sleep_log(log, logging.WARNING),  # <-- logs retries
)
def process_batch(batch):
    log.info("Processing batch with %d UUIDs", len(batch))
    # run your Hive query here
    return run_query(batch)

def process_all(uuids, batch_size=1000):
    for batch in itertools.batched(uuids, batch_size):
        try:
            yield process_batch(batch)
        except Exception as e:
            log.error("Batch failed after retries: %s", e)
            # re-raise if you want the whole job to stop
            raise
```

### What happens now:

* On **each retryable failure**, Tenacity logs a warning:

  ```
  2025-09-22 15:45:12 [WARNING] __main__: RetryError caught ... Retrying in 1 seconds ...
  ```
* On **successful processing**, you’ll see:

  ```
  2025-09-22 15:45:13 [INFO] __main__: Processing batch with 1000 UUIDs
  ```
* On **final failure** (all retries exhausted), you’ll see:

  ```
  2025-09-22 15:45:16 [ERROR] __main__: Batch failed after retries: ConnectionError('...')
  ```

That gives you clear observability without extra boilerplate.


In summary, for batch processing:

* `stop` + `wait` → control attempts and pacing.
* `retry` → ensure you only retry on transient errors.
* `reraise=True` → caller just sees the “real” Hive error if all retries fail.
