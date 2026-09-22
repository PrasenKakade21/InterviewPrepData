# Validation

Validation clicks as: `reject at the boundary` → `400 with details` → `handlers trust input`. Never let raw `req.body` reach business logic — schemas decide what is acceptable.

The core idea:

```text
Raw body/query/params
    ↓
zod schema.parse (throws on bad)
    ↓
typed clean data for Kakade handlers
```

Install block (needed — third-party package):

```bash
npm install zod
```

---

# 1. Why Validate at Boundary, zod Basics for Bodies, and 400 Error Patterns

`Why validate at boundary`: one check at entry beats scattered `if (!x)` in every handler. `zod basics for bodies` (`z.object`, `safeParse`) validate + type in one declaration. `400 error patterns`: return field-level details so clients fix requests fast.

```js
import { z } from "zod";

const User = z.object({ name: z.string().min(1), age: z.number().int().min(0) });
const ok = User.safeParse({ name: "Kakade", age: 30 });
console.log(ok.success);
const bad = User.safeParse({ name: "" });
console.log(bad.success);
```

Output:

```text
true
false
```

Route shape:

```js
// app.post("/users", (req, res) => {
//   const r = User.safeParse(req.body);
//   if (!r.success) return res.status(400).json({ error: r.error.flatten() });
//   res.status(201).json(r.data); // trusted for Kakade
// });
```

Think: `safeParse` returns `{ success, data/error }` — no try/catch needed for control flow.

---

# 2. Query Parsing and Pagination and Validating Params

`Query parsing and pagination` (`?page=&limit=`) coerces strings to numbers with defaults and caps. `Validating params` (`/users/:id`) rejects non-numeric ids before DB hits.

```js
import { z } from "zod";

const Page = z.object({
  page: z.coerce.number().int().min(1).default(1),
  limit: z.coerce.number().int().min(1).max(100).catch(10),
});
const q = Page.parse({ page: "2", limit: "200" });
console.log(q);
```

Output:

```text
{ page: 2, limit: 10 }
```

Param sketch:

```js
// const Id = z.object({ id: z.coerce.number().int().positive() });
// const { id } = Id.parse(req.params); // "abc" → 400 for Kakade.
```

Notice: `z.coerce` tames query strings (`"2"` → `2`); `.max(100)` caps abuse — pagination without caps is a DoS invitation.

---

# Real-World Uses

Kakade's API validates every POST body with zod, coerces pagination with caps, and returns `{ error: flattened }` 400s — handlers never see unvalidated input.

```js
import { z } from "zod";

const Note = z.object({ title: z.string().min(1) });
console.log(Note.safeParse({ title: "Kakade todo" }).success);
```

Output:

```text
true
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **Why validate at boundary** | One check beats scattered ifs |
| **zod basics for bodies** | `z.object` + `safeParse` |
| **400 error patterns** | Field details in `{ error }` |
| **Query parsing and pagination** | Coerce + default + cap |
| **Validating params** | Reject bad ids pre-DB |

## The Mental Model

```text
Raw input (body/query/params)
  │
  ├── zod schema → safeParse
  │     ├── fail → 400 { error } for Kakade
  │     └── pass → trusted data → handler
  │
  └── pagination: coerce + cap (max 100)
```

### The 3 things to remember

**1. Body schema:**

```js
const User = z.object({ name: z.string().min(1) });
```

**2. Safe gate:**

```js
const r = User.safeParse(req.body);
if (!r.success) return res.status(400).json({ error: r.error.flatten() });
```

**3. Paged query:**

```js
z.object({ page: z.coerce.number().int().min(1).default(1) });
```

> **Schemas at the boundary, 400s with details, and handlers that trust their input.**
