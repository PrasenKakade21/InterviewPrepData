# JSX

JSX clicks once you see the chain: `HTML-like syntax` → `Babel compiles to calls` → `React renders elements`. It looks like HTML in JavaScript, but it is really a concise way to describe UI trees.

The core idea:

```text
JSX
    ↓
compiles to React.createElement()
    ↓
creates element objects
    ↓
React renders to DOM
```

JSX is **not required** to use React, but almost everyone uses it because nested `createElement` calls are painful to read.

---

# 1. What is JSX and JSX vs JS

`JSX` (JavaScript XML) is syntax extension that lets you write markup inside JavaScript. `JSX vs JS` is simple: JSX describes what the UI looks like; plain JS is the compiled output that creates element objects.

```jsx
function Greet() {
  return <h1>Hello Kakade</h1>;
}
```

Output:

```text
Hello Kakade
```

Compare — this JSX:

```jsx
<h1>Hello Kakade</h1>
```

compiles roughly to:

```js
React.createElement("h1", null, "Hello Kakade");
```

Think:

```text
You write <h1>Hello Kakade</h1>
    ↓
Babel transforms it
    ↓
JS object { type: "h1", props: {...} }
```

Notice: JSX is neither a string nor HTML — it becomes a plain object React understands.

---

# 2. Embedding Expressions and Conditional JSX

Curly braces `{}` let you embed any **expression** — variables, function calls, ternaries, `map()` — inside markup. `Conditional JSX` just uses normal JS conditionals: ternary for either/or, `&&` for show-or-hide.

```jsx
function Welcome() {
  const name = "Kakade";
  const isMember = true;
  return <h1>{isMember ? `Welcome ${name}` : "Please join"}</h1>;
}
```

Output:

```text
Welcome Kakade
```

Statements like `if` or `for` cannot go directly inside `{}` — only expressions. So compute first, then embed:

```jsx
function Badge({ count }) {
  return <p>{count > 5 ? "Pro Kakade" : "New Kakade"}</p>;
}
```

Output:

```text
When count is 8: Pro Kakade
When count is 2: New Kakade
```

Interview trap: `{}` takes expressions (`a + b`, `user.name`, `cond ? x : y`), not statements (`if`, `for`, `return`).

---

# 3. className vs class and Self-Closing Tags

Because `class` is a reserved word in JS, JSX uses `className` for CSS classes. `Self-closing tags` must end with `/>` when an element has no children — `<img>`, `<input>`, `<br>`, and your own components.

```jsx
function Avatar() {
  return <img className="avatar" src="/kakade.png" alt="Kakade" />;
}
```

Output:

```text
Image with CSS class avatar rendered.
```

Bad:

```jsx
// Bad: class is ignored / warns in JSX
// <h1 class="title">Hello Kakade</h1>
// Bad: <img> without closing slash warns
```

Good:

```jsx
function Title() {
  return <h1 className="title">Hello Kakade</h1>;
}
```

Output:

```text
Hello Kakade (with .title styles applied)
```

Other renames to remember: `for` becomes `htmlFor`, `tabindex` becomes `tabIndex`. JSX attributes are camelCase.

---

# 4. Fragments, Keys in Fragments, and JSX Comments

`Fragments` (`<>...</>`) group multiple elements without adding an extra DOM node. `JSX comments` use `{/* ... */}` because `//` alone is not valid inside markup. If you map fragments, give them explicit `<Fragment key={...}>`.

```jsx
function Row() {
  return (
    <>
      <td>Kakade</td>
      <td>Member</td>
    </>
  );
}
```

Output:

```text
Two table cells with no wrapper div in DOM.
```

Comments and keyed fragments:

```jsx
import { Fragment } from "react";

function List({ users }) {
  return users.map((u) => (
    <Fragment key={u.id}>
      {/* show Kakade first */}
      <p>{u.name}</p>
    </Fragment>
  ));
}
```

Output:

```text
One <p> per user, comment not rendered.
```

Notice: short `<>...</>` cannot take a `key` — switch to `<Fragment key={...}>` when you need one.

---

# 5. JSX Gotchas

Three traps catch beginners: one parent rule, JS keywords, and falsy rendering. JSX must return **one parent element**, `0` renders as text, and `undefined`/`null` render nothing.

```jsx
function Card() {
  return (
    <div>
      <h2>Kakade</h2>
      <p>Member since 2026</p>
    </div>
  );
}
```

Output:

```text
Kakade
Member since 2026
```

Common gotchas table:

| Trap | What happens | Fix |
| ---- | ------------ | --- |
| **Two roots** | Syntax error | Wrap in `div` or `<>` |
| **`0 && <p>`** | Renders `0` | Use `count > 0 && <p>` |
| **`class` / `for`** | Warning, styles break | Use `className` / `htmlFor` |
| **Unclosed tag** | Compile error | Close every tag or `/>` |

```jsx
function Score({ points }) {
  return <p>{points > 0 && `Score: ${points}`}</p>;
}
```

Output:

```text
When points is 0: empty <p>, no stray 0 on screen.
```

---

# 6. Babel Role

`Babel` is the compiler that turns JSX into plain JS browsers and Vite understand. Vite uses esbuild/SWC under the hood with the same idea: transform `<h1>` into `jsx()` / `createElement()` calls before running.

```jsx
function Hi() {
  return <p>Hi Kakade</p>;
}
```

Output:

```text
Hi Kakade (after Babel transforms JSX to JS calls)
```

Think:

```text
Source JSX
    ↓
Babel / SWC transform
    ↓
React.createElement or jsx() calls
    ↓
browser runs plain JS
```

You never call Babel yourself in Vite/CRA — it runs automatically. Interview one-liner: JSX is developer syntax; Babel makes it runnable JS.

---

# Real-World Uses

A user card pulls every idea together: expressions for names, `className` for styling, fragment to avoid wrapper divs, ternary for membership, and comments for teammates.

```jsx
function UserCard({ user }) {
  return (
    <>
      {/* Kakade card */}
      <h2 className="name">{user.name}</h2>
      <p>{user.isPro ? "Pro member" : "Free member"}</p>
    </>
  );
}
```

Output:

```text
Kakade
Pro member
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **What is JSX** | HTML-like syntax for describing UI |
| **JSX vs JS** | JSX compiles to `createElement` objects |
| **Embedding expressions** | `{}` holds any expression, not statements |
| **className vs class** | Use `className`, `htmlFor`, camelCase props |
| **Self-closing tags** | `<img />`, `<input />`, `<MyComp />` need `/>` |
| **Fragments** | `<>...</>` groups without extra DOM node |
| **JSX comments** | `{/* comment */}` |
| **Conditional JSX** | Ternary for either/or, `&&` with boolean guard |
| **JSX gotchas** | One root, `0` renders, close all tags |
| **Babel role** | Compiles JSX to runnable JS |

## The Mental Model

```text
JSX markup
  │
  ├── { expressions } → dynamic values
  ├── className + /> → valid syntax
  ├── <> fragments → clean DOM
  ├── ternary / && → conditional UI
  │       │
  │       ↓
  │   Babel transform
  │       │
  │       ↓
  └── element objects → React renders
```

### The 3 things to remember

**1. Embed with braces:**

```jsx
<h1>Hello {name}</h1>
```

**2. Fragment + className:**

```jsx
<>
  <p className="hi">Hi Kakade</p>
</>
```

**3. Guarded conditional:**

```jsx
{isMember ? <p>Welcome Kakade</p> : <p>Join us</p>}
```

> **JSX is just a readable shortcut for element objects — braces for JS, one root, and Babel handles the rest.**
