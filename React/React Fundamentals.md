# React Fundamentals

React makes sense once you see the chain: `UI as components` → `Virtual DOM diffing` → `fast, predictable updates`. Instead of manually touching the DOM, you describe what the UI should look like and React figures out how to update it efficiently.

The core idea:

```text
Describe UI
    ↓
React compares Virtual DOM
    ↓
updates only what changed in real DOM
    ↓
fast SPA experience
```

React is a **JavaScript library for building user interfaces**, maintained by Meta. You build encapsulated pieces called components, and React re-renders them when data changes.

---

# 1. What is React

React is a **declarative library** for building UI from small, reusable pieces called `components`. You do not manipulate the DOM directly — you return what the UI should show, and React updates the browser for you.

```jsx
function Greet() {
  return <h1>Hello Kakade</h1>;
}
```

Output:

```text
Hello Kakade
```

Think:

```text
Component function
    ↓
returns UI description
    ↓
React renders to DOM
```

React only handles the view layer. Routing, fetching, and global state come from other tools.

---

# 2. React vs Vanilla JS

In `Vanilla JS` you imperatively select elements and mutate them step by step. In React you declaratively describe the final UI for a given state, and React handles the DOM operations.

```jsx
function Welcome() {
  const name = "Kakade";
  return <p>Hello {name}</p>;
}
```

Output:

```text
Hello Kakade
```

Compare:

```text
Vanilla JS: find element → set text → add listener → update manually
React:      state → describe UI → React updates DOM
```

Notice: in Vanilla JS, UI logic scatters across event handlers. In React, UI is a function of state: `UI = f(state)`. Interview trap: React is a library, not a full framework — it does not ship a router or HTTP client.

---

# 3. Single Page Applications

A `Single Page Application (SPA)` loads one HTML page and updates the view in JavaScript without full page reloads. Clicking a link swaps components instead of asking the server for a new document.

```jsx
function App() {
  const page = "home";
  return page === "home" ? <h1>Home</h1> : <h1>About</h1>;
}
```

Output:

```text
Home
```

Think:

```text
First load → download JS bundle
    ↓
navigation → swap components
    ↓
no full reload → feels instant
```

Tradeoff: SPAs feel fast after first load, but initial bundle is bigger and SEO needs extra care. That is why tools like Next.js exist for server rendering.

---

# 4. Virtual DOM and Reconciliation

The `Virtual DOM` is a lightweight JavaScript copy of the real DOM. When state changes, React builds a new Virtual DOM tree, compares it with the previous one (this comparison is called `reconciliation`), and applies only the differences to the real DOM.

```jsx
function Counter({ count }) {
  return <p>Count: {count}</p>;
}
```

Output:

```text
When count goes 0 → 1, only the text node updates, not the whole page.
```

Think:

```text
state changes
    ↓
new Virtual DOM tree
    ↓
diff vs old tree (reconciliation)
    ↓
patch only changed real DOM nodes
```

Interview trap: Virtual DOM is not what makes React fast by itself — skipping unnecessary real DOM work is what makes it fast. `Reconciliation` uses heuristics like comparing element type and `key` props to keep the diff cheap.

---

# 5. React Fiber Overview

`React Fiber` is the reconciliation engine rewritten in React 16. It splits rendering work into small units that can pause, resume, prioritize, and abort. High-priority updates like typing stay responsive while low-priority work waits.

```jsx
function SearchBox() {
  return <input placeholder="Type, Kakade" />;
}
```

Output:

```text
Typing stays smooth even while a big list renders in background.
```

Think:

```text
Big render
    ↓
Fiber splits into units
    ↓
urgent update interrupts → renders first
    ↓
background work resumes
```

You rarely use Fiber directly, but it powers `useTransition`, `useDeferredValue`, and `Suspense`. Interview one-liner: Fiber enables concurrent rendering.

---

# 6. Declarative vs Imperative UI

`Imperative` UI tells the browser how to update step by step. `Declarative` UI tells React what to show for the current state. React is declarative — you describe the end result.

```jsx
function Status({ isOnline }) {
  return <p>{isOnline ? "Kakade is online" : "Kakade is offline"}</p>;
}
```

Output:

```text
When isOnline is true: Kakade is online
When isOnline is false: Kakade is offline
```

Bad (imperative Vanilla JS mindset):

```js
// Bad: manual DOM steps
// document.querySelector("p").textContent = "Kakade is online";
```

Good (declarative React):

```jsx
function Status({ isOnline }) {
  return <p>{isOnline ? "Kakade is online" : "Kakade is offline"}</p>;
}
```

Output:

```text
Kakade is online
```

Less branching in your head, fewer DOM bugs.

---

# 7. Component-Based Architecture and One-Way Data Flow

React apps are trees of `components` — small functions that return UI. Data flows one way: from parent down to child via `props`. A child never directly mutates its parent's data; it asks the parent to update via callbacks.

```jsx
function Child({ name }) {
  return <p>Hello {name}</p>;
}

function Parent() {
  return <Child name="Kakade" />;
}
```

Output:

```text
Hello Kakade
```

Think:

```text
Parent state
    ↓ props down
Child renders
    ↓ events up (callback)
Parent updates
```

Notice: `one-way data flow` makes debugging easy — you always know where data came from. If a child needs to change something, lift the state to the parent and pass a setter down.

---

# 8. React 18 vs React 19 Basics

`React 18` added concurrent features like automatic batching, `Suspense` for data, and `useTransition`. `React 19` keeps that model and adds Actions, `useActionState`, `useOptimistic`, the `use` API, and ref-as-prop for function components.

```jsx
function Hello() {
  return <h1>Hello Kakade on React 19</h1>;
}
```

Output:

```text
Hello Kakade on React 19
```

| Version | Remember |
| ------- | -------- |
| **React 18** | Automatic batching, `createRoot`, transitions, Suspense improvements |
| **React 19** | Actions + `useActionState`, `useOptimistic`, `use` API, no need for `forwardRef` |

Interview trap: `ReactDOM.render` is React 17 style. React 18+ uses `createRoot`. In React 19, function components receive `ref` as a normal prop.

---

# Real-World Uses

A profile page shows why fundamentals matter: component tree for layout, SPA navigation between tabs, Virtual DOM for fast likes/comments updates, and one-way data flow so the parent owns the user object.

```jsx
function Profile({ user }) {
  return <h2>{user.name} from Kakade.dev</h2>;
}
```

Output:

```text
Kakade-style name rendered without full page reload.
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **What is React** | Library for component-based UI, you describe what to show |
| **React vs Vanilla JS** | `UI = f(state)` vs manual DOM mutation |
| **Single Page Applications** | One HTML file, JS swaps views, no full reloads |
| **Virtual DOM** | JS copy of DOM used to compute minimal updates |
| **Reconciliation basics** | Diff new vs old tree, patch only changes |
| **React Fiber overview** | Splits work into pausable units, enables concurrency |
| **Declarative vs imperative UI** | Describe result vs list steps |
| **Component-based architecture** | App is a tree of reusable functions |
| **One-way data flow** | Props down, events up |
| **React 18 vs React 19 basics** | 18: batching + root API; 19: Actions, `use`, ref-as-prop |

## The Mental Model

```text
State
  │
  ↓
Describe UI (declarative components)
  │
  ↓
Virtual DOM diff (reconciliation + Fiber)
  │
  ↓
Minimal real DOM patch
  │
  ↓
SPA with one-way data flow
```

### The 3 things to remember

**1. UI is a function of state:**

```jsx
function Greet() {
  return <h1>Hello Kakade</h1>;
}
```

**2. Data flows down:**

```jsx
function Parent() {
  return <Child name="Kakade" />;
}
```

**3. React 18 root API:**

```js
createRoot(document.getElementById("root")).render(<App />);
```

> **Describe the UI for each state, and let React diff and patch the DOM for you.**
