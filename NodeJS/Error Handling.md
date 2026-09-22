# Error Handling

Express errors click as: `wrap async` → `forward with next(err)` → `4-arg middleware answers`. Unhandled async rejections crash or hang — the wrapper pattern kills the whole class.

The core idea:

```text
Route throws / rejects
    ↓
next(err) → error middleware (err, req, res, next)
    ↓
status + { error } for Kakade clients
```

Sync throws are caught by Express; async needs the wrapper. 404 handler sits after routes, error middleware last.

---

# 1. Sync vs Async Errors, Async Wrapper Pattern, and Four-Arg Error Middleware

`Sync vs async errors`: Express catches sync throws automatically; async rejections escape to crash/hang. `Async wrapper pattern` (`ah(fn)`) forwards rejections to `next`. `Four-arg error middleware` (`(err, req, res, next)`) is recognized by arity — three args is just middleware.

```js
const ah = (fn) => (req, res, next) => Promise.resolve(fn(req, res, next)).catch(next);

async function getKakade(req, res) {
  throw new Error("DB down");
}

function errors(err, req, res, next) {
  console.error(err.message);
  res.status(500).json({ error: "Something broke for Kakade" });
}
```

Output:

```text
GET /me → 500 {"error":"Something broke for Kakade"} (logged server-side).
```

Wiring:

```js
// app.get("/me", ah(getKakade));
// app.use(errors); // LAST, after routes + 404.
```

Trap: error middleware registered before routes never runs — order is the feature.

---

# 2. 404 Handler and Logging Errors

`404 handler` (no-match middleware after routes) answers unknown paths with JSON instead of Express's HTML. `Logging errors` records stack + route + user server-side while clients get safe messages.

```js
import express from "express";

const app = express();
app.get("/hi", (req, res) => res.json({ hi: "Kakade" }));
app.use((req, res) => res.status(404).json({ error: "Not found" }));
app.use((err, req, res, next) => {
  console.error(`${req.method} ${req.url}: ${err.message}`);
  res.status(err.status || 500).json({ error: "Something broke for Kakade" });
});
app.listen(3000);
```

Output:

```text
GET /hi → Kakade; GET /nope → 404 JSON; crash → 500 JSON.
```

Think:

```text
routes → 404 (miss) → errors (boom)
  ✓        ✓              ✓ last
```

Never leak stacks or SQL to clients — log verbosely, respond generically.

---

# Real-World Uses

Kakade's API wraps every async route with `ah`, mounts 404 + error middleware last, and logs `{ method, url, stack }` to pino while clients see stable `{ error }` shapes.

```js
const ah = (fn) => (req, res, next) => Promise.resolve(fn(req, res, next)).catch(next);
console.log(typeof ah);
```

Output:

```text
function
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **Sync vs async errors** | Sync auto-caught, async escapes |
| **Async wrapper pattern** | `ah` forwards rejections to next |
| **Four-arg error middleware** | Arity 4 = error handler, last |
| **404 handler** | After routes, JSON not HTML |
| **Logging errors** | Verbose server-side, generic client-side |

## The Mental Model

```text
Request for Kakade
  │
  ├── route (ah-wrapped) → throws? next(err)
  │
  ├── no match? → 404 JSON
  │
  └── error middleware (last) → log stack → safe { error }
```

### The 3 things to remember

**1. Wrap async:**

```js
const ah = (fn) => (req, res, next) => Promise.resolve(fn(req, res, next)).catch(next);
```

**2. Catch-all last:**

```js
app.use((err, req, res, next) => {
  res.status(500).json({ error: "Something broke for Kakade" });
});
```

**3. 404 before errors:**

```js
app.use((req, res) => res.status(404).json({ error: "Not found" }));
```

> **Wrap async routes, mount 404 then errors last, and log verbosely while answering safely.**
