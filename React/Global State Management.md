# Global State Management

Global state clicks as: `local first` → `lift or context` → `library when shared + updated widely`. Most state stays local; only cross-cutting data (auth, cart) goes global.

The core idea:

```text
Component state (default)
    ↓ shared by few? → lift / Context
    ↓ shared by many + frequent? → Redux / Zustand
```

Libraries add a single store outside React with subscribed readers — pick the smallest one that fits.

---

# 1. Local vs Global State, When to Add a Library, and Context vs Redux vs Zustand

`Local vs global state`: form input and toggles are local; logged-in user and cart are global. `When to add a library`: prop drilling hurts, many distant components read/write the same data, or updates need DevTools/time-travel. `Context vs Redux vs Zustand`: Context broadcasts (no selectors), Redux standardizes large flows, Zustand gives tiny selector-based stores.

```jsx
import { useState } from "react";

function Toggle() {
  const [open, setOpen] = useState(false);
  return <button onClick={() => setOpen(!open)}>{open ? "Hide" : "Show"} Kakade</button>;
}
```

Output:

```text
Show/Hide Kakade — local, no library needed.
```

| Tool | Best for |
| ---- | -------- |
| **Local state** | **Toggles, inputs, page-only data** |
| **Context + reducer** | **Theme, auth in small apps** |
| **Zustand** | **Small/medium shared state, minimal boilerplate** |
| **Redux Toolkit** | **Large teams, complex flows, strict conventions** |

Rule: do not globalize fetch caches or form drafts — server cache belongs to Query, ephemeral UI stays local.

---

# 2. Redux Toolkit Basics, Slices, and Store and Provider

`Redux Toolkit basics`: `configureStore` + `createSlice` remove classic Redux boilerplate. `Slices` bundle `name + initialState + reducers` and auto-generate actions. `Store and Provider` expose the store via `<Provider store={store}>`.

```jsx
import { configureStore } from "@reduxjs/toolkit";

const store = configureStore({
  reducer: {},
});
```

Output:

```text
Empty store ready; slices plug into reducer map.
```

Slice shape:

```js
// import { createSlice } from "@reduxjs/toolkit";
// const userSlice = createSlice({
//   name: "user",
//   initialState: { name: "Kakade" },
//   reducers: {
//     rename(state, action) { state.name = action.payload; },
//   },
// });
// export const { rename } = userSlice.actions;
```

Think:

```text
createSlice → { reducer, actions }
    ↓
configureStore({ reducer: { user: userSlice.reducer } })
    ↓
<Provider store={store}> → tree reads it
```

RTK uses Immer inside, so "mutating" syntax in slice reducers is safe and idiomatic.

---

# 3. useSelector and useDispatch, Zustand Basics, and Normalizing State

`useSelector and useDispatch`: selector reads a slice (`state.user.name`), dispatch sends actions. `Zustand basics`: `create(set => ({...}))` makes a hook-store with per-component selectors. `Normalizing state` stores entities by id (`{ byId, ids }`) instead of nested trees.

```jsx
import { useSelector, useDispatch } from "react-redux";

function Name() {
  const name = useSelector((s) => s.user.name);
  const dispatch = useDispatch();
  return <button onClick={() => dispatch({ type: "user/rename", payload: "Kakade" })}>{name}</button>;
}
```

Output:

```text
Button shows Kakade; click dispatches rename.
```

Zustand shape:

```js
// import { create } from "zustand";
// export const useUser = create((set) => ({
//   name: "Kakade",
//   rename: (name) => set({ name }),
// }));
// useUser((s) => s.name) → subscribes only to name.
```

Normalized sketch:

```js
// { byId: { kakade: { id: "kakade", name: "Kakade" } }, ids: ["kakade"] }
```

Selectors prevent over-rendering: `useSelector(s => s.user.name)` re-renders only when `name` changes, unlike whole-object Context reads.

---

# 4. DevTools

`DevTools`: Redux DevTools (extension) logs every action + state diff with time-travel; Zustand plugs into the same extension. Name actions clearly and keep state serializable.

```jsx
function CartCount() {
  return <p>Cart for Kakade: 2</p>;
}
```

Output:

```text
Cart for Kakade: 2 ( DevTools shows cart/add → count 2 ).
```

Checklist: install extension, avoid non-serializable values (class instances, functions) in store, and use action prefixes (`cart/add`) so logs read like a story.

---

# Real-World Uses

A shop keeps the cart in Zustand (`items`, `add`, `clear`) with normalized products by id, while Kakade's auth user lives in a Redux slice so the team gets DevTools traces across checkout steps.

```jsx
function Badge() {
  return <p>Hi Kakade: 3 items</p>;
}
```

Output:

```text
Hi Kakade: 3 items (read from global store).
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **Local vs global state** | Local default; global = cross-cutting |
| **Redux Toolkit basics** | `configureStore` + slices, less boilerplate |
| **Slices** | Name + state + reducers → actions |
| **Store and Provider** | `<Provider store>` exposes store |
| **useSelector and useDispatch** | Read slice, send actions |
| **Zustand basics** | `create()` hook-store with selectors |
| **Context vs Redux vs Zustand** | Broadcast vs conventions vs minimal |
| **When to add a library** | Drilling pain + wide shared updates |
| **Normalizing state** | `{ byId, ids }` flat entities |
| **DevTools** | Action log + time-travel debugging |

## The Mental Model

```text
Local state (toggles, forms)
  │
  ├── few sharers → lift / Context + reducer
  │
  └── many sharers + frequent updates
        ├── Zustand → tiny selector store
        └── Redux Toolkit → slices + store + DevTools
              │
              └── normalized { byId, ids }
```

### The 3 things to remember

**1. Stay local first:**

```jsx
const [open, setOpen] = useState(false);
```

**2. Slice + store:**

```jsx
const store = configureStore({ reducer: {} });
```

**3. Select narrowly:**

```jsx
const name = useSelector((s) => s.user.name);
```

> **Keep state as local as possible; globalize only widely shared data with selectors and DevTools.**
