# Project Setup

Setup clicks as: `scaffold` → `run dev` → `env + scripts for deploys`. One command gives Kakade a TypeScript, linted, routed app — the rest is conventions.

The core idea:

```text
npx create-next-app → kakade-app/
    ↓
npm run dev → localhost:3000
    ↓
.env.local + build → ship
```

Accept the TypeScript + ESLint + App Router defaults; customize only Tailwind choice.

---

# 1. create-next-app and Folder Structure

`create-next-app` scaffolds everything (App Router, TS, ESLint) in one prompt flow. `Folder structure`: `app/` holds routes, `public/` static assets, `next.config` + `tsconfig` at root.

```bash
npx create-next-app@latest kakade-app
cd kakade-app
npm run dev
```

Output:

```text
Ready on http://localhost:3000 for Kakade.
```

Structure:

```text
kakade-app/
├── app/ (page.tsx, layout.tsx, globals.css)
├── public/ (kakade.png)
├── .env.local (secrets, gitignored)
└── next.config.mjs
```

Keep route files (`page`, `layout`, `loading`, `error`) reserved-named; everything else colocates freely beside them.

---

# 2. Dev Build and Start Scripts, Environment Variables, and TypeScript Default Setup

`Dev build and start scripts`: `dev` (HMR), `build` (production bundle + prerender), `start` (serve prod). `Environment variables`: `.env.local` for secrets, `NEXT_PUBLIC_` prefix only for browser-safe values. `TypeScript default setup`: strict TS + path alias `@/*` out of the box.

```bash
npm run dev
npm run build
npm start
```

Output:

```text
dev → HMR; build → prerendered; start → prod for Kakade.
```

Env shape:

```bash
DATABASE_URL=postgres://… (server only)
NEXT_PUBLIC_SITE=https://kakade.dev (browser ok)
```

```jsx
const url = process.env.DATABASE_URL;

export default function Page() {
  return <p>Hi Kakade</p>;
}
```

Output:

```text
Hi Kakade (server read DATABASE_URL safely).
```

Never `NEXT_PUBLIC_` a secret — the prefix literally ships it to browsers. Validate server env at startup like the Node track teaches.

---

# Real-World Uses

Kakade scaffolds with defaults, develops on `dev`, previews `build + start` before pushing, and keeps prod secrets in host env — not in the repo.

```bash
npm run build
```

Output:

```text
✓ Generating static pages for Kakade (prerendered).
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **create-next-app** | One-command scaffold |
| **Folder structure** | `app/` routes, `public/` assets |
| **Dev build and start scripts** | dev / build / start |
| **Environment variables** | `.env.local`, `NEXT_PUBLIC_` browser only |
| **TypeScript default setup** | Strict + `@/*` alias |

## The Mental Model

```text
Scaffold (defaults)
  │
  ├── dev → iterate for Kakade
  │
  ├── build + start → verify prod locally
  │
  └── env: server secrets vs NEXT_PUBLIC_
```

### The 3 things to remember

**1. Scaffold:**

```bash
npx create-next-app@latest kakade-app
```

**2. Verify prod:**

```bash
npm run build
npm start
```

**3. Prefix discipline:**

```bash
DATABASE_URL=… (server)
NEXT_PUBLIC_SITE=… (browser)
```

> **Scaffold with defaults, verify with build + start, and never leak server secrets behind NEXT_PUBLIC_.**
