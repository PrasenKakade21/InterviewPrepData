# CSS Fundamentals — Revision Notes (Beginner)
> One-line mental model: select elements, declare styles, and let the cascade turn rules into computed values.

## Core recall
- CSS = Cascading Style Sheets; declarative presentation language, not a general-purpose programming language.
- Rule anatomy: `selector { property: value; }`; at-rules start with `@` (`@media`, `@supports`, `@font-face`).
- Three attach points: external `<link>` (cached, preferred), `<style>` (per document), inline `style=""` (per element, high specificity).
- `@import` pulls CSS from CSS but adds a late request; prefer `<link rel="stylesheet">`.
- Only `/* ... */` comments are valid in plain CSS; `//` is Sass-only.
- Property names are case-insensitive; class names, ids, and custom properties are case-sensitive.
- Render chain: DOM + CSSOM → render tree → layout → paint.
- `display:none` removes from render tree; `visibility:hidden` keeps layout but skips paint.
- Preprocessors (Sass) add variables/nesting/mixins and compile to CSS; modern CSS covers much of that.
- DevTools: Styles (matched rules), Computed (final values), Layout (grid/flex overlays), `:hov` state toggle.

## Must-remember code
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
  .card { padding: calc(var(--space) * 2); }
}
```
```html
<link rel="stylesheet" href="/css/app.css" />
```

## Interview traps
- Calling CSS a programming language — it is declarative and rule-based.
- Using `@import` instead of `<link>` and delaying render.
- Inline styles overriding classes and resisting theming.
- Expecting `visibility:hidden` to remove layout space.
- `//` comments in plain CSS.
- Assuming class names are case-insensitive.

## 3 things to remember
1. `selector { property: value; }` is the whole language shape.
2. External CSS + cascade beats inline hacks.
3. CSSOM blocks first paint — keep critical CSS small.
