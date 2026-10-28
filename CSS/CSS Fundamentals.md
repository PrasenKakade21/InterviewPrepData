# CSS Fundamentals

CSS clicks once you see the chain: `selectors` → `declarations` → `cascade` → `computed styles`. It is a declarative language: you describe how something should look, and the browser resolves conflicts and applies the result.

The core idea:

```text
CSS source
    ↓
parse rules
    ↓
match selectors to DOM
    ↓
cascade + inheritance
    ↓
computed styles → layout → paint
```

HTML gives structure, CSS gives presentation, JavaScript gives behavior. CSS is not "styling afterthought" — it is the layout engine's instruction set.

---

# 1. What is CSS and How CSS Works

`CSS` (Cascading Style Sheets) attaches visual rules to elements. "Cascading" means multiple rules can target the same element, and the browser resolves them by origin, specificity, and source order.

```css
h1 {
  color: #0f172a;
  font-size: 2rem;
}

.note {
  color: #475569;
}
```

```html
<h1>Kakade</h1>
<p class="note">Frontend learner</p>
```

Output:

```text
Kakade          → dark, 2rem
Frontend learner → muted gray
```

Think:

```text
selector { property: value; }
    │          │        │
    │          │        └── what to apply
    │          └── what aspect to change
    └── which elements
```

Interview trap: CSS is not a programming language in the loop/function sense — it is declarative and rule-based, though modern CSS has functions, variables, and conditionals.

---

# 2. CSS Syntax

A rule set has a selector and a declaration block. Declarations are `property: value` pairs separated by semicolons. At-rules (`@media`, `@supports`, `@font-face`) start with `@` and have their own syntax.

```css
/* Rule set */
.card {
  padding: 1rem;
  border: 1px solid #e2e8f0;
}

/* At-rule */
@media (min-width: 768px) {
  .card {
    padding: 2rem;
  }
}
```

Output:

```text
Card: 1rem padding, thin border
At 768px+: 2rem padding
```

Syntax rules:

- The last declaration's semicolon is optional but recommended.
- Property names are case-insensitive; custom property names are case-sensitive.
- Unknown properties are ignored, not errors that stop the sheet.
- A missing `}` can swallow the rest of the file.

---

# 3. External, Internal and Inline CSS, and @import

There are three ways to attach CSS. External files are cached and reusable; `<style>` is inline in the document; `style=""` applies to one element. `@import` pulls another stylesheet from inside CSS.

```html
<!-- External -->
<link rel="stylesheet" href="/css/app.css" />

<!-- Internal -->
<style>
  .hero { padding: 2rem; }
</style>

<!-- Inline -->
<p style="color: #0f172a">Kakade</p>
```

```css
/* Inside app.css */
@import url("/css/print.css") print;
```

Output:

```text
External: cached, reused across pages
Internal: one document, no extra request
Inline: one element, highest specificity
@import: extra request, blocks later rules
```

Interview trap: `@import` is render-blocking and slower than `<link>`; prefer `<link>` in HTML. Inline styles win over most selectors and are hard to override.

---

# 4. Comments and Case Sensitivity

CSS comments use `/* ... */` and can span lines. Property names and most keywords are case-insensitive, but class names, ids, and custom properties are case-sensitive.

```css
/* Kakade's theme */
.card {
  COLOR: #0f172a;      /* works */
  Background-Color: #fff;
}

.Kakade { color: red; }  /* different from .kakade */
```

Output:

```text
.card gets dark text on white.
.Kakade and .kakade are different selectors.
```

Notice: `//` comments are invalid in plain CSS (Sass allows them). Custom properties like `--brand-Color` are case-sensitive too.

---

# 5. CSSOM and Rendering

The browser parses CSS into the CSSOM, combines it with the DOM into a render tree, computes layout, then paints. Render-blocking CSS delays first paint, which is why critical CSS matters.

```css
.hero {
  display: grid;
  gap: 2rem;
  min-height: 60vh;
}
```

Output:

```text
Render tree includes .hero as a grid container,
layout computes tracks, then paint draws the page.
```

Think:

```text
HTML → DOM
CSS  → CSSOM
        ↓
   render tree
        ↓
 layout (boxes, sizes)
        ↓
 paint (pixels)
```

Interview trap: `display: none` removes an element from the render tree; `visibility: hidden` keeps its box but does not paint it.

---

# 6. CSS vs Preprocessors and DevTools

Preprocessors like Sass add variables, nesting, mixins, and functions, then compile to CSS. Modern CSS now covers much of that (custom properties, nesting, `color-mix()`), but Sass remains common in large codebases. DevTools shows matched rules, computed values, and layout overlays.

```scss
// Sass
$brand: #0f172a;

.card {
  color: $brand;
  &:hover { color: lighten($brand, 20%); }
}
```

```css
/* Compiled CSS */
.card { color: #0f172a; }
.card:hover { color: #334155; }
```

Output:

```text
Same final CSS either way; Sass adds authoring conveniences.
```

DevTools checklist:

- **Styles** pane: matched rules and overrides.
- **Computed** pane: final values.
- **Layout** pane: grid/flex overlays.
- Toggle `:hov` to test hover/focus states.

---

# Real-World Uses

A page combines an external stylesheet, a scoped component block, custom properties, and a media query.

```css
:root {
  --brand: #0f172a;
  --space: 1rem;
}

.card {
  padding: var(--space);
  color: var(--brand);
  border: 1px solid #e2e8f0;
}

@media (min-width: 768px) {
  .card {
    padding: calc(var(--space) * 2);
  }
}
```

Output:

```text
Card with brand text, thin border, and responsive padding.
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **What is CSS** | Declarative presentation language |
| **How CSS works** | Selectors + declarations + cascade |
| **CSS syntax** | `selector { property: value; }` |
| **Attaching CSS** | External `<link>`, `<style>`, inline `style` |
| **@import** | Pulls CSS from CSS; slower than `<link>` |
| **Comments** | `/* ... */` only |
| **Case sensitivity** | Properties insensitive; classes/custom props sensitive |
| **CSSOM** | Parsed CSS used to build the render tree |
| **Preprocessors** | Sass adds authoring features, compiles to CSS |
| **DevTools** | Matched rules, computed values, layout overlays |

## The Mental Model

```text
CSS
  │
  ├── attach → link / style / inline
  ├── select → selector matches DOM nodes
  ├── declare → property: value
  ├── cascade → origin + specificity + order
  └── compute → CSSOM → render → layout → paint
```

### The 3 things to remember

**1. Rule anatomy:**

```css
.card {
  padding: 1rem;
  color: #0f172a;
}
```

**2. External CSS first:**

```html
<link rel="stylesheet" href="/css/app.css" />
```

**3. Cascade decides:**

```css
.card { color: #475569; }
.card--active { color: #0f172a; } /* wins when both match */
```

> **CSS is declarative: select elements, declare styles, and let the cascade resolve conflicts into computed values.**
