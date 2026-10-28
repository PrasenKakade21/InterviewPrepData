# Custom Properties and Theming

Custom properties (`--name`) are CSS variables. They make design tokens possible, enable runtime theming, and reduce repetition. Unlike Sass variables, they are live in the browser and participate in the cascade and inheritance.

The core idea:

```text
:root { --brand: #0f172a; }
    ↓
var(--brand) resolves at use time
    ↓
override the variable to re-theme a subtree
```

---

# 1. Declaring Custom Properties and var() Fallbacks

Declare variables on any selector; read them with `var()`. A fallback is the second argument.

```css
:root {
  --brand: #0f172a;
  --surface: #ffffff;
  --radius: 12px;
}

.card {
  background: var(--surface);
  color: var(--brand);
  border-radius: var(--radius, 8px);
  border: 1px solid var(--border, #e2e8f0);
}
```

Output:

```text
Card uses the brand/surface tokens; if --border is missing,
#e2e8f0 is used.
```

Gotchas: custom property names are case-sensitive; an invalid value makes the declaration invalid at computed-value time (often falling back to inherited/initial behavior).

---

# 2. Scope and Inheritance

Custom properties inherit. Declaring one on a component scopes it to that subtree; children can override it locally without affecting siblings.

```css
:root {
  --space: 1rem;
}

.card {
  --space: 1.5rem;
  padding: var(--space);
}

.compact {
  --space: 0.75rem;
}
```

Output:

```text
Default spacing 1rem globally; cards use 1.5rem;
.compact cards use 0.75rem.
```

Think:

```text
:root --space: 1rem
  ├── .card --space: 1.5rem → padding 1.5rem
  └── .compact --space: 0.75rem → padding 0.75rem
```

Interview trap: inheritance means a variable declared on `body` reaches everything; override at the narrowest scope that needs it.

---

# 3. Theming with Data Attributes

A theme switch is just a variable override on a root attribute. Components keep using tokens and never know which theme is active.

```css
:root {
  --bg: #ffffff;
  --text: #0f172a;
  --brand: #0f172a;
}

[data-theme="dark"] {
  --bg: #0f172a;
  --text: #e2e8f0;
  --brand: #93c5fd;
}

body {
  background: var(--bg);
  color: var(--text);
}
```

```html
<html data-theme="dark">
```

Output:

```text
Dark theme swaps three variables; every component updates.
```

This is the standard design-token approach. Keep semantic names (`--surface`, `--text-muted`) rather than raw colors so themes can change values independently.

---

# 4. Dark Mode with prefers-color-scheme

Respect the OS preference by default, and allow an explicit override.

```css
:root {
  --bg: #ffffff;
  --text: #0f172a;
}

@media (prefers-color-scheme: dark) {
  :root {
    --bg: #0f172a;
    --text: #e2e8f0;
  }
}

[data-theme="light"] {
  --bg: #ffffff;
  --text: #0f172a;
}
```

Output:

```text
Default follows OS; data-theme="light" forces light even on a dark OS.
```

Interview trap: `color-scheme: dark` also styles native controls and scrollbars; set it alongside your tokens.

---

# 5. calc() with Variables and @property

Variables can hold numbers, lengths, colors, or fragments. `calc()` combines them. `@property` registers a typed custom property with syntax, inheritance, and an initial value.

```css
:root {
  --gap: 16px;
  --scale: 1.5;
}

.grid {
  gap: calc(var(--gap) * 2);
}

.title {
  font-size: calc(1rem * var(--scale));
}

@property --angle {
  syntax: "<angle>";
  inherits: false;
  initial-value: 0deg;
}

.ring {
  background: conic-gradient(#0f172a var(--angle), #e2e8f0 0);
  transition: --angle 500ms ease;
}
```

Output:

```text
Gap 32px; title 1.5rem; conic ring can animate --angle.
```

Interview trap: unregistered custom properties are untyped strings, so they cannot be smoothly interpolated. `@property` gives them a type and enables animation.

---

# 6. Custom Property Pitfalls

Variables are resolved at computed-value time. Common traps: invalid values, spaces in token lists, and scope surprises.

```css
/* Bad: trailing space breaks the color */
:root { --brand: #0f172a ; }

/* Good */
:root { --brand: #0f172a; }

/* Space-separated token list */
:root { --shadow: 0 12px 24px rgb(15 23 42 / 0.12); }
.card { box-shadow: var(--shadow); }
```

Output:

```text
The shadow token expands into a complete box-shadow value.
```

Gotchas:

- A variable containing an invalid value fails the whole declaration.
- Fallbacks are evaluated only when the variable is missing/invalid, not for partial values.
- Do not store secrets in CSS — it ships to the browser.

---

# Real-World Uses

A design system defines semantic tokens, themes via `data-theme`, and respects the OS preference.

```css
:root {
  --color-bg: #ffffff;
  --color-surface: #f8fafc;
  --color-text: #0f172a;
  --color-muted: #475569;
  --color-brand: #0f172a;
  --radius: 12px;
  --shadow: 0 12px 24px rgb(15 23 42 / 0.12);
}

@media (prefers-color-scheme: dark) {
  :root {
    --color-bg: #0f172a;
    --color-surface: #1e293b;
    --color-text: #e2e8f0;
    --color-muted: #94a3b8;
    --color-brand: #93c5fd;
  }
}

.card {
  background: var(--color-surface);
  color: var(--color-text);
  border-radius: var(--radius);
  box-shadow: var(--shadow);
}
```

Output:

```text
The same card works in light/dark with zero component changes.
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **Declaring** | `--name: value;` on any selector |
| **var() and fallbacks** | `var(--x, fallback)` |
| **Scope and inheritance** | Inherit; override in a subtree |
| **Theming** | Override tokens on `[data-theme]` |
| **Dark mode** | `prefers-color-scheme` + explicit override |
| **calc() with variables** | Combine numbers/lengths |
| **@property** | Typed, animatable custom properties |
| **Pitfalls** | Invalid values, spaces, secrets |

## The Mental Model

```text
tokens
  ├── declare at :root → global defaults
  ├── inherit into components
  ├── override at [data-theme] → theme swap
  ├── read with var(--token, fallback)
  └── combine with calc() / register with @property
```

### The 3 things to remember

**1. Semantic tokens:**

```css
:root {
  --color-text: #0f172a;
  --color-bg: #ffffff;
}
```

**2. Theme by override:**

```css
[data-theme="dark"] {
  --color-text: #e2e8f0;
  --color-bg: #0f172a;
}
```

**3. Fallbacks:**

```css
color: var(--color-text, #0f172a);
```

> **Custom properties are inherited tokens — declare semantic names at the root, override them for themes, and read them with var() fallbacks.**
