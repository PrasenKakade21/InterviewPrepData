# CSS Architecture — Revision Notes (Intermediate)
> One-line mental model: architecture is naming + order + tokens — flat specificity and clear ownership beat clever selectors.

## Core recall
- BEM: `.block`, `.block__element`, `.block--modifier`; ownership obvious, specificity flat.
- OOCSS: separate structure/skin, container/content.
- SMACSS: base, layout, module, state, theme categories.
- ITCSS: settings → tools → generic → elements → objects → components → utilities.
- Utility-first: small single-purpose classes composed in markup; Tailwind adds tokens/build.
- Utility tradeoffs: speed/consistency vs noisy markup and redesign cost.
- Organize files by responsibility (`tokens`, `reset`, `base`, `layout`, `components`, `utilities`).
- Design tokens named by role (`--color-danger`), not value (`--red`).
- Manage specificity with `@layer`, class selectors, and no `!important`.
- Review for dead CSS, deep selectors, and undocumented state classes.

## Must-remember code
```css
@layer tokens, reset, components, utilities;
```
```css
.card { padding: var(--space-4); }
.card__title { color: var(--color-brand); margin: 0; }
.card--featured { border-color: var(--color-brand); }
```
```css
:root {
  --color-bg: #ffffff;
  --color-text: #0f172a;
  --space-4: 1rem;
  --radius-md: 12px;
}
```
```html
<div class="grid gap-4 p-6 md:grid-cols-2">...</div>
```

## Interview traps
- Deep descendant selectors instead of BEM/flat classes.
- Confusing the three methodologies (OOCSS/SMACSS/ITCSS).
- Assuming utility-first is always better or always worse.
- Naming tokens by raw color/value.
- Creating a token for every one-off value.
- Reaching for `!important` instead of layers.
- No cleanup plan for dead CSS.

## 3 things to remember
1. Name by role and ownership (BEM or tokens).
2. Order layers once; keep specificity flat.
3. Tokens for repeated decisions, not every value.
