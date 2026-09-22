# Middleware

Middleware clicks as: `request enters` → `chain of (req, res, next)` → `route or error out`. Order is everything — Express runs middleware top-to-bottom in registration order.

The core idea:

```text
Request → use(logger) → use(json) → route
    ↓ forgot next()? → hangs forever
    ↓ threw? → error middleware
```

Every cross-cutting concern (logging, auth, parsing) is middleware; routes stay business-only.

---

# 1. app.use and Next, Middleware Order, and Built-in json and Static

`app.use and next`: `use(fn)` registers; `next()` passes control onward — omitting it stalls the request. `Middleware order`: JSON parser before routes, logger first, error handlers last. `Built-in json and static`: `express.json()` parses bodies; `express.static()` serves files.

```js
import express from "express";

const app = express();
app.use(express.json());
app.use((req, res, next) => {
  console.log(`${req.method} ${req.url} for Kakade`);
  next();
});
app.post("/echo", (req, res) => res.json({ you: req.body }));
app.listen(3000);
```

Output:

```text
POST /echo {"name":"Kakade"} → {"you":{"name":"Kakade"}} (logged first).
```

Order diagram:

```text
logger → json → auth → route → 404 → error
  1       2       3       4       5       6
```

Trap: `express.json()` after the route means `req.body` is `undefined` — parsers must precede users.

---

# 2. Custom Logger Middleware and Route-Level Middleware

`Custom logger middleware` stamps method/url/time per request. `Route-level middleware` (`app.get("/x", checkAuth, handler)`) guards single endpoints without touching the global chain.

```js
function logger(req, res, next) {
  const t = Date.now();
  res.on("finish", () => {
    console.log(`${req.method} ${req.url} ${Date.now() - t}ms for Kakade`);
  });
  next();
}
```

Output:

```text
GET /hi 3ms for Kakade (logged on finish).
```

Guard shape:

```js
// function onlyKakade(req, res, next) {
//   if (req.headers["x-user"] !== "Kakade") return res.status(401).json({ error: "Nope" });
//   next();
// }
// app.get("/admin", onlyKakade, (req, res) => res.json({ ok: true }));
```

Notice: middleware either responds or calls `next()` — doing both sends double responses and crashes.

---

# Real-World Uses

Kakade's API chains request-id → logger → json → auth globally, with per-route admin guards — each concern one small testable function.

```js
import express from "express";

const app = express();
app.use(express.json());
app.get("/ping", (req, res) => res.json({ pong: "Kakade" }));
app.listen(3000);
```

Output:

```text
GET /ping → {"pong":"Kakade"}.
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **app.use and next** | Register chain, pass control on |
| **Middleware order** | Logger → parsers → auth → routes → errors |
| **Built-in json and static** | Body parsing + file serving |
| **Custom logger middleware** | Method/url/time on finish |
| **Route-level middleware** | Per-endpoint guards |

## The Mental Model

```text
Request for Kakade
  │
  ├── global: logger → json → auth
  │
  ├── route (+ guards): handler → res.json
  │
  └── miss → 404 → error middleware
        └── no next()? → hangs
```

### The 3 things to remember

**1. Parse before routes:**

```js
app.use(express.json());
```

**2. Log + pass:**

```js
console.log(`${req.method} ${req.url} for Kakade`);
next();
```

**3. Guard routes:**

```js
app.get("/admin", onlyKakade, handler);
```

> **Order the chain deliberately, respond or next (never both), and keep routes business-only.**
