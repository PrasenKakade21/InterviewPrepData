# Conditional Rendering

Conditional rendering clicks as: `state` → `choose branch` → `React shows it`. Same component, different UI depending on login, loading, data, or toggles.

The core idea:

```text
Condition value
    ↓
pick JSX branch
    ↓
React renders only that branch
    ↓
UI matches state
```

React reuses normal JS — `if`, ternary, `&&`, early returns — to decide what appears.

---

# 1. if / else Rendering, Ternary Rendering, and Logical && Rendering

`if / else` outside JSX picks big branches. `Ternary` (`cond ? a : b`) picks either/or inline. `Logical &&` (`cond && <UI />`) shows something or nothing.

```jsx
function Status({ isMember }) {
  if (isMember) {
    return <p>Welcome back Kakade</p>;
  } else {
    return <p>Please join us</p>;
  }
}
```

Output:

```text
When isMember is true: Welcome back Kakade
When false: Please join us
```

Ternary + `&&` inline:

```jsx
function Inbox({ unread }) {
  return (
    <div>
      <p>{unread > 0 ? `${unread} new` : "All caught up"}</p>
      {unread > 0 && <button>Mark read for Kakade</button>}
    </div>
  );
}
```

Output:

```text
When unread is 3: 3 new + button. When 0: All caught up, no button.
```

Think:

```text
if/else → whole returns differ
ternary → two inline options
&& → show or hide one thing
```

---

# 2. Early Returns and Null Rendering

`Early returns` exit before the main JSX for loading, error, or permission cases — flattest way to handle branches. `Null rendering` returns `null` to render nothing while keeping hooks and component alive.

```jsx
function Profile({ user }) {
  if (!user) {
    return <p>Please log in, Kakade</p>;
  }
  return <h2>Hello {user}</h2>;
}
```

Output:

```text
When user is null: Please log in, Kakade
When user is Kakade: Hello Kakade
```

Null example:

```jsx
function Warning({ show }) {
  if (!show) {
    return null;
  }
  return <p>Careful, Kakade!</p>;
}
```

Output:

```text
When show is false: renders nothing. When true: Careful, Kakade!
```

Notice: hooks must stay before any early return — otherwise hook order breaks between renders.

---

# 3. Loading States, Empty States, and Toggle UI

`Loading states` show a spinner while waiting. `Empty states` guide users when lists have no data. `Toggle UI` flips between two views with a boolean.

```jsx
function Users({ loading, users }) {
  if (loading) {
    return <p>Loading Kakade…</p>;
  }
  if (users.length === 0) {
    return <p>No users yet. Invite Kakade!</p>;
  }
  return <p>Found {users.length} users</p>;
}
```

Output:

```text
Loading… → No users yet… → Found 2 users (as state changes).
```

Toggle:

```jsx
import { useState } from "react";

function Panel() {
  const [open, setOpen] = useState(false);
  return (
    <div>
      <button onClick={() => setOpen((o) => !o)}>
        {open ? "Hide" : "Show"} Kakade
      </button>
      {open && <p>Secret details</p>}
    </div>
  );
}
```

Output:

```text
Button toggles Show/Hide; details appear only when open.
```

Good empty states tell users what to do next, not just what is missing.

---

# 4. 0 and Falsy Gotchas

JSX renders `0` as text but ignores `null`, `undefined`, `false`, and `""`. So `{count && <p>}` shows a stray `0` when count is zero — guard with an explicit comparison.

```jsx
function Cart({ count }) {
  return <div>{count > 0 && <p>{count} items for Kakade</p>}</div>;
}
```

Output:

```text
When count is 0: empty div. When 2: 2 items for Kakade.
```

Bad vs Good:

```jsx
// Bad: renders 0 on screen
// {count && <p>Items</p>}
```

```jsx
function CartFixed({ count }) {
  return <div>{count > 0 && <p>Items for Kakade</p>}</div>;
}
```

Output:

```text
When count is 0: nothing. No stray 0.
```

| Value | Renders? |
| ----- | -------- |
| **`0`** | Yes — shows `0` |
| **`null` / `undefined` / `false`** | No — renders nothing |
| **`""`** | No visible output |

Interview trap: any numeric `&&` needs `> 0` or `!== 0` guard. Same for `count` from APIs.

---

# Real-World Uses

A dashboard combines all patterns: early return for no session, ternary for role badges, `&&` for admin tools, and loading/empty states for the orders list.

```jsx
function Dashboard({ user, orders }) {
  if (!user) {
    return <p>Log in, Kakade</p>;
  }
  return <p>{orders.length === 0 ? "No orders" : `${orders.length} orders`}</p>;
}
```

Output:

```text
Log in, Kakade / No orders / 3 orders depending on state.
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **if / else rendering** | Branch whole returns outside JSX |
| **Ternary rendering** | `cond ? <A /> : <B />` for either/or |
| **Logical && rendering** | `cond && <UI />` for show-or-hide |
| **Early returns** | Exit first for loading / error / no-access |
| **Null rendering** | `return null` renders nothing |
| **Loading states** | Spinner while `loading` is true |
| **Empty states** | Helpful message + next action |
| **Toggle UI** | Boolean flip with `!prev` |
| **0 and falsy gotchas** | `0` renders; use `count > 0 &&` |

## The Mental Model

```text
State (user / loading / list / toggle)
  │
  ├── no user → early return login
  ├── loading → spinner
  ├── empty list → empty-state help
  │       │
  │       └── has data → ternary / && picks UI
  │               │
  │               └── guard numbers (count > 0)
```

### The 3 things to remember

**1. Either/or inline:**

```jsx
{isMember ? <p>Hi Kakade</p> : <p>Join us</p>}
```

**2. Show-or-hide guarded:**

```jsx
{count > 0 && <p>Items for Kakade</p>}
```

**3. Early exit first:**

```jsx
if (!user) {
  return <p>Log in, Kakade</p>;
}
```

> **Pick the branch that matches state — and guard numbers so a stray 0 never leaks onto the screen.**
