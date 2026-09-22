# Refs and DOM Patterns

Refs click as: `ref holds node or value` → `parent asks, child exposes` → `measure then act`. Refs escape React for focus, scroll, and measure — everything else stays declarative.

The core idea:

```text
useRef / callback ref → DOM node
    ↓
forwardRef + useImperativeHandle → curated API
    ↓
focus / scroll / measure on demand
```

Never read layout during render; measure in effects or events.

---

# 1. forwardRef, useImperativeHandle Basics, and Controlled vs Uncontrolled Ref Use

`forwardRef` lets a child receive a parent's ref (needed before React 19 patterns). `useImperativeHandle basics` exposes only chosen methods (`focus`, `clear`) instead of the whole node. `Controlled vs uncontrolled ref use`: controlled reads values via state; uncontrolled grabs them via ref at submit.

```jsx
import { forwardRef, useRef } from "react";

const Input = forwardRef(function Input(props, ref) {
  return <input ref={ref} placeholder="Kakade" {...props} />;
});

function Form() {
  const ref = useRef(null);
  return <button onClick={() => ref.current.focus()}>Focus {ref.current ? "" : "Kakade"}</button>;
}
```

Output:

```text
Click Focus Kakade → input focuses (parent drives child node).
```

Curated API:

```js
// useImperativeHandle(ref, () => ({ focus: () => node.current.focus() }));
// Parent gets { focus } only — not the raw input.
```

Use controlled inputs for validation; uncontrolled + ref for one-shot reads (file inputs, "scroll to error").

---

# 2. Focus Management, Scroll to Element, and Measuring Elements

`Focus management` moves focus to errors, dialogs, or first fields. `Scroll to element` uses `ref.current.scrollIntoView({ behavior: "smooth" })`. `Measuring elements` reads `getBoundingClientRect()` in effects/events for tooltips and popovers.

```jsx
import { useRef } from "react";

function Chat() {
  const end = useRef(null);
  function jump() {
    end.current.scrollIntoView({ behavior: "smooth" });
  }
  return (
    <div>
      <button onClick={jump}>Jump to Kakade</button>
      <div ref={end}>Kakade is here</div>
    </div>
  );
}
```

Output:

```text
Click Jump to Kakade → smooth scroll to target.
```

Measure sketch:

```js
// const r = ref.current.getBoundingClientRect();
// setPos({ top: r.bottom, left: r.left }) → place tooltip for Kakade.
```

Focus on mount belongs in `useEffect`; focus after action belongs in the event handler — both need null-guards.

---

# 3. Refs in Lists and Callback Refs

`Refs in lists` store nodes in a `Map` keyed by id for dynamic rows. `Callback refs` (`ref={node => …}`) run on mount/unmount — ideal for observers and cleanup per element.

```jsx
import { useRef } from "react";

function List({ users }) {
  const map = useRef(new Map());
  function jump(id) {
    map.current.get(id)?.scrollIntoView();
  }
  return (
    <div>
      <button onClick={() => jump("kakade")}>Find Kakade</button>
      {users.map((u) => (
        <div key={u.id} ref={(n) => (n ? map.current.set(u.id, n) : map.current.delete(u.id))}>
          {u.name}
        </div>
      ))}
    </div>
  );
}
```

Output:

```text
Find Kakade → scrolls to that row (Map of nodes).
```

Think:

```text
mount → callback(node) → map.set(id, node)
unmount → callback(null) → map.delete(id)
```

One `useRef` per static node; a Map + callback refs for dynamic lists. Never store nodes in state — refs avoid re-renders.

---

# Real-World Uses

A Kakade form focuses the first invalid field via a ref Map, scrolls the chat to new messages, and measures a button to anchor a portal tooltip with observer cleanup.

```jsx
import { useRef } from "react";

function Box() {
  const ref = useRef(null);
  return <button onClick={() => ref.current?.focus()}>Hi Kakade</button>;
}
```

Output:

```text
Hi Kakade (safe focus, no crash if unmounted).
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **forwardRef** | Pass parent ref into child |
| **useImperativeHandle basics** | Expose `{ focus }`, hide node |
| **Focus management** | Errors, dialogs, first fields |
| **Scroll to element** | `scrollIntoView({ smooth })` |
| **Measuring elements** | `getBoundingClientRect` in effect/event |
| **Refs in lists** | `Map<id, node>` pattern |
| **Callback refs** | Mount/unmount hook per node |
| **Controlled vs uncontrolled ref use** | State validation vs one-shot reads |

## The Mental Model

```text
Need the DOM?
  │
  ├── single node → useRef → focus / scroll / measure
  │
  ├── child node → forwardRef + imperative handle
  │
  └── many nodes → callback refs → Map<id, node>
        └── effects/events only, never render
```

### The 3 things to remember

**1. Expose narrowly:**

```jsx
useImperativeHandle(ref, () => ({ focus: () => node.current.focus() }));
```

**2. Scroll on demand:**

```jsx
end.current.scrollIntoView({ behavior: "smooth" });
```

**3. List refs as Map:**

```jsx
ref={(n) => (n ? map.current.set(u.id, n) : map.current.delete(u.id))}
```

> **Refs escape React for focus, scroll, and measure — expose little, measure late, clean up.**
