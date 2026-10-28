# Accessibility in HTML — Revision Notes (Intermediate)
> One-line mental model: correct HTML gives names, roles, and focus for free — ARIA is only a patch for gaps.

## Core recall
- Native elements expose role and keyboard behavior; the first rule of ARIA is not to use ARIA when HTML already works.
- Heading order creates the outline screen-reader users navigate; never skip levels for size.
- `alt` describes purpose; `alt=""` for decorative; functional images describe the action.
- Every form control needs a `label` tied by `for`/`id`; placeholder is not a label.
- Landmarks: header/banner, nav/navigation, main/main, aside/complementary, footer/contentinfo, search/search.
- Skip link is the first focusable element and targets `#main`.
- Tables need `caption` + `th` + `scope` so cells are announced with headers.
- `aria-label` = direct name, `aria-labelledby` = referenced name (higher priority), `aria-describedby` = extra description.
- `aria-hidden="true"` removes from the accessibility tree but not from keyboard focus — avoid on focusable elements.
- `aria-live="polite"` announces dynamic updates without moving focus.
- Contrast AA: 4.5:1 normal text, 3:1 large text; support 200% zoom and never block pinch-zoom.

## Must-remember code
```html
<label for="email">Email</label>
<input id="email" name="email" type="email" autocomplete="email" required />
```
```html
<button aria-label="Close settings">
  <svg aria-hidden="true" width="16" height="16">...</svg>
</button>
```
```html
<section aria-labelledby="billing-heading">
  <h2 id="billing-heading">Billing</h2>
</section>
```
```html
<input id="password" type="password" aria-describedby="password-help" />
<p id="password-help">Use at least 8 characters.</p>
```
```html
<a href="#main" class="skip-link">Skip to content</a>
<main id="main">...</main>
```

## Interview traps
- `<div role="button">` without tabindex and keyboard handlers.
- `aria-hidden` on focusable elements (ghost controls).
- `aria-label` disagreeing with visible text.
- Placeholder used as the only label.
- Missing `scope`/`caption` on data tables.
- Blocking zoom with `user-scalable=no`.
- Positive `tabindex` breaking natural focus order.
- Icon-only buttons with no accessible name.

## 3 things to remember
1. Native element first; ARIA only when nothing native fits.
2. Every control and image needs a meaningful name.
3. Keep focus visible, ordered, and never hidden from AT while focusable.
