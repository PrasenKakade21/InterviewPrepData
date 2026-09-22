# Postgres with Prisma

Prisma clicks as: `schema declares models` → `migrate creates tables` → `client queries typed`. Postgres enforces relational truth; Prisma makes it feel like typed JavaScript.

The core idea:

```text
schema.prisma (User model)
    ↓
prisma migrate dev (SQL applied)
    ↓
prisma.user.create/findMany for Kakade
```

Install block (needed — third-party packages):

```bash
npm install prisma @prisma/client
npx prisma init
```

---

# 1. Schema Basics, Migrate Overview, and Client CRUD

`Schema basics` (`model User { id, email @unique, name }`) declare tables in Prisma syntax with a Postgres datasource. `Migrate overview` (`npx prisma migrate dev --name init`) versions SQL changes — reviewable, repeatable, team-safe. `Client CRUD` (`prisma.user.create/findMany/update/delete`) queries with autocompletion.

```prisma
model User {
  id    Int    @id @default(autoincrement())
  email String @unique
  name  String
}
```

Output:

```text
Table users for Kakade: id PK, unique email, name.
```

Client shape:

```js
// import { PrismaClient } from "@prisma/client";
// const prisma = new PrismaClient();
// const u = await prisma.user.create({ data: { email: "k@dev.io", name: "Kakade" } });
// const all = await prisma.user.findMany({ where: { name: "Kakade" } });
```

Think: schema is the single source — DB, client types, and migrations all derive from it.

---

# 2. Relations Brief and Postgres vs Mongo Choice

`Relations brief`: `@relation` + foreign keys model one-to-many (`User` → `Post[]`) with `include` for joins. `Postgres vs Mongo choice`: relations + transactions + constraints → Postgres; flexible evolving docs → Mongo.

```prisma
model Post {
  id       Int    @id @default(autoincrement())
  title    String
  authorId Int
  author   User   @relation(fields: [authorId], references: [id])
}
```

Output:

```text
Posts belong to Kakade: join via include, enforced by FK.
```

Join sketch:

```js
// const u = await prisma.user.findUnique({
//   where: { email: "k@dev.io" },
//   include: { posts: true }, // Kakade + posts in one query.
// });
```

| Pick | When |
| ---- | ---- |
| **Postgres + Prisma** | **Relations, transactions, constraints** |
| **Mongo + Mongoose** | **Flexible docs, fast iteration** |

Rule: one `PrismaClient` instance app-wide (singleton) — new clients per request exhaust the pool.

---

# Real-World Uses

Kakade's shop runs Postgres with Prisma migrations in CI, `include` for order+items joins, and unique emails enforced at the DB — not hoped for in code.

```js
// const prisma = new PrismaClient(); // singleton for Kakade.
// await prisma.$connect().then(() => console.log("Postgres ready"));
```

Output:

```text
Postgres ready (one shared client).
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **Schema basics** | Models + datasource in one file |
| **Migrate overview** | Versioned SQL, team-safe |
| **Client CRUD** | Typed create/find/update/delete |
| **Relations brief** | FK + include for joins |

## The Mental Model

```text
schema.prisma (models for Kakade)
  │
  ├── migrate dev → versioned tables
  │
  └── singleton client → typed queries
        └── relations → include (FK enforced)
```

### The 3 things to remember

**1. Model:**

```prisma
model User {
  id    Int    @id @default(autoincrement())
  email String @unique
}
```

**2. Migrate:**

```bash
npx prisma migrate dev --name init
```

**3. Query:**

```js
const u = await prisma.user.create({ data: { email: "k@dev.io", name: "Kakade" } });
```

> **Schema once, migrate always, query typed — one client for the whole app.**
