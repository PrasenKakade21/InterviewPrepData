# Global Attributes Reference

Global attributes can be used on **any** HTML element. They control identity, styling hooks, language, direction, visibility, focus, editing, dragging, custom data, and a handful of platform hints.

The core idea:

```text
global attribute
    ├── identity → id, class
    ├── presentation → style, title, hidden
    ├── language → lang, dir, translate, spellcheck
    ├── focus → tabindex, autofocus, accesskey, inert
    ├── editing → contenteditable, draggable
    ├── data → data-*, item*
    ├── platform → popover, slot, part, is, nonce, inputmode
    └── events → on* handlers
```

Only `id` must be unique.

---

# Identity and Styling

## `id`

Unique identifier. Used for fragment links, `label for`, `aria-labelledby`, and `getElementById`.

```html
<h1 id="page-title">Kakade</h1>
<a href="#page-title">Back to top</a>
```

Output:

```text
Clicking "Back to top" scrolls to the heading.
```

Gotchas: duplicates break references; values are case-sensitive; no spaces (use hyphens).

## `class`

Reusable space-separated group names for CSS and JS.

```html
<p class="card card--featured">Kakade</p>
```

Output:

```text
Both .card and .card--featured apply.
```

Gotchas: class names are case-sensitive; avoid appearance-based names.

## `style`

Inline CSS declarations for one element.

```html
<p style="color: #0f172a; font-weight: 600">Kakade</p>
```

Output:

```text
Dark, semibold text.
```

Gotchas: high specificity, no reuse, mixes presentation with markup; reserve for dynamic values.

## `title`

Advisory tooltip text.

```html
<abbr title="HyperText Markup Language">HTML</abbr>
```

Output:

```text
Tooltip on hover: HyperText Markup Language
```

Gotchas: unreliable on touch, inconsistent in screen readers; never the only label.

## `hidden`

Removes the element from rendering and the accessibility tree.

```html
<p hidden>Draft not ready.</p>
```

Output:

```text
Nothing rendered.
```

Gotchas: author CSS can override it (`display: block`); use `hidden="until-found"` for find-in-page reveal.

---

# Language and Text

## `lang`

Language of the element's content; inherited.

```html
<p lang="hi">नमस्ते काकड़े</p>
```

Output:

```text
Correct pronunciation and spellcheck behavior.
```

Gotchas: set on `<html>` globally; override inline for language switches.

## `dir`

Text direction: `ltr`, `rtl`, or `auto`.

```html
<p dir="rtl" lang="ar">مرحبا</p>
```

Output:

```text
Right-to-left text layout.
```

Gotchas: `auto` infers from the first strong character; useful for user-generated content.

## `translate`

Hints whether content should be translated (`yes`/`no`).

```html
<p translate="no">Kakade.dev</p>
```

Output:

```text
Brand name kept untranslated by translation tools.
```

Gotchas: a hint, not a guarantee; some tools ignore it.

## `spellcheck`

Enables/disables spellchecking on editable content.

```html
<textarea spellcheck="true">Kakade is revising HTML.</textarea>
```

Output:

```text
Misspellings underlined in the textarea.
```

Gotchas: meaningful mainly on inputs, textareas, and `contenteditable`.

---

# Focus and Interaction

## `tabindex`

Controls keyboard focus order.

| Value | Behavior |
| ----- | -------- |
| `0` | Natural tab order |
| `-1` | Programmatic focus only |
| positive | Explicit order (avoid) |

```html
<div tabindex="0" role="button">Kakade's custom button</div>
```

Output:

```text
Reachable by Tab, but still needs keyboard handlers.
```

Gotchas: prefer native controls; positive values override natural order and confuse users.

## `autofocus`

Focuses one element on page load.

```html
<input autofocus aria-label="Search notes" />
```

Output:

```text
Search field focused on load.
```

Gotchas: only one per document; can disorient keyboard/screen-reader users.

## `accesskey`

Defines a keyboard shortcut.

```html
<button accesskey="s">Save</button>
```

Output:

```text
Platform-dependent shortcut (often Alt+Shift+S).
```

Gotchas: conflicts with browser/OS shortcuts; discoverability is poor; use sparingly.

## `inert`

Makes a subtree non-interactive and hidden from assistive tech.

```html
<div inert>
  <button>Unavailable</button>
</div>
```

Output:

```text
Button not clickable, not focusable, not announced.
```

Gotchas: still occupies layout; ideal for background content behind a modal.

## `contenteditable`

Makes an element editable.

```html
<div contenteditable="true" role="textbox" aria-label="Kakade's note">Edit me.</div>
```

Output:

```text
Text editable in place.
```

Gotchas: not a form control (no name/validation/submission); sanitize pasted HTML.

## `draggable`

Enables native drag behavior.

```html
<div draggable="true" data-id="1">Kakade's task</div>
```

Output:

```text
Element can be dragged; drop handling needs JS.
```

Gotchas: no keyboard equivalent by default; provide buttons/menus for reordering.

---

# Custom Data and Microdata

## `data-*`

Custom data attributes, readable via `dataset`.

```html
<button data-user-id="42" data-plan="pro">Kakade</button>
<script>
  console.log(document.querySelector("button").dataset.userId); // "42"
</script>
```

Output:

```text
42
```

Gotchas: kebab-case becomes camelCase in `dataset`; values are strings; never store secrets.

## `itemid`, `itemprop`, `itemref`, `itemscope`, `itemtype`

Microdata vocabulary for structured data in the body.

```html
<div itemscope itemtype="https://schema.org/Person">
  <span itemprop="name">Kakade</span>
</div>
```

Output:

```text
Search engines can read the Person/name pair.
```

Gotchas: JSON-LD is generally preferred; microdata must match visible content.

---

# Platform Hints

## `popover`

Turns an element into a top-layer popup.

```html
<button popovertarget="menu">Menu</button>
<div id="menu" popover>...</div>
```

Output:

```text
Popup with light dismiss and Escape close.
```

Gotchas: non-modal; use `dialog` for focus-trapped modals; `popover="manual"` disables light dismiss.

## `slot`

Assigns a child to a named shadow-DOM slot.

```html
<span slot="title">Kakade</span>
```

Output:

```text
Child projected into <slot name="title">.
```

Gotchas: only meaningful inside a shadow host.

## `part`

Exposes a shadow-DOM element for external styling via `::part()`.

```html
<button part="button">Kakade</button>
```

Output:

```text
Host page can style ::part(button).
```

Gotchas: requires shadow DOM; keep part names semantic.

## `is`

Upgrades a built-in element to a customized built-in element.

```html
<button is="kakade-button">Save</button>
```

Output:

```text
Button keeps native semantics plus custom behavior.
```

Gotchas: browser support is limited; autonomous custom elements are more common.

## `nonce`

CSP nonce for inline scripts/styles.

```html
<script nonce="r4nd0m">console.log("Kakade")</script>
```

Output:

```text
Script allowed only if the CSP includes the same nonce.
```

Gotchas: the nonce must be unpredictable per response; don't reuse it.

## `inputmode`, `enterkeyhint`, `autocapitalize`

Mobile keyboard hints.

```html
<input inputmode="numeric" enterkeyhint="search" autocapitalize="none" />
```

Output:

```text
Numeric keypad, "search" enter key, no auto-capitalization.
```

Gotchas: hints only — always validate the actual value.

---

# Event Handler Attributes

`onclick`, `oninput`, `onsubmit`, etc. attach inline JavaScript.

```html
<button onclick="alert('Kakade')">Click</button>
```

Output:

```text
Alert on click.
```

Gotchas: mixes behavior with markup, blocks CSP, hard to test; use `addEventListener` in JS instead.

---

# Quick Reference

| Attribute | Category | Purpose |
| --------- | -------- | ------- |
| `id` | Identity | Unique identifier |
| `class` | Identity | Reusable group |
| `style` | Styling | Inline CSS |
| `title` | Advisory | Tooltip |
| `hidden` | Visibility | Remove from render |
| `lang` / `dir` | Language | Language / direction |
| `translate` / `spellcheck` | Language | Translation / spellcheck hints |
| `tabindex` | Focus | Focus order |
| `autofocus` | Focus | Initial focus |
| `accesskey` | Focus | Keyboard shortcut |
| `inert` | Focus | Freeze subtree |
| `contenteditable` | Editing | Editable region |
| `draggable` | Interaction | Drag support |
| `data-*` | Data | Custom data |
| `item*` | Data | Microdata |
| `popover` | Platform | Native popup |
| `slot` / `part` | Platform | Shadow DOM |
| `is` | Platform | Customized built-in |
| `nonce` | Security | CSP nonce |
| `inputmode` etc. | Mobile | Keyboard hints |
| `on*` | Events | Inline handlers |

> **Global attributes are identity, language, focus, data, and platform hints — only `id` is unique, and boolean attributes are true by presence.**
