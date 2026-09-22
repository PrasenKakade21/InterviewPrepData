# useEffect and Lifecycle

Effects click as: `render first` → `then sync outside world` → `clean up before next`. Rendering shows UI; effects handle timers, subscriptions, and fetching that live outside React.

The core idea:

```text
Component renders
    ↓
useEffect runs after paint
    ↓
syncs with outside (fetch / timer)
    ↓
cleanup before re-run / unmount
```

`What is useEffect`: a hook for side effects — anything render should not do directly.

---

# 1. Mount Behavior, Dependency Array, and Multiple Effects

`Mount behavior` with `[]` runs once after first render. The `dependency array` lists values that re-trigger the effect. `Multiple effects` split unrelated logic — one per job — instead of one giant effect.

```jsx
import { useEffect, useState } from "react";

function Hello() {
  const [name, setName] = useState("Kakade");
  useEffect(() => {
    console.log(`Hello ${name}`);
  }, [name]);
  return <p>{name}</p>;
}
```

Output:

```text
Logs Hello Kakade on mount, again whenever name changes.
```

Mount-only + split effects:

```jsx
import { useEffect } from "react";

function Welcome() {
  useEffect(() => {
    console.log("Mounted for Kakade");
  }, []);
  useEffect(() => {
    document.title = "Kakade.dev";
  }, []);
  return <h1>Hi Kakade</h1>;
}
```

Output:

```text
Hi Kakade; both logs run once after first paint.
```

| Deps | When it runs |
| ---- | ------------ |
| **No array** | After every render (rarely wanted) |
| **`[]`** | Once on mount |
| **`[a, b]`** | On mount + when `a` or `b` changes |

---

# 2. Cleanup Functions and Fetching in Effects

`Cleanup functions` (the returned function) remove timers, listeners, and abort fetches before the next run or unmount. `Fetching in effects` loads data on mount or when a query changes, with loading and error states.

```jsx
import { useEffect, useState } from "react";

function Timer() {
  const [n, setN] = useState(0);
  useEffect(() => {
    const id = setInterval(() => setN((v) => v + 1), 1000);
    return () => clearInterval(id);
  }, []);
  return <p>{n}s for Kakade</p>;
}
```

Output:

```text
0s → 1s → 2s…; timer cleared on unmount.
```

Fetch with abort:

```jsx
import { useEffect, useState } from "react";

function User({ id }) {
  const [name, setName] = useState("…");
  useEffect(() => {
    const ctrl = new AbortController();
    fetch(`https://jsonplaceholder.typicode.com/users/${id}`, {
      signal: ctrl.signal,
    })
      .then((r) => r.json())
      .then((u) => setName(u.name))
      .catch((e) => { if (e.name !== "AbortError") console.log(e); });
    return () => ctrl.abort();
  }, [id]);
  return <p>User: {name} (friend of Kakade)</p>;
}
```

Output:

```text
User: Leanne Graham (friend of Kakade) after load.
```

Think: every effect that starts something must stop it in cleanup.

---

# 3. Effects vs Events and When Not to Use Effects

`Effects vs events`: events handle user actions directly; effects sync after render. `When not to use effects`: do not fetch on button clicks via effects, do not store derivable values, and do not sync what an event handler can set immediately.

```jsx
import { useState } from "react";

function Cart({ items }) {
  const total = items.reduce((s, i) => s + i.price, 0);
  return <p>Total for Kakade: {total}</p>;
}
```

Output:

```text
Total for Kakade: 300 (computed, no effect needed).
```

Bad (effect for a click fetch):

```jsx
// Bad: extra state + effect just to fetch on click
// const [go, setGo] = useState(false);
// useEffect(() => { if (go) fetchData(); }, [go]);
```

Good (fetch in the event):

```jsx
function Saver() {
  function handleSave() {
    fetch("https://jsonplaceholder.typicode.com/posts/1")
      .then((r) => r.json())
      .then((d) => console.log(`Saved for Kakade: ${d.id}`));
  }
  return <button onClick={handleSave}>Save</button>;
}
```

Output:

```text
Click Save → logs Saved for Kakade: 1
```

Rule: if the user caused it, use an event. If render revealed stale outside data, use an effect.

---

# 4. StrictMode Double Effects and Infinite Loop Pitfalls

`StrictMode double effects` intentionally mount → unmount → remount in development to surface missing cleanup — not a bug. `Infinite loop pitfalls` happen when an effect sets state it also depends on without a guard.

```jsx
import { useEffect } from "react";

function Probe() {
  useEffect(() => {
    console.log("Effect for Kakade");
    return () => console.log("Cleanup for Kakade");
  }, []);
  return <p>Check console</p>;
}
```

Output:

```text
Dev StrictMode: Effect → Cleanup → Effect. Prod: Effect once.
```

Loop trap:

```jsx
import { useEffect, useState } from "react";

function Loopy({ id }) {
  const [user, setUser] = useState(null);
  useEffect(() => {
    fetch(`https://jsonplaceholder.typicode.com/users/${id}`)
      .then((r) => r.json())
      .then(setUser);
  }, [id]);
  return <p>{user ? user.name : "…"}</p>;
}
```

Output:

```text
Fetches when id changes only — no loop.
```

Bad would be deps `[user]` while setting `user` — every set re-fires forever. Fix: depend on the input (`id`), not the result (`user`). Also watch object deps: `{...}` literals change every render and refire effects.

---

# Real-World Uses

A profile page fetches the user when `userId` changes, sets the document title in a second effect, and aborts stale requests so fast navigation never shows the wrong Kakade.

```jsx
import { useEffect, useState } from "react";

function ProfileTitle({ name }) {
  useEffect(() => {
    document.title = `${name} — Kakade.dev`;
  }, [name]);
  return <h2>{name}</h2>;
}
```

Output:

```text
Tab title follows the profile name.
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **What is useEffect** | Sync outside world after render |
| **Mount behavior** | `[]` runs once after first paint |
| **Dependency array** | Values that re-run the effect |
| **Cleanup functions** | `return () => …` stops timers / aborts |
| **Effects vs events** | Events: user caused; effects: render revealed |
| **Fetching in effects** | On mount / query change + abort |
| **StrictMode double effects** | Dev-only mount-unmount-remount check |
| **Infinite loop pitfalls** | Do not set what you depend on; fix deps |
| **Multiple effects** | One effect per job |
| **When not to use effects** | Derive during render; fetch in events |

## The Mental Model

```text
Render UI (pure, no side work)
  │
  ↓
useEffect([deps]) → sync outside
  │
  ├── fetch / timer / listener starts
  │       │
  │       └── cleanup stops it (re-run / unmount)
  │
  └── StrictMode dev → run + cleanup + run (checks cleanup)
```

### The 3 things to remember

**1. Mount once:**

```jsx
useEffect(() => {
  console.log("Hi Kakade");
}, []);
```

**2. Always clean up:**

```jsx
useEffect(() => {
  const id = setInterval(tick, 1000);
  return () => clearInterval(id);
}, []);
```

**3. Depend on inputs, not results:**

```jsx
useEffect(() => {
  fetchUser(id).then(setUser);
}, [id]);
```

> **Render first, sync in effects with exact deps, and clean up everything you start.**
