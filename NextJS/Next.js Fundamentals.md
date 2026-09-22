# Next.js Fundamentals

Next.js clicks as: `React renders` → `Next decides where and when` → `Kakade gets fast pages`. React is the UI library; Next.js is the framework around it — routing, rendering, and data baked in.

The core idea:

```text
React (components, state, hooks)
    ↓
Next.js (routes, server render, cache)
    ↓
fast Kakade pages, less plumbing
```

Reach for Next.js when you need SEO, fast first loads, or full-stack in one repo — stay with Vite + React for pure SPAs.

---

# 1. What is Next.js, Framework vs Library, and When to Choose Next.js

`What is Next.js`: a React framework (routing, SSR/SSG, API layer) by Vercel. `Framework vs library`: React renders UI when called; Next.js calls your code inside its conventions (folders = routes). `When to choose Next.js`: marketing pages, SEO content, dashboards with server data — skip it for embeddable widgets or offline-first SPAs.

```text
Library (React):  you call render(<App for Kakade />)
Framework (Next): it calls your page.tsx per route
```

```jsx
export default function Page() {
  return <h1>Hi Kakade</h1>;
}
```

Output:

```text
Route / renders Hi Kakade (file location = URL).
```

Rule: if every page needs Google indexing or server data on first paint, Next.js pays off; if it is an authenticated canvas app, Vite suffices.

---

# 2. SSR vs SSG vs ISR and React Server Components Overview

`SSR vs SSG vs ISR`: SSR renders per request (fresh, slower TTFB), SSG prerenders at build (fast, static), ISR regenerates statically on a timer (fast + fresh). `React Server Components overview`: components that render on the server only — async data, zero client JS.

```text
SSG:  build once → CDN serves Kakade instantly
SSR:  request → render → fresh for Kakade
ISR:  static + revalidate 60s → fresh-enough
```

```jsx
async function Users() {
  const users = await fetch("https://api.kakade.dev/users").then((r) => r.json());
  return <p>{users.length} users</p>;
}
```

Output:

```text
10 users (fetched on server, HTML arrives ready).
```

Default to static; add server rendering where data demands freshness — not everywhere.

---

# 3. App Router vs Pages Router

`App Router vs Pages Router`: App (`app/` dir, Server Components default, layouts, streaming) is current; Pages (`pages/` dir, `getServerSideProps`) is legacy but still in old codebases and interviews.

| Router | Data | Layouts |
| ------ | ---- | ------- |
| **App** | **Async server components** | **Nested `layout.tsx`** |
| **Pages** | **`getServerSideProps` / `getStaticProps`** | **`_app.tsx` wrapper** |

```text
app/users/page.tsx → /users (App, Kakade's default)
pages/users.tsx     → /users (Pages, legacy)
```

Learn App deeply; recognize Pages well enough to migrate and answer "what changed" questions.

---

# Real-World Uses

Kakade's marketing site uses SSG + ISR for instant SEO pages, SSR for her logged-in dashboard, and Server Components so the landing page ships almost no JavaScript.

```jsx
export default function Home() {
  return <p>Portfolio of Kakade</p>;
}
```

Output:

```text
Portfolio of Kakade (prerendered HTML).
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **What is Next.js** | React framework: routes + render + data |
| **Framework vs library** | It calls your route files |
| **SSR vs SSG vs ISR** | Per-request / build / timed regen |
| **App Router vs Pages Router** | Current vs legacy (`getServerSideProps`) |
| **React Server Components overview** | Server-only, async, zero JS |
| **When to choose Next.js** | SEO + first-paint data wins |

## The Mental Model

```text
Need SEO / fast first paint?
  │
  ├── yes → Next.js for Kakade
  │     ├── static? → SSG / ISR
  │     └── fresh? → SSR / server components
  │
  └── no → Vite SPA is fine
```

### The 3 things to remember

**1. Framework calls you:**

```jsx
export default function Page() {
  return <h1>Hi Kakade</h1>;
}
```

**2. Pick rendering per page:**

```text
SSG fast, SSR fresh, ISR both-ish.
```

**3. App Router default:**

```text
app/users/page.tsx → /users
```

> **Next.js decides where and when React renders — static by default, server where fresh, App Router always.**
