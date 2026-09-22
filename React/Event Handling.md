# Event Handling

Event handling clicks when you see: `user acts` → `handler runs` → `state updates`. React wraps browser events in a consistent system, so one pattern covers clicks, typing, submits, and keys.

The core idea:

```text
JSX event prop
    ↓
user interacts
    ↓
handler function runs
    ↓
state updates → re-render
```

React events are props like `onClick` and `onChange` — camelCase, receiving a function, not a string.

---

# 1. onClick, onChange, and onSubmit

`onClick` fires on clicks, `onChange` fires on every input change, and `onSubmit` fires when a form submits. All three take a function reference you define in the component.

```jsx
import { useState } from "react";

function Clicker() {
  const [count, setCount] = useState(0);
  function handleClick() {
    setCount((c) => c + 1);
  }
  return <button onClick={handleClick}>Hi Kakade: {count}</button>;
}
```

Output:

```text
Hi Kakade: 0 → 1 → 2 on each click.
```

`onChange` + `onSubmit` together:

```jsx
import { useState } from "react";

function NameForm() {
  const [name, setName] = useState("");
  function handleSubmit(e) {
    e.preventDefault();
    alert(`Hello ${name || "Kakade"}`);
  }
  return (
    <form onSubmit={handleSubmit}>
      <input value={name} onChange={(e) => setName(e.target.value)} />
      <button type="submit">Save</button>
    </form>
  );
}
```

Output:

```text
Typing updates name; submit shows alert without page reload.
```

Notice: pass `handleClick`, not `handleClick()` — parentheses would call it during render.

---

# 2. Event Objects, Passing Arguments, and Handler Naming

The `event object` (`e`) carries details like `e.target.value`. To pass extra data, wrap the call in an arrow function. Name handlers `handleX` for definitions and `onX` for props.

```jsx
function Greeter() {
  function handleGreet(name) {
    alert(`Hello ${name}`);
  }
  return <button onClick={() => handleGreet("Kakade")}>Greet</button>;
}
```

Output:

```text
Click → alert: Hello Kakade
```

Think:

```text
onClick={fn} → no args, gets event
onClick={() => fn(id)} → with args
onClick={fn(id)} → WRONG, runs on render
```

Naming table:

| Role | Convention | Example |
| ---- | ---------- | ------- |
| **Definition** | `handle` + Event | `handleClick`, `handleChange` |
| **Prop** | `on` + Event | `onSelect`, `onDelete` |
| **With value** | `handleUserDelete` | `onClick={() => handleUserDelete(id)}` |

Interview trap: `onClick={handleDelete(user.id)}` deletes on render. Always wrap with `() =>` when arguments are needed.

---

# 3. Prevent Default and Stop Propagation

`Prevent default` (`e.preventDefault()`) stops the browser's built-in action like form reload or link navigation. `Stop propagation` (`e.stopPropagation()`) stops the event bubbling to parent handlers.

```jsx
function SignupForm() {
  function handleSubmit(e) {
    e.preventDefault();
    console.log("Saved Kakade, no reload");
  }
  return (
    <form onSubmit={handleSubmit}>
      <button type="submit">Join</button>
    </form>
  );
}
```

Output:

```text
Saved Kakade, no reload
```

Propagation example:

```jsx
function Card() {
  function handleCardClick() {
    console.log("card");
  }
  function handleButtonClick(e) {
    e.stopPropagation();
    console.log("button only");
  }
  return (
    <div onClick={handleCardClick}>
      <button onClick={handleButtonClick}>Hi Kakade</button>
    </div>
  );
}
```

Output:

```text
Click button → button only (card handler skipped).
```

Think:

```text
button click
    ↓
button handler → stopPropagation?
    ├── yes → parent skipped
    └── no → parent handler also runs
```

---

# 4. Synthetic Events and Keyboard Events

`Synthetic events` are React's cross-browser wrapper around native events — same `e.target`, `e.preventDefault()`, and `e.stopPropagation()` everywhere. `Keyboard events` (`onKeyDown`) let you react to Enter, Escape, and shortcuts.

```jsx
function SearchBox() {
  function handleKeyDown(e) {
    if (e.key === "Enter") {
      console.log(`Search for ${e.target.value}`);
    }
  }
  return <input placeholder="Search Kakade" onKeyDown={handleKeyDown} />;
}
```

Output:

```text
Press Enter → logs Search for <typed text>.
```

Compare:

```text
Native: e.keyCode varies, attach via addEventListener
React:  e.key is normalized ("Enter"), attach via onKeyDown prop
```

Notice: use `e.key` (`"Enter"`, `"Escape"`), not deprecated `e.keyCode`. For form-wide Enter handling, prefer `onSubmit` on the `<form>` — it also covers mobile keyboards.

---

# Real-World Uses

A todo row uses everything: `onClick` to toggle, argument passing for the id, `stopPropagation` so the delete button does not toggle, and `onKeyDown` for Enter-to-save in the edit box.

```jsx
function TodoRow({ todo, onToggle, onDelete }) {
  return (
    <div onClick={() => onToggle(todo.id)}>
      <span>{todo.text} for Kakade</span>
      <button onClick={(e) => { e.stopPropagation(); onDelete(todo.id); }}>
        Delete
      </button>
    </div>
  );
}
```

Output:

```text
Click row toggles; Delete removes without toggling.
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **onClick** | `onClick={handleClick}`, no parens |
| **onChange** | Every keystroke, pair with `value` |
| **onSubmit** | Form submit, needs `preventDefault` |
| **Event objects** | `e.target.value`, `e.key`, `e.preventDefault()` |
| **Passing arguments** | `onClick={() => fn(id)}` |
| **Prevent default** | Stops reload / navigation |
| **Stop propagation** | Stops parent handlers |
| **Synthetic events** | React's normalized cross-browser wrapper |
| **Keyboard events** | `onKeyDown`, check `e.key === "Enter"` |
| **Handler naming** | `handleX` defines, `onX` props |

## The Mental Model

```text
User action (click / type / submit / key)
  │
  ↓
Synthetic event (e)
  │
  ├── preventDefault → block browser default
  ├── stopPropagation → block parent handlers
  │       │
  │       ↓
  └── handler(name, args) → setState → re-render
```

### The 3 things to remember

**1. Reference, not call:**

```jsx
<button onClick={handleClick}>Hi Kakade</button>
```

**2. Args need a wrapper:**

```jsx
<button onClick={() => handleDelete(user.id)}>Delete</button>
```

**3. Forms need preventDefault:**

```jsx
function handleSubmit(e) {
  e.preventDefault();
}
```

> **Attach a handler reference, use the event object wisely, and let the handler update state.**
