# Forms and Controlled Components

Forms click as: `state holds values` → `inputs show state` → `onChange updates state`. React becomes the single source of truth instead of the DOM.

The core idea:

```text
useState holds value
    ↓
input value={state}
    ↓
onChange → setState
    ↓
submit reads state
```

Most interview forms are controlled — predictable, validatable, and resettable from code.

---

# 1. Controlled Inputs, Uncontrolled Inputs, and useState with Forms

`Controlled inputs` bind `value` + `onChange` to state. `Uncontrolled inputs` let the DOM hold the value and read it via `ref` on submit. `useState with forms` means every keystroke flows through React state.

```jsx
import { useState } from "react";

function NameInput() {
  const [name, setName] = useState("Kakade");
  return <input value={name} onChange={(e) => setName(e.target.value)} />;
}
```

Output:

```text
Input shows Kakade; typing updates state instantly.
```

Uncontrolled contrast:

```jsx
import { useRef } from "react";

function QuickInput() {
  const ref = useRef(null);
  function show() {
    alert(ref.current.value);
  }
  return (
    <>
      <input ref={ref} defaultValue="Kakade" />
      <button onClick={show}>Show</button>
    </>
  );
}
```

Output:

```text
Click Show → alerts whatever is typed.
```

| Type | Remember |
| ---- | -------- |
| **Controlled** | `value` + `onChange`, React owns data |
| **Uncontrolled** | `defaultValue` + `ref`, DOM owns data |

Prefer controlled for validation and live feedback; uncontrolled only for simple one-shot reads.

---

# 2. Handling Multiple Fields and Form Submit Handling

`Handling multiple fields` uses one state object keyed by input `name`. `Form submit handling` reads that state in `onSubmit` after `preventDefault()`.

```jsx
import { useState } from "react";

function Signup() {
  const [form, setForm] = useState({ name: "Kakade", city: "Pune" });
  function handleChange(e) {
    const { name, value } = e.target;
    setForm({ ...form, [name]: value });
  }
  function handleSubmit(e) {
    e.preventDefault();
    console.log(`Save ${form.name} from ${form.city}`);
  }
  return (
    <form onSubmit={handleSubmit}>
      <input name="name" value={form.name} onChange={handleChange} />
      <input name="city" value={form.city} onChange={handleChange} />
      <button type="submit">Save</button>
    </form>
  );
}
```

Output:

```text
Save Kakade from Pune (with edited values after typing).
```

Think:

```text
input name="city" → [name]: value → { ...form, city: new }
```

One generic `handleChange` scales to ten fields without ten setters.

---

# 3. Checkboxes, Radio Buttons, and Select Dropdowns

`Checkboxes` use `checked` + `e.target.checked`. `Radio buttons` share a `name` and compare `value`. `Select dropdowns` bind `value` on the `<select>` itself.

```jsx
import { useState } from "react";

function Prefs() {
  const [member, setMember] = useState(true);
  return (
    <label>
      <input checked={member} onChange={(e) => setMember(e.target.checked)} />
      Member Kakade
    </label>
  );
}
```

Output:

```text
Checkbox toggles true/false with clicks.
```

Radio + select:

```jsx
import { useState } from "react";

function Survey() {
  const [plan, setPlan] = useState("pro");
  const [city, setCity] = useState("Pune");
  return (
    <>
      <label>
        <input value="pro" checked={plan === "pro"} name="plan" type="radio"
          onChange={(e) => setPlan(e.target.value)} /> Pro
      </label>
      <select value={city} onChange={(e) => setCity(e.target.value)}>
        <option value="Pune">Pune</option>
        <option value="Mumbai">Mumbai</option>
      </select>
    </>
  );
}
```

Output:

```text
Radio picks pro/free; select picks Pune/Mumbai for Kakade.
```

Notice: text inputs use `value`; checkboxes use `checked`. Mixing them is the most common form bug.

---

# 4. Form Validation Basics and Resetting Forms

`Form validation basics`: check state on submit (and optionally on change), store an `errors` object, and block submit when invalid. `Resetting forms` restores the initial object.

```jsx
import { useState } from "react";

function JoinForm() {
  const [name, setName] = useState("");
  const [error, setError] = useState("");
  function handleSubmit(e) {
    e.preventDefault();
    if (name.trim().length < 3) {
      setError("Name needs 3+ letters");
      return;
    }
    setError("");
    console.log(`Welcome ${name}`);
  }
  return (
    <form onSubmit={handleSubmit}>
      <input value={name} onChange={(e) => setName(e.target.value)} />
      {error && <p>{error}</p>}
      <button type="submit">Join Kakade.dev</button>
    </form>
  );
}
```

Output:

```text
Short name → Name needs 3+ letters. Valid → Welcome <name>.
```

Reset:

```jsx
import { useState } from "react";

const empty = { name: "" };

function ResetDemo() {
  const [form, setForm] = useState({ name: "Kakade" });
  return <button onClick={() => setForm(empty)}>Reset</button>;
}
```

Output:

```text
Click Reset → name clears to empty string.
```

Keep validation derived where possible: `const invalid = name.length < 3` beats a second state that drifts.

---

# Real-World Uses

A checkout form combines text, select, checkbox, validation, and reset: one state object, one change handler, errors shown inline, and submit blocked until valid.

```jsx
import { useState } from "react";

function Checkout() {
  const [city, setCity] = useState("Pune");
  return <p>Ship to {city} for Kakade</p>;
}
```

Output:

```text
Ship to Pune for Kakade (updates with select).
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **Controlled inputs** | `value` + `onChange`, React owns value |
| **Uncontrolled inputs** | `defaultValue` + `ref`, DOM owns value |
| **useState with forms** | Keystroke → setter → re-render |
| **Handling multiple fields** | One object + `[name]: value` |
| **Checkboxes** | `checked` + `e.target.checked` |
| **Radio buttons** | Shared `name`, `checked={val === opt}` |
| **Select dropdowns** | `value` on `<select>` |
| **Form submit handling** | `onSubmit` + `preventDefault`, read state |
| **Form validation basics** | Validate state, show `errors`, block submit |
| **Resetting forms** | `setForm(initialObject)` |

## The Mental Model

```text
State object (values + errors)
  │
  ├── value/checked → inputs show it
  ├── onChange → setState copy
  │       │
  │       ↓
  └── onSubmit → validate → save / show errors → reset
```

### The 3 things to remember

**1. Controlled text:**

```jsx
<input value={name} onChange={(e) => setName(e.target.value)} />
```

**2. One handler, many fields:**

```jsx
setForm({ ...form, [e.target.name]: e.target.value });
```

**3. Checkbox binds checked:**

```jsx
<input checked={member} onChange={(e) => setMember(e.target.checked)} />
```

> **Let state own every field, validate that state, and submit or reset from code.**
