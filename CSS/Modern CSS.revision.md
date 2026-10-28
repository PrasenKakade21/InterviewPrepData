# Modern CSS — Revision Notes (Advanced)
> One-line mental model: modern CSS natively covers nesting, parent selection, layers, containers, color mixing, and view transitions.

## Core recall
- Native nesting: child rules inside a parent, `&` refers to the parent; still increases specificity.
- `:has()` selects by descendants/siblings — parent selection, state styling, scroll locking.
- `@layer` controls cascade priority; declare order once, fill layers.
- `@container` + `container-type` makes components respond to container size.
- `color-mix(in oklab, colorA 20%, white)` blends colors; `oklch()` is perceptually uniform.
- `aspect-ratio` locks proportions; logical properties (`margin-inline`, `padding-block`) adapt to direction.
- `dvh`/`svh` fix mobile `100vh` overflow.
- `subgrid` inherits parent tracks for cross-component alignment.
- View transitions (`::view-transition-*`, `startViewTransition`) animate state/page changes as enhancement.
- Keep `:has()` selectors specific; broad ones can be expensive.

## Must-remember code
```css
.card {
  padding: 1.25rem;
  &__title { font-size: 1.25rem; }
  &:hover { border-color: #94a3b8; }
}
```
```css
.card:has(img) { padding-top: 0; }
body:has(dialog[open]) { overflow: hidden; }
```
```css
@layer reset, base, components, utilities;
.card { container-type: inline-size; }
@container (min-width: 480px) {
  .card { grid-template-columns: 160px 1fr; }
}
```
```css
.card {
  margin-inline: auto;
  padding-block: 1.5rem;
  border: 1px solid color-mix(in oklab, #0f172a 12%, white);
}
```

## Interview traps
- Thinking native nesting is free — it adds specificity.
- Using `:has()` broadly on large DOMs.
- Confusing `@layer` with `@container`.
- Mixing physical and logical properties.
- Continuing to use `100vh` on mobile.
- Assuming `subgrid` works without parent tracks.
- Shipping view transitions without a non-animated fallback.

## 3 things to remember
1. `:has()` for parent/state logic.
2. Layers for precedence, containers for responsiveness.
3. Logical properties + dvh + color-mix for modern, resilient UI.
