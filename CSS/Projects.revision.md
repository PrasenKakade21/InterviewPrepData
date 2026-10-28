# Projects — Revision Notes (Intermediate)
> One-line mental model: build small projects that each add one skill — layout, components, motion, theming — and the system emerges.

## Core recall
- Build raw CSS first; frameworks hide box model, cascade, and layout behavior.
- Landing page: mobile-first, grid/flex header, `clamp()` type, hero, focus states, 320px reflow.
- Dashboard: named grid areas, fixed sidebar, `auto-fit` widget grid, mobile collapse.
- Pricing cards: grid `stretch` + flex column + `margin-top:auto` CTA.
- Modal: native `dialog` + `showModal()`, `::backdrop`, entrance animation, reduced-motion override.
- Theme: semantic tokens at `:root`, `[data-theme]` overrides, `prefers-color-scheme`.
- Navbar: flex wrap; stack links on small screens.
- Illustration: pseudo-elements, gradients, transforms, `aspect-ratio`.
- Design system: tokens, type scale, buttons, forms, states, documentation from one source.
- Avoid documentation drift by importing the real CSS/tokens.

## Must-remember code
```css
.shell {
  display: grid;
  grid-template-columns: 240px 1fr;
  gap: 1rem;
}
```
```css
.plan {
  display: flex;
  flex-direction: column;
}
.plan__cta { margin-top: auto; }
```
```css
dialog::backdrop { background: rgb(15 23 42 / 0.55); }
dialog[open] { animation: modal-in 180ms ease-out; }
@media (prefers-reduced-motion: reduce) {
  dialog[open] { animation: none; }
}
```
```css
:root {
  --space-4: 1rem;
  --radius-md: 12px;
  --color-brand: #0f172a;
}
```

## Interview traps
- Jumping to a framework before understanding layout.
- Fixed widths that break at 320px.
- Cards with unequal heights and floating CTAs.
- Hand-rolled modals without focus management.
- Copying tokens into docs instead of sharing them.
- Animations without reduced-motion handling.
- Hiding mobile navigation with no accessible alternative.

## 3 things to remember
1. One project, one skill cluster.
2. Tokens + native dialog + reduced motion from the start.
3. Test at 320px, 200% zoom, and keyboard only.
