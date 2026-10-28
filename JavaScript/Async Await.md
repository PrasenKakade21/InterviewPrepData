# Async Await

An `async` function always returns a **Promise**, and `await` pauses **that function only** until a Promise settles. The chain is **async function → await a Promise → resume as a microtask → return a Promise**, and interviews test what runs while you wait, what runs after, and when to parallelise.

```text
async function
    ↓
returns a Promise immediately
    ↓
await pauses THIS function only
    ↓
other code keeps running
    ↓
Promise settles → resume in a microtask
    ↓
return value → resolved Promise
```

`async`/`await` is syntax over Promises, not a new thread — it makes waiting readable, not faster.

---

# 1. async and await

### async functions

Adding `async` does three things: it wraps the return value in a resolved Promise, converts a thrown error into a rejection, and allows `await`. It does **not** run the function on another thread.

```javascript
async function greet(name) {
  return `Hello ${name}`;
}

const result = greet("Kakade");
console.log(result instanceof Promise);
result.then((message) => console.log(message));
```

Output:

```text
true
Hello Kakade
```

Notice: `greet("Kakade")` does not hand you a string. A non-async caller must `await` it or attach `.then`. Returning a Promise from an async function **adopts** it, so `return Promise.resolve(1)` still resolves to `1`, not to a nested Promise.

### await

`await` waits for a thenable and evaluates to its fulfilled value; if it rejects, the `await` expression throws. Awaiting a plain value still costs one microtask tick because it is wrapped in `Promise.resolve`.

```javascript
const wait = (ms) => new Promise((resolve) => setTimeout(resolve, ms));

async function load() {
  console.log("loading");
  await wait(50);
  console.log("done");
}

load();
console.log("after load()");
```

Output:

```text
loading
after load()
done
```

Think:

```text
load() called
   │
   ├── runs until first await  → prints "loading"
   │
   ├── returns a pending Promise to caller
   │
   └── caller continues → prints "after load()"
             │
             └── timer fires → resume load() → prints "done"
```

`await` only suspends the async function. The rest of the program — the caller, timers, events — keeps running, which is why `"after load()"` appears before `"done"`.

---

# 2. Handling Errors

### try/catch with await

A rejected Promise becomes a thrown error at the `await`, so a normal `try/catch` works. This is the main win over `.then().catch()` chains.

```javascript
async function loadUser() {
  try {
    const res = await Promise.reject(new Error("404"));
    console.log(res);
  } catch (err) {
    console.log("caught:", err.message);
  } finally {
    console.log("cleanup");
  }
}

loadUser();
```

Output:

```text
caught: 404
cleanup
```

Bad: `loadUser().then(...)` with no `.catch` → an unhandled rejection. Good: `try/catch` around the awaits, or a `.catch` on the returned Promise. `finally` runs on both success and failure — use it to hide a spinner.

### Error handling

`Promise.all` rejects as soon as one input rejects, and the other results are discarded. When every result matters, use `Promise.allSettled`, which never rejects and reports `status` per item.

```javascript
async function load(id) {
  if (id === 2) throw new Error("user 2 missing");
  return { id, name: id === 1 ? "Kakade" : "Rohan" };
}

(async () => {
  const results = await Promise.allSettled([load(1), load(2), load(3)]);
  for (const r of results) {
    console.log(r.status, r.status === "fulfilled" ? r.value.name : r.reason.message);
  }
})();
```

Output:

```text
fulfilled Kakade
rejected user 2 missing
fulfilled Rohan
```

Notice: the loop sees results **in input order**, not completion order. In Node, an unhandled rejection terminates the process by default; in browsers it logs. Always handle or await the rejection.

---

# 3. Concurrency

### Sequential vs parallel

Independent awaits written one after another are **sequential**: each waits for the previous one. Start the Promises first, then `await` them together, and the waits overlap.

```javascript
const wait = (ms, label) =>
  new Promise((resolve) => setTimeout(() => resolve(label), ms));

async function sequential() {
  const a = await wait(60, "Kakade");
  const b = await wait(40, "Asha");
  console.log("sequential:", a, b);
}

async function parallel() {
  const [a, b] = await Promise.all([wait(60, "Kakade"), wait(40, "Asha")]);
  console.log("parallel:", a, b);
}

sequential().then(parallel);
```

Output:

```text
sequential: Kakade Asha
parallel: Kakade Asha
```

| Approach | Starts | Approx. time |
| --- | --- | --- |
| `await` one by one | each after the previous resolves | 100 ms |
| `Promise.all` | all together | 60 ms |

`Promise.all` preserves input order, so `[a, b]` is stable even when `b` finishes first.

### Promise.all with await

`Promise.all` takes an iterable of Promises and resolves with an array of values. It rejects on the **first** rejection, but the other Promises are not cancelled — they keep running and their results are ignored.

```javascript
const ok = (value) => Promise.resolve(value);
const fail = (message) => Promise.reject(new Error(message));

async function run() {
  try {
    const values = await Promise.all([
      ok("Kakade"),
      fail("Asha failed"),
      ok("Rohan"),
    ]);
    console.log(values);
  } catch (err) {
    console.log("stopped:", err.message);
  }
}

run();
```

Output:

```text
stopped: Asha failed
```

Think:

```text
Promise.all([p1, p2, p3])
        │
        ├── all start immediately
        ├── p2 rejects  →  the await throws
        └── p1 / p3 keep running (no cancellation)
```

If you need every outcome, use `allSettled`; if you need the first success, use `Promise.any`; if you need the fastest (success or failure), use `Promise.race`.

### Performance

Sequential awaits multiply latency; parallel awaits overlap it. A quick way to prove the difference is to measure peak concurrency instead of wall-clock time, which keeps the output stable.

```javascript
const wait = (ms) => new Promise((resolve) => setTimeout(resolve, ms));
let inFlight = 0;
let peak = 0;

async function task() {
  inFlight++;
  peak = Math.max(peak, inFlight);
  await wait(20);
  inFlight--;
}

async function run(parallel) {
  peak = 0;
  const ids = [1, 2, 3];
  if (parallel) await Promise.all(ids.map(task));
  else for (const id of ids) await task(id);
  console.log(parallel ? "Promise.all" : "for await", "peak:", peak);
}

(async () => {
  await run(false);
  await run(true);
})();
```

Output:

```text
for await peak: 1
Promise.all peak: 3
```

Notice: `await` inside `for...of` is correct when each step depends on the previous one (pagination, rate limits). It is wasteful when the calls are independent. For hundreds of requests, cap concurrency in batches rather than firing all of them at once.

---

# 4. Modules and Iteration

### Top-level await

`await` at the top level is allowed only in **ES modules** — a `.mjs` file, a package with `"type": "module"`, or `<script type="module">` in a browser. It pauses the module's evaluation and anything importing it, so importers see a fully initialised module.

```javascript
// app.mjs — run with: node app.mjs
const wait = (ms) => new Promise((resolve) => setTimeout(resolve, ms));

const user = await Promise.resolve("Kakade");
await wait(10);
console.log("ready:", user);
```

Output:

```text
ready: Kakade
```

Bad: putting that code in a CommonJS `.js` file → `SyntaxError: await is only valid in async functions`. Loading such a module with `require()` also fails (`ERR_REQUIRE_ASYNC_MODULE`); use `await import()` from an async context instead. Top-level await suits config and connections but delays every importer.

### Async iteration

`for await...of` consumes an async iterable: each `next()` returns a Promise, and the loop awaits it before running the body. Async generators (`async function*`) are the easiest way to produce one.

```javascript
async function* pages() {
  yield "page 1: Kakade";
  await new Promise((resolve) => setTimeout(resolve, 10));
  yield "page 2: Asha";
}

async function read() {
  for await (const page of pages()) {
    console.log(page);
  }
}

read();
```

Output:

```text
page 1: Kakade
page 2: Asha
```

This is the right shape for paginated APIs, Node streams and large files: the next chunk is fetched only after the previous one is handled, so memory stays flat. A plain `for...of` over the same generator would give you Promises instead of values.

---

# 5. Traps

### Common traps

The most common async bug is using `forEach` with an async callback: `forEach` ignores the returned Promises, so nothing is awaited and the loop "finishes" immediately.

```javascript
const ids = [1, 2, 3];
const fetchName = async (id) => {
  await new Promise((resolve) => setTimeout(resolve, 10));
  return `user-${id}`;
};

ids.forEach(async (id) => console.log(await fetchName(id)));
console.log("forEach done");

(async () => {
  const names = await Promise.all(ids.map(fetchName));
  console.log("collected:", names.join(", "));
})();
```

Output:

```text
forEach done
user-1
user-2
user-3
collected: user-1, user-2, user-3
```

Bad:

```javascript
const names = ids.map(fetchName); // [Promise, Promise, Promise]
for (const id of ids) await fetchName(id); // sequential, slower than needed
```

Good:

```javascript
const names = await Promise.all(ids.map(fetchName));
```

Other traps:

- **Missing `await`** → you log a pending Promise, and a `try/catch` never sees the rejection.
- **`await` in a loop when calls are independent** → serial latency; use `Promise.all` or batches.
- **`return await`** is redundant outside `try/catch`, but inside `try` it keeps the rejection in that scope.
- **Async functions cannot be constructors** — `new` on one throws `TypeError`.
- **`await` in a non-async function** is a syntax error, not a runtime one.
- **Cancellation is manual** — `Promise.all` does not stop the other tasks; pass an `AbortSignal` to `fetch`.

---

# Real-World Uses

- **API calls** — `await fetch(...)` reads top-to-bottom instead of nested callbacks.
- **Startup** — top-level await for config, database clients, feature flags.
- **Parallel dashboards** — `Promise.all` for profile, orders and notifications at once.
- **Pagination** — `for await` over an async generator of pages.
- **Graceful degradation** — `allSettled` so one failed widget does not blank the page.

---

# Quick Revision

| Concept | Remember |
| --- | --- |
| **async functions** | Always return a Promise; thrown errors become rejections; no new thread |
| **await** | Pauses only the async function; resumes in a microtask; throws on rejection |
| **try/catch with await** | Rejected Promise becomes a thrown error; `finally` always runs |
| **Sequential vs parallel** | Independent calls start together, then `Promise.all`; dependent calls await in order |
| **Promise.all with await** | Input-order results; fails fast on first rejection; other tasks keep running |
| **Top-level await** | ES modules only (`.mjs`, `type="module"`); delays importers |
| **Async iteration** | `for await...of` + async generators; one chunk at a time |
| **Error handling** | `allSettled` for every outcome; never leave a rejection unhandled |
| **Common traps** | `forEach(async ...)` is not awaited; `map` returns Promises; missing `await` hides errors |
| **Performance** | One `await` per dependency; parallelise independent work; cap concurrency |

## The Mental Model

```text
                    async function
                          │
                          ↓
                 returns a Promise
                          │
              ┌───────────┴───────────┐
              ↓                       ↓
          await p1                 await p2
       (pause this fn)          (pause this fn)
              │                       │
              └───────────┬───────────┘
                          ↓
                microtask resumes
                          │
              ┌───────────┴───────────┐
              ↓                       ↓
        value → return          throw → reject
              │                       │
              └───────────┬───────────┘
                          ↓
                 caller's Promise settles

  Independent? → start both, then Promise.all
  Dependent?   → await one, then the next
```

### The 3 things to remember

**1. `async` returns a Promise, `await` unwraps it.**

```javascript
async function get() { return 42; }
get() instanceof Promise; // true
await get();              // 42 (inside an async function)
```

**2. Parallelise independent work.**

```javascript
// slow: 300 ms
for (const id of ids) await load(id);
// fast: about 100 ms
await Promise.all(ids.map(load));
```

**3. Never fire and forget.**

```javascript
load().catch((err) => console.error(err)); // handled
await load();                              // or await inside try/catch
```

> **`async`/`await` is Promise syntax: it pauses one function, not the program — so `await` what you depend on and run the rest in parallel.**
