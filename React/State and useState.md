# State and useState

State clicks when you see: `event happens` → `state updates` → `React re-renders`. Props are inputs from parents; state is memory a component owns and changes over time.

The core idea:

```text
User action
    ↓
setState(new value)
    ↓
React re-renders
    ↓
UI shows new state
```

`State` is **data that changes over renders** — counters, form text, toggles. `useState` is the hook that gives a function component that memory.

---

# 1. What is State and useState Basics

`What is state`: private, mutable data owned by one component. `useState basics`: call `useState(initial)` to get the current value plus a setter; calling the setter schedules a re-render.

```jsx
import { useState } from "react";

function Counter() {
  const [count, setCount] = useState(0);
  return <button onClick={() => setCount(count + 1)}>{count}</button>;
}
```

Output:

```text
Button shows 0, then 1, 2… on each click.
```

Think:

```text
useState(0)
  ├── count → current render value
  └── setCount → queues update + re-render
```

Rules: call hooks at the top level only, never inside `if` or loops. Each component instance gets its own isolated state.

---

# 2. State vs Props and Local vs Shared State

`State vs props`: props are read-only inputs from a parent; state is owned memory the component can change. `Local vs shared state`: keep state as low as possible, lift it only when two components need the same value.

```jsx
function Hello({ name }) {
  return <p>Hello {name}</p>;
}

function App() {
  return <Hello name="Kakade" />;
}
```

Output:

```text
Hello Kakade (name is a prop, App cannot be changed by Hello)
```

Local vs shared:

```jsx
import { useState } from "react";

function Chat({ user, message }) {
  return <p>{user}: {message}</p>;
}

function ChatApp() {
  const [message, setMessage] = useState("Hi Kakade");
  return <Chat user="Asha" message={message} />;
}
```

Output:

```text
Asha: Hi Kakade
```

| Question | Answer |
| -------- | ------ |
| **Can child change it?** | Props: no. State: yes, via setter |
| **Where does it live?** | Props: parent. State: this component |
| **When to lift?** | When siblings need the same value |

Colocate state — lifting everything to `App` causes needless re-renders.

---

# 3. State Batching and Functional Updates

`State batching` means React groups multiple setters in one event into a single re-render. `Functional updates` (`setCount(c => c + 1)`) read the latest queued value, so rapid updates do not overwrite each other.

```jsx
import { useState } from "react";

function Triple() {
  const [count, setCount] = useState(0);
  function addThree() {
    setCount((c) => c + 1);
    setCount((c) => c + 1);
    setCount((c) => c + 1);
  }
  return <button onClick={addThree}>{count}</button>;
}
```

Output:

```text
Each click adds 3 (0 → 3 → 6…).
```

Bad (stale value — all three read the same old `count`):

```jsx
function Stale() {
  const [count, setCount] = useState(0);
  function addThree() {
    setCount(count + 1);
    setCount(count + 1);
    setCount(count + 1);
  }
  return <button onClick={addThree}>{count}</button>;
}
```

Output:

```text
Each click adds only 1 (0 → 1 → 2…).
```

Interview trap: use the `c => c + 1` form whenever the next state depends on the previous state.

---

# 4. State with Objects and State with Arrays

State can hold `objects` (forms, users) and `arrays` (todos, lists). React compares by reference, so you must **replace** them with new copies — never mutate with `push` or direct assignment.

```jsx
import { useState } from "react";

function Profile() {
  const [user, setUser] = useState({ name: "Kakade", age: 22 });
  function birthday() {
    setUser({ ...user, age: user.age + 1 });
  }
  return <button onClick={birthday}>{user.age}</button>;
}
```

Output:

```text
Button shows 22, then 23, 24… on each click.
```

Arrays:

```jsx
import { useState } from "react";

function TodoList() {
  const [todos, setTodos] = useState(["Learn JSX"]);
  function add() {
    setTodos([...todos, "Learn state"]);
  }
  return <button onClick={add}>Todos: {todos.length}</button>;
}
```

Output:

```text
Button shows Todos: 1, then Todos: 2…
```

Patterns: update with `map`/`filter`, remove with `filter`, toggle with `map`. For deeply nested objects, consider `useReducer` or Immer.

---

# 5. Lifting State and State Update Gotchas

`Lifting state` means moving it to the nearest common parent so children share one source of truth. `State update gotchas`: state is async/snapshot-like, mutating objects fails silently, and stale closures capture old values.

```jsx
import { useState } from "react";

function Input({ value, onValue }) {
  return <input value={value} onChange={(e) => onValue(e.target.value)} />;
}

function Mirror({ value }) {
  return <p>Hi {value}</p>;
}

function Form() {
  const [name, setName] = useState("Kakade");
  return (
    <>
      <Input value={name} onValue={setName} />
      <Mirror value={name} />
    </>
  );
}
```

Output:

```text
Typing in input instantly updates Hi … mirror text.
```

Gotchas table:

| Trap | Why | Fix |
| ---- | --- | --- |
| **Read state right after setter** | Setter is async | Use the value you computed, or `useEffect` |
| **`arr.push()` then setter** | Same reference, no re-render | `setArr([...arr, item])` |
| **Stale closure in timeout** | Timeout captured old state | Functional update or ref |
| **Derived copy in state** | Gets out of sync | Compute during render instead |

```jsx
import { useState } from "react";

function Fixed({ items }) {
  const [filter, setFilter] = useState("");
  const visible = items.filter((i) => i.includes(filter));
  return <p>Visible: {visible.length}</p>;
}
```

Output:

```text
Visible count always matches current filter.
```

Notice: do not store what you can derive — `visible` is computed, not a second `useState`.

---

# Real-World Uses

A signup form uses all of it: object state for fields, array state for errors, functional updates for counters, and lifted state so the preview card mirrors the form.

```jsx
import { useState } from "react";

function Signup() {
  const [form, setForm] = useState({ name: "Kakade" });
  return <p>Signing up: {form.name}</p>;
}
```

Output:

```text
Signing up: Kakade
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **What is state** | Owned, changeable memory that triggers re-render |
| **useState basics** | `const [v, setV] = useState(init)` |
| **State vs props** | State owned + mutable; props parent-owned + read-only |
| **State batching** | Multiple setters → one re-render |
| **Functional updates** | `setC(c => c + 1)` when next depends on prev |
| **State with objects** | Replace via `{...obj, key: val}` |
| **State with arrays** | Replace via `[...arr]`, `map`, `filter` |
| **Lifting state** | Move shared value to common parent |
| **Local vs shared state** | Keep low, lift only when shared |
| **State update gotchas** | Async setter, no mutation, no derived copies |

## The Mental Model

```text
Event / interaction
  │
  ↓
setState (functional + immutable copy)
  │
  ↓
React batches → re-renders
  │
  ├── local state → this component only
  └── lifted state → parent + all children sync
```

### The 3 things to remember

**1. Declare state:**

```jsx
const [count, setCount] = useState(0);
```

**2. Update functionally + immutably:**

```jsx
setCount((c) => c + 1);
setTodos([...todos, "Learn Kakade"]);
```

**3. Lift when shared:**

```jsx
function Parent() {
  const [name, setName] = useState("Kakade");
  return <Child value={name} onValue={setName} />;
}
```

> **Own state locally, update it immutably with the setter, and lift it only when someone else needs it.**
