# Props

Props click when you see: `parent data` → `passed as attributes` → `child renders`. They are how components talk to each other — always flowing down from parent to child.

The core idea:

```text
Parent owns data
    ↓
passes props like attributes
    ↓
child reads props
    ↓
UI renders
```

`Props` (properties) are **read-only inputs** to a component. Think of them like function arguments: the caller provides values, the component uses them but never mutates them.

---

# 1. What are Props and Passing Props

`What are props`: objects containing the values a parent gives a child. `Passing props` looks like HTML attributes — strings in quotes, dynamic values in braces.

```jsx
function Welcome({ name }) {
  return <h1>Hello {name}</h1>;
}

function App() {
  return <Welcome name="Kakade" />;
}
```

Output:

```text
Hello Kakade
```

Think:

```text
<App /> → name="Kakade"
    ↓
Welcome({ name }) → reads it
    ↓
renders Hello Kakade
```

Anything can be a prop: strings, numbers, arrays, objects, even functions. Dynamic values need braces: `<Welcome name={user.name} />`.

---

# 2. Children Prop and Prop Destructuring

The special `children prop` is whatever you nest between a component's tags — used for layout wrappers like cards and modals. `Prop destructuring` unpacks props directly in the parameter list for cleaner code.

```jsx
function Card({ children }) {
  return <div className="card">{children}</div>;
}

function App() {
  return (
    <Card>
      <h2>Hello Kakade</h2>
    </Card>
  );
}
```

Output:

```text
Hello Kakade (wrapped in .card div)
```

Destructuring vs long form:

```jsx
// Without destructuring
function Hi(props) {
  return <p>Hi {props.name}</p>;
}

// With destructuring
function Hello({ name }) {
  return <p>Hi {name}</p>;
}
```

Output:

```text
Hi Kakade (both versions identical)
```

Notice: `children` can be text, elements, or even a function. Destructure with defaults: `function Hi({ name = "Kakade" })`.

---

# 3. Default Props, Spread Props, and Conditional Props

`Default props` supply fallbacks when a parent omits a value. `Spread props` (`{...obj}`) pass a whole object at once. `Conditional props` pass values only when a condition holds.

```jsx
function Badge({ name = "Kakade", role = "Member" }) {
  return <p>{name} — {role}</p>;
}

function App() {
  return <Badge />;
}
```

Output:

```text
Kakade — Member
```

Spread and conditional:

```jsx
function ProfileCard(props) {
  return <p>{props.name} is {props.role}</p>;
}

function App() {
  const user = { name: "Kakade", role: "Admin" };
  const showRole = true;
  return <ProfileCard {...user} title={showRole ? "Lead" : "Guest"} />;
}
```

Output:

```text
Kakade is Admin
```

Notice: explicit props after a spread win: `<Comp {...user} name="Asha" />` uses `Asha`. Interview trap: spreading everything (`{...props}`) through many layers hides which props a component actually needs.

---

# 4. Props are Read-Only

`Props are read-only` — a child must never assign to them. If a child needs to change something, it calls a callback prop and the parent (who owns the data) updates.

```jsx
function Name({ value }) {
  return <p>Name: {value}</p>;
}
```

Output:

```text
Name: Kakade
```

Bad (mutating props):

```jsx
// Bad: never do this
// function Name({ value }) {
//   value = "Asha";
//   return <p>{value}</p>;
// }
```

Good (parent owns update via callback):

```jsx
function NameInput({ value, onChange }) {
  return <input value={value} onChange={onChange} />;
}
```

Output:

```text
Input shows parent value; typing calls parent onChange.
```

Think: props flow down like cold water in pipes — children cannot push water back up, they ring the doorbell (callback) instead.

---

# 5. Prop Drilling and Lifting State Up

`Prop drilling` is passing props through intermediate components that do not use them, just to reach a deep child. `Lifting state up` moves shared state to the nearest common parent so siblings stay in sync.

```jsx
function GrandChild({ name }) {
  return <p>Hi {name}</p>;
}

function Child({ name }) {
  return <GrandChild name={name} />;
}

function App() {
  return <Child name="Kakade" />;
}
```

Output:

```text
Hi Kakade (drilled App → Child → GrandChild)
```

Lifting state up:

```jsx
import { useState } from "react";

function NameInput({ value, onName }) {
  return <input value={value} onChange={(e) => onName(e.target.value)} />;
}

function Preview({ value }) {
  return <p>Preview: {value}</p>;
}

function Form() {
  const [name, setName] = useState("Kakade");
  return (
    <>
      <NameInput value={name} onName={setName} />
      <Preview value={name} />
    </>
  );
}
```

Output:

```text
Typing in input instantly updates Preview.
```

Think:

```text
Before: state in Child A, Child B out of sync
After:  state in Parent → both children get props → always synced
```

Interview trap: drilling 1–2 levels is fine. Deep drilling (4+ levels) signals you need `Context` or a store.

---

# Real-World Uses

A product list uses every pattern: `ProductCard` takes props, `children` for the description slot, defaults for missing images, spread for product objects, and lifted cart state so cards and header stay synced.

```jsx
function ProductCard({ title = "Course", children }) {
  return (
    <div>
      <h3>{title} for Kakade</h3>
      {children}
    </div>
  );
}
```

Output:

```text
Course for Kakade plus nested description.
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **What are props** | Read-only inputs, like function args |
| **Passing props** | `<Comp name="Kakade" count={3} />` |
| **Children prop** | Nested JSX between tags, for wrappers |
| **Default props** | `({ name = "Kakade" })` fallback |
| **Prop destructuring** | `({ name })` instead of `(props)` |
| **Props are read-only** | Never mutate; use callbacks to update |
| **Prop drilling** | Passing through unused middlemen |
| **Lifting state up** | Shared state lives in common parent |
| **Spread props** | `<Comp {...user} />` passes whole object |
| **Conditional props** | `title={isLead ? "Lead" : "Guest"}` |

## The Mental Model

```text
Parent (owns data)
  │
  ├── props → Child A (renders)
  │       │
  │       └── callback ↑ (asks to update)
  │
  └── props → Child B (renders)
        │
        └── children → nested UI slot
```

### The 3 things to remember

**1. Pass like attributes:**

```jsx
<Welcome name="Kakade" />
```

**2. Read + default:**

```jsx
function Welcome({ name = "Kakade" }) {
  return <h1>Hello {name}</h1>;
}
```

**3. Lift shared state:**

```jsx
function Form() {
  const [name, setName] = useState("Kakade");
  return <Preview value={name} />;
}
```

> **Props flow down read-only — lift shared data up and let callbacks flow changes back.**
