# Performance Optimization

Performance clicks as: `measure first` → `render less` → `split + defer the rest`. Most apps are fast enough; optimize proven bottlenecks, not every component.

The core idea:

```text
Profile slow interaction
    ↓
skip re-renders (memo, keys, stable props)
    ↓
split bundles + defer non-urgent UI
```

React DevTools Profiler before `memo` — the fastest render is the one skipped.

---

# 1. Re-render Causes, React.memo, and Avoiding Inline Object Traps

`Re-render causes`: state change, parent re-render, or context value change. `React.memo` skips a component when props are shallow-equal. `Avoiding inline object traps`: `style={{...}}` and `onX={() => …}` create new identities each render, defeating memo.

```jsx
import { memo } from "react";

const Hello = memo(function Hello({ name }) {
  return <p>Hi {name || "Kakade"}</p>;
});
```

Output:

```text
Hello re-renders only when name changes.
```

Bad (memo defeated):

```jsx
// Bad: new object + fn each render → Hello always re-renders
// <Hello name="Kakade" style={{ color: "red" }} onHi={() => {}} />
```

Good (stable props):

```jsx
const style = { color: "red" };

function Page({ onHi }) {
  return <Hello name="Kakade" style={style} onHi={onHi} />;
}
```

Output:

```text
Same style/onHi references → memo can skip.
```

Think: `memo` compares props with `===` — stabilize objects, arrays, and callbacks passed down.

---

# 2. Keys and Reconciliation, Code splitting, React.lazy, and Suspense

`Keys and reconciliation`: stable keys let React move rows instead of remounting them. `Code splitting` ships routes separately; `React.lazy` + `Suspense` load them on demand with a fallback.

```jsx
function Users({ users }) {
  return (
    <ul>
      {users.map((u) => (
        <li key={u.id}>{u.name}</li>
      ))}
    </ul>
  );
}
```

Output:

```text
Kakade row keeps state when list reorders (stable key).
```

Lazy route:

```jsx
import { Suspense, lazy } from "react";

const Dashboard = lazy(() => import("./Dashboard"));

function App() {
  return (
    <Suspense fallback={<p>Loading Kakade…</p>}>
      <Dashboard />
    </Suspense>
  );
}
```

Output:

```text
Loading Kakade… → Dashboard chunk after download.
```

Rules: keys must be stable ids (never index for reorderable lists); split at routes and heavy modals first.

---

# 3. useTransition Basics, useDeferredValue Basics, and Virtualization Concept

`useTransition basics`: `startTransition` marks updates non-urgent (filtering a huge list) so typing stays responsive. `useDeferredValue basics` defers a value (search query) while showing stale UI. `Virtualization concept` renders only visible rows of long lists (react-window).

```jsx
import { useState, useTransition } from "react";

function Filter({ items }) {
  const [q, setQ] = useState("");
  const [isPending, startTransition] = useTransition();
  function onChange(e) {
    setQ(e.target.value);
    startTransition(() => {
      // heavy filter for Kakade list here
    });
  }
  return <input value={q} onChange={onChange} placeholder={isPending ? "…" : "Kakade"} />;
}
```

Output:

```text
Input stays smooth; heavy filter runs as transition.
```

Deferred shape:

```js
// const deferredQ = useDeferredValue(q);
// <SlowList query={deferredQ} /> keeps old list while typing.
```

Virtualization sketch:

```js
// 10,000 rows → render ~20 visible + overscan; scroll swaps windows.
```

Use transitions for state updates you trigger; deferred values for props you receive. Both need `Suspense`-friendly or memo-split slow children to shine.

---

# Real-World Uses

A Kakade admin table virtualizes 10k rows, memoizes each row, stabilizes handlers with `useCallback`, lazy-loads the chart tab, and wraps filtering in a transition so search never janks.

```jsx
import { memo } from "react";

const Row = memo(function Row({ name }) {
  return <p>{name}</p>;
});
```

Output:

```text
Only changed rows re-render for Kakade.
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **Re-render causes** | State, parent, context change |
| **React.memo** | Skip on same props (shallow) |
| **Keys and reconciliation** | Stable ids preserve rows |
| **Code splitting** | Ship routes separately |
| **React.lazy** | `lazy(() => import())` on demand |
| **Suspense** | Fallback while chunk loads |
| **useTransition basics** | Non-urgent `startTransition` updates |
| **useDeferredValue basics** | Deferred prop value, stale UI ok |
| **Avoiding inline object traps** | Hoist objects, stabilize callbacks |
| **Virtualization concept** | Render visible window only |

## The Mental Model

```text
Slow interaction (profiled)
  │
  ├── memo + stable props + keys → skip renders
  │
  ├── lazy + Suspense → smaller initial bundle
  │
  └── transition / deferred → urgent typing first
        │
        └── virtualize → 10k rows → ~20 mounted
```

### The 3 things to remember

**1. Memo + stable props:**

```jsx
const Hello = memo(function Hello({ name }) {
  return <p>Hi {name}</p>;
});
```

**2. Split heavy routes:**

```jsx
const Dashboard = lazy(() => import("./Dashboard"));
```

**3. Defer non-urgent work:**

```jsx
startTransition(() => setQuery("Kakade"));
```

> **Profile first, skip renders with stable props, split bundles, and defer what can wait.**
