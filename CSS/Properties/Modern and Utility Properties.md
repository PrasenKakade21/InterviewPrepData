# Modern and Utility Properties

This reference collects modern CSS at-rules, typed custom properties, logical properties, and utility properties that do not fit the other categories.

```text
modern & utility
  ├── tokens → custom properties, var(), @property
  ├── precedence → @layer
  ├── responsiveness → @container
  ├── selection → :has()
  ├── direction → logical properties
  └── behavior → scroll, pointer, selection, sizing
```

---

# Custom properties, var() and @property

Declare reusable values and register types.

```css
:root {
  --brand: #0f172a;
  --space-4: 1rem;
}

.card {
  color: var(--brand);
  padding: var(--space-4);
}

@property --angle {
  syntax: "<angle>";
  inherits: false;
  initial-value: 0deg;
}
```

Gotchas: unregistered properties are untyped strings and cannot interpolate; invalid values fail the declaration.

---

# @layer

Declares cascade layers and their precedence order.

```css
@layer reset, base, components, utilities;

@layer components {
  .card { padding: 1rem; }
}

@layer utilities {
  .p-0 { padding: 0; }
}
```

Gotchas: later layers win regardless of specificity; unlayered normal styles beat layered normal styles.

---

# @container and container-type

Enables component-level responsive rules.

```css
.card-wrap {
  container-type: inline-size;
  container-name: card;
}

@container card (min-width: 480px) {
  .card { grid-template-columns: 160px 1fr; }
}
```

Gotchas: requires a containment ancestor; responds to container size, not viewport.

---

# :has()

Selects an element based on descendants/siblings.

```css
.card:has(img) { padding-top: 0; }
body:has(dialog[open]) { overflow: hidden; }
label:has(input:checked) { font-weight: 600; }
```

Gotchas: broad `:has()` selectors can be expensive on large DOMs.

---

# Logical properties

Direction-aware spacing and sizing.

```css
margin-inline: auto;
margin-block: 1rem;
padding-inline: 1.25rem;
border-inline-start: 4px solid #0f172a;
inset-inline-end: 8px;
inline-size: 100%;
block-size: auto;
```

Gotchas: do not mix physical and logical properties for the same side; pick one system.

---

# scroll-behavior, overscroll-behavior

Smooth scrolling and scroll chaining control.

```css
html { scroll-behavior: smooth; }
.modal { overscroll-behavior: contain; }
```

Gotchas: respect reduced motion (`scroll-behavior: auto`); `overscroll-behavior` is not a scrollbar style.

---

# scrollbar-width, scrollbar-color, ::-webkit-scrollbar

Scrollbar styling (limited and browser-specific).

```css
* {
  scrollbar-width: thin;
  scrollbar-color: #94a3b8 #e2e8f0;
}
```

Gotchas: `::-webkit-scrollbar` is non-standard; ensure scrollbars remain visible and usable.

---

# user-select, pointer-events, cursor

Interaction behavior.

```css
.select-none { user-select: none; }
.disabled { pointer-events: none; }
.clickable { cursor: pointer; }
```

Gotchas: `pointer-events:none` also blocks focus/clicks on children; do not remove text selection from content users may want to copy.

---

# resize, appearance, touch-action

Control resizing, native styling, and touch gestures.

```css
textarea { resize: vertical; }
button { appearance: none; }
.canvas { touch-action: none; }
```

Gotchas: `appearance:none` removes native affordances — restyle carefully; `touch-action:none` disables browser gestures.

---

# will-change, contain, content-visibility

Rendering hints and isolation.

```css
.drawer { will-change: transform; }
.card { contain: layout paint; }
.section {
  content-visibility: auto;
  contain-intrinsic-size: 600px;
}
```

Gotchas: permanent `will-change` wastes memory; `contain` can affect sticky/overflow; `content-visibility` needs intrinsic size to avoid jumps.

---

# object-fit, object-position, aspect-ratio

Media sizing.

```css
.avatar {
  width: 96px;
  height: 96px;
  object-fit: cover;
  object-position: center;
  border-radius: 50%;
}

.media { aspect-ratio: 16 / 9; }
```

Gotchas: `object-fit` applies to replaced elements; `aspect-ratio` is overridden by explicit width and height.

---

# Real-World Uses

A card system combining layers, containers, logical spacing, and typed properties.

```css
@layer tokens, components, utilities;

@layer tokens {
  :root {
    --brand: #0f172a;
    --space-4: 1rem;
  }
}

@layer components {
  .card-wrap {
    container-type: inline-size;
  }

  .card {
    padding-inline: var(--space-4);
    padding-block: var(--space-4);
    border-inline-start: 4px solid var(--brand);
  }

  .card:has(img) {
    padding-block-start: 0;
  }
}

@container (min-width: 480px) {
  .card {
    display: grid;
    grid-template-columns: 160px 1fr;
  }
}
```

---

# Quick Revision

| Feature | Purpose |
| ------- | ------- |
| **Custom properties/var()** | Reusable tokens |
| **@property** | Typed, animatable tokens |
| **@layer** | Cascade precedence |
| **@container** | Component responsiveness |
| **:has()** | Parent/sibling selection |
| **Logical properties** | Direction-aware spacing |
| **scroll-behavior** | Smooth scrolling |
| **user-select/pointer-events** | Interaction control |
| **resize/appearance/touch-action** | Native control |
| **will-change/contain/content-visibility** | Rendering hints |
| **object-fit/aspect-ratio** | Media sizing |

## The Mental Model

```text
modern utilities
  ├── tokens → --vars, var(), @property
  ├── order → @layer
  ├── responsiveness → @container
  ├── selection → :has()
  ├── direction → logical properties
  └── behavior → scroll/pointer/selection/rendering hints
```

### The 3 things to remember

**1. Tokens + layers:**

```css
@layer tokens, components, utilities;
:root { --brand: #0f172a; }
```

**2. Container queries:**

```css
.card-wrap { container-type: inline-size; }
@container (min-width: 480px) { ... }
```

**3. Logical spacing:**

```css
padding-inline: 1.25rem;
margin-block: 1rem;
```

> **Modern CSS gives you tokens, layers, containers, parent selection, and direction-aware properties — use them before adding tooling or hacks.**
