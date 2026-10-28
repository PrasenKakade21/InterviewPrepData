# HTML5 Modern Features — Revision Notes (Intermediate)
> One-line mental model: HTML5 ships native components and performance features — use them before adding JavaScript or libraries.

## Core recall
- `data-*` stores custom data; read via `dataset` (kebab → camelCase), values are strings, never secrets.
- `template` markup is inert: not rendered, assets not loaded; clone `template.content` with `cloneNode(true)`.
- `dialog` + `showModal()` = native modal with backdrop/focus trap; `<form method="dialog">` closes it.
- `details` + `summary` = native accordion, no JS.
- `picture` + `source` = format/art-direction fallback; must end with `img`.
- `srcset` + `sizes` = resolution switching.
- `loading="lazy"` defers offscreen images — not the hero/LCP image.
- `contenteditable` is editable but not a form control; needs ARIA and syncing.
- `draggable` + `dataTransfer` = native drag-and-drop, no keyboard support by default.
- Custom elements: hyphenated names + lifecycle callbacks via `customElements.define`.
- `popover` + `popovertarget` = native non-modal top-layer popup.
- `canvas` = pixels via JS, needs text alternative; inline `svg` = vector DOM, labelable.

## Must-remember code
```html
<button class="plan" data-plan="pro" data-price="499">Choose Pro</button>
<script>
  const btn = document.querySelector(".plan");
  console.log(btn.dataset.plan, btn.dataset.price); // pro 499
</script>
```
```html
<template id="card">
  <article class="card"><h2 class="card__title"></h2></article>
</template>
<script>
  const card = document.getElementById("card").content.cloneNode(true);
  card.querySelector(".card__title").textContent = "Kakade";
  document.body.append(card);
</script>
```
```html
<dialog id="confirm">
  <form method="dialog">
    <button value="yes">Delete</button>
    <button value="no">Cancel</button>
  </form>
</dialog>
```
```html
<picture>
  <source srcset="/img/hero.avif" type="image/avif" />
  <img src="/img/hero.jpg" alt="Kakade's desk" width="1200" height="675" loading="lazy" />
</picture>
```
```html
<button popovertarget="menu">Menu</button>
<div id="menu" popover>...</div>
```

## Interview traps
- Lazy-loading the LCP hero image.
- Using `picture` without a fallback `img`.
- Forgetting `cloneNode(true)` and moving template nodes.
- Treating `contenteditable` as a submittable field.
- Custom element names without a hyphen (invalid).
- Canvas with no accessible alternative.
- `popover` used where a modal `dialog` is required.
- Storing sensitive data in `data-*`.

## 3 things to remember
1. Native first: `details`, `dialog`, `popover`, `template`.
2. `data-*` for data, `dataset` to read, strings only.
3. Lazy-load offscreen media; keep the hero eager.
