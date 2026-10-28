# Interactive and Scripting Tags

These tags create navigation, disclosure, modals, reusable markup, and script hooks. Most of them replace JavaScript widgets with native behavior.

The core idea:

```text
interaction
    ├── navigation → a
    ├── disclosure → details + summary
    ├── modal → dialog
    ├── reusable markup → template + slot
    └── behavior → script (+ noscript)
```

---

# `<a>`

A hyperlink. Requires `href` to be a real, focusable link.

| Attribute | Purpose |
| --------- | ------- |
| `href` | Destination URL |
| `target` | `_blank`, `_self`, frame name |
| `rel` | Relationship (`noopener`, `nofollow`, `sponsored`) |
| `download` | Save same-origin file |
| `hreflang` | Language of the destination |
| `type` | MIME type hint |

```html
<a href="/notes">Read Kakade's notes</a>
<a href="https://kakade.dev" target="_blank" rel="noopener noreferrer">Kakade.dev</a>
<a href="/files/resume.pdf" download>Download resume</a>
```

Output:

```text
Internal link, safe external link, and a download.
```

Gotchas: no `href` = not a link; use `button` for actions; descriptive link text; `mailto:`/`tel:` are also anchors.

---

# `<details>` and `<summary>`

A native disclosure widget. `summary` is the always-visible label and must be the first child.

```html
<details>
  <summary>What is the box model?</summary>
  <p>Content, padding, border, and margin.</p>
</details>

<details open>
  <summary>Native by default</summary>
  <p>No JavaScript required.</p>
</details>
```

Output:

```text
▸ What is the box model?   (expands on click)
▾ Native by default        (starts open)
```

Gotchas: only the first `summary` is the label; `open` toggles the initial state; keyboard support is built in.

---

# `<dialog>`

A native dialog for modals and non-modal panels.

```html
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

<script>
  document.getElementById("settings").showModal();
</script>
```

Output:

```text
Modal with backdrop; Escape closes; form method=dialog sets returnValue.
```

Gotchas: `open` attribute alone is non-modal; `showModal()` gives focus trapping and inert background; style `::backdrop` for the overlay.

---

# `<template>`

Holds inert markup that is parsed but not rendered or loaded.

```html
<template id="row">
  <li class="row"><span class="name"></span></li>
</template>

<ul id="list"></ul>

<script>
  const row = document.getElementById("row").content.cloneNode(true);
  row.querySelector(".name").textContent = "Kakade";
  document.getElementById("list").append(row);
</script>
```

Output:

```text
• Kakade
```

Gotchas: access `template.content` (DocumentFragment); clone before inserting; images/scripts inside do not load until cloned.

---

# `<slot>`

A placeholder inside shadow DOM where light-DOM children are projected.

```html
<template id="card">
  <div class="card">
    <h2><slot name="title">Default title</slot></h2>
    <slot>Default body</slot>
  </div>
</template>

<kakade-card>
  <span slot="title">Kakade</span>
  <p>Frontend learner</p>
</kakade-card>
```

Output:

```text
Card renders "Kakade" in the title slot and the paragraph in the default slot.
```

Gotchas: `slot` only works with shadow DOM; named slots require matching `slot="name"` on the child; fallback content shows when nothing is projected.

---

# The `popover` Attribute

Not a tag, but the native popup system: an element with `popover` is shown/hidden by a `popovertarget` button.

```html
<button popovertarget="menu">Kakade's menu</button>
<div id="menu" popover>
  <a href="/notes">Notes</a>
  <a href="/projects">Projects</a>
</div>
```

Output:

```text
Top-layer popup; closes on outside click or Escape.
```

Gotchas: `popover="manual"` disables light dismiss; use `dialog` for modal behavior; popovers are not focus traps.

---

# `<script>`

Embeds or loads JavaScript; placement and attributes control execution timing.

```html
<script src="/js/app.js" defer></script>
<script type="module" src="/js/search.js"></script>
<script>
  console.log("Inline Kakade script");
</script>
```

Output:

```text
Deferred app code, deferred module, and an inline script at parse point.
```

Gotchas: inline scripts cannot be deferred; modules are strict and scoped; `async` scripts run out of order.

---

# `<noscript>`

Fallback content when scripting is disabled.

```html
<noscript>
  <p>Search requires JavaScript. Browse the full index instead.</p>
</noscript>
```

Output:

```text
Visible only when JS is off or blocked.
```

Gotchas: do not hide essential content behind JS and rely on `noscript` to fix it; core content should be server-rendered HTML.

---

# Quick Reference

| Tag | Purpose |
| --- | ------- |
| `a` | Hyperlink |
| `details` / `summary` | Native disclosure |
| `dialog` | Native modal/non-modal dialog |
| `template` | Inert reusable markup |
| `slot` | Shadow DOM content projection |
| `popover` | Native top-layer popup (attribute) |
| `script` | JavaScript |
| `noscript` | No-JS fallback |

> **HTML has native navigation, disclosure, modals, templates, and popups — use the platform before writing a widget.**
