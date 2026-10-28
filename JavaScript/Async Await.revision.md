# Async Await — Revision Notes (Intermediate)

> One-line mental model: `async` wraps a return value in a Promise; `await` pauses only that function, then resumes in a microtask.

## Core recall

- **async function**: always returns a Promise; returned values are wrapped, returned Promises are adopted; thrown errors become rejections. The body runs **synchronously until the first `await`** — no new thread.
- **await**: unwraps a thenable; a rejection throws at the await site. Awaiting a plain value (`await null`) still yields one microtask.
- **try/catch with await**: a rejected Promise becomes a caught error; `finally` runs on both paths. Missing `await` means `catch` never sees the rejection.
- **Sequential vs parallel**: independent calls should start first and be awaited together; dependent calls stay in a loop. Three 100 ms calls = ~300 ms sequential vs ~100 ms with `Promise.all`.
- **Promise.all with await**: resolves to an input-order array; rejects on the first rejection; the other tasks keep running (no cancellation). `allSettled` reports every outcome; `any` gives the first success; `race` the first settle.
- **Top-level await**: ES modules only (`.mjs`, `"type": "module"`, `<script type="module">`). Pauses the module and its importers; `require()` of it throws `ERR_REQUIRE_ASYNC_MODULE`.
- **Async iteration**: `for await (const x of iterable)` pulls one Promise at a time; pair with `async function*` for pagination and streams — flat memory, natural backpressure.
- **Error handling**: `try/catch/finally` around awaits; `allSettled` when partial success is fine; never leave a fire-and-forget rejection unhandled (Node exits by default).
- **Common traps**: `forEach(async ...)` is not awaited; `map(async ...)` returns Promises; `return await` is redundant outside `try`; async functions cannot be constructed with `new`.
- **Performance**: parallelise independent I/O, keep CPU work off the main thread (Worker), cap concurrency, and pass an `AbortSignal` for real cancellation.

## Must-remember code

```javascript
// sequential vs parallel
for (const id of ids) await load(id);              // one at a time
await Promise.all(ids.map(load));                  // overlap independent calls
```

```javascript
// safe error handling
try {
  const user = await loadUser(id);
  console.log(user.name);
} catch (err) {
  console.error("failed:", err.message);
} finally {
  spinner.hide();
}
```

```javascript
// top-level await — app.mjs only
const config = await fetch("https://example.com/config.json").then((r) => r.json());
console.log("ready", config.env);
```

```javascript
// async iteration
async function* pages() {
  yield "page 1";
  await new Promise((r) => setTimeout(r, 10));
  yield "page 2";
}
for await (const page of pages()) console.log(page);
```

## Interview traps

- `await` delays only the async function; the caller keeps running → order is `3, 1, 4, 2` for the classic snippet.
- `forEach(async fn)` returns immediately and produces unhandled rejections → use `Promise.all(map)` or `for...of`.
- `Promise.all` does **not** cancel the losers; they keep running after the first rejection.
- A missing `await` logs a pending Promise and bypasses `try/catch`; the error surfaces as an unhandled rejection.
- Top-level await is a syntax error in CommonJS/classic scripts — it is module-only.
- `async` does not make a CPU loop non-blocking; use a Worker or chunk the work.
- Returning a Promise from an async function adopts it - you never get a nested Promise.
- `await` inside `reduce` without care serialises everything; build the Promises first, then `Promise.all`.

## 3 things to remember

1. **`async` returns a Promise; `await` unwraps it.**
   ```javascript
   async function get() { return 42; }
   get() instanceof Promise; // true
   await get();              // 42
   ```

2. **Parallelise independent work.**
   ```javascript
   await Promise.all(ids.map(load)); // ~100 ms
   for (const id of ids) await load(id); // ~300 ms
   ```

3. **Awaited or caught — never floating.**
   ```javascript
   load().catch(console.error);       // handled
   await load();                      // inside try/catch
   ```
