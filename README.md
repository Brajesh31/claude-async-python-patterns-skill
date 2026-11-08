# claude-async-python-patterns-skill

A Claude Code skill for building efficient, reliable asynchronous Python applications with asyncio and async/await.

## Overview

This skill covers concurrency patterns for I/O-bound applications such as web APIs, network clients, scrapers, background workers, and real-time services. It helps structure asynchronous code, control concurrency, handle cancellation, and avoid common event-loop problems.

Async code is not automatically faster. It is most useful when tasks spend substantial time waiting on I/O.

## Main Capabilities

- Use async functions and await correctly.
- Coordinate concurrent tasks.
- Apply task groups and structured concurrency patterns.
- Limit concurrency with semaphores and queues.
- Handle timeouts and cancellation.
- Manage async context managers and resource cleanup.
- Implement backpressure.
- Integrate async database and HTTP clients.
- Test asynchronous code.
- Add observability for concurrent workflows.

## When to Use It

Use this skill for:

- Async web APIs.
- Concurrent database or network operations.
- Web scraping and API aggregation.
- WebSocket and real-time services.
- Background task processing.
- I/O-bound microservices.
- Workloads that need bounded concurrency.

For CPU-bound processing, use an appropriate process pool, native extension, or separate compute service rather than expecting asyncio alone to accelerate the work.

## Installation

```bash
git clone https://github.com/Brajesh31/claude-async-python-patterns-skill.git ~/.claude/skills/async-python-patterns
```

## Usage

Invoke the skill for asynchronous Python development:

```text
/async-python-patterns
```

Provide the Python version, async framework, workload type, concurrency target, and failure-handling requirements.

## Example

```python
import asyncio

async def fetch_one(item: str, semaphore: asyncio.Semaphore) -> str:
    async with semaphore:
        await asyncio.sleep(0.1)  # Replace with an async I/O operation.
        return f"processed:{item}"

async def main() -> None:
    items = ["a", "b", "c", "d"]
    semaphore = asyncio.Semaphore(2)

    results = await asyncio.gather(
        *(fetch_one(item, semaphore) for item in items)
    )
    print(results)

if __name__ == "__main__":
    asyncio.run(main())
```

This example demonstrates bounded concurrency. Replace the placeholder operation with a real asynchronous client call.

## Recommended Workflow

1. Confirm that the workload is I/O-bound.
2. Identify the event-loop owner and framework.
3. Use async-compatible libraries.
4. Bound concurrent operations.
5. Set explicit timeouts.
6. Handle cancellation and cleanup.
7. Add retry policies only for safe, retryable operations.
8. Test error, timeout, and cancellation paths.
9. Measure throughput and resource use.

## Reference Playbook

The repository includes `resources/implementation-playbook.md`, covering concurrency limits, timeouts, cancellation, backpressure, testing, and observability.

## Reliability Checklist

- No blocking calls run on the event loop unintentionally.
- Concurrency is bounded.
- Timeouts are explicit.
- Cancellation is propagated correctly.
- Async resources are closed reliably.
- Retries do not duplicate unsafe side effects.
- Queue growth and backpressure are controlled.
- Tests cover timeouts and failures.
- Metrics capture latency and task failures.

## Repository Contents

- `SKILL.md`: Main async Python instructions.
- `resources/implementation-playbook.md`: Implementation and review playbook.
- `README.md`: Patterns, examples, and reliability guidance.

## Practical Principle

Use concurrency to overlap waiting, not to hide slow work. Measure the actual workload and keep the async design as simple as possible.
