# HTML5 Modern Features

HTML5 is not one feature — it is a set of native upgrades: custom data with `data-*`, reusable markup with `template`, native modals and accordions, responsive images, lazy loading, editable regions, custom elements, and popovers. Knowing which are native saves entire libraries of JavaScript.

The core idea:

```text
modern HTML
    ├── data-*        → custom data on elements
    ├── template      → inert reusable markup
    ├── dialog        → native modal
    ├── picture       → responsive art direction
    ├── loading=lazy  → deferred images
    ├── contenteditable → editable regions
    ├── custom elements → author-defined tags
    └── popover       → native top-layer popup
```

Use the platform before adding a dependency.

---

# 1. data-* Attributes

`data-*` stores custom data directly on an element. It is readable in CSS via `attr()` (limited) and in JavaScript through `dataset`, with kebab-case names converted to camelCase.

```html
<button class="plan" data-plan="pro" data-price="499" data-currency="INR">
  Choose Pro
</button>

<script>
  const btn = document.querySelector(".plan");
  console.log(btn.dataset.plan, btn.dataset.price);
</script>
```

Output:

```text
pro 499
```

`data-user-id` becomes `dataset.userId`. Values are always strings — parse numbers explicitly. Do not store secrets: anything in HTML is public.

---

# 2. template, dialog and details

These three cover most "I need a component" cases: `template` for cloning markup, `dialog` for modals, and `details`/`summary` for accordions.

```html
<template id="card">
  <article class="card">
    <h2 class="card__title"></h2>
    <p class="card__body"></p>
  </article>
</template>

<details>
  <summary>Why use template?</summary>
  <p>It parses markup without rendering or loading assets.</p>
</details>

<dialog id="confirm">
  <form method="dialog">
    <p>Delete Kakade's draft?</p>
    <button value="yes">Delete</button>
    <button value="no">Cancel</button>
  </form>
</dialog>
```

Output:

```text
Template stays inert until cloned.
Details expands natively on click.
Dialog opens modally with showModal().
```

Interview trap: `template.content` is a DocumentFragment — clone it with `cloneNode(true)` before inserting, or you move the original nodes.

---

# 3. picture, srcset and loading="lazy"

Responsive images and lazy loading are native performance features: `picture`/`srcset` pick the right file, `loading="lazy"` defers offscreen images.

```html
<picture>
  <source srcset="/img/hero.avif" type="image/avif" />
  <source srcset="/img/hero.webp" type="image/webp" />
  <img
    src="/img/hero.jpg"
    alt="Kakade's desk setup"
    width="1200"
    height="675"
    loading="lazy"
    decoding="async"
  />
</picture>
```

Output:

```text
AVIF browser → hero.avif
WebP browser → hero.webp
Else         → hero.jpg
Image loads only when near the viewport.
```

Keep above-the-fold hero images eager: lazy loading the LCP image delays the largest paint.

---

# 4. contenteditable and Drag-and-Drop Attributes

`contenteditable` makes an element editable. `draggable="true"` plus `dragstart`/`dragover`/`drop` handlers enables native drag-and-drop. Both need deliberate accessibility handling.

```html
<div contenteditable="true" role="textbox" aria-multiline="true" aria-label="Kakade's note">
  Edit this note.
</div>

<ul id="board">
  <li draggable="true" data-id="1">HTML task</li>
  <li draggable="true" data-id="2">CSS task</li>
</ul>

<script>
  document.querySelectorAll("[draggable]").forEach((el) => {
    el.addEventListener("dragstart", (e) => {
      e.dataTransfer.setData("text/plain", el.dataset.id);
    });
  });
</script>
```

Output:

```text
Note becomes editable in place.
Tasks can be dragged; the id travels in dataTransfer.
```

Drag-and-drop has no keyboard equivalent by default — provide buttons or a menu for reordering.

---

# 5. Custom Elements Overview and popover

Custom elements let you define your own tags (`<kakade-card>`) with lifecycle callbacks. `popover` gives a native top-layer popup without custom positioning code.

```html
<kakade-badge label="Pro"></kakade-badge>

<button popovertarget="menu">Kakade's menu</button>
<div id="menu" popover>
  <a href="/notes">Notes</a>
  <a href="/projects">Projects</a>
</div>

<script>
  class KakadeBadge extends HTMLElement {
    connectedCallback() {
      this.textContent = `[${this.getAttribute("label")}]`;
    }
  }
  customElements.define("kakade-badge", KakadeBadge);
</script>
```

Output:

```text
[Pro] rendered by the custom element
Menu opens in the top layer, closes on outside click or Escape
```

Custom element names must contain a hyphen. They are not a replacement for semantic HTML — a badge should still expose meaning to assistive tech.

---

# 6. canvas and Inline SVG

`canvas` draws pixels through JavaScript; inline `svg` is vector markup in the DOM. Use canvas for high-volume rendering (games, large charts) and SVG for icons, diagrams, and accessible graphics.

```html
<canvas id="spark" width="240" height="60" aria-label="Study trend: rising"></canvas>

<svg viewBox="0 0 24 24" width="24" height="24" role="img" aria-label="Checkmark">
  <path d="M4 12l5 5L20 6" fill="none" stroke="currentColor" stroke-width="2" />
</svg>
```

Output:

```text
Canvas stays blank until JS draws on the 2D context.
SVG checkmark renders immediately and scales cleanly.
```

Accessibility: canvas needs a text alternative or data table; SVG can carry `role="img"` + `aria-label` and individual shapes are inspectable.

---

# Real-World Uses

A product card uses a template, data attributes, a responsive image, a popover menu, and a custom element.

```html
<template id="product-template">
  <article class="product" data-sku="">
    <img src="" alt="" width="320" height="320" loading="lazy" />
    <h2 class="product__name"></h2>
    <button popovertarget="actions">Actions</button>
  </article>
</template>

<div id="products"></div>

<div id="actions" popover>
  <button>Add to cart</button>
  <button>Save for later</button>
</div>
```

Output:

```text
Cards cloned from the template, images lazy-loaded,
actions available in a native popover.
```

---

# Quick Revision

| Feature | Remember |
| ------- | -------- |
| **data-*** | Custom data; `dataset` camelCase; always strings |
| **template** | Inert markup; clone before insert |
| **dialog** | Native modal with `showModal()` |
| **details** | Native accordion with `summary` |
| **picture/srcset** | Format fallback and responsive sources |
| **loading=lazy** | Defer offscreen images; not the hero |
| **contenteditable** | Editable but not a form control |
| **Drag and drop** | `draggable` + dataTransfer + handlers |
| **Custom elements** | Hyphenated tags with lifecycle callbacks |
| **popover** | Native non-modal top-layer popup |
| **canvas/svg** | Pixels via JS vs vector DOM |

## The Mental Model

```text
modern HTML toolbox
  ├── data → data-*
  ├── reusable → template
  ├── modal → dialog
  ├── disclosure → details/summary
  ├── responsive media → picture + srcset + lazy
  ├── editable → contenteditable
  ├── drag → draggable + DnD events
  ├── custom UI → custom elements
  ├── popup → popover
  └── graphics → canvas / svg
```

### The 3 things to remember

**1. Native first:**

```html
<details><summary>FAQ</summary><p>Answer.</p></details>
```

**2. Data lives in data-*:**

```html
<button data-plan="pro" data-price="499">Pro</button>
```

**3. Lazy, but not the hero:**

```html
<img src="/hero.jpg" alt="Kakade's desk" width="1200" height="675" />
<img src="/thumb.jpg" alt="" width="200" height="200" loading="lazy" />
```

> **HTML5 ships native components — template, dialog, details, popover, lazy media — so reach for markup before JavaScript.**
