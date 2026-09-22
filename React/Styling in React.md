# Styling in React

Styling clicks as: `scope decides method` → `conditionals compose classes` → `utilities vs components for scale`. No single winner — match the approach to global vs scoped needs.

The core idea:

```text
One-off tweak → inline style
    ↓
shared + static → CSS / Modules
    ↓
rapid UI + variants → Tailwind / styled
```

Prefer Modules or Tailwind for scoped work; reserve global CSS for resets and tokens.

---

# 1. Inline Styles, CSS Classes, and Global vs Scoped Styles

`Inline styles` (`style={{}}` camelCase objects) suit dynamic values. `CSS classes` (`className`) suit reusable static styles. `Global vs scoped styles`: global CSS leaks everywhere; Modules/Tailwind scope by default.

```jsx
function Hi() {
  return <p style={{ color: "tomato", fontWeight: "bold" }}>Hi Kakade</p>;
}
```

Output:

```text
Tomato bold Hi Kakade (dynamic, no stylesheet).
```

Class version:

```jsx
function Card() {
  return <div className="card">Card for Kakade</div>;
}
```

Output:

```text
.card styles apply (shared, cached by browser).
```

Think: inline for `width: ${pct}%`; classes for everything repeated. Never put design tokens only in inline objects — they cannot be themed or overridden cleanly.

---

# 2. CSS Modules, Conditional Classes, and clsx / classnames Pattern

`CSS Modules` (`Card.module.css` → `styles.card`) scope class names per file. `Conditional classes` toggle variants; `clsx / classnames pattern` composes them without string soup.

```jsx
import styles from "./Card.module.css";

function Card({ active }) {
  return <div className={active ? `${styles.card} ${styles.on}` : styles.card}>Kakade</div>;
}
```

Output:

```text
.card always; .on only when active (scoped hashes).
```

clsx shape:

```js
// import clsx from "clsx";
// clsx(styles.card, { [styles.on]: active }) → "card_hash on_hash" or "card_hash".
```

Convention: base + variant (`card`, `card--on`); compute once per render, keep logic out of JSX strings.

---

# 3. Tailwind with React, Styled-components Basics, and CSS-in-JS Tradeoffs

`Tailwind with React` composes utilities (`flex gap-2 rounded`) with variant props. `Styled-components basics` (`styled.button\`\``) co-locates CSS with components via props. `CSS-in-JS tradeoffs`: colocation + theming vs runtime cost and larger bundles.

```jsx
function Badge({ vip }) {
  return (
    <span className={vip ? "bg-gold px-2 rounded" : "bg-gray-200 px-2 rounded"}>
      {vip ? "VIP Kakade" : "Kakade"}
    </span>
  );
}
```

Output:

```text
VIP Kakade gets gold pill; default gets gray.
```

Styled shape:

```js
// import styled from "styled-components";
// const Btn = styled.button`
//   padding: 8px 12px;
//   background: ${(p) => (p.primary ? "black" : "white")};
// `;
// <Btn primary>Hi Kakade</Btn>
```

| Approach | Strength | Watch out |
| -------- | -------- | --------- |
| **Modules** | **Zero runtime, scoped** | **Verbose variants** |
| **Tailwind** | **Fast variants, tiny CSS** | **Long class strings** |
| **styled** | **Props-driven, themed** | **Runtime + bundle cost** |

Pick one primary system per app — mixing all three creates inconsistency.

---

# Real-World Uses

A Kakade profile card uses a Module for layout, Tailwind utilities for spacing/variants, and a single styled primary button — tokens in CSS variables so dark mode flips in one place.

```jsx
function Chip() {
  return <span className="px-2 rounded bg-gray-200">Kakade</span>;
}
```

Output:

```text
Gray chip for Kakade (utility, no new CSS file).
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **Inline styles** | Dynamic values, camelCase |
| **CSS classes** | Reusable static styles |
| **CSS Modules** | `*.module.css`, scoped hashes |
| **Conditional classes** | Ternary / variant props |
| **clsx / classnames pattern** | Clean class composition |
| **Tailwind with React** | Utility variants fast |
| **Styled-components basics** | `styled.x` + props CSS |
| **CSS-in-JS tradeoffs** | Colocation vs runtime cost |
| **Global vs scoped styles** | Reset global, scope components |

## The Mental Model

```text
Style need
  │
  ├── dynamic value? → inline style
  │
  ├── scoped file? → CSS Module + clsx
  │
  └── system choice → Tailwind XOR styled
        └── tokens global, components scoped
```

### The 3 things to remember

**1. Dynamic inline:**

```jsx
<p style={{ color: "tomato" }}>Hi Kakade</p>
```

**2. Scoped + conditional:**

```jsx
<div className={clsx(styles.card, { [styles.on]: active })}>Kakade</div>
```

**3. One system:**

```jsx
<span className="px-2 rounded bg-gray-200">Kakade</span>
```

> **Scope by default, compose variants cleanly, and standardize on one styling system.**
