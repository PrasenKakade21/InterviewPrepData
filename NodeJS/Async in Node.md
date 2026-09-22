# Async in Node

Async clicks as: `callback style is legacy` → `promises compose` → `async/await reads sync`. Node APIs come in all three shapes; promisify the old, await the new.

The core idea:

```text
Error-first callback (legacy)
    ↓
promise (.then / .catch)
    ↓
async/await + try/catch for Kakade
```

Write `async/await` everywhere; reach for `Promise.all` for parallel work.

---

# 1. Error-First Callbacks, Promises, and async and Await

`Error-first callbacks` (`(err, data) => …`) are the legacy Node convention — err always first. `Promises` represent future values with `.then/.catch`. `async and await` pause without blocking, with `try/catch` for failures.

```js
async function getUser(id) {
  if (id !== 1) throw new Error("Not found");
  return { id, name: "Kakade" };
}

const user = await getUser(1);
console.log(user);
```

Output:

```text
{ id: 1, name: 'Kakade' }
```

Callback shape (legacy):

```js
// fs.readFile("notes.txt", "utf8", (err, data) => {
//   if (err) return console.error(err);
//   console.log(data); // Kakade's notes
// });
```

Think: callbacks nest (pyramid of doom), promises chain, async/await reads top-to-bottom — migrate outward from the oldest code.

---

# 2. Promise.all and allSettled, Top-Level Await in ESM, and Callback to Promise with util.promisify

`Promise.all and allSettled`: `all` fails fast on first rejection; `allSettled` reports every outcome. `Top-level await in ESM` allows `await` at file scope (no wrapper). `Callback to promise with util.promisify` converts legacy APIs to awaitable ones.

```js
const a = Promise.resolve("Kakade");
const b = Promise.resolve("Pro");
const both = await Promise.all([a, b]);
console.log(both);
```

Output:

```text
[ 'Kakade', 'Pro' ]
```

Promisify sketch:

```js
// import { promisify } from "node:util";
// import fs from "node:fs";
// const readFile = promisify(fs.readFile);
// const notes = await readFile("notes.txt", "utf8");
```

| Tool | Use when |
| ---- | -------- |
| **`Promise.all`** | **All must succeed (fail fast)** |
| **`allSettled`** | **Need every result, tolerate fails** |
| **`promisify`** | **Legacy callback API, one-shot upgrade** |

Prefer `node:fs/promises` directly over promisifying `fs` — it is already promise-native.

---

# Real-World Uses

Kakade's bootstrap awaits config + DB in parallel with `Promise.all`, reads files via `fs/promises`, and wraps the one legacy SDK with `promisify` instead of nesting callbacks.

```js
const results = await Promise.allSettled([
  Promise.resolve("Kakade"),
  Promise.reject(new Error("oops")),
]);
console.log(results.map((r) => r.status));
```

Output:

```text
[ 'fulfilled', 'rejected' ]
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **Error-first callbacks** | `(err, data)`, legacy style |
| **Promises** | Future value + chain |
| **async and await** | Sync-looking, try/catch errors |
| **Promise.all and allSettled** | Fail-fast vs report-all |
| **Top-level await in ESM** | Await at file scope |
| **Callback to promise** | `util.promisify` upgrade path |

## The Mental Model

```text
Legacy callback (err, data)
  │
  ├── promisify → promise
  │
  └── async/await + try/catch
        ├── parallel? → Promise.all (strict)
        └── tolerant? → allSettled
```

### The 3 things to remember

**1. Await with catch:**

```js
const user = await getUser(1);
```

**2. Parallel strict:**

```js
const both = await Promise.all([a, b]);
```

**3. Legacy upgrade:**

```js
const readFile = promisify(fs.readFile);
```

> **Await by default, parallelize with all, tolerate with allSettled — promisify the past.**
