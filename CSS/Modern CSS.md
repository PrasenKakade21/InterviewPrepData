# Modern CSS

Modern CSS absorbed many preprocessor and JavaScript features: native nesting, parent selection with `:has()`, cascade layers, container queries, color mixing, logical properties, and view transitions. Interviewers increasingly ask which features are native now.

The core idea:

```text
modern CSS
    ├── structure → nesting, @layer
    ├── selection → :has(), :is(), :where()
    ├── layout → container queries, subgrid, aspect-ratio
    ├── color → color-mix(), oklch()
    └── motion → view transitions
```

---

# 1. Native Nesting

Nesting lets you write child rules inside a parent without Sass. `&` refers to the parent selector.

```css
.card {
  padding: 1.25rem;

  &__title {
    font-size: 1.25rem;
  }

  &:hover {
    border-color: #94a3b8;
  }

  @media (min-width: 768px) {
    padding: 2rem;
  }
}
```

Output:

```text
Compiles to .card, .card__title, .card:hover,
and a media query scoped to .card.
```

Gotchas: nesting increases specificity like a descendant selector; deeply nested rules become hard to override. Browser support is now broad, but check build targets.

---

# 2. :has()

`:has()` selects an element based on its descendants or siblings — the long-missing "parent selector".

```css
.card:has(img) {
  padding-top: 0;
}

.field:has(input:invalid) {
  border-color: #b91c1c;
}

body:has(dialog[open]) {
  overflow: hidden;
}

label:has(input:checked) {
  font-weight: 600;
}
```

Output:

```text
Cards with images lose top padding; fields with invalid input go red;
page scroll locks while a dialog is open; checked labels bold.
```

Interview trap: `:has()` is powerful but can be expensive on large DOMs if the argument is broad; keep selectors specific.

---

# 3. @layer and @container

`@layer` controls cascade priority without specificity wars. `@container` makes components respond to their container.

```css
@layer reset, base, components, utilities;

@layer components {
  .card {
    container-type: inline-size;
    padding: 1rem;
  }
}

@container (min-width: 480px) {
  .card__body {
    display: grid;
    grid-template-columns: 160px 1fr;
  }
}
```

Output:

```text
Utilities beat components regardless of selector weight;
card layout changes when its own container is wide.
```

Layers and containers solve different problems: layers = precedence, containers = responsive components.

---

# 4. color-mix(), aspect-ratio and Logical Properties

`color-mix()` blends colors. `aspect-ratio` locks proportions. Logical properties (`margin-inline`, `padding-block`, `inset-inline`) adapt to writing direction.

```css
.button {
  background: color-mix(in oklab, #0f172a 85%, white);
  color: #fff;
}

.media {
  aspect-ratio: 16 / 9;
}

.card {
  margin-inline: auto;
  padding-block: 1.5rem;
  border-inline-start: 4px solid #0f172a;
}
```

Output:

```text
Button uses a lighter brand mix; media is 16:9;
card centers horizontally and adapts to RTL with logical properties.
```

Interview trap: physical (`margin-left`) and logical (`margin-inline-start`) properties can conflict; pick one system per codebase.

---

# 5. Dynamic Viewport Units, Subgrid and View Transitions

`dvh`/`svh` fix mobile viewport height. `subgrid` inherits parent tracks. View transitions animate between page states with `view-transition-name`.

```css
.hero {
  min-height: 100dvh;
}

.card {
  display: grid;
  grid-template-rows: subgrid;
  grid-row: span 2;
}

::view-transition-old(root),
::view-transition-new(root) {
  animation-duration: 250ms;
}
```

Output:

```text
Hero fits the visible mobile viewport; nested card aligns to parent rows;
route changes can cross-fade with the View Transitions API.
```

View transitions require `document.startViewTransition()` and are progressive enhancement; always keep the non-animated path working.

---

# Real-World Uses

A modern card grid uses layers, nesting, container queries, logical properties, and color mixing.

```css
@layer reset, base, components;

@layer components {
  .grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(240px, 1fr));
    gap: 1.5rem;
  }

  .card {
    container-type: inline-size;
    padding-block: 1.25rem;
    padding-inline: 1.25rem;
    border: 1px solid color-mix(in oklab, #0f172a 12%, white);

    &:has(img) {
      padding-block-start: 0;
    }

    &__title {
      font-size: 1.125rem;
      text-wrap: balance;
    }
  }
}

@container (min-width: 420px) {
  .card {
    display: grid;
    grid-template-columns: 140px 1fr;
    gap: 1rem;
  }
}
```

Output:

```text
Responsive cards with native nesting, tokenized borders,
container-driven layout, and no preprocessor.
```

---

# Quick Revision

| Feature | Remember |
| ------- | -------- |
| **Nesting** | Child rules inside a parent, `&` for parent |
| **:has()** | Select by descendants/siblings |
| **@layer** | Cascade priority without specificity |
| **@container** | Component responds to container size |
| **color-mix()** | Blend two colors |
| **aspect-ratio** | Lock width-to-height |
| **Logical properties** | inline/block adapt to direction |
| **dvh/svh** | Honest mobile viewport height |
| **Subgrid** | Inherit parent tracks |
| **View transitions** | Animate page/state changes |

## The Mental Model

```text
modern CSS
  ├── authoring → nesting
  ├── selection → :has(), :is(), :where()
  ├── precedence → @layer
  ├── responsiveness → @container, clamp, dvh
  ├── design → color-mix, logical props, aspect-ratio
  └── motion → view transitions
```

### The 3 things to remember

**1. :has() for parent logic:**

```css
.card:has(img) { padding-top: 0; }
```

**2. Layers before specificity:**

```css
@layer reset, base, components, utilities;
```

**3. Container queries for components:**

```css
.card { container-type: inline-size; }
@container (min-width: 480px) { ... }
```

> **Modern CSS covers nesting, parent selection, layers, containers, and color mixing natively — use the platform before adding tooling.**
