# Accessibility in React

Accessibility clicks as: `semantic first` → `label + keyboard + focus` → `test with tools`. Most a11y wins are plain HTML done right — React just must not break them.

The core idea:

```text
<button>/<label>/<h1> semantics
    ↓
aria only for custom widgets
    ↓
keyboard + focus + contrast → audit
```

If Kakade can use it with keyboard + screen reader, you are most of the way there.

---

# 1. Semantic HTML, Labels and Inputs, and Alt Text

`Semantic HTML` (`header/main/nav/button`) gives free roles and keyboard behavior. `Labels and inputs` (`<label htmlFor>` or wrapping) name every field. `Alt text` describes images; empty `alt=""` marks decoration.

```jsx
function Signup() {
  return (
    <main>
      <h1>Join Kakade</h1>
      <label htmlFor="email">Email</label>
      <input id="email" type="email" />
      <button>Sign up</button>
    </main>
  );
}
```

Output:

```text
Screen reader: heading Join Kakade → labeled email field → sign-up button.
```

Image:

```jsx
<img src="/kakade.png" alt="Kakade smiling at meetup" />
```

Output:

```text
Meaningful alt; use alt="" for dividers/icons.
```

Never `<div onClick>` for buttons — you lose focus, Enter/Space, and roles in one line.

---

# 2. aria-* Attributes, Keyboard Navigation, and Focus Management

`aria-* attributes` (`aria-label`, `aria-expanded`, `aria-live`) fill gaps for custom UI. `Keyboard navigation` means all actions work via Tab/Enter/Escape/arrows. `Focus management` moves focus into dialogs and back on close, with visible focus rings.

```jsx
import { useState } from "react";

function Menu() {
  const [open, setOpen] = useState(false);
  return (
    <div>
      <button aria-expanded={open} onClick={() => setOpen(!open)}>
        Options for Kakade
      </button>
      {open && <p role="status">Menu open</p>}
    </div>
  );
}
```

Output:

```text
Button announces expanded state; status reads politely.
```

Dialog focus sketch:

```js
// open → ref.current.focus(); close → triggerRef.current.focus().
// Escape closes; Tab trapped inside for Kakade.
```

First rule of ARIA: do not use it when native elements already do the job.

---

# 3. Color Contrast Basics, Skip Links, and Testing Accessibility Basics

`Color contrast basics` (4.5:1 text, 3:1 large/UI) keeps Kakade reading in sunlight. `Skip links` (`Skip to content`) let keyboard users bypass nav. `Testing accessibility basics` (`axe-core`, eslint-plugin-jsx-a11y, manual keyboard pass) catches regressions.

```jsx
function Page() {
  return (
    <div>
      <a href="#main">Skip to content for Kakade</a>
      <main id="main" tabIndex={-1}>
        <p>Hi Kakade</p>
      </main>
    </div>
  );
}
```

Output:

```text
Tab → Skip link → jumps to main content.
```

Test sketch:

```js
// import { axe } from "vitest-axe";
// const res = await axe(<Page />); expect(res).toHaveNoViolations();
```

Checklist: keyboard-only run, 200% zoom, forced-colors, and one screen-reader pass before release.

---

# Real-World Uses

A Kakade form pairs every input with labels, announces errors via `aria-live`, traps dialog focus with Escape, and gates PRs on axe + jsx-a11y so regressions fail CI.

```jsx
function Err() {
  return <p role="alert">Email for Kakade is required</p>;
}
```

Output:

```text
Error announced immediately by screen readers.
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **Semantic HTML** | Native roles + keyboard free |
| **Labels and inputs** | `htmlFor` every field |
| **aria-* attributes** | Custom widgets only |
| **Keyboard navigation** | Tab/Enter/Escape/arrows all work |
| **Focus management** | Into dialog, back on close |
| **Alt text** | Meaningful or `""` decorative |
| **Color contrast basics** | 4.5:1 text minimum |
| **Skip links** | Bypass nav to `#main` |
| **Testing accessibility basics** | axe + keyboard pass |

## The Mental Model

```text
Semantic elements + labels + alt
  │
  ├── custom UI? → aria + keyboard + focus trap
  │
  └── polish: contrast + skip link → axe/keyboard test
```

### The 3 things to remember

**1. Label fields:**

```jsx
<label htmlFor="email">Email</label>
<input id="email" type="email" />
```

**2. Announce state:**

```jsx
<button aria-expanded={open}>Options for Kakade</button>
```

**3. Skip + test:**

```jsx
<a href="#main">Skip to content for Kakade</a>
```

> **Use native semantics, label everything, manage focus, and verify with keyboard + axe.**
