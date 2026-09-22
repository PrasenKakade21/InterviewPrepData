# Custom Hooks

Custom hooks click as: `repeated stateful logic` → `extract to useX()` → `reuse everywhere`. They share behavior (fetching, toggles, storage) without sharing UI.

The core idea:

```text
Component logic repeats
    ↓
move state + effects to useName()
    ↓
return values / handlers
    ↓
components call it like a hook
```

A custom hook is just a function starting with `use` that calls other hooks inside.

---

# 1. What are Custom Hooks, Rules of Hooks, and Hook Naming

`What are custom hooks`: reusable functions encapsulating stateful logic. `Rules of hooks`: call hooks at top level only, and only in React functions or other hooks. `Hook naming with use prefix` (`useFetch`, `useToggle`) lets React enforce rules via lint and signals hook behavior.

```jsx
import { useState } from "react";

function useToggle(initial = false) {
  const [on, setOn] = useState(initial);
  function toggle() {
    setOn((o) => !o);
  }
  return [on, toggle];
}

function Panel() {
  const [open, toggle] = useToggle(false);
  return <button onClick={toggle}>{open ? "Hide" : "Show"} Kakade</button>;
}
```

Output:

```text
Button toggles Show/Hide Kakade.
```

Think:

```text
useToggle() → [value, helpers]
    ├── Panel instance → own state
    └── Sidebar instance → separate state
```

Each call gets isolated state — hooks share logic, not data. Interview trap: naming a hook without `use` breaks lint rules and confuses readers.

---

# 2. Extracting Reusable Logic and Sharing Stateful Logic

`Extracting reusable logic` moves copy-pasted `useState` + `useEffect` pairs into one hook returning a tidy API. `Sharing stateful logic` means two components use the same behavior with independent state — no render props or HOCs needed.

```jsx
import { useState } from "react";

function useCounter(start = 0) {
  const [n, setN] = useState(start);
  function inc() {
    setN((v) => v + 1);
  }
  return [n, inc];
}

function A() {
  const [n, inc] = useCounter(0);
  return <button onClick={inc}>A: {n} for Kakade</button>;
}
```

Output:

```text
Each component using useCounter tracks its own n.
```

Notice: if you need shared *data* (not just shared logic), lift state up or add Context — hooks alone do not sync instances.

---

# 3. useFetch Example, useToggle Example, and useLocalStorage Example

`useFetch example` wraps loading/data/error for GET requests. `useToggle example` condenses booleans. `useLocalStorage example` persists state across reloads.

```jsx
import { useEffect, useState } from "react";

function useFetch(url) {
  const [data, setData] = useState(null);
  useEffect(() => {
    fetch(url).then((r) => r.json()).then(setData);
  }, [url]);
  return data;
}

function User() {
  const user = useFetch("https://jsonplaceholder.typicode.com/users/1");
  return <p>{user ? user.name : "…"} friend of Kakade</p>;
}
```

Output:

```text
Leanne Graham friend of Kakade after load.
```

Storage hook:

```jsx
import { useState } from "react";

function useLocalStorage(key, initial) {
  const [value, setValue] = useState(() => {
    return localStorage.getItem(key) || initial;
  });
  function save(next) {
    setValue(next);
    localStorage.setItem(key, next);
  }
  return [value, save];
}

function NameSaver() {
  const [name, setName] = useLocalStorage("name", "Kakade");
  return <input value={name} onChange={(e) => setName(e.target.value)} />;
}
```

Output:

```text
Input persists Kakade across page reloads.
```

Design hooks to return consistent shapes: tuples for pairs (`[v, set]`), objects for 3+ fields (`{ data, loading, error }`).

---

# 4. Custom Hook Testing Basics

`Custom hook testing basics`: render the hook in isolation with React Testing Library's `renderHook`, then assert return values and state transitions via `act`.

```jsx
import { useState } from "react";

function useUpper(name) {
  const [v] = useState(name.toUpperCase());
  return v;
}
```

Output:

```text
useUpper("kakade") → "KAKADE"
```

Test shape:

```js
// useToggle.test.js
// const { result } = renderHook(() => useToggle(false));
// act(() => result.current[1]());
// expect(result.current[0]).toBe(true);
```

Cover initial value, transitions, and cleanup (unmount cancels listeners). Pure helpers inside hooks can be unit-tested without rendering at all.

---

# Real-World Uses

A settings page reuses `useLocalStorage` for theme, `useToggle` for panels, and `useFetch` for the Kakade profile — three one-liners replacing duplicated effect blocks.

```jsx
function Settings() {
  const [open, toggle] = useToggle(false);
  return <button onClick={toggle}>{open ? "Close" : "Open"} settings</button>;
}
```

Output:

```text
Open/Close settings toggle for Kakade.
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **What are custom hooks** | `useX()` sharing stateful logic |
| **Rules of hooks** | Top-level, React fns / hooks only |
| **Extracting reusable logic** | Move state + effects out of components |
| **useFetch example** | `{ data, loading, error }` wrapper |
| **useToggle example** | `[on, toggle]` boolean helper |
| **useLocalStorage example** | State synced to `localStorage` |
| **Hook naming with use prefix** | Required `use` + lint enforcement |
| **Sharing stateful logic** | Shared code, isolated state per call |
| **Custom hook testing basics** | `renderHook` + `act` assertions |

## The Mental Model

```text
Duplicated useState + useEffect
  │
  ↓
useName() custom hook (use-prefixed, rules-safe)
  │
  ├── useFetch → remote data
  ├── useToggle → booleans
  └── useLocalStorage → persisted state
        │
        ↓
  components share logic, keep own state
```

### The 3 things to remember

**1. use-prefix + top-level:**

```jsx
function useToggle(initial = false) {
  const [on, setOn] = useState(initial);
  return [on, () => setOn((o) => !o)];
}
```

**2. Fetch wrapper:**

```jsx
const user = useFetch("/api/kakade");
```

**3. Persisted state:**

```jsx
const [name, setName] = useLocalStorage("name", "Kakade");
```

> **When stateful logic repeats, extract it to a use-prefixed hook that each component calls independently.**
