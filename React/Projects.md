# Projects

Projects click as: `small → integrated → deployed`. Each build reuses earlier topics — state, effects, router, queries, auth, and styling stack into portfolio proof.

The core idea:

```text
Counter/todo → hooks + lists
    ↓
notes/weather/shop → persist + fetch + routes
    ↓
blog/dashboard/redux/forms → auth + global + validation → deploy
```

Ship every project with README, error/loading states, and a live URL.

---

# 1. Counter and Todo App, Notes App with localStorage, and Weather App with API Fetching

`Counter and todo app` practices `useState`, lists/keys, and conditional UI. `Notes app with localStorage` adds `useEffect` persistence + search. `Weather app with API fetching` adds abort, loading/error, and env keys.

```jsx
import { useState } from "react";

function Todo() {
  const [items, setItems] = useState(["Learn React"]);
  const [text, setText] = useState("");
  function add() {
    setItems([...items, text || "Hi Kakade"]);
    setText("");
  }
  return <button onClick={add}>Todos: {items.length}</button>;
}
```

Output:

```text
Todos: 1 → 2 (Kakade item appended).
```

Notes sketch:

```js
// useEffect(() => localStorage.setItem("notes-kakade", JSON.stringify(notes)), [notes]);
// Load lazily: useState(() => JSON.parse(localStorage.getItem(...) || "[]")).
```

Weather sketch:

```js
// fetch(`${import.meta.env.VITE_WEATHER}/?q=kakade`) + AbortController + status/error.
```

Each project should demo one new hard skill: keys, persistence, then network.

---

# 2. E-commerce Product Listing, Blog with React Router, and Auth Protected Dashboard

`E-commerce product listing` combines filters, pagination, and cart state (Zustand/Context). `Blog with React Router` adds nested routes, params (`/posts/:id`), and 404s. `Auth protected dashboard` adds login, guards, and `/me` persistence.

```jsx
import { Link } from "react-router-dom";

function Shop() {
  return <Link to="/product/kakade-1">Buy Kakade Tee</Link>;
}
```

Output:

```text
Buy Kakade Tee → /product/kakade-1 detail.
```

Dashboard shape:

```js
// <Guard><Dashboard /></Guard>; login → nav(state.from || "/dashboard").
// Cart: useCart(s => s.items) with add/clear for Kakade.
```

These three prove routing + shared state + auth — the freelance/employer filter.

---

# 3. Redux Task Manager, Form Builder with Validation, and Deployed Portfolio Project

`Redux task manager` shows slices, selectors, and DevTools on non-trivial flows. `Form builder with validation` shows schemas (zod), field errors, and accessible labels. `Deployed portfolio project` ties everything: Vite build, rewrites, CDN, and README demo.

```jsx
function Task() {
  return <p>Task for Kakade: ship it</p>;
}
```

Output:

```text
Task for Kakade: ship it (RTK slice drives status).
```

Portfolio checklist:

```text
[ ] Live URL + preview per PR for Kakade
[ ] README: stack, runs, screenshots
[ ] Loading/error/empty states everywhere
[ ] Lighthouse + axe clean
```

One polished deployed app beats five localhost demos — interviewers click links.

---

# Real-World Uses

Kakade ships a notes PWA, a weather app with cached queries, a routed blog, an auth dashboard, and a Redux task board — each deployed with env configs and a one-paragraph README story.

```jsx
function Link2() {
  return <p>Portfolio of Kakade: live</p>;
}
```

Output:

```text
Portfolio of Kakade: live (deployed, not localhost).
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **Counter and todo app** | State + keys + conditional |
| **Notes app with localStorage** | Persist + search |
| **Weather app with API fetching** | Abort + status/error + env |
| **E-commerce product listing** | Filter + page + cart |
| **Blog with React Router** | Nested + params + 404 |
| **Auth protected dashboard** | Guard + `/me` + refresh |
| **Redux task manager** | Slices + DevTools |
| **Form builder with validation** | Schema + accessible errors |
| **Deployed portfolio project** | Live URL + README + budgets |

## The Mental Model

```text
Foundations (todo/notes/weather)
  │
  ├── integration (shop/blog/dashboard)
  │
  └── senior signals (redux/forms/deployed)
        └── every app: loading/error/empty + live link
```

### The 3 things to remember

**1. Start stateful:**

```jsx
const [items, setItems] = useState(["Learn React"]);
```

**2. Route + guard:**

```jsx
<Link to="/product/kakade-1">Buy Kakade Tee</Link>
```

**3. Ship live:**

```bash
npm run build
```

> **Build progressively harder apps and deploy every one with states, README, and a live URL.**
