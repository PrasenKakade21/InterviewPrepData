# Best Practices and Interview Gotchas

Gotchas click as: `closure captures old` → `batching delays` → `deps/keys lie`. Nearly every React interview trap is stale data or misplaced state — this page collects them.

The core idea:

```text
Small components + colocated state
    ↓
derive, don't duplicate (no mirror state)
    ↓
functional updates + honest deps + stable keys
```

When in doubt: colocate, derive, and use the functional updater.

---

# 1. Stale Closures, State Batching Traps, and Effect Dependency Traps

`Stale closures` capture old state in timeouts/handlers — fix with functional updates or refs. `State batching traps` (`setN(n+1); setN(n+1)` → +1) need `setN(v => v+1)`. `Effect dependency traps` (missing/extra deps) cause stale reads or loops — obey the linter.

```jsx
import { useState } from "react";

function Stuck() {
  const [n, setN] = useState(0);
  function add() {
    setN(n + 1);
    setN(n + 1);
  }
  return <button onClick={add}>Kakade {n}</button>;
}
```

Output:

```text
Click → +1 only (both used stale n).
```

Fixed:

```jsx
function Fixed() {
  const [n, setN] = useState(0);
  function add() {
    setN((v) => v + 1);
    setN((v) => v + 1);
  }
  return <button onClick={add}>Kakade {n}</button>;
}
```

Output:

```text
Click → +2 (each sees latest).
```

Effect rule: every reactive value used inside goes in deps — or move constants/handlers out.

---

# 2. Keys Pitfalls, Derived State, and Lifting vs Colocating State

`Keys pitfalls`: index keys scramble inputs on reorder; random keys remount every render. `Derived state` (filtering, totals) computes during render — never mirror props in state. `Lifting vs colocating state`: lift shared, colocate private to avoid re-render blast radius.

```jsx
function Total({ items }) {
  const total = items.reduce((s, i) => s + i.price, 0);
  return <p>Total for Kakade: {total}</p>;
}
```

Output:

```text
Total for Kakade: always fresh (derived, no useState).
```

| Choice | Use when |
| ------ | -------- |
| **Lift** | **Two+ siblings share it** |
| **Colocate** | **One component owns it** |
| **Stable id key** | **Lists reorder/filter** |
| **Derive** | **Computable from props/state** |

Smell: `useState(prop)` + `useEffect(setState(prop))` — replace with direct derivation or `key={prop.id}` reset.

---

# 3. Keeping Components Small, Naming Conventions, and Code Review Checklist

`Keeping components small` (one job, early returns) makes bugs local. `Naming conventions` (`UserCard`, `useAuth`, `handleSave`, `isOpen`) signal roles. `Code review checklist` gates merges on deps, keys, a11y, and perf basics.

```jsx
function UserCard({ user }) {
  if (!user) return <p>No Kakade yet</p>;
  return <p>{user.name}</p>;
}
```

Output:

```text
No Kakade yet → name when loaded (early return).
```

Review checklist:

```text
[ ] deps honest (linter clean) for Kakade effects
[ ] keys stable, no index/random
[ ] no mirror state; derived inline
[ ] handlers stable for memo children
[ ] labels, alt, keyboard path exist
```

Small + named + checked beats clever — reviewers (and interviewers) notice.

---

# Real-World Uses

A Kakade PR fixes a triple-click counter with functional updates, replaces index keys with ids, deletes mirrored filter state, and splits a 400-line page into focused cards — all checklist items.

```jsx
function Chip({ name = "Kakade" }) {
  return <span>{name}</span>;
}
```

Output:

```text
Kakade (tiny, named, testable).
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **Stale closures** | Functional update / ref latest |
| **State batching traps** | `setV(v => …)` for chains |
| **Effect dependency traps** | All reactives in deps |
| **Keys pitfalls** | Stable ids, never index/random |
| **Lifting vs colocating state** | Share up, own down |
| **Derived state** | Compute, do not mirror |
| **Keeping components small** | One job + early returns |
| **Naming conventions** | `Noun`, `useX`, `handleX`, `isX` |
| **Code review checklist** | Deps, keys, a11y, perf |

## The Mental Model

```text
Bug? ask in order
  │
  ├── stale closure/batch? → (v => …)
  │
  ├── effect loop/stale? → honest deps
  │
  └── wrong row/reset? → stable keys + derive
        └── small components + checklist gate
```

### The 3 things to remember

**1. Chain updates:**

```jsx
setN((v) => v + 1);
```

**2. Derive:**

```jsx
const total = items.reduce((s, i) => s + i.price, 0);
```

**3. Key + colocate:**

```jsx
<li key={u.id}>Kakade</li>
```

> **Update functionally, depend honestly, key stably, and keep state close to its owner.**
