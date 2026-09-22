# Projects

Projects click as: `CRUD first` → `auth + DB` → `deployed with docs`. Each build stacks earlier topics — routing, validation, errors, tests, and Docker compound into proof.

The core idea:

```text
Todo API → routes + validation
    ↓
notes/blog/shop → auth + Mongo/Postgres
    ↓
deployed portfolio API for Kakade
```

Ship every project with README, tests, and a live URL.

---

# 1. Todo API and Notes API with Auth

`Todo API` (in-memory CRUD with zod + error middleware) proves REST fundamentals. `Notes API with auth` adds bcrypt login, JWT guard, and per-user data — Kakade only sees her notes.

```js
import express from "express";

const app = express();
app.use(express.json());
const todos = [{ id: 1, title: "Learn Node", owner: "Kakade" }];
app.get("/todos", (req, res) => res.json(todos));
const server = app.listen(3000);
const res = await fetch("http://localhost:3000/todos");
console.log((await res.json()).length);
server.close();
```

Output:

```text
1
```

Notes shape:

```js
// POST /auth/login → { token }; GET /notes (guard) → Kakade's notes only.
// Owner filter: Note.find({ owner: req.user.id }) — never all notes.
```

These two prove routes + validation + auth — the hiring filter.

---

# 2. Blog API with MongoDB and Shop API with Postgres

`Blog API with MongoDB` (Mongoose posts + comments, population) proves document modeling. `Shop API with Postgres` (Prisma products + orders, relations) proves relational modeling — one of each answers any stack question.

```js
import mongoose from "mongoose";

const post = new mongoose.Schema({
  title: { type: String, required: true },
  author: { type: String, default: "Kakade" },
});
console.log(Object.keys(post.paths).join(","));
```

Output:

```text
title,author,_id
```

Shop sketch:

```prisma
model Product {
  id    Int     @id @default(autoincrement())
  name  String
  price Int
  orders OrderItem[]
}
```

Both need pagination, search, tests, and Docker Compose for reviewers to run in one command.

---

# 3. Deployed Portfolio API

`Deployed portfolio API`: one polished build — auth, uploads, caching, docs (README + OpenAPI sketch), CI tests, Docker image, live URL.

```text
[ ] Live URL + /health green for Kakade
[ ] README: stack, runs, screenshots
[ ] Tests: 201/404/401 paths covered
[ ] Dockerfile + Compose one-command boot
[ ] Rate limits, helmet, validation on
```

One deployed API beats five localhost demos — interviewers click links.

```js
console.log("Portfolio API live for Kakade");
```

Output:

```text
Portfolio API live for Kakade
```

---

# Real-World Uses

Kakade ships todo → notes → blog → shop → portfolio, each deployed with env configs and a one-paragraph README story — the full track in five links.

```js
const stack = ["todo", "notes", "blog", "shop", "portfolio"];
console.log(stack.length, "projects for Kakade");
```

Output:

```text
5 projects for Kakade
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **Todo API** | CRUD + validation + errors |
| **Notes API with auth** | JWT guard + owner filter |
| **Blog API with MongoDB** | Documents + population |
| **Shop API with Postgres** | Relations + transactions |
| **Deployed portfolio API** | Live + docs + Docker |

## The Mental Model

```text
Foundations (todo/notes)
  │
  ├── data (blog/Mongo, shop/Postgres)
  │
  └── portfolio (deployed, tested, docs)
        └── every app: 201/404/401 + live link
```

### The 3 things to remember

**1. Start CRUD:**

```js
app.get("/todos", (req, res) => res.json(todos));
```

**2. Scope data:**

```js
Note.find({ owner: req.user.id });
```

**3. Ship live:**

```bash
docker build -t kakade-api .
```

> **Build progressively harder APIs and deploy every one with tests, README, and a live URL.**
