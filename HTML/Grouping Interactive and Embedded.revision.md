# Grouping, Interactive and Embedded — Revision Notes (Intermediate)
> One-line mental model: HTML ships native widgets — details, dialog, popover, template — use them before writing JavaScript.

## Core recall
- `div` blocks and `span` inlines are generic; correct only when no semantic element fits.
- `details` + `summary` = native accordion; `summary` must be the first child; `open` starts expanded.
- `dialog` + `showModal()` = modal with backdrop, focus trap, Escape close; `open` alone is non-modal.
- `<form method="dialog">` closes the dialog and sets `dialog.returnValue` from the button value.
- `template` content is inert — parsed but not rendered or loaded until cloned.
- `slot` is a placeholder used inside shadow DOM web components.
- `canvas` = bitmap drawn by JS, not in the accessibility tree; needs a text/data alternative.
- Inline `svg` = vector DOM, scalable, styleable, focusable, labelable with `role="img"` + `aria-label`.
- `noscript` shows fallback content when scripting is disabled.
- `popover` + `popovertarget` = lightweight non-modal top-layer popup with light dismiss.

## Must-remember code
```html
<details>
  <summary>What is the box model?</summary>
  <p>Content, padding, border, and margin.</p>
</details>
```
```html
<dialog id="settings">
  <form method="dialog">
    <h2>Kakade's settings</h2>
    <button value="save">Save</button>
    <button value="cancel">Cancel</button>
  </form>
</dialog>
```
```html
<template id="row-template">
  <li class="row"><span class="name"></span></li>
</template>
```
```html
<svg width="120" height="12" role="img" aria-label="Progress: 80 percent">
  <rect width="96" height="12" fill="#0f172a" />
</svg>
```
```html
<button popovertarget="menu">Menu</button>
<div id="menu" popover>...</div>
```

## Interview traps
- Using a div + click handler instead of `details`/`dialog`.
- Styling away the `summary` marker and losing the affordance.
- Opening `dialog` with only the `open` attribute and expecting a backdrop/focus trap.
- Canvas charts with no accessible alternative.
- Treating `popover` as a modal.
- Expecting `template` images/scripts to load before cloning.
- `noscript` used as the only content path instead of a fallback.

## 3 things to remember
1. Native accordion = `details` + `summary`.
2. Native modal = `dialog` + `showModal()`.
3. Canvas needs an accessible text/table alternative; SVG is DOM.
