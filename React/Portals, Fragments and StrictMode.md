# Portals, Fragments and StrictMode

These utilities click as: `fragments group` → `portals relocate` → `StrictMode exposes bugs`. They solve DOM-structure and dev-safety problems without extra components.

The core idea:

```text
<> groups, no wrapper <div>
    ↓
createPortal renders elsewhere (body)
    ↓
StrictMode double-invokes to surface impurity
```

Use them for clean markup, overlay stacking, and catching side-effect bugs early.

---

# 1. Fragments and Keys in Fragments

`Fragments` (`<>…</>` or `<Fragment>`) group siblings without adding nodes — fixing table/ul markup and extra divs. `Keys in fragments`: only `<Fragment key={…}>` (long form) accepts keys for mapped groups.

```jsx
function Row({ name }) {
  return (
    <>
      <td>{name}</td>
      <td>Friend of Kakade</td>
    </>
  );
}
```

Output:

```text
Two <td>s with no wrapper — valid <tr> children.
```

Keyed groups:

```jsx
import { Fragment } from "react";

function Pairs({ pairs }) {
  return (
    <dl>
      {pairs.map((p) => (
        <Fragment key={p.id}>
          <dt>{p.term}</dt>
          <dd>{p.def} for Kakade</dd>
        </Fragment>
      ))}
    </dl>
  );
}
```

Output:

```text
Each term/definition pair keyed without extra divs.
```

Notice: shorthand `<>` cannot take keys or props — switch to `<Fragment>` when mapping groups.

---

# 2. Portals with createPortal, Modal Pattern, and Event Bubbling in Portals

`Portals with createPortal` render children into another DOM node (usually `document.body`) while keeping React context and state. `Modal pattern` combines portal + overlay + Escape/close for dialogs. `Event bubbling in portals`: events bubble through the React tree, not the DOM location — portal clicks still trigger React parents.

```jsx
import { createPortal } from "react-dom";

function Modal({ onClose }) {
  return createPortal(
    <div className="overlay" onClick={onClose}>
      <div className="box" onClick={(e) => e.stopPropagation()}>
        <p>Hi Kakade</p>
        <button onClick={onClose}>Close</button>
      </div>
    </div>,
    document.body
  );
}
```

Output:

```text
Dialog renders at body level (above app stacking), Close hides it.
```

Think:

```text
React tree: App → Modal (state, context kept)
DOM: body → .overlay (escapes overflow/z-index)
events: portal click → bubbles to App handlers
```

Trap: `stopPropagation` on the box prevents overlay-click close from firing on inner clicks; Escape key needs its own listener.

---

# 3. StrictMode and Why Double-Invoke Effects

`StrictMode` (`<StrictMode>` in dev root) surfaces bugs by double-invoking renders, effects (mount → cleanup → mount), and refs callbacks. `Why double-invoke effects`: missing cleanup or impure setup (double subscribe, double fetch) becomes visible immediately.

```jsx
import { StrictMode } from "react";
import { createRoot } from "react-dom/client";

createRoot(document.getElementById("root")).render(
  <StrictMode>
    <p>App for Kakade</p>
  </StrictMode>
);
```

Output:

```text
Dev only: effects run setup → cleanup → setup (prod unaffected).
```

Effect that survives StrictMode:

```jsx
import { useEffect } from "react";

function Timer() {
  useEffect(() => {
    const id = setInterval(() => {}, 1000);
    return () => clearInterval(id);
  }, []);
  return <p>Tick for Kakade</p>;
}
```

Output:

```text
One interval in dev and prod (cleanup removes the extra).
```

Rule: make effects idempotent with cleanup; never "fix" StrictMode by removing it — fix the impure effect.

---

# Real-World Uses

A Kakade settings dialog uses a keyed fragment list for options, a portal modal to escape a clipped sidebar, and StrictMode in dev caught a missing Escape-listener cleanup before production.

```jsx
function Note() {
  return (
    <>
      <p>Hi Kakade</p>
      <p>No wrapper div</p>
    </>
  );
}
```

Output:

```text
Two paragraphs, zero extra nodes.
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **Fragments** | `<>` groups without nodes |
| **Keys in fragments** | `<Fragment key>` for mapped groups |
| **Portals with createPortal** | Render into `document.body`, keep React tree |
| **Modal pattern** | Portal + overlay + close/Escape |
| **Event bubbling in portals** | Bubbles via React tree, not DOM spot |
| **StrictMode** | Dev-only bug spotlight, no prod effect |
| **Why double-invoke effects** | Exposes missing cleanup / impurity |

## The Mental Model

```text
JSX structure
  │
  ├── <> / <Fragment key> → group, no div
  │
  ├── createPortal(child, document.body) → overlay elsewhere
  │       └── events bubble to React parents
  │
  └── <StrictMode> → double setup/cleanup → pure effects only
```

### The 3 things to remember

**1. Group without nodes:**

```jsx
<>
  <td>Kakade</td>
  <td>Pro</td>
</>
```

**2. Portal modal:**

```jsx
createPortal(<div>Hi Kakade</div>, document.body);
```

**3. StrictMode-safe effect:**

```jsx
useEffect(() => {
  const id = setInterval(() => {}, 1000);
  return () => clearInterval(id);
}, []);
```

> **Group with fragments, relocate overlays with portals, and let StrictMode prove your effects clean up.**
