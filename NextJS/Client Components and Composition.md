# Client Components and Composition

Client components click as: `"use client" opts in` → `boundary = JS shipped` → `compose to shrink it`. Every client file adds to Kakade's bundle — place boundaries like you pay for them.

The core idea:

```text
"use client" at top → hydrated + interactive
    ↓
children props stay server-rendered
    ↓
islands, not oceans, for Kakade
```

State, effects, events, and browser APIs need client; everything else stays server.

---

# 1. use client Directive, When to Go Client, and Boundary Placement

`use client directive`: top-of-file opt-in marking module + imports as client. `When to go client`: clicks, inputs, `useState`/`useEffect`, `window`/`localStorage`. `Boundary placement`: mark the smallest leaf needing interactivity — parents stay server.

```jsx
"use client";
import { useState } from "react";

export function Counter() {
  const [n, setN] = useState(0);
  return <button onClick={() => setN(n + 1)}>Kakade {n}</button>;
}
```

Output:

```text
Kakade 0 → 1 per click (hydrated island).
```

Boundary rule:

```jsx
// app/page.tsx (server) → imports <Counter/> (client leaf)
// Only Counter + react ship JS; page HTML stays static.
```

`"use client"` at a page root client-ifies the whole subtree — push it down.

---

# 2. Hooks and Events and Interleaving Server and Client

`Hooks and events` work normally inside client components (same React track knowledge applies). `Interleaving server and client`: pass server components as `children`/props into client components — they render server-side, slots stay static.

```jsx
"use client";

export function Tabs({ children }) {
  return <div>{children} — tabs for Kakade</div>;
}
```

```jsx
// <Tabs><Bio /></Tabs> → Bio renders on server, Tabs hydrates.
```

Output:

```text
Tabs interactive, bio HTML — best of both for Kakade.
```

Forbidden direction: never import a server component directly into client code (it would drag server code to the browser) — always via children/props.

---

# Real-World Uses

Kakade's docs page is fully server except a client search box and theme toggle — two tiny islands in a static sea.

```jsx
"use client";

export function Hi() {
  return <p>Hi Kakade (interactive)</p>;
}
```

Output:

```text
Hi Kakade (hydrated on demand).
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **use client directive** | Top-of-file opt-in |
| **When to go client** | State, events, browser APIs |
| **Boundary placement** | Smallest leaf possible |
| **Hooks and events** | Normal React inside |
| **Interleaving server and client** | Server children via props |

## The Mental Model

```text
Needs clicks/state/effects?
  │
  ├── yes → "use client" leaf for Kakade
  │     └── accept server children as props
  │
  └── no → server stays server
```

### The 3 things to remember

**1. Opt in at leaves:**

```jsx
"use client";
```

**2. Slot pattern:**

```jsx
<Tabs><Bio /></Tabs>
```

**3. Never import down:**

```jsx
// server → client import = forbidden; use children
```

> **Client is a paid upgrade per boundary — buy small islands, keep the sea server.**
