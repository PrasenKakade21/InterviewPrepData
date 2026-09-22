# Context API

Context clicks as: `create once` → `provide at top` → `consume anywhere`. It broadcasts shared values (theme, user) without threading props through every level.

The core idea:

```text
createContext()
    ↓
<Provider value={…}> wraps tree
    ↓
useContext() reads it deep down
    ↓
no prop drilling
```

Context is dependency injection for React trees — best for slow-changing, widely needed data.

---

# 1. Prop Drilling Problem, createContext, Provider, and useContext

The `prop drilling problem` is passing `user` or `theme` through five middlemen that never use it. `createContext` makes a channel, `Provider` supplies the value, and `useContext` reads it wherever needed.

```jsx
import { createContext, useContext } from "react";

const UserCtx = createContext("Guest");

function DeepChild() {
  const user = useContext(UserCtx);
  return <p>Hi {user}</p>;
}

function App() {
  return (
    <UserCtx.Provider value="Kakade">
      <DeepChild />
    </UserCtx.Provider>
  );
}
```

Output:

```text
Hi Kakade (no props passed through middlemen).
```

Think:

```text
Provider value="Kakade"
    ↓
…any depth, no forwarding…
    ↓
useContext(UserCtx) → "Kakade"
```

Notice: default (`"Guest"`) only applies when no Provider wraps the reader — useful for tests. `Context vs props`: props for direct parent-child data, context for global-ish data many levels need.

---

# 2. Multiple Contexts, Theme Context Example, and Auth Context Example

`Multiple contexts` separate independent concerns — theme, auth, locale — so changing one does not re-render readers of another. `Theme context` toggles styling; `auth context` shares user + login/logout.

```jsx
import { createContext, useContext, useState } from "react";

const ThemeCtx = createContext("light");

function Card() {
  const theme = useContext(ThemeCtx);
  return <div className={theme}>Card for Kakade</div>;
}

function App() {
  const [theme, setTheme] = useState("light");
  return (
    <ThemeCtx.Provider value={theme}>
      <button onClick={() => setTheme(theme === "light" ? "dark" : "light")}>
        Toggle
      </button>
      <Card />
    </ThemeCtx.Provider>
  );
}
```

Output:

```text
Button toggles Card between light and dark classes.
```

Auth example:

```jsx
import { createContext, useContext } from "react";

const AuthCtx = createContext({ user: null });

function Who() {
  const { user } = useContext(AuthCtx);
  return <p>{user ? `Hi ${user}` : "Guest"}</p>;
}

function App2() {
  return (
    <AuthCtx.Provider value={{ user: "Kakade" }}>
      <Who />
    </AuthCtx.Provider>
  );
}
```

Output:

```text
Hi Kakade (auth shared without props).
```

Keep contexts focused and small — one giant object forces all readers to re-render on any field change.

---

# 3. Context Re-render Gotchas and When Not to Use Context

`Context re-render gotchas`: every Provider value change re-renders all readers, and inline `{...}` objects change identity each render causing extra renders. `When not to use context`: fast-changing values (keystrokes), page-only data, or performance-critical lists — use props, local state, or a store instead.

```jsx
function StaticApp({ children }) {
  return <p>Shell for Kakade: {children}</p>;
}
```

Output:

```text
Shell for Kakade: nested content (props fine here, no context needed).
```

Bad (new object every render):

```jsx
// Bad: value={{ user }} recreates → all readers re-render each time
// <AuthCtx.Provider value={{ user }}>
```

Good (memoized value):

```jsx
import { useMemo } from "react";

function FixedProvider({ user, children }) {
  const value = useMemo(() => ({ user }), [user]);
  return <AuthCtx.Provider value={value}>{children}</AuthCtx.Provider>;
}
```

Output:

```text
Readers re-render only when user actually changes.
```

| Use context | Avoid context |
| ----------- | ------------- |
| **Theme, locale, auth user** | **Form keystrokes, scroll position** |
| **Rarely changing, widely read** | **Rapidly changing or narrowly used** |

Interview trap: context is not state management — it has no selectors or batching. Pair with `useReducer` or Zustand for complex global state.

---

# Real-World Uses

An app shell provides `ThemeCtx` + `AuthCtx` at the root: header reads auth for the avatar, cards read theme for classes, and only those readers re-render when their context changes.

```jsx
import { useContext } from "react";

function Header() {
  const { user } = useContext(AuthCtx);
  return <p>Logged in: {user || "Guest Kakade"}</p>;
}
```

Output:

```text
Logged in: Kakade
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **Prop drilling problem** | Middlemen forward unused props |
| **createContext** | `createContext(default)` makes channel |
| **Provider** | `<Ctx.Provider value={v}>` supplies |
| **useContext** | `useContext(Ctx)` reads anywhere below |
| **Context vs props** | Props: direct; context: broadcast |
| **Multiple contexts** | Split theme / auth / locale |
| **Theme context example** | String toggle consumed by styling |
| **Auth context example** | `{ user, login, logout }` object |
| **Context re-render gotchas** | New object → all readers re-render; memoize |
| **When not to use context** | Fast-changing or narrow data |

## The Mental Model

```text
Root providers (theme, auth)
  │
  ├── value memoized → stable identity
  │       │
  │       └── deep readers (useContext) → re-render on change
  │
  └── props → local, fast, narrow data (no context)
```

### The 3 things to remember

**1. Create + provide:**

```jsx
const UserCtx = createContext("Guest");
<UserCtx.Provider value="Kakade">…</UserCtx.Provider>
```

**2. Consume deep:**

```jsx
const user = useContext(UserCtx);
```

**3. Memoize object values:**

```jsx
const value = useMemo(() => ({ user }), [user]);
```

> **Provide slow-changing shared values once, consume them anywhere, and keep fast data in props or state.**
