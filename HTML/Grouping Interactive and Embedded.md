# Grouping, Interactive and Embedded Content

Not every piece of HTML is a heading or a paragraph. `div` and `span` group things, `details`/`summary` build native accordions, `dialog` creates modals, `template` holds inert markup, and `canvas`/`svg` draw graphics.

The core idea:

```text
need
    ├── group / layout      → div, span
    ├── show-hide content   → details + summary
    ├── modal               → dialog
    ├── reusable markup     → template + slot
    ├── drawing             → canvas / svg
    └── fallback/scripting  → noscript, script
```

Native elements come with keyboard behavior and accessibility built in — use them before reaching for JavaScript.

---

# 1. div and span

`div` is a block-level generic container; `span` is inline. They carry no meaning, which makes them correct for layout wrappers and styling hooks — and wrong when a semantic element exists.

```html
<div class="card-grid">
  <article class="card">
    <h2>Kakade's Notes</h2>
    <p>Price: <span class="price">₹499</span></p>
  </article>
</div>
```

Output:

```text
Two-column card grid (CSS handles layout)
Price: ₹499 (price styled inline)
```

| Element | Display | Use |
| ------- | ------- | --- |
| `div` | Block | Layout grouping |
| `span` | Inline | Styling a text fragment |

Interview trap: div soup is not a CSS problem — it is a lost-semantics problem. Replace wrappers with `header`, `nav`, `main`, `section`, `article`, `aside`, `footer` where they fit.

---

# 2. details and summary

`details` is a native disclosure widget; `summary` is its always-visible label. Clicking toggles the content, keyboard support is free, and `open` starts it expanded.

```html
<details>
  <summary>What is the box model?</summary>
  <p>Content, padding, border, and margin around an element.</p>
</details>

<details open>
  <summary>Do I need JavaScript for this?</summary>
  <p>No — details and summary are native HTML.</p>
</details>
```

Output:

```text
▸ What is the box model?          ← collapsed, expands on click
▾ Do I need JavaScript for this?  ← starts open
  No — details and summary are native HTML.
```

Interview trap: `summary` must be the first child of `details`. Styling the default triangle requires `summary::-webkit-details-marker` and `summary::marker`, not `display:none` on the summary.

---

# 3. The dialog Element

`dialog` is a native modal/non-modal dialog. `showModal()` opens it modally with a backdrop, focus trapping, and Escape-to-close; `show()` opens it non-modally.

```html
<button id="open">Open settings</button>

<dialog id="settings">
  <form method="dialog">
    <h2>Kakade's settings</h2>
    <label for="theme">Theme</label>
    <select id="theme" name="theme">
      <option>Light</option>
      <option>Dark</option>
    </select>
    <button value="save">Save</button>
    <button value="cancel">Cancel</button>
  </form>
</dialog>
```

Output:

```text
Click "Open settings" → modal with backdrop
Escape or Cancel → closes without saving
Save → closes and returns "save" as the dialog result
```

`<form method="dialog">` closes the dialog on submit and exposes the button's `value` as `dialog.returnValue`. The `open` attribute alone opens it non-modally and does not add a backdrop.

---

# 4. template and slot

`template` holds markup that is parsed but inert — not rendered, not loaded. It is the standard way to clone repeatable UI. `slot` defines a placeholder inside a web component's shadow DOM.

```html
<template id="row-template">
  <li class="row">
    <span class="name"></span>
    <span class="score"></span>
  </li>
</template>

<ul id="results"></ul>

<script>
  const tpl = document.getElementById("row-template");
  const list = document.getElementById("results");
  const row = tpl.content.cloneNode(true);
  row.querySelector(".name").textContent = "Kakade";
  row.querySelector(".score").textContent = "92";
  list.append(row);
</script>
```

Output:

```text
• Kakade 92
```

A `template`'s images and scripts do not load until the content is cloned into the document. `slot` is only meaningful inside shadow DOM:

```html
<template id="card">
  <div class="card">
    <h2><slot name="title">Default title</slot></h2>
    <slot>Default body</slot>
  </div>
</template>
```

---

# 5. canvas and Inline SVG

`canvas` is a bitmap drawing surface controlled by JavaScript. Inline `svg` is vector markup that scales, is styleable with CSS, and is accessible as DOM. Choose canvas for performance-heavy rendering; SVG for icons, charts, and diagrams.

```html
<canvas id="chart" width="320" height="120" aria-label="Kakade's study hours"></canvas>

<svg width="120" height="40" role="img" aria-label="Progress: 80 percent">
  <rect width="120" height="12" rx="6" fill="#e2e8f0" />
  <rect width="96" height="12" rx="6" fill="#0f172a" />
</svg>
```

Output:

```text
Canvas: blank until JS draws on its 2D context
SVG: two rounded bars visible immediately, 80% filled
```

Interview trap: canvas pixels are not in the accessibility tree. Add `aria-label`, a text alternative, or a data table if the graphic carries information. SVG shapes are DOM nodes and can be inspected, styled, and focused.

---

# 6. noscript and script

`script` loads or embeds JavaScript. `noscript` shows fallback content when scripting is disabled. Placement and attributes (`defer`, `async`, `type="module"`) decide when the code executes.

```html
<noscript>
  <p>Kakade's notes need JavaScript for search. All articles are still readable.</p>
</noscript>

<script src="/js/app.js" defer></script>
<script type="module" src="/js/search.js"></script>
```

Output:

```text
JS enabled: scripts run; noscript is ignored
JS disabled: the fallback paragraph appears
```

`defer` preserves order and runs after parsing; `async` runs as soon as downloaded; modules are deferred by default. Details are covered in Scripts, Styles and Loading.

---

# 7. The popover Attribute

`popover` turns an element into a lightweight top-layer popup toggled by `popovertarget`. It is good for menus, tooltips, and non-modal panels without custom JavaScript.

```html
<button popovertarget="menu">Kakade's menu</button>

<div id="menu" popover>
  <a href="/notes">Notes</a>
  <a href="/projects">Projects</a>
</div>
```

Output:

```text
Clicking the button opens the popover above other content
Clicking outside or pressing Escape closes it
```

`popover="manual"` prevents light-dismiss and auto-closing other popovers. Popovers are not modal dialogs — use `dialog` when you need focus trapping and a backdrop.

---

# Real-World Uses

A FAQ page combines native details, a dialog for settings, an SVG icon, and a deferred script.

```html
<section aria-labelledby="faq">
  <h2 id="faq">FAQ</h2>

  <details>
    <summary>Is HTML a programming language?</summary>
    <p>No — it is a markup language.</p>
  </details>

  <details>
    <summary>Do I need JavaScript for accordions?</summary>
    <p>No — details and summary handle it.</p>
  </details>
</section>

<dialog id="help">
  <form method="dialog">
    <h2>Need help?</h2>
    <p>Email Kakade at kakade@example.com.</p>
    <button>Close</button>
  </form>
</dialog>

<script src="/js/help.js" defer></script>
```

Output:

```text
Two native accordions, a modal help dialog, and deferred JS.
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **div and span** | Generic block/inline containers |
| **details and summary** | Native accordion; summary first child |
| **dialog** | Native modal; `showModal()` + `method="dialog"` |
| **template and slot** | Inert reusable markup; shadow DOM placeholder |
| **canvas** | Bitmap surface drawn by JS |
| **Inline SVG** | Vector DOM, scalable and accessible |
| **noscript** | Fallback when JS is off |
| **script** | Loads JS; behavior depends on attributes |
| **popover** | Lightweight top-layer popup, no modal |

## The Mental Model

```text
grouping / interactive / embedded
  │
  ├── div / span → layout only
  ├── details/summary → native accordion
  ├── dialog → modal + focus trap
  ├── template → inert markup
  ├── canvas → pixels (JS)
  ├── svg → vector DOM
  ├── noscript → no-JS fallback
  ├── script → behavior
  └── popover → non-modal popup
```

### The 3 things to remember

**1. Native before custom:**

```html
<details>
  <summary>What is the box model?</summary>
  <p>Content, padding, border, margin.</p>
</details>
```

**2. Modal needs dialog:**

```html
<dialog id="settings">
  <form method="dialog"><button>Close</button></form>
</dialog>
```

**3. Graphics need alternatives:**

```html
<svg role="img" aria-label="Progress: 80 percent">...</svg>
```

> **HTML has native widgets for accordions, modals, popups, and templates — reach for them before writing JavaScript.**
