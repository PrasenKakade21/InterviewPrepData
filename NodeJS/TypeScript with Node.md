# TypeScript with Node

Types click as: `tsx for speed` → `type the boundaries` → `tsc ships JS`. Type requests, env, and DB models — infer everything inside handlers.

The core idea:

```text
tsx watch (dev, no build step)
    ↓
Request/Response generics + env type
    ↓
tsc → dist/ for Kakade prod
```

Install block (needed — third-party packages):

```bash
npm install -D typescript tsx @types/node @types/express
```

---

# 1. tsx Setup for Dev and Build with tsc

`tsx setup for dev` (`tsx watch src/server.ts`) runs TypeScript directly with hot reload. `Build with tsc` (`tsc` → `dist/`, then `node dist/server.js`) ships plain JS — types erased, zero runtime cost.

```bash
npx tsx watch src/server.ts
```

```text
Watching for Kakade… restarts on save, no build step.
```

Build config:

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "NodeNext",
    "outDir": "dist",
    "strict": true
  }
}
```

Scripts:

```json
{
  "scripts": {
    "dev": "tsx watch src/server.ts",
    "build": "tsc",
    "start": "node dist/server.js"
  }
}
```

Dev on tsx, prod on compiled JS — never run `tsx` in production.

---

# 2. Typing req and res and Env Typing

`Typing req and res` (`Request<Params, Body, Query>`, `Response<Body>`) types params, bodies, and queries at the boundary. `Env typing` (a validated `env.ts` exporting typed constants) kills `string | undefined` sprawl.

```ts
import type { Request, Response } from "express";

type HiParams = { name: string };

export function hi(req: Request<HiParams>, res: Response) {
  res.json({ hi: req.params.name || "Kakade" });
}
console.log(typeof hi);
```

Output:

```text
function
```

Env shape:

```ts
// function need(k: string): string {
//   const v = process.env[k];
//   if (!v) throw new Error(`Missing ${k} for Kakade`);
//   return v;
// }
// export const env = { port: Number(need("PORT")), jwt: need("JWT_SECRET") };
```

Import `env` everywhere instead of `process.env` — one validation point, typed values out.

---

# Real-World Uses

Kakade's API develops on `tsx watch`, types every route boundary, validates env once, and deploys `tsc` output — prod runs plain JS with dev-time safety.

```ts
const user: { name: string } = { name: "Kakade" };
console.log(user.name);
```

Output:

```text
Kakade
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **tsx setup for dev** | Direct TS, watch mode |
| **Typing req and res** | Generics at the boundary |
| **Env typing** | One validated `env` module |
| **Build with tsc** | `dist/` JS for prod |

## The Mental Model

```text
Dev: tsx watch (Kakade iterates fast)
  │
  ├── boundaries typed (req/res/env)
  │
  └── prod: tsc → node dist/ (types erased)
```

### The 3 things to remember

**1. Dev loop:**

```bash
npx tsx watch src/server.ts
```

**2. Type the edge:**

```ts
function hi(req: Request<HiParams>, res: Response) {}
```

**3. Ship JS:**

```bash
npx tsc
node dist/server.js
```

> **Run TS directly in dev, type the boundaries, and ship compiled JS.**
