# useReducer and Advanced State

This topic clicks as: `actions describe what happened` → `reducer computes next state` → `dispatch triggers it`. When one object has many update paths, a reducer centralizes the rules instead of scattering setters.

The core idea:

```text
Event → dispatch({ type })
    ↓
reducer(state, action) → new state
    ↓
re-render with consistent state
```

`useReducer` is `useState` with explicit transitions — ideal for forms, carts, and wizards.

---

# 1. useReducer Basics, Reducer Functions, and Dispatch Actions

`useReducer basics`: `const [state, dispatch] = useReducer(reducer, initial)`. `Reducer functions` are pure `(state, action) => newState`. `Dispatch actions` are plain objects like `{ type: "add", name }` that request a change.

```jsx
import { useReducer } from "react";

function reducer(state, action) {
  if (action.type === "add") {
    return { count: state.count + 1 };
  }
  return state;
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, { count: 0 });
  return <button onClick={() => dispatch({ type: "add" })}>{state.count}</button>;
}
```

Output:

```text
Button shows 0 → 1 → 2 for Kakade on each click.
```

Think:

```text
dispatch({ type: "add" })
    ↓
reducer reads type → returns { count + 1 }
    ↓
React replaces state → re-renders
```

Rules: reducers must be pure — same inputs give same output, no mutation, no API calls inside.

---

# 2. useState vs useReducer, Complex State Objects, and Action Conventions

`useState vs useReducer`: one independent value favors `useState`; linked fields with many transitions favor `useReducer`. `Complex state objects` (cart, form with status) stay consistent via single transitions. `Action conventions` use `type` strings plus payload fields, often `type: "domain/event"`.

```jsx
import { useReducer } from "react";

function cartReducer(state, action) {
  if (action.type === "cart/add") {
    return { items: [...state.items, action.item] };
  }
  if (action.type === "cart/clear") {
    return { items: [] };
  }
  return state;
}

function Cart() {
  const [state, dispatch] = useReducer(cartReducer, { items: [] });
  return <button onClick={() => dispatch({ type: "cart/add", item: "Course" })}>
    Items: {state.items.length} for Kakade
  </button>;
}
```

Output:

```text
Items: 0 → 1 → 2 as Kakade adds courses.
```

| Signal | Choice |
| ------ | ------ |
| **Single toggle / string** | `useState` |
| **Object + many actions** | `useReducer` |
| **Shared across tree** | Reducer + Context |

Convention: `switch` on `type`, throw on unknown types in dev so typos surface fast.

---

# 3. Immer Basics and State Machines Concept

`Immer basics`: write "mutating" drafts (`draft.items.push(x)`) and Immer produces the immutable copy — removes spread noise for nested state. `State machines concept`: model state as finite modes (`idle → loading → done`) with allowed transitions, so impossible states cannot exist.

```jsx
import { useReducer } from "react";

function statusReducer(state, action) {
  if (action.type === "load") {
    return { mode: "loading" };
  }
  if (action.type === "done") {
    return { mode: "done" };
  }
  return state;
}

function Loader() {
  const [state, dispatch] = useReducer(statusReducer, { mode: "idle" });
  return <button onClick={() => dispatch({ type: "load" })}>{state.mode}</button>;
}
```

Output:

```text
Button shows idle → loading for Kakade.
```

Immer shape (needs `use-immer` package):

```js
// With Immer: draft.items.push("Course") replaces spreads.
// Without: { ...state, items: [...state.items, "Course"] }
```

Think:

```text
idle →(load)→ loading →(done)→ done
  └── no jump idle → done (machine forbids)
```

Machines shine for checkout, auth, and upload flows where status enums beat loose booleans.

---

# 4. Testing Reducers

`Testing reducers`: because reducers are pure functions, tests call them directly — no rendering needed. Assert each action returns the expected next state.

```jsx
function counterReducer(state, action) {
  if (action.type === "inc") {
    return { count: state.count + 1 };
  }
  return state;
}
```

Output:

```text
counterReducer({ count: 0 }, { type: "inc" }) → { count: 1 }
```

Example test shape (Vitest):

```js
// counter.test.js
// expect(counterReducer({ count: 0 }, { type: "inc" }))
//   .toEqual({ count: 1 });
// expect(counterReducer({ count: 1 }, { type: "reset" }))
//   .toEqual({ count: 0 });
```

Notice: export the reducer separately from the component so tests import logic without React. Cover happy paths plus unknown-action fallback.

---

# Real-World Uses

A checkout wizard uses a machine-style reducer: `cart → address → pay → done`, each step dispatching typed actions, with Immer keeping nested address updates readable.

```jsx
import { useReducer } from "react";

function Wizard() {
  const [state, dispatch] = useReducer(statusReducer, { mode: "idle" });
  return <p>Step: {state.mode} for Kakade</p>;
}
```

Output:

```text
Step: idle for Kakade (advances via dispatch).
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **useReducer basics** | `[state, dispatch] = useReducer(fn, init)` |
| **Reducer functions** | Pure `(state, action) => next` |
| **Dispatch actions** | `{ type, ...payload }` objects |
| **useState vs useReducer** | Single value vs linked transitions |
| **Complex state objects** | One transition keeps fields consistent |
| **Action conventions** | `domain/event` types, `switch` on type |
| **Immer basics** | Mutate draft, get immutable copy |
| **State machines concept** | Finite modes + allowed transitions |
| **Testing reducers** | Pure calls, no render needed |

## The Mental Model

```text
UI events
  │
  ↓
dispatch({ type, payload })
  │
  ↓
reducer (pure + machine rules) → next state
  │
  └── ImmerDraft → immutable copy (nested-safe)
        │
        ↓
    re-render + easy unit tests
```

### The 3 things to remember

**1. Declare reducer + dispatch:**

```jsx
const [state, dispatch] = useReducer(reducer, { count: 0 });
```

**2. Dispatch typed actions:**

```jsx
dispatch({ type: "cart/add", item: "Course" });
```

**3. Keep reducer pure:**

```jsx
function reducer(state, action) {
  return { count: state.count + 1 };
}
```

> **Describe changes as actions, compute next state in one pure place, and test that function directly.**
