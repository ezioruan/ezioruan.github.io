---
title: "Async/Await: A Tale of Two Runtimes (Node.js vs. Python)"
date: 2021-12-15 11:00:00
tags: Python, Node.js, asyncio, Concurrency
---

Both Node.js and Python have embraced the `async/await` syntax to simplify asynchronous programming. While the keywords are the same and the high-level concept is similar—making asynchronous code look synchronous—the underlying models and ecosystems have key differences. Understanding these distinctions is crucial for developers working in both environments.

<!--more-->

### The Core Concept: The Event Loop

At the heart of both Node.js's and Python's `asyncio` is the **event loop**. It's a single-threaded construct that manages and distributes events or messages in a program. When an asynchronous operation (like a network request or a database query) is initiated, it's handed off to the system. The event loop is then free to run other code. Once the operation is complete, the event loop picks up the result and resumes the execution of the code that was waiting for it.

`async/await` is syntactic sugar that makes this process more intuitive. An `async` function always returns a promise-like object (a `Promise` in JavaScript, a `Coroutine` or `Future` in Python). The `await` keyword pauses the function's execution until that object is resolved.

### Node.js: Asynchronous by Default

Node.js was designed from the ground up to be non-blocking and asynchronous. Its entire standard library and the vast majority of third-party packages are built with this model in mind. This makes asynchronous programming feel very natural and integrated into the ecosystem.

**Key Characteristics in Node.js:**

*   **Pervasive Asynchronicity:** Almost all I/O operations are non-blocking by default (e.g., `fs.readFile`, `http.request`). You don't need a special library for async I/O.
*   **Promise-based:** Modern Node.js has fully embraced Promises. The `async/await` syntax is built directly on top of the Promise API.
*   **Single, Integrated Event Loop:** The JavaScript runtime (V8) has one primary event loop that handles everything. You don't manage it directly; it's just part of the environment.

```javascript
// javascript (Node.js)
const fs = require('fs.promises');

async function readFile() {
    try {
        const data = await fs.readFile('./file.txt', 'utf8');
        console.log(data);
    } catch (err) {
        console.error('Error:', err);
    }
}

readFile();
```

### Python: Asynchronous as an Option

In contrast, Python started as a synchronous language. Asynchronous capabilities were added later with the `asyncio` library. As a result, the async ecosystem is separate from the standard synchronous one.

**Key Characteristics in Python:**

*   **Explicit Async Ecosystem:** To perform non-blocking I/O, you must use `asyncio`-compatible libraries (e.g., `aiohttp` for HTTP requests instead of `requests`, `asyncpg` for PostgreSQL instead of `psycopg2`). Using a standard synchronous library inside an `async` function will block the event loop.
*   **Coroutine-based:** `async def` functions in Python create coroutine objects. While they serve a similar purpose to Promises, the terminology and some behaviors are different.
*   **Explicit Event Loop Management:** You typically start and run the event loop explicitly using `asyncio.run()`. This gives you more control but also adds a layer of boilerplate.

```python
# python
import asyncio
import aiofiles

async def read_file():
    try:
        async with aiofiles.open('./file.txt', mode='r') as f:
            contents = await f.read()
            print(contents)
    except IOError as err:
        print(f"Error: {err}")

asyncio.run(read_file())
```

### Key Differences Summarized

| Feature                  | Node.js                                        | Python (`asyncio`)                                     |
| ------------------------ | ---------------------------------------------- | ------------------------------------------------------ |
| **Core Philosophy**      | Asynchronous by default                        | Opt-in asynchronous model                              |
| **Standard Library**     | Most I/O is non-blocking                       | Standard I/O is blocking; requires `aio*` alternatives |
| **Underlying Primitive** | `Promise`                                      | `Coroutine` / `Future`                                 |
| **Event Loop**           | Implicit, always running                       | Explicitly managed with `asyncio.run()`                |
| **Ecosystem**            | Vast majority of libraries are async-aware     | Separate ecosystems for sync and async libraries       |

### Conclusion

While Node.js and Python share the `async/await` syntax, their approaches reflect their histories. Node.js offers a deeply integrated, "async-first" world. Python provides a powerful but separate `asyncio` ecosystem that you must consciously opt into. Both are incredibly effective for building high-performance, I/O-bound applications. For developers, the key is to recognize which environment you're in and choose the right libraries and patterns for the job, ensuring you never block the all-important event loop.