# CSS Architecture

CSS architecture is how you keep styles predictable as a codebase grows. Naming conventions, layers, tokens, and file structure decide whether a new feature takes five minutes or a specificity war. Interviewers ask BEM, utility-first tradeoffs, and how you prevent regressions.

The core idea:

```text
architecture
    ├── naming → BEM / utilities
    ├── structure → ITCSS / feature folders
    ├── tokens → variables for color/space/type
    └── precedence → layers + flat specificity
```

---

# 1. BEM

BEM names blocks, elements, and modifiers: `.block`, `.block__element`, `.block--modifier`. It makes ownership obvious and keeps specificity flat.

```css
.card { padding: 1.25rem; }
.card__title { font-size: 1.25rem; }
.card__title--large { font-size: 1.5rem; }
.card--featured { border-color: #0f172a; }
```

```html
<article class="card card--featured">
  <h2 class="card__title card__title--large">Kakade</h2>
</article>
```

Output:

```text
Clear ownership: everything card-related starts with card.
All selectors stay at one class of specificity.
```

Gotchas: long names for deep nesting; modifiers should not be used as standalone blocks.

---

# 2. OOCSS, SMACSS and ITCSS

- **OOCSS**: separate structure from skin, and containers from content.
- **SMACSS**: categorize rules into base, layout, module, state, theme.
- **ITCSS**: order CSS from generic to specific in layers (settings, tools, generic, elements, objects, components, utilities).

```text
ITCSS order
  1. settings   → tokens
  2. tools      → mixins/functions
  3. generic    → reset
  4. elements   → bare tags
  5. objects    → layout patterns
  6. components → UI pieces
  7. utilities  → overrides
```

Output:

```text
Later layers are more specific in purpose and precedence.
```

These are mental models, not strict rules; ITCSS pairs well with `@layer`.

---

# 3. Utility-First CSS and Tailwind

Utility-first uses small single-purpose classes (`p-4`, `text-center`, `grid`) composed in markup. Tailwind popularized it with design tokens and a build step.

```html
<div class="grid gap-4 p-6 md:grid-cols-2">
  <article class="rounded-xl border p-4">Kakade</article>
</div>
```

```css
/* Equivalent utilities */
.grid { display: grid; }
.gap-4 { gap: 1rem; }
.p-6 { padding: 1.5rem; }
```

Output:

```text
Layout expressed in markup; no custom class per component.
```

Tradeoffs:

| Pros | Cons |
| ---- | ---- |
| Fast iteration | Markup gets noisy |
| Consistent tokens | Harder to read at scale |
| Fewer naming debates | Requires build/purge setup |
| Easy responsive variants | Custom design can feel constrained |

Interview answer: "Utility-first is great for product UI and consistency; component CSS/BEM is better when the design is bespoke or markup must stay clean."

---

# 4. File Organization and Design Tokens

Structure files by responsibility, not by page. Tokens centralize color, spacing, type, radius, and shadows.

```text
styles/
  tokens.css
  reset.css
  base.css
  layout.css
  components/
    card.css
    button.css
  utilities.css
```

```css
:root {
  --color-bg: #ffffff;
  --color-text: #0f172a;
  --space-1: 0.25rem;
  --space-2: 0.5rem;
  --space-4: 1rem;
  --radius-md: 12px;
}
```

Output:

```text
One source of truth for design values; components consume tokens.
```

Gotchas: too many one-off tokens is just hardcoding with extra steps; name by role (`--color-danger`) not by value (`--red`).

---

# 5. Specificity Management and Maintainability

Keep specificity flat, use layers for precedence, and avoid `!important`. Prefer classes over ids, avoid deep descendant chains, and scope component styles.

```css
@layer base, components, utilities;

@layer components {
  .card { padding: 1.25rem; }
  .card__title { margin: 0; }
}

@layer utilities {
  .mt-0 { margin-top: 0; }
}
```

Output:

```text
Utilities always win; components stay independent;
no !important required.
```

Review checklist:

- One class per selector where possible.
- No ids for styling.
- No `!important` in new code.
- No styles targeting bare tags inside components.
- State classes (`is-open`, `has-error`) documented.
- Dead CSS removed regularly.

---

# Real-World Uses

A design system combines tokens, layers, BEM components, and a small utility layer.

```css
@layer tokens, reset, components, utilities;

@layer tokens {
  :root {
    --color-brand: #0f172a;
    --space-4: 1rem;
    --radius-md: 12px;
  }
}

@layer components {
  .card {
    padding: var(--space-4);
    border-radius: var(--radius-md);
    border: 1px solid color-mix(in oklab, var(--color-brand) 12%, white);
  }
  .card__title {
    color: var(--color-brand);
    margin: 0;
  }
}

@layer utilities {
  .visually-hidden {
    position: absolute;
    width: 1px;
    height: 1px;
    overflow: hidden;
    clip: rect(0 0 0 0);
  }
}
```

Output:

```text
Tokens drive components; utilities override safely;
no specificity escalation.
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **BEM** | block, block__element, block--modifier |
| **OOCSS and SMACSS** | Separate structure/skin; categorize rules |
| **ITCSS** | Generic → specific layer order |
| **Utility-first** | Small classes composed in markup |
| **Tailwind** | Utility-first + tokens + build |
| **File organization** | By responsibility, not page |
| **Design tokens** | Central color/space/type/radius |
| **Specificity management** | Flat selectors, layers, no !important |
| **Maintainability** | Naming, docs, dead-code cleanup |

## The Mental Model

```text
architecture
  ├── naming → BEM or utilities
  ├── layers → tokens → reset → components → utilities
  ├── tokens → one source of truth
  └── discipline → flat specificity, no !important, cleanup
```

### The 3 things to remember

**1. Name by role:**

```css
.card__title { color: var(--color-brand); }
```

**2. Order by specificity purpose:**

```css
@layer tokens, reset, components, utilities;
```

**3. Keep selectors flat:**

```css
.card { }
.card__title { }
```

> **CSS architecture is naming + order + tokens — flat specificity and clear ownership beat clever selectors.**
