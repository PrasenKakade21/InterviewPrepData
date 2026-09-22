# REST APIs with Express

REST clicks as: `nouns in URLs` → `verbs in methods` → `status codes tell the story`. Resources (`/users`) with GET/POST/PUT/PATCH/DELETE — consistent shapes beat clever endpoints.

The core idea:

```text
/users (collection) + /users/:id (item)
    ↓
GET read, POST create, PUT/PATCH update, DELETE remove
    ↓
201/200/204/404 for Kakade clients
```

Plural nouns, JSON everywhere, and status codes clients can branch on.

---

# 1. Resources and Routes, CRUD with Router, and Params Query and Body

`Resources and routes`: collections (`GET /users`) vs items (`GET /users/:id`). `CRUD with Router` (`express.Router()`) groups one resource in its own file. `Params query and body`: `req.params` identifies, `req.query` filters, `req.body` carries payloads.

```js
import express from "express";

const users = [{ id: 1, name: "Kakade" }];
const router = express.Router();
router.get("/", (req, res) => res.json(users));
router.get("/:id", (req, res) => {
  const u = users.find((x) => x.id === Number(req.params.id));
  if (!u) return res.status(404).json({ error: "No Kakade here" });
  res.json(u);
});
const app = express();
app.use("/users", router);
app.listen(3000);
```

Output:

```text
GET /users → list; GET /users/1 → Kakade; GET /users/9 → 404 JSON.
```

Think:

```text
/users?q=kakade → query filters collection
/users/1 → param picks item
POST /users {name} → body creates
```

Rule: params for identity, query for filtering/sorting, body for creation — never identity in query when a param fits.

---

# 2. Status Code Conventions and JSON Response Shapes

`Status code conventions`: `201` created, `200` ok, `204` deleted (no body), `400` bad input, `404` missing. `JSON response shapes`: consistent envelopes (`{ data }` / `{ error }`) so clients parse once.

```js
router.post("/", (req, res) => {
  const u = { id: 2, name: req.body.name || "Kakade" };
  users.push(u);
  res.status(201).json(u);
});
router.delete("/:id", (req, res) => {
  const i = users.findIndex((x) => x.id === Number(req.params.id));
  if (i === -1) return res.status(404).json({ error: "No Kakade here" });
  users.splice(i, 1);
  res.status(204).end();
});
```

Output:

```text
POST /users → 201 + new user; DELETE /users/2 → 204 empty.
```

| Action | Code | Body |
| ------ | ---- | ---- |
| **Create** | **201** | **New resource** |
| **Read/update** | **200** | **Resource** |
| **Delete** | **204** | **Empty** |
| **Bad input** | **400** | **{ error }** |
| **Missing** | **404** | **{ error }** |

Interview trap: `PUT` replaces whole, `PATCH` updates partial — same URL, different semantics.

---

# Real-World Uses

Kakade's notes API mounts `/notes` and `/users` routers, returns 201 on create with the new object, and keeps every error as `{ error: "message" }` so the frontend has one parser.

```js
router.get("/", (req, res) => res.json([{ id: 1, name: "Kakade" }]));
```

Output:

```text
[{"id":1,"name":"Kakade"}] (collection envelope).
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **Resources and routes** | Plural nouns, collection vs item |
| **CRUD with Router** | One router per resource |
| **Params query and body** | Identity / filter / payload |
| **Status code conventions** | 201/200/204/400/404 |
| **JSON response shapes** | Consistent data/error envelopes |

## The Mental Model

```text
/users (GET list, POST create → 201)
  │
  └── /users/:id (GET → 200/404, PATCH → 200, DELETE → 204)
        ├── params = who, query = which, body = what
        └── errors always { error } for Kakade
```

### The 3 things to remember

**1. Router per resource:**

```js
const router = express.Router();
app.use("/users", router);
```

**2. Item lookup:**

```js
const u = users.find((x) => x.id === Number(req.params.id));
```

**3. Create + delete codes:**

```js
res.status(201).json(u);
res.status(204).end();
```

> **Nouns in URLs, verbs in methods, and status codes clients can trust.**
