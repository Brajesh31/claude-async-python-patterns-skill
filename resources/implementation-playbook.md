# Async Python Implementation Playbook

This playbook focuses on I/O-bound concurrency with `asyncio`. Use synchronous code for simple workflows and process pools or native/vectorized code for CPU-bound work.

## 1. Structure async entry points
```python
import asyncio

async def main():
    ...

if __name__ == "__main__":
    asyncio.run(main())
```
Do not call `asyncio.run` from inside an already-running event loop. In notebooks and async frameworks, await the coroutine in the existing loop.

## 2. Concurrent independent I/O
Use `asyncio.TaskGroup` (Python 3.11+) for related tasks that should be cancelled together, or `asyncio.gather` when its result and exception semantics are appropriate. Bound concurrency with a semaphore when the remote service has rate or connection limits.

```python
import asyncio

async def bounded_call(item, semaphore):
    async with semaphore:
        return await fetch_item(item)

async def collect(items):
    semaphore = asyncio.Semaphore(10)
    async with asyncio.TaskGroup() as group:
        tasks = [group.create_task(bounded_call(x, semaphore)) for x in items]
    return [task.result() for task in tasks]
```

## 3. Timeouts, retries, and cancellation
- Set explicit timeouts for network and database operations.
- Retry only transient failures, with bounded exponential backoff and jitter.
- Do not retry validation errors or non-idempotent operations unless protected by idempotency keys.
- Propagate cancellation; do not swallow `CancelledError`.
- Ensure cleanup uses `async with` or `try/finally`.

## 4. Queues and backpressure
Use bounded `asyncio.Queue(maxsize=...)` to prevent producers from overwhelming consumers. Define worker count, shutdown behavior, poison-message handling, and what happens when a consumer fails.

## 5. Blocking work
Never run blocking file, network, or CPU-heavy work directly on the event loop. Use an async library where available, `asyncio.to_thread` for blocking I/O, or a process pool for CPU-bound Python work. Measure before adding complexity.

## 6. Resource lifecycle
Use async context managers for HTTP clients, database pools, and streams. Reuse connection pools instead of creating a client per request. Close resources during application shutdown.

## 7. Testing
- Test success, timeout, cancellation, retry exhaustion, and partial failure.
- Use deterministic fakes for external services.
- Avoid arbitrary sleeps; coordinate tests with events or controlled futures.
- Assert concurrency limits and cleanup behavior.
- Enable asyncio debug mode when investigating leaked tasks or un-awaited coroutines.

## 8. Observability
Log task identifiers and correlation IDs, but never secrets. Track latency, active tasks, queue depth, retry counts, timeout rates, and cancellation counts.

## Design checklist
- Is the workload I/O-bound?
- Is concurrency bounded?
- Are timeouts and cancellation defined?
- Are retries safe and limited?
- Is backpressure implemented?
- Are clients and tasks cleaned up?
- Are async paths covered by tests?
