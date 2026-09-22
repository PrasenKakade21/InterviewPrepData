# Testing APIs

Tests click as: `hit the route` → `assert status + body` → `mock the DB`. Test HTTP behavior, not internals — Kakade's client only sees status codes and JSON.

The core idea:

```text
supertest(app).get("/users")
    ↓
expect 200 + shape
    ↓
DB mocked → fast, isolated for Kakade
```

Install block (needed — third-party packages):

```bash
npm install -D vitest supertest
```

---

# 1. Vitest Basics and supertest for Routes

`Vitest basics` (`describe/it/expect`) run fast with ESM support. `supertest for routes` fires requests against the Express app without binding a port.

```js
import express from "express";
import request from "supertest";

const app = express();
app.get("/hi", (req, res) => res.json({ hi: "Kakade" }));
const res = await request(app).get("/hi");
console.log(res.status, res.body.hi);
```

Output:

```text
200 Kakade
```

Vitest shape:

```js
// import { describe, expect, it } from "vitest";
// describe("GET /hi", () => {
//   it("greets Kakade", async () => {
//     const res = await request(app).get("/hi");
//     expect(res.status).toBe(200);
//     expect(res.body.hi).toBe("Kakade");
//   });
// });
// Run: npx vitest run
```

Rule: one behavior per test; assert status first, then body — status tells Kakade's client what happened.

---

# 2. Mocking the Database and Coverage Basics

`Mocking the database` (stub model methods with `vi.fn()`) keeps route tests off real Mongo/Postgres. `Coverage basics` (`vitest run --coverage`) flags untested branches — gate critical paths, not 100% theater.

```js
const users = {
  find: async () => [{ name: "Kakade" }],
};
const list = await users.find();
console.log(list[0].name);
```

Output:

```text
Kakade
```

Mock sketch:

```js
// import { vi } from "vitest";
// vi.spyOn(User, "find").mockResolvedValue([{ name: "Kakade" }]);
// Restore after: vi.restoreAllMocks();
```

Coverage command:

```bash
npx vitest run --coverage
```

Mock at the model boundary (Mongoose/Prisma), never inside route logic — tests then cover real handler code with fake data.

---

# Real-World Uses

Kakade's CI runs vitest + supertest per PR: 201 on create, 404 on missing, 401 without token — DB mocked, coverage gating auth and validation paths.

```js
import express from "express";
import request from "supertest";

const app = express();
app.get("/missing", (req, res) => res.status(404).json({ error: "No Kakade here" }));
const res = await request(app).get("/missing");
console.log(res.status, res.body.error);
```

Output:

```text
404 No Kakade here
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **Vitest basics** | `describe/it/expect`, ESM-fast |
| **supertest for routes** | No port, assert status + body |
| **Mocking the database** | Stub models, cover handlers |
| **Coverage basics** | Gate critical paths |

## The Mental Model

```text
Route test for Kakade
  │
  ├── supertest(app) → status + body asserts
  │
  └── models mocked → fast, isolated
        └── coverage on auth + validation paths
```

### The 3 things to remember

**1. Fire without ports:**

```js
const res = await request(app).get("/hi");
```

**2. Assert status first:**

```js
expect(res.status).toBe(200);
```

**3. Stub data:**

```js
vi.spyOn(User, "find").mockResolvedValue([{ name: "Kakade" }]);
```

> **Test routes over HTTP with mocked data — status first, body second, DB never.**
