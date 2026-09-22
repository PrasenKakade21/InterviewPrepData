# MongoDB with Mongoose

Mongoose clicks as: `connect once` → `schema shapes docs` → `model CRDs`. Mongo stores flexible JSON-like docs; Mongoose adds validation, types, and query helpers on top.

The core idea:

```text
mongoose.connect(uri)
    ↓
schema → model ("User")
    ↓
create/find/update for Kakade docs
```

Install block (needed — third-party package):

```bash
npm install mongoose
```

---

# 1. Connect Basics, Schema and Model, and Schema Validation

`Connect basics` (`mongoose.connect(process.env.MONGO_URI)`) opens one shared connection at boot. `Schema and model` (`new Schema({...})` → `model("User", schema)`) define shape once, query everywhere. `Schema validation` (`required`, `minlength`, `enum`) rejects bad docs before they save.

```js
import mongoose from "mongoose";

const userSchema = new mongoose.Schema({
  name: { type: String, required: true, minlength: 1 },
  role: { type: String, enum: ["user", "admin"], default: "user" },
});
const User = mongoose.model("User", userSchema);
console.log(Object.keys(User.schema.paths).join(","));
```

Output:

```text
name,role,_id,__v
```

Connect sketch:

```js
// await mongoose.connect(process.env.MONGO_URI);
// console.log("Mongo for Kakade connected");
```

Think: schema is the contract, model is the API — define constraints in the schema, not scattered in routes.

---

# 2. CRUD Operations and ObjectId and 404 Handling

`CRUD operations` (`create`, `find`, `findById`, `findByIdAndUpdate`, `findByIdAndDelete`) cover the resource lifecycle. `ObjectId and 404 handling`: malformed ids throw `CastError` (400), valid-but-missing returns `null` (404) — handle both.

```js
// POST /users
// const u = await User.create({ name: "Kakade" }); // 201
// GET /users/:id
// let u;
// try {
//   u = await User.findById(req.params.id);
// } catch {
//   return res.status(400).json({ error: "Bad id" }); // CastError.
// }
// if (!u) return res.status(404).json({ error: "No Kakade here" });
// res.json(u);
```

Output:

```text
POST → 201 Kakade; GET bad id → 400; GET missing → 404.
```

| Id case | Result | Code |
| ------- | ------ | ---- |
| **Valid + exists** | **Doc** | **200** |
| **Malformed (`abc`)** | **CastError** | **400** |
| **Valid + missing** | **null** | **404** |

Rule: `findByIdAndUpdate(id, patch, { new: true, runValidators: true })` — return the new doc and validate updates too.

---

# Real-World Uses

Kakade's blog stores posts in Mongo with required titles, enum statuses, and id-guarded routes — CastError mapped to 400, nulls to 404, validation failures to 422/400.

```js
import mongoose from "mongoose";

const note = new mongoose.Schema({ title: { type: String, required: true } });
console.log(note.path("title").isRequired);
```

Output:

```text
true
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **Connect basics** | One shared connect at boot |
| **Schema and model** | Shape once, query everywhere |
| **Schema validation** | required/enum/minlength pre-save |
| **CRUD operations** | create/find/update/delete helpers |
| **ObjectId and 404 handling** | Malformed 400, missing 404 |

## The Mental Model

```text
connect (once at boot)
  │
  ├── schema (contract) → model (API)
  │
  └── routes for Kakade
        ├── create → 201 (validation first)
        └── :id → CastError 400 / null 404 / doc 200
```

### The 3 things to remember

**1. Shape:**

```js
const User = mongoose.model("User", userSchema);
```

**2. Create:**

```js
const u = await User.create({ name: "Kakade" });
```

**3. Guard ids:**

```js
try {
  u = await User.findById(req.params.id);
} catch {
  return res.status(400).json({ error: "Bad id" });
}
```

> **Connect once, constrain in the schema, and separate malformed ids from missing docs.**
