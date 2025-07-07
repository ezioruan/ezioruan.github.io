---
title: "Python's Async/Await: Common Pitfalls and How to Avoid Them"
date: 2021-11-12 09:20:00
tags: Python, asyncio, Concurrency
---

Python's `asyncio` library, along with the `async` and `await` keywords, has brought powerful concurrency capabilities to the language. It allows developers to write highly scalable I/O-bound applications. However, the asynchronous paradigm introduces new challenges and potential pitfalls that can trip up even experienced developers. This article highlights some of the most common traps and how to avoid them.

<!--more-->

### Pitfall 1: Forgetting to `await` a Coroutine

This is perhaps the most common mistake. When you call a coroutine function (defined with `async def`), it doesn't execute the function. Instead, it returns a coroutine object. If you forget to `await` it, the code inside the coroutine will never run.

**The Trap:**
```python
import asyncio

async def my_coroutine():
    print("Coroutine is running!")
    await asyncio.sleep(1)
    print("Coroutine finished.")

async def main():
    my_coroutine() # Forgot to await!
    print("Main function finished.")

asyncio.run(main())
```

**Output:**
```
Main function finished.
<stdin>:11: RuntimeWarning: coroutine 'my_coroutine' was never awaited
```
Notice that "Coroutine is running!" is never printed. Python's runtime is smart enough to warn you about this, but it's crucial to understand why it happens.

**The Fix:**
Always use `await` when you call a coroutine.
```python
async def main():
    await my_coroutine() # Correctly awaited
    print("Main function finished.")
```

### Pitfall 2: Blocking the Event Loop

The entire `asyncio` model is built on a single-threaded event loop. This loop is responsible for scheduling and running all your asynchronous tasks. If you run a long-running, synchronous (blocking) function in your async code, it will block the entire event loop. No other tasks can run until the blocking call is finished.

**The Trap:**
```python
import asyncio
import time

async def other_task():
    print("Other task starts")
    await asyncio.sleep(1)
    print("Other task ends")

async def main():
    asyncio.create_task(other_task())
    print("Main: starting blocking call")
    time.sleep(5) # Blocking call!
    print("Main: finished blocking call")

asyncio.run(main())
```

**Output:**
```
Main: starting blocking call
# (5-second pause)
Main: finished blocking call
Other task starts
Other task ends
```
`other_task` couldn't even start until the `time.sleep(5)` was complete, defeating the purpose of concurrency.

**The Fix:**
For I/O-bound operations, use `asyncio`-compatible libraries (e.g., `aiohttp` instead of `requests`, `asyncpg` instead of `psycopg2`). For CPU-bound work, run it in a separate process or thread pool using `asyncio.to_thread` (Python 3.9+) or `loop.run_in_executor()`.

```python
# For Python 3.9+
await asyncio.to_thread(time.sleep, 5)
```

### Pitfall 3: Not Running Tasks Concurrently with `asyncio.gather`

Simply using `await` one after another on multiple coroutines will execute them sequentially, not concurrently.

**The Trap (Sequential Execution):**
```python
async def fetch_data(url):
    print(f"Fetching {url}...")
    await asyncio.sleep(2)
    print(f"Done fetching {url}.")
    return {url: "some data"}

async def main():
    result1 = await fetch_data("url1")
    result2 = await fetch_data("url2")
    # Total time will be ~4 seconds
```

**The Fix (Concurrent Execution):**
To run tasks concurrently, you should use `asyncio.gather` or `asyncio.create_task`.

```python
async def main():
    # Create tasks to run them concurrently
    task1 = asyncio.create_task(fetch_data("url1"))
    task2 = asyncio.create_task(fetch_data("url2"))

    # Wait for both to complete
    results = await asyncio.gather(task1, task2)
    # Total time will be ~2 seconds
```

### Conclusion

`asyncio` is a powerful tool, but it requires a shift in mindset. By being mindful of these common pitfalls—always awaiting coroutines, avoiding blocking calls, and using tools like `asyncio.gather` for concurrency—you can write efficient, scalable, and correct asynchronous Python code.