# Positioning and Stacking — Revision Notes (Intermediate)
> One-line mental model: positioning is relative to a containing block, and z-index is relative to a stacking context.

## Core recall
- `static` = normal flow; `inset`/`z-index` do nothing.
- `relative` keeps space, shifts visually, and creates a containing block for absolute children.
- `absolute` leaves flow and positions to the nearest positioned ancestor (padding box).
- `fixed` positions to the viewport; `sticky` is relative until a scroll threshold.
- Sticky needs an inset threshold, a scrolling ancestor, and no `overflow:hidden` ancestor.
- `inset` = top/right/bottom/left shorthand; logical `inset-block`/`inset-inline` adapt to direction.
- `inset: 0` + `position: fixed` = full-screen overlay.
- Stacking contexts: positioned + z-index, opacity < 1, transform, filter, will-change, isolation, contain.
- `z-index` only competes within its stacking context; raising a number can never escape a parent context.
- Prefer `dialog` for modals; it handles top layer, backdrop, focus trap, and Escape.

## Must-remember code
```css
.card { position: relative; }
.card__badge {
  position: absolute;
  inset-block-start: 8px;
  inset-inline-end: 8px;
}
```
```css
.backdrop {
  position: fixed;
  inset: 0;
  background: rgb(15 23 42 / 0.5);
  z-index: 90;
}
.modal {
  position: fixed;
  inset: 0;
  margin: auto;
  width: min(90vw, 480px);
  z-index: 100;
}
```
```css
.header {
  position: sticky;
  top: 0;
  z-index: 20;
}
```

## Interview traps
- Absolute element flying to the viewport because the parent is not positioned.
- `z-index:9999` inside a trapped stacking context.
- Sticky failing due to `overflow:hidden` or no threshold.
- Using `fixed` for a modal and skipping focus management.
- Forgetting sticky needs room to move.
- Assuming `transform` is purely visual — it creates a stacking context and containing block.

## 3 things to remember
1. Absolute = nearest positioned ancestor.
2. z-index only works within one stacking context.
3. `inset:0` + fixed = overlay; `dialog` = real modal.
