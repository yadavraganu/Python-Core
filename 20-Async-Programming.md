Asynchronous programming lets you handle thousands of concurrent I/O-bound tasks without the overhead of creating thousands of threads. In Python this is primarily done with the `asyncio` library.

## 1. The Mental Model: Event Loop and Cooperative Multitasking

`asyncio` runs on a **single thread** with a single **event loop**. The loop keeps a queue of ready tasks, runs one until it hits an `await` on something that isn't ready yet (a network response, a sleep), then switches to another ready task. When the awaited thing completes, the loop resumes the original task.

- **Cooperative multitasking:** tasks voluntarily yield control at `await` points. Nothing interrupts a task mid-execution.
- **Threading is preemptive** by contrast: the OS can pause a thread at almost any moment, which is why threads need locks for shared data.

Consequences worth being able to state:
- Between two `await` points your code runs uninterrupted, so many race conditions that plague threads can't happen (though logical races across an `await` still can).
- A task that never awaits (a tight CPU loop, `time.sleep()`) **blocks every other task**, since nothing can preempt it.
- There is no parallelism: everything runs on one core. Async gives **concurrency**, not parallel speedup.

## 2. The Core Syntax

* **`async def`**: declares a coroutine function. Calling it doesn't run the body; it returns a **coroutine object**.
* **`await`**: pauses the current coroutine until the awaited object finishes, handing control back to the event loop so other tasks can run.
* **`asyncio.run()`**: the entry point; creates the event loop, runs the main coroutine to completion, then closes the loop.

```python
import asyncio

async def say_hello():
    print("Hello...")
    await asyncio.sleep(1)   # non-blocking pause
    print("...World!")

if __name__ == "__main__":
    asyncio.run(say_hello())
```

### The Classic Bug: Forgetting `await`

```python
async def main():
    asyncio.sleep(1)        # BUG: creates a coroutine object, never runs it
    await asyncio.sleep(1)  # correct
```
Python emits `RuntimeWarning: coroutine 'sleep' was never awaited`. The coroutine object was created but never scheduled, so nothing happened. This is the most common async beginner bug, and it's also the reason `result = some_async_fn()` gives you a coroutine object instead of a value.

## 3. Coroutines vs. Tasks vs. Futures

| Concept | What it is |
|---|---|
| **Coroutine** | The object returned by calling an `async def` function. Does nothing until awaited or wrapped in a Task. |
| **Task** | A coroutine **scheduled** on the event loop to run concurrently. Created via `asyncio.create_task()`. |
| **Future** | A low-level placeholder for a result that will be available later. Tasks are a subclass of Future. You rarely create these directly. |

Awaiting a bare coroutine runs it **inline**, to completion, before moving on. Wrapping it in a Task starts it running **in the background**:

```python
async def work(name):
    await asyncio.sleep(1)
    return name

async def main():
    # Sequential: takes ~2 seconds
    a = await work("a")
    b = await work("b")

    # Concurrent: takes ~1 second
    t1 = asyncio.create_task(work("a"))
    t2 = asyncio.create_task(work("b"))
    a = await t1
    b = await t2
```

This is the crux of the topic: `await`ing coroutines one after another is just synchronous code with extra steps. Concurrency only appears once tasks are scheduled together.

## 4. Managing Multiple Tasks

### `asyncio.gather()`
Runs multiple awaitables concurrently and returns their results **in the order given** (not completion order).

```python
async def process_data(id):
    await asyncio.sleep(1)
    return f"Data {id} processed"

async def main():
    results = await asyncio.gather(
        process_data(1),
        process_data(2),
        process_data(3),
    )
    print(results)   # ['Data 1 processed', 'Data 2 processed', 'Data 3 processed']

asyncio.run(main())
```

**Error behavior:** by default, if one awaitable raises, `gather` propagates that exception immediately, but the *other* tasks keep running in the background (they are not cancelled). With `return_exceptions=True`, exceptions are returned as items in the result list instead of raised:

```python
results = await asyncio.gather(good(), bad(), return_exceptions=True)
# [ 'ok', ValueError('boom') ]
```

### `asyncio.TaskGroup` (Python 3.11+): Structured Concurrency
A safer alternative to `gather`. If any task in the group fails, the remaining tasks are **cancelled**, and the failures surface together as an `ExceptionGroup`. All tasks are guaranteed finished when the `async with` block exits.

```python
async def main():
    async with asyncio.TaskGroup() as tg:
        t1 = tg.create_task(process_data(1))
        t2 = tg.create_task(process_data(2))
    print(t1.result(), t2.result())   # both are done here
```

### `asyncio.as_completed()` and `asyncio.wait()`
- `as_completed(aws)` yields results in **completion order**, handy for processing whichever finishes first.
- `wait(tasks, return_when=FIRST_COMPLETED)` gives finer control over done/pending sets.

## 5. Real-World Example: Async HTTP Requests

Libraries like `requests` are **blocking**: they stop the whole event loop while waiting on the network. For async networking use an async library such as **`aiohttp`** (or `httpx`'s async client).

```python
import asyncio
import aiohttp
import time

async def fetch_status(session, url):
    async with session.get(url) as response:
        status = response.status
        print(f"Finished {url} with status {status}")
        return status

async def main():
    urls = ["https://google.com", "https://python.org", "https://github.com"]

    async with aiohttp.ClientSession() as session:
        tasks = [fetch_status(session, url) for url in urls]
        await asyncio.gather(*tasks)

if __name__ == "__main__":
    start = time.perf_counter()
    asyncio.run(main())
    print(f"Total time: {time.perf_counter() - start:.2f} seconds")
```

## 6. Timeouts and Cancellation

### Timeouts
Always bound network calls so one slow endpoint can't hang the pipeline.

```python
# Works on all supported versions
try:
    result = await asyncio.wait_for(fetch(), timeout=5)
except asyncio.TimeoutError:
    print("too slow")

# Python 3.11+: context-manager style
async with asyncio.timeout(5):
    result = await fetch()
```

### Cancellation
`task.cancel()` requests cancellation by raising `asyncio.CancelledError` **inside** the task at its current `await`. Cleanup goes in `try/finally` or an `except` that **re-raises**:

```python
async def worker():
    try:
        await asyncio.sleep(10)
    except asyncio.CancelledError:
        print("cleaning up")
        raise                      # always re-raise, or the cancellation is swallowed

async def main():
    task = asyncio.create_task(worker())
    await asyncio.sleep(1)
    task.cancel()
    try:
        await task
    except asyncio.CancelledError:
        print("worker was cancelled")
```

Note that `CancelledError` inherits from `BaseException` (not `Exception`) since Python 3.8, so a broad `except Exception:` won't accidentally swallow it.

## 7. Synchronization and Throttling

Unbounded concurrency is a bug: launching 10,000 requests at once can exhaust sockets or get you rate-limited.

### `asyncio.Semaphore`: Cap Concurrency
```python
sem = asyncio.Semaphore(10)     # at most 10 in flight

async def limited_fetch(session, url):
    async with sem:
        return await fetch_status(session, url)
```

### `asyncio.Lock`
Protects a critical section that spans an `await`, where two tasks could interleave. Lock use is rarer than in threading, but needed when shared state is modified across an `await`.

### `asyncio.Queue`: Async Producer-Consumer
```python
async def producer(q):
    for i in range(5):
        await q.put(i)
        await asyncio.sleep(0.1)

async def consumer(q):
    while True:
        item = await q.get()
        print(f"consumed {item}")
        q.task_done()

async def main():
    q = asyncio.Queue()
    consumer_task = asyncio.create_task(consumer(q))
    await producer(q)
    await q.join()               # wait until every item has been processed
    consumer_task.cancel()       # consumer loops forever, so stop it explicitly
```
This is the async analogue of `queue.Queue` from the threading notes, but it is **not** thread-safe. Use `asyncio.Queue` only within one event loop.

## 8. Async Iterators, Generators, and Context Managers

- **`async with`** calls `__aenter__`/`__aexit__`, which may themselves await (opening/closing a connection).
- **`async for`** iterates over an **async iterator** (`__aiter__`/`__anext__`), where each step can await.
- An **async generator** is an `async def` containing `yield`:

```python
async def ticker(n):
    for i in range(n):
        await asyncio.sleep(0.1)
        yield i

async def main():
    async for value in ticker(3):
        print(value)               # 0, 1, 2
```

## 9. Dealing with Blocking Code

If you must call blocking or CPU-heavy code from async code, push it off the event loop thread:

```python
import asyncio, time

def blocking_io():
    time.sleep(2)
    return "done"

async def main():
    # Python 3.9+
    result = await asyncio.to_thread(blocking_io)

    # Equivalent, older style (also used for a ProcessPoolExecutor for CPU-bound work)
    loop = asyncio.get_running_loop()
    result = await loop.run_in_executor(None, blocking_io)
```
`to_thread` uses a thread pool, which suits blocking **I/O**. For genuinely **CPU-bound** work, pass a `ProcessPoolExecutor` to `run_in_executor` instead, since threads are still limited by the GIL.

## 10. When to Use Async vs. Alternatives

| Model | Best Use Case | Performance Bottleneck |
| :--- | :--- | :--- |
| **Synchronous** | Simple scripts, linear logic. | I/O-bound (waiting for disk/network). |
| **Threading** | Blocking I/O libraries you can't change, GUI apps. | GIL for CPU work; per-thread memory and locking complexity. |
| **Multiprocessing** | Heavy math, image processing, CPU-bound work. | Memory overhead (each process has its own RAM), pickling costs. |
| **Asyncio** | Web servers (FastAPI), scraping, API integrations, many concurrent connections. | CPU-bound tasks (single core); requires async-compatible libraries throughout. |

**Function coloring:** async is "viral." An `async def` can only be awaited from another `async def`, so adopting it tends to spread through a codebase, and every I/O library on the hot path must have an async version.

## 11. Pro-Tips for Production

* **Never block the loop:** avoid `time.sleep()`, `requests.get()`, or heavy CPU logic inside `async def`. Use `asyncio.sleep()`, an async library, or `to_thread`/`run_in_executor`.
* **Always set timeouts** on network calls.
* **Bound your concurrency** with a `Semaphore` when fanning out over many items.
* **Use `async with`** for sessions and connections so they close even on error.
* **Keep references to tasks** you create with `create_task()`. The event loop holds only weak references, so a task with no other reference can be garbage-collected mid-run. Store them in a set/list or use a `TaskGroup`.
* **Don't call `asyncio.run()` from inside a running loop** (common in Jupyter, which already runs one). Use `await` directly there.

## Notes & Gotchas

- **Async = concurrency, not parallelism.** One thread, one core; the win is overlapping *waiting*, not computing.
- **`await` only yields control if the awaited thing actually suspends.** Awaiting a coroutine that never truly waits just runs it inline, so `await` alone doesn't make code concurrent; `create_task`/`gather`/`TaskGroup` do.
- **Forgetting `await` returns a coroutine object** and triggers a "never awaited" warning. The work silently doesn't happen.
- **A blocking call inside a coroutine freezes every task**, because scheduling is cooperative and nothing can preempt it.
- **`gather` doesn't cancel siblings on failure; `TaskGroup` does.** That difference is the core argument for structured concurrency.
- **`CancelledError` must be re-raised** after cleanup, and it derives from `BaseException`, not `Exception`.
- **Keep strong references to fire-and-forget tasks**, or they can be garbage-collected before finishing.
- **Threads vs. asyncio at scale:** each thread costs real OS memory and context-switch time, while a coroutine is a small heap object, which is why async scales to tens of thousands of concurrent connections.
