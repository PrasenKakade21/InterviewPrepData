# useRef, useMemo and useCallback

These three hooks click as: `useRef` → `hold without re-render` → `useMemo/useCallback` → `skip redoing work`. Refs escape the render cycle; memo hooks avoid repeating expensive work across renders.

The core idea:

```text
Render happens
    ↓
ref persists silently (no re-render)
    ↓
memo reuses last value / function
    ↓
less wasted work
```

All three are optimizations or escape hatches — reach for state and plain code first.

---

# 1. useRef Basics, Refs vs State, and Persisting Values

`useRef basics`: `useRef(initial)` returns `{ current: initial }` — a box that survives re-renders. `Refs vs state`: changing state re-renders, changing a ref does not. `Persisting values` uses refs for timers, counts, or previous values you read imperatively.

```jsx
import { useRef, useState } from "react";

function Clicks() {
  const [n, setN] = useState(0);
  const renders = useRef(0);
  renders.current += 1;
  return <button onClick={() => setN(n + 1)}>{n} ({renders.current}x)</button>;
}
```

Output:

```text
Button shows count and render tally; tally updates without triggering renders.
```

Think:

```text
useState → change → re-render
useRef → change → silent, read later
```

Notice: never read or write `ref.current` during render for output — it breaks purity. Update refs in events or effects. Interview trap: `useRef` does not notify React; UI will not update until something else re-renders.

---

# 2. DOM Access with Refs

`DOM access with refs` attaches a ref to an element via `<input ref={myRef} />`, then uses `myRef.current.focus()` or measures sizes. Refs are the only safe way to touch a DOM node directly.

```jsx
import { useRef } from "react";

function FocusBox() {
  const inputRef = useRef(null);
  function focus() {
    inputRef.current.focus();
  }
  return (
    <>
      <input ref={inputRef} placeholder="Name, Kakade" />
      <button onClick={focus}>Focus</button>
    </>
  );
}
```

Output:

```text
Click Focus → cursor jumps into the input.
```

Think:

```text
ref={inputRef} → inputRef.current = DOM node
    ↓
button → inputRef.current.focus()
```

Rule: `ref.current` is `null` on first render — access it in events or effects, never during render. For lists, use a `Map` or callback refs instead of one shared ref.

---

# 3. useMemo Basics and Memoization When It Helps

`useMemo basics`: `useMemo(() => compute(a, b), [a, b])` caches the result until deps change. `Memoization when it helps`: expensive sorts, filters, or derived totals recomputed every render.

```jsx
import { useMemo, useState } from "react";

function Total({ prices }) {
  const [name, setName] = useState("Kakade");
  const total = useMemo(() => {
    return prices.reduce((s, p) => s + p, 0);
  }, [prices]);
  return <p>{name}: {total}</p>;
}
```

Output:

```text
Typing name re-renders but does not recompute total unless prices change.
```

Think:

```text
render → deps same? → reuse cached value
       → deps changed? → recompute
```

Use `useMemo` for genuinely costly work or referential stability (passing objects to memoized children). Cheap math like `a + b` costs more to memoize than to redo.

---

# 4. useCallback Basics and Referential Equality

`useCallback basics`: `useCallback(fn, deps)` caches the function identity across renders. `Referential equality` (`===` on objects/functions) is why it matters — memoized children and effects see a new function every render unless it is stabilized.

```jsx
import { useCallback, useState } from "react";

function Sender({ onSend }) {
  return <button onClick={() => onSend("Hi Kakade")}>Send</button>;
}

function Chat() {
  const [text, setText] = useState("");
  const handleSend = useCallback((msg) => {
    console.log(msg);
  }, []);
  return <Sender onSend={handleSend} />;
}
```

Output:

```text
Typing text re-renders Chat, but Sender gets the same onSend reference.
```

Think:

```text
inline () => … → new function each render → child re-renders
useCallback → same reference → memo child skips
```

Notice: `useCallback(fn, deps)` is `useMemo(() => fn, deps)` for functions. Only needed when the function goes to a memoized child or an effect dependency.

---

# 5. Over-Memoization Pitfalls

`Over-memoization pitfalls`: wrapping everything adds code, dependency bugs, and memory cost for zero visible gain. Memoize at measured bottlenecks, keep deps honest, and prefer simpler fixes first.

```jsx
import { useState } from "react";

function Hello({ name }) {
  return <p>Hello {name || "Kakade"}</p>;
}
```

Output:

```text
Hello Kakade — no memo needed for trivial output.
```

Bad vs Good:

```jsx
// Bad: memo theater — cheap work, stale-risk deps
// const upper = useMemo(() => name.toUpperCase(), []);
```

```jsx
import { useMemo } from "react";

function Fixed({ name }) {
  const upper = useMemo(() => name.toUpperCase(), [name]);
  return <p>{upper} here</p>;
}
```

Output:

```text
KAKADE here, always fresh because name is in deps.
```

Checklist before memoizing: is it slow (measured)? Does identity matter (memo child / effect dep)? Are deps complete? If no, skip it.

---

# Real-World Uses

A searchable product table uses all three: ref focuses search on mount, `useMemo` caches filtered rows, `useCallback` stabilizes the add-to-cart handler passed to memoized rows.

```jsx
import { useMemo } from "react";

function Results({ items, query }) {
  const shown = useMemo(() => {
    return items.filter((i) => i.includes(query));
  }, [items, query]);
  return <p>{shown.length} for Kakade</p>;
}
```

Output:

```text
Filtered count updates only when items or query change.
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **useRef basics** | `{ current }` box surviving renders |
| **Refs vs state** | Ref: silent; state: re-renders |
| **DOM access with refs** | `ref={r}` + `r.current.focus()` in events/effects |
| **Persisting values with refs** | Timers, ids, previous values |
| **useMemo basics** | Cache value: `useMemo(() => v, deps)` |
| **useCallback basics** | Cache fn: `useCallback(fn, deps)` |
| **Memoization when it helps** | Slow compute or stable identity need |
| **Over-memoization pitfalls** | Clutter + stale deps + no gain |
| **Referential equality** | New `{}`/`() =>` each render breaks `===` |

## The Mental Model

```text
Render
  │
  ├── ref (mute box, DOM node, timer id) → no re-render
  │
  ├── useMemo (cached value) → recompute only on dep change
  │
  └── useCallback (stable fn) → memo child / effect sees same ===
```

### The 3 things to remember

**1. Silent box + DOM:**

```jsx
const inputRef = useRef(null);
inputRef.current.focus();
```

**2. Cache expensive value:**

```jsx
const total = useMemo(() => sum(prices), [prices]);
```

**3. Stabilize passed functions:**

```jsx
const handleSend = useCallback((m) => send(m), []);
```

> **Refs hold without re-rendering; memoize only costly values or identities that must stay stable.**
