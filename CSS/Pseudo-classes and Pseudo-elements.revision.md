# Pseudo-classes and Pseudo-elements — Revision Notes (Intermediate)
> One-line mental model: pseudo-classes react to state/position; pseudo-elements generate decoration — keep meaning in the DOM.

## Core recall
- Pseudo-class `:state` selects elements by state/position; pseudo-element `::part` selects a generated box.
- Interactive: `:hover`, `:focus`, `:active`, `:focus-visible`.
- `:focus-visible` keeps keyboard focus rings without mouse-click rings; never use `outline: none` without a replacement.
- Structural: `:first-child`, `:last-child`, `:nth-child()`, `:nth-of-type()`.
- Formulas: `2n` even, `2n+1` odd, `-n+3` first three, `n+4` from fourth.
- Form states: `:checked`, `:disabled`, `:required`, `:invalid`, `:user-invalid`.
- `:invalid` matches empty required fields on load; prefer `:user-invalid`.
- `::before`/`::after` need `content`, are not in the DOM, and are decorative.
- `::placeholder`, `::selection`, `::marker` style specific generated parts.
- Counters: `counter-reset`, `counter-increment`, `content: counter(name)`.

## Must-remember code
```css
.button:focus-visible {
  outline: 2px solid #0f172a;
  outline-offset: 2px;
}
```
```css
li:nth-child(2n) { background: #f8fafc; }
li:nth-of-type(3) { font-weight: 600; }
```
```css
.card::before {
  content: "";
  display: block;
  height: 4px;
  background: #0f172a;
}
```
```css
.faq { counter-reset: q; }
.faq details { counter-increment: q; }
.faq summary::before {
  content: counter(q) ". ";
  font-weight: 700;
}
```

## Interview traps
- `outline:none` with no replacement.
- Confusing `:nth-child` with `:nth-of-type`.
- `:invalid` styling errors before the user types.
- Using `::before` for essential text.
- Forgetting `content` so the pseudo-element never renders.
- Expecting `::marker` to accept all properties.
- Relying on generated arrows to communicate new tabs.

## 3 things to remember
1. State → single colon; generated part → double colon.
2. `:focus-visible` + visible ring, never remove focus.
3. `::before` is decoration; counters need reset + increment.
