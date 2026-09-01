---
title: "Error Handling and Cleanup with try / except / finally in Async Python"
updated: 2026-08-31
sort_date: 2026-08-31
---

Today I revisited a very basic Python pattern, but one that becomes especially important in async code:

```python
try:
    await do_something()
finally:
    await cleanup()
```

The main idea is simple: **`finally` runs regardless of how the `try` block exits.**

That makes it the right place for cleanup work that must happen whether an operation succeeds, fails, or gets cancelled.

## 1. Use `finally` for Guaranteed Cleanup

Consider a resource that must always be closed:

```python
async def process():
    resource = await acquire_resource()

    try:
        await use_resource(resource)
    finally:
        await resource.close()
```

If `use_resource()` succeeds, the resource is closed.

If `use_resource()` raises an exception, the resource is still closed.

Without `finally`:

```python
await use_resource(resource)
await resource.close()
```

`resource.close()` would never run if `use_resource()` failed.

This pattern is useful for things like:

- connections
- locks
- streams
- subscriptions
- temporary resources
- other cleanup that must not be skipped

## 2. `finally` Does Not Swallow the Original Exception

If the code inside `try` raises an exception and the `finally` block completes successfully, the original exception keeps propagating:

```python
try:
    await do_something()  # raises ValueError
finally:
    await cleanup()

# ValueError still propagates upward
```

So `finally` is about **cleanup**, not error handling.

## 3. Add `except` When We Want to Handle an Error

If we want to react to a specific exception, we can combine `except` and `finally`:

```python
try:
    await do_something()
except ValueError as exc:
    logger.exception("Invalid value")
finally:
    await cleanup()
```

The flow is:

- success -> skip `except` -> run `finally`
- `ValueError` -> run `except` -> run `finally`
- another exception -> skip this `except` -> run `finally` -> propagate the exception

## 4. Decide Whether an Exception Is Handled or Re-Raised

Catching an exception without raising it again means the error has been handled:

```python
try:
    await do_something()
except ValueError:
    logger.warning("Invalid value")
finally:
    await cleanup()

# execution continues here
```

That may be correct for an expected, recoverable error.

But if we only want to log an error while still treating the operation as failed, we should re-raise it:

```python
try:
    await do_something()
except ValueError:
    logger.exception("Invalid value")
    raise
finally:
    await cleanup()
```

A bare `raise` inside an `except` block re-raises the same exception and preserves its traceback.

## 5. Handle Different Errors Differently

We can use multiple `except` blocks:

```python
try:
    await do_something()
except ValueError:
    logger.warning("Invalid input")
except TimeoutError:
    logger.warning("Operation timed out")
except SomeCustomError:
    logger.warning("Known application error")
finally:
    await cleanup()
```

If several exceptions should be handled in the same way, they can be grouped:

```python
try:
    await do_something()
except (ValueError, TimeoutError) as exc:
    logger.warning("Expected operation failure: %s", exc)
finally:
    await cleanup()
```

## 6. Unexpected Errors Should Usually Be Logged and Re-Raised

A useful default is:

```python
try:
    await do_something()
except ValueError:
    # Expected error: handle intentionally
    logger.warning("Invalid value")
except Exception:
    # Unexpected error: record it, but do not hide it
    logger.exception("Unexpected error")
    raise
finally:
    await cleanup()
```

The distinction is important:

- **Expected error** -> handle it intentionally if the application can recover.
- **Unexpected error** -> usually log and re-raise it.
- **Cleanup** -> always happens in `finally`.

Silently swallowing an unexpected exception can make a real failure look like success and make bugs much harder to diagnose.

Inside an `except` block, `logger.exception(...)` is useful because it includes the traceback automatically.

## 7. Re-Raising an Error Does Not Mean the Frontend Has to Crash

One thing I initially wondered was: if an error is re-raised, could that eventually crash the frontend?

The better way to think about this is to separate **error propagation inside the program** from **error presentation at an application boundary**.

A lower-level service should not pretend an operation succeeded when it actually failed:

```python
async def service_operation():
    try:
        await do_something()
    except Exception:
        logger.exception("Unexpected failure")
        raise
    finally:
        await cleanup()
```

A higher-level API or application boundary can then translate that failure into something the frontend understands:

```python
try:
    result = await service_operation()
    return {"status": "success", "data": result}
except Exception:
    logger.exception("Request failed")
    return {
        "status": "error",
        "message": "Something went wrong",
    }
```

And the frontend should handle that failure gracefully:

```typescript
try {
  const result = await doSomething();
  updateUI(result);
} catch (error) {
  console.error(error);
  showErrorState();
}
```

So the goal is not to hide unexpected errors. It is to:

> **preserve the error internally, but handle it gracefully at the application boundary.**

That way:

- the user does not get a broken UI
- the system does not falsely report success
- logs and telemetry still capture the real failure
- engineers retain enough information to debug it

## 8. Avoid Bare `except`

In most application code, prefer:

```python
except Exception:
```

over:

```python
except:
```

A bare `except` catches `BaseException`, which includes exceptions such as `KeyboardInterrupt` and `SystemExit` that usually should not be swallowed.

There is also an async-specific detail: in modern Python, `asyncio.CancelledError` inherits from `BaseException`, not `Exception`.

That means:

```python
except Exception:
```

does not accidentally swallow task cancellation, while the `finally` block can still run cleanup code.

## 9. `async with` Often Encapsulates the Same Pattern

When a resource supports an async context manager, this:

```python
async with resource:
    await do_something(resource)
```

is often the higher-level version of an acquire -> `try` -> `finally` -> cleanup lifecycle.

Context managers are preferable when the resource provides them because they make the ownership and cleanup rules explicit.

## Mental Model

I find this the easiest way to remember the four blocks:

| Block | Responsibility |
| --- | --- |
| `try` | Attempt the work |
| `except` | Handle specific failures |
| `else` | Run only when the `try` block succeeds |
| `finally` | Run regardless of success or failure |

And for application-level error handling:

> **Handle expected errors intentionally. Re-raise unexpected errors unless you are at the boundary responsible for turning them into a safe user-facing failure. Always use `finally` for cleanup that must happen regardless.**

This is basic Python, but revisiting it was useful. In async applications especially, this small pattern sits underneath resource management, cancellation, logging, and reliable error propagation.
