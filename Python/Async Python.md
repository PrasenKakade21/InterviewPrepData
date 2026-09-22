# Async Python

Async looks confusing, but once you understand **sync waits → async switches while waiting**, it clicks.

The core idea:

```text
Sync: task A waits → CPU idle
Async: task A waits → CPU runs task B
```

Use async for **I/O-bound** work (API calls, DB queries). Not for CPU math.

```bash
pip install httpx
```

---

# 1. Synchronous vs Asynchronous Code

Sync: one thing at a time, waits block.

```python
import time


def fetch(user):
    time.sleep(1)
    return f"done {user}"


start = time.time()
print(fetch("Kakade"))
print(fetch("Asha"))
print(time.time() - start)
```

Output:

```text
done Kakade
done Asha
2.0...
```

2 seconds total. Second call waits for first.

Async version (preview) finishes both in ~1 second by overlapping waits.

Rule:

```text
CPU-bound (math) → threads/multiprocessing
I/O-bound (wait) → async
```

---

# 2. async def

Marks a function as coroutine. Calling it does NOT run it — returns coroutine object.

```python
async def greet():
    return "Hello Kakade"


print(greet())
```

Output:

```text
<coroutine object greet at ...>
```

Need `await` or `asyncio.run()` to execute. Forgetting this is the #1 beginner error.

---

# 3. await

`await` pauses current coroutine until result ready, letting event loop run others.

```python
import asyncio


async def greet():
    await asyncio.sleep(1)
    return "Hello Kakade"


print(asyncio.run(greet()))
```

Output:

```text
Hello Kakade
```

Think:

```text
await asyncio.sleep(1)
    ↓
pause here, run other tasks
    ↓
resume after 1s
```

`await` only works inside `async def`.

---

# 4. asyncio

`asyncio` is stdlib async framework. Entry point: `asyncio.run()`.

```python
import asyncio


async def main():
    print("Hello Kakade")


asyncio.run(main())
```

Output:

```text
Hello Kakade
```

One `asyncio.run()` per program (it creates and closes the loop). Inside FastAPI, loop already runs — just use `async def` routes.

---

# 5. Event Loop

Loop is a scheduler: tracks paused coroutines, resumes when ready.

```text
Task A waits for API
    ↓
loop runs Task B
    ↓
A ready → loop resumes A
```

You rarely touch loop directly. Know the term for interviews: "single-threaded scheduler multiplexing I/O."

---

# 6. Coroutines

Coroutine = paused-able function. Two types:

```python
async def fetch_user():
    await asyncio.sleep(1)
    return {"name": "Kakade"}
```

Calling `fetch_user()` creates coroutine. `await` executes it.

```text
def → returns value
async def → returns coroutine, needs await
```

---

# 7. Tasks

`Task` schedules coroutine to run concurrently.

```python
import asyncio


async def fetch(user):
    await asyncio.sleep(1)
    return f"done {user}"


async def main():
    task1 = asyncio.create_task(fetch("Kakade"))
    task2 = asyncio.create_task(fetch("Asha"))
    print(await task1)
    print(await task2)


asyncio.run(main())
```

Output (total ~1s, not 2s):

```text
done Kakade
done Asha
```

Think:

```text
create_task() → start in background
await task → wait for result
```

---

# 8. Concurrency

Concurrency ≠ parallelism.

```text
Concurrency (async)
    ↓
one thread, switches on wait

Parallelism (threads/processes)
    ↓
multiple threads/cores, truly simultaneous
```

Async wins when tasks spend time waiting. Loses for heavy math (use multiprocessing).

---

# 9. Concurrent I/O

Sequential (slow):

```python
async def main():
    a = await fetch("Kakade")  # 1s
    b = await fetch("Asha")    # +1s = 2s
```

Concurrent (fast):

```python
async def main():
    a, b = await asyncio.gather(
        fetch("Kakade"),
        fetch("Asha")
    )  # ~1s total
```

Same work, half time. This is the entire pitch for async APIs.

---

# 10. asyncio.gather()

Runs many coroutines, returns results in order.

```python
import asyncio


async def fetch(n):
    await asyncio.sleep(1)
    return n * 10


async def main():
    results = await asyncio.gather(fetch(1), fetch(2), fetch(3))
    print(results)


asyncio.run(main())
```

Output:

```text
[10, 20, 30]
```

If one fails, others cancel unless `return_exceptions=True`.

```python
results = await asyncio.gather(fetch(1), fetch(2), return_exceptions=True)
```

---

# 11. Async HTTP

`requests` blocks the loop. Use `httpx` async client.

```python
import asyncio
import httpx


async def fetch(user_id: int):
    async with httpx.AsyncClient() as client:
        r = await client.get(
            f"https://jsonplaceholder.typicode.com/users/{user_id}",
            timeout=5
        )
        return r.json()["name"]


async def main():
    names = await asyncio.gather(fetch(1), fetch(2))
    print(names)


asyncio.run(main())
```

Output:

```text
['Leanne Graham', 'Ervin Howell']
```

Rules:

```python
httpx.get() → sync, blocks loop, avoid in async
httpx.AsyncClient() → async, use with await
```

FastAPI `async def` routes + `httpx` = concurrent API fan-out.

---

# 12. httpx

`httpx` mirrors `requests` API, sync + async.

```python
import httpx

r = httpx.get("https://jsonplaceholder.typicode.com/users/1", timeout=5)

print(r.status_code)
print(r.json()["name"])
```

Output:

```text
200
Leanne Graham
```

| Library | Sync | Async |
| ------- | ---- | ----- |
| `requests` | Yes | No |
| `httpx` | Yes | Yes |

New code talking to APIs: prefer `httpx` so sync/async both work.

---

# 13. Async Database Operations

Async DB needs async driver + async session.

Concept (needs `asyncpg` for Postgres):

```python
from sqlalchemy.ext.asyncio import create_async_engine, async_sessionmaker

engine = create_async_engine("postgresql+asyncpg://localhost/mydb")
Session = async_sessionmaker(engine)


async def get_user(user_id: int):
    async with Session() as session:
        return await session.get(User, user_id)
```

Think:

```text
await session.get() → pause while DB responds
loop serves other requests meanwhile
```

Don't mix sync `sqlite3` calls inside `async def` — they block the loop. Use async driver or run sync code in threadpool.

---

# Real-World Uses

```python
FastAPI async route → gather 3 API calls + 2 DB queries concurrently
```

Typical backend: one request fans out to auth service, tasks service, and DB — all overlap, total time = slowest, not sum.

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **Sync vs async** | Async overlaps waits |
| **async def** | Returns coroutine |
| **await** | Pause, let others run |
| **asyncio** | `asyncio.run(main())` entry |
| **Loop** | Schedules resumption |
| **Coroutines** | Pausable functions |
| **Tasks** | `create_task()` background |
| **Concurrency** | One thread, many waits |
| **Concurrent I/O** | Overlap, not sequence |
| **gather()** | Run many, ordered results |
| **Async HTTP** | `httpx.AsyncClient` |
| **httpx** | Requests-compatible + async |
| **Async DB** | Async driver + `await session` |

## The Mental Model

```text
        SYNC (2s)
   A wait 1s → B wait 1s

        ASYNC (1s)
   A wait ─┐
           ├→ overlap
   B wait ─┘

async def → coroutine
    ↓
await → pause/resume
    ↓
create_task/gather → concurrent
    ↓
httpx + async DB → no blocking
```

### The 3 things to remember

**1. Define and run:**

```python
async def main():
    await asyncio.sleep(1)

asyncio.run(main())
```

**2. Overlap waits:**

```python
await asyncio.gather(fetch(1), fetch(2))
```

**3. Never block loop:**

```python
async with httpx.AsyncClient() as c:
    await c.get(url)
```

> **Async Python overlaps waiting: mark I/O with async/await, fan out with gather, and use httpx plus async drivers so the loop never blocks.**
