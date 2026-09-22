# Error Handling

Error handling clicks as: `boundary catches render crashes` → `fallback shows` → `reset recovers`. Boundaries are class-only safety nets; async and event errors need manual handling.

The core idea:

```text
Child throws during render
    ↓
nearest boundary catches
    ↓
fallback UI + log → reset
```

Wrap routes and widgets — never rely on `try/catch` around JSX.

---

# 1. Error Boundaries, componentDidCatch, and getDerivedStateFromError

`Error boundaries` are class components catching descendant render/lifecycle errors. `getDerivedStateFromError` swaps to fallback UI; `componentDidCatch` logs details. Only class components can be boundaries (no hook equivalent).

```jsx
import { Component } from "react";

class Boundary extends Component {
  state = { bad: false };
  static getDerivedStateFromError() {
    return { bad: true };
  }
  componentDidCatch(err, info) {
    console.error(err, info);
  }
  render() {
    if (this.state.bad) {
      return <p>Something broke for Kakade</p>;
    }
    return this.props.children;
  }
}
```

Output:

```text
Crash inside → Something broke for Kakade instead of blank page.
```

Think:

```text
render throws
    ├── getDerivedStateFromError → bad: true → fallback
    └── componentDidCatch → log stack
```

Boundaries catch render, lifecycle, and constructor errors — not events, async code, or the boundary's own errors.

---

# 2. react-error-boundary Library, Error Fallback UI, and Resetting Errors

`react-error-boundary library` (`<ErrorBoundary FallbackComponent onReset>`) removes class boilerplate. `Error fallback UI` explains + offers retry. `Resetting errors` clears state via `resetKeys` or `resetErrorBoundary()`.

```jsx
import { ErrorBoundary } from "react-error-boundary";

function Fallback({ error, resetErrorBoundary }) {
  return (
    <div>
      <p>Failed for Kakade: {error.message}</p>
      <button onClick={resetErrorBoundary}>Retry</button>
    </div>
  );
}

function App({ children }) {
  return <ErrorBoundary FallbackComponent={Fallback}>{children}</ErrorBoundary>;
}
```

Output:

```text
Crash → Failed for Kakade: … + Retry button.
```

Reset on route change:

```jsx
// <ErrorBoundary FallbackComponent={Fallback} resetKeys={[location.pathname]}>
```

Think: `resetKeys` change → boundary auto-resets; manual `resetErrorBoundary()` for button retries. Always reset related query caches too.

---

# 3. Async Error Limits, 404 vs Crash Handling, and Logging Errors

`Async error limits`: boundaries do not catch promises, timers, or event handlers — use local `error` state there. `404 vs crash handling`: missing resources render a 404 route; broken code triggers a boundary. `Logging errors` sends boundary + async errors to Sentry/Datadog with route context.

```jsx
import { useState } from "react";

function Saver() {
  const [error, setError] = useState(null);
  async function save() {
    try {
      await fetch("/api/kakade");
    } catch (e) {
      setError(e.message);
    }
  }
  return <button onClick={save}>{error ? `Retry (${error})` : "Save"}</button>;
}
```

Output:

```text
Failed save shows Retry instead of crashing.
```

| Case | Tool |
| ---- | ---- |
| **Render crash** | **Error boundary + fallback** |
| **Missing page/data** | **404 route / empty state** |
| **Event / async fail** | **Local error state + retry** |

Log with context (`route`, `user: "Kakade"`, `action`) — raw stacks alone are undebuggable in production.

---

# Real-World Uses

An app wraps each route in a boundary with Sentry logging, shows per-widget fallbacks so one chart cannot kill Kakade's dashboard, and resets on navigation while async forms use local retry states.

```jsx
function Widget() {
  return <p>Chart for Kakade</p>;
}
```

Output:

```text
Chart for Kakade — or isolated fallback if it throws.
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **Error boundaries** | Class-only render crash nets |
| **componentDidCatch** | Log error + info side effect |
| **getDerivedStateFromError** | Return fallback state |
| **react-error-boundary library** | Function-friendly wrapper |
| **Error fallback UI** | Explain + retry action |
| **Resetting errors** | `resetKeys` / `resetErrorBoundary` |
| **Async error limits** | Events/async need manual state |
| **404 vs crash handling** | Missing → 404; broken → boundary |
| **Logging errors** | Sentry with route + user context |

## The Mental Model

```text
Render tree
  │
  ├── boundary per route/widget
  │     ├── throws? → fallback + log → reset
  │     └── 404? → Not Found route (not a crash)
  │
  └── events / async → try/catch → local error state
```

### The 3 things to remember

**1. Class boundary pair:**

```jsx
static getDerivedStateFromError() {
  return { bad: true };
}
```

**2. Library fallback + reset:**

```jsx
<ErrorBoundary FallbackComponent={Fallback} resetKeys={[path]}>
```

**3. Async needs manual catch:**

```jsx
try {
  await fetch("/api/kakade");
} catch (e) {
  setError(e.message);
}
```

> **Boundaries catch render crashes with resettable fallbacks; handle async, events, and 404s separately.**
