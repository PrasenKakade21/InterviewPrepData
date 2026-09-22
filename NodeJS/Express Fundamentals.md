# Express Fundamentals

Express clicks as: `app + routes` → `req/res helpers` → `listen`. It is thin routing over raw `http` — the most-interviewed Node framework by far.

The core idea:

```text
npm install express
    ↓
app.get/post ("/path", handler)
    ↓
app.listen(3000) serves Kakade
```

Install block (needed — third-party package):

```bash
npm install express
```

---

# 1. App and Listen, Basic Routes, and req and Res Helpers

`App and listen` (`express()` + `listen(port)`) boots the server. `Basic routes` (`app.get/post/put/delete`) map method+path to handlers. `req and res helpers` (`req.params/query/body`, `res.json/status/send`) replace raw header juggling.

```js
import express from "express";

const app = express();
app.get("/hi", (req, res) => {
  res.json({ hi: "Kakade" });
});
app.listen(3000, () => console.log("Express for Kakade on 3000"));
```

Output:

```text
Express for Kakade on 3000
```

Fetch check:

```js
// const r = await fetch("http://localhost:3000/hi");
// console.log(await r.json()); // { hi: "Kakade" }
```

Think: `res.json()` sets content-type + stringifies — one line replacing three raw-`http` lines.

---

# 2. Nodemon for Dev and Project Structure

`Nodemon for dev` (or `node --watch`) restarts on save — no manual kill/rerun loop. `Project structure` separates routes, middleware, and data so `server.js` stays thin.

```bash
npm install -D nodemon
```

```text
npx nodemon server.js → restarts for Kakade on every save.
```

Structure:

```text
kakade-api/
├── server.js (app + listen)
├── routes/ (users.js, notes.js)
├── middleware/ (logger.js, auth.js)
└── data/ (store.js)
```

Rule: `server.js` wires; `routes/` handles; nothing else lives at top level. Nodemon is dev-only — production uses `node server.js` or PM2.

---

# Real-World Uses

Kakade's todo API is Express with one router per resource, `node --watch` locally, and `npm start` in production — raw `http` knowledge reserved for debugging.

```js
import express from "express";

const app = express();
app.get("/me", (req, res) => res.json({ user: "Kakade" }));
app.listen(3000);
```

Output:

```text
GET /me → {"user":"Kakade"}.
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **App and listen** | `express()` + port boot |
| **Basic routes** | get/post/put/delete per path |
| **req and res helpers** | params/query/body, json/status |
| **Nodemon for dev** | Auto-restart on save |
| **Project structure** | server + routes + middleware |

## The Mental Model

```text
express()
  │
  ├── routes: get/post ("/x", handler)
  │     └── res.json for Kakade
  │
  └── listen(3000)
        ├── dev → --watch / nodemon
        └── prod → node server.js
```

### The 3 things to remember

**1. Boot:**

```js
const app = express();
```

**2. Route:**

```js
app.get("/hi", (req, res) => {
  res.json({ hi: "Kakade" });
});
```

**3. Serve:**

```js
app.listen(3000);
```

> **Routes map method+path to handlers, res.json answers, and structure keeps server.js thin.**
