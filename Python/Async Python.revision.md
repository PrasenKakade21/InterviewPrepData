# Async Python — Revision Notes (Intermediate)
> One-line mental model: Sync idles while waiting → async switches to other tasks while waiting.
## Core recall
- I/O-bound (API/DB wait) → async; CPU math → threads/multiprocessing.
- `async def` returns coroutine (not result); `await` pauses, lets loop run others; `await` only in `async def`.
- `asyncio.run(main())` one entry per program; FastAPI already runs loop (just use `async def` routes).
- Event loop: single-thread scheduler resuming coroutines when ready.
- `asyncio.create_task(coro)` background; `await task` collects result.
- Concurrency (one thread switching) ≠ parallelism (multi-core simultaneous).
- Sequential `await a; await b` = 2s; `await asyncio.gather(a,b)` = ~1s, ordered results.
- `gather(..., return_exceptions=True)` prevents one failure cancelling all.
- HTTP: `httpx.AsyncClient` + `await`; never blocking `requests`/`httpx.get()` inside async.
- DB: async driver (`asyncpg`) + `await session.get()`; sync `sqlite3` blocks loop → threadpool or async driver.
## Must-remember code
```python
import asyncio
async def greet():
    await asyncio.sleep(1); return "Hello Kakade"
print(asyncio.run(greet()))
```
```python
async def main():
    a, b = await asyncio.gather(fetch("Kakade"), fetch("Asha"))
```
```python
import httpx
async with httpx.AsyncClient() as c:
    r = await c.get("https://jsonplaceholder.typicode.com/users/1", timeout=5)
    print(r.json()["name"])
```
```python
async def main():
    t1 = asyncio.create_task(fetch("Kakade"))
    print(await t1)
```
## Interview traps
- Calling `async def` without `await` → coroutine object, nothing runs.
- Blocking call (`time.sleep`, `requests.get`) in async freezes all requests → `asyncio.sleep`, `httpx.AsyncClient`.
- `await` serially instead of `gather` → 2x slow fan-out.
- Multiple `asyncio.run()` in one program / nested in Jupyter → RuntimeError; one loop only.
- `gather` cancels siblings on error → `return_exceptions=True` for independent calls.
## 3 things to remember
1. `async def` + `await` + `asyncio.run()`.
2. Overlap waits with `gather` / `create_task`.
3. Never block loop: async HTTP + async DB drivers.
