# Components

Components make sense as: `function` → `returns UI` → `composes into app`. Everything on screen — buttons, cards, pages — is just components nested inside components.

The core idea:

```text
Small function
    ↓
returns JSX
    ↓
compose into tree
    ↓
full app UI
```

A component is a **reusable, independent piece of UI** with its own markup and optionally logic. React apps are trees with `App` at the root.

---

# 1. Functional Components and Class Components Overview

`Functional components` are plain JS functions that return JSX — this is the modern standard. `Class components` use `class extends React.Component` with `render()` and lifecycle methods; you mainly read them in legacy code.

```jsx
function Greet() {
  return <h1>Hello Kakade</h1>;
}
```

Output:

```text
Hello Kakade
```

Class version (overview only):

```jsx
import { Component } from "react";

class GreetOld extends Component {
  render() {
    return <h1>Hello Kakade</h1>;
  }
}
```

Output:

```text
Hello Kakade
```

Think:

```text
Function → return UI (modern, hooks)
Class → render() + this.state (legacy)
```

Interview trap: new code uses functions + hooks. Know classes exist, but do not write new ones. Component names must start with a capital letter or React treats them as HTML tags.

---

# 2. Component Composition and Parent and Child Components

`Composition` means building big UI by nesting small components. A `parent` renders one or more `children` — data flows down, and layout flows by nesting.

```jsx
function Avatar() {
  return <p>Kakade</p>;
}

function Profile() {
  return (
    <div>
      <h2>Profile</h2>
      <Avatar />
    </div>
  );
}
```

Output:

```text
Profile
Kakade
```

Think:

```text
Parent (Profile)
    ├── <h2> title
    └── Child (Avatar)
```

Keep each component focused on one job. If `Profile` grows, extract `Avatar`, `Bio`, `FollowButton` instead of one giant file.

---

# 3. Reusable Components and Presentational vs Container Pattern

A `reusable component` takes props and renders the same shape for different data. The `presentational vs container` pattern splits look (how it renders) from logic (how it fetches or stores data).

```jsx
function UserBadge({ name }) {
  return <p>Member: {name}</p>;
}

function Team() {
  return (
    <div>
      <UserBadge name="Kakade" />
      <UserBadge name="Asha" />
    </div>
  );
}
```

Output:

```text
Member: Kakade
Member: Asha
```

Presentational (dumb look) + container (smart data):

```jsx
function UserListView({ users }) {
  return users.map((u) => <p key={u}>{u}</p>);
}

function UserList() {
  const users = ["Kakade", "Rohan"];
  return <UserListView users={users} />;
}
```

Output:

```text
Kakade
Rohan
```

Interview one-liner: presentational is props-in-UI-out; containers own state and effects.

---

# 4. Component File Structure and Default Exports vs Named Exports

A clean `file structure` keeps one component per file with styles and tests nearby. `Default exports` allow any import name but only one per file; `named exports` allow multiple, strictly-named imports.

```jsx
// Avatar.jsx
export default function Avatar() {
  return <p>Kakade</p>;
}
```

Output:

```text
Avatar renders Kakade wherever imported.
```

Compare:

```js
// Default: any name works
import Avatar from "./Avatar.jsx";

// Named: exact names in braces
import { Avatar, Bio } from "./people.jsx";
```

Think:

```text
Avatar.jsx → default Avatar
people.jsx → named { Avatar, Bio }
App.jsx → imports and composes
```

Tip: colocate — `Avatar.jsx`, `Avatar.css`, `Avatar.test.jsx` in one folder. Interview trap: mixing default and named imports (`import { Avatar }` when it was `export default`) gives `undefined`.

---

# 5. Pure Components Concept

A `pure component` always returns the same UI for the same props and causes no side effects during render. Predictable rendering is what lets React skip work safely with memoization.

```jsx
function Double({ n }) {
  return <p>{n * 2}</p>;
}
```

Output:

```text
When n is 4: 8
```

Bad (impure — mutates outside value during render):

```jsx
let total = 0;

function BadCounter({ n }) {
  total += n;
  return <p>{total}</p>;
}
```

Output:

```text
Unpredictable — total keeps growing across renders.
```

Good (pure — computes from props only):

```jsx
function GoodCounter({ n }) {
  return <p>{n + 1}</p>;
}
```

Output:

```text
When n is 4: 5, every time.
```

Rule: render should only compute JSX — no fetching, timers, or DOM writes. Push side effects into event handlers or `useEffect`.

---

# Real-World Uses

A blog page composes `Header`, `PostCard`, and `Footer`. Each card is reusable, presentational `PostCardView` gets data from container `Blog`, and files use named exports for the shared UI kit.

```jsx
function PostCard({ title }) {
  return <h3>{title} by Kakade</h3>;
}
```

Output:

```text
Post title by Kakade, repeated per post.
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **Functional components** | Function returning JSX, modern default |
| **Class components overview** | `extends Component` + `render()`, legacy only |
| **Component composition** | Nest small pieces into big UI |
| **Reusable components** | Same shape, different props |
| **Parent and child components** | Parent renders child, passes props down |
| **Component file structure** | One component per file, colocate CSS/tests |
| **Default exports vs named exports** | Default: one, any name; named: many, exact names |
| **Pure components concept** | Same props → same UI, no render side effects |
| **Presentational vs container pattern** | Look vs data-fetching split |

## The Mental Model

```text
App (parent/container)
  │
  ├── Header (reusable, presentational)
  ├── Profile
  │     └── Avatar (child)
  └── Footer
        │
        ↓
  pure functions of props
        │
        ↓
  composed tree = full page
```

### The 3 things to remember

**1. Capitalized function:**

```jsx
function Greet() {
  return <h1>Hello Kakade</h1>;
}
```

**2. Compose parents and children:**

```jsx
function Profile() {
  return <Avatar />;
}
```

**3. Stay pure in render:**

```jsx
function Double({ n }) {
  return <p>{n * 2}</p>;
}
```

> **Build small pure functions of UI, then compose them parent-inside-child into a full app.**
