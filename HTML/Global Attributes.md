# Global Attributes

Global attributes work on **every** HTML element. They are the knobs for identity, styling hooks, language, visibility, focus, editability, and custom data. Interviewers love `id` vs `class`, `hidden` vs `display:none`, and what makes an attribute boolean.

The core idea:

```text
global attribute
    ├── identity       → id
    ├── grouping       → class
    ├── inline style   → style
    ├── advisory text  → title
    ├── language       → lang, dir
    ├── visibility     → hidden
    ├── focus order    → tabindex
    ├── editing        → contenteditable
    └── custom data    → data-*
```

Only `id` must be unique; everything else is reusable.

---

# 1. id and class

`id` uniquely identifies one element and is used for fragment links, labels, and `getElementById`. `class` groups many elements for CSS and JavaScript.

```html
<h1 id="page-title" class="heading heading--xl">Kakade's Notes</h1>
<p class="text text--muted">Updated today</p>
```

Output:

```text
#page-title → one element; URL #page-title scrolls here
.heading    → shared styling for any element with that class
```

| Attribute | Uniqueness | Access | Typical use |
| --------- | ---------- | ------ | ----------- |
| `id` | Must be unique | `#id`, `getElementById` | Anchors, labels |
| `class` | Reusable | `.class`, `querySelectorAll` | Styling, grouping |

Bad:

```html
<p id="note">First</p>
<p id="note">Second</p>
```

Good:

```html
<p class="note">First</p>
<p class="note">Second</p>
```

Duplicate ids break fragment links, `label for`, and `aria-labelledby`.

---

# 2. style and title

`style` applies inline CSS to one element and wins over most stylesheet rules — which is why it is hard to maintain. `title` provides advisory text shown as a tooltip and sometimes read by screen readers.

```html
<p style="color: #0f172a; font-weight: 600">Kakade</p>
<p title="HyperText Markup Language">HTML</p>
```

Output:

```text
Kakade (dark, semibold)
HTML (tooltip on hover: HyperText Markup Language)
```

Interview trap: `title` is not a replacement for `alt` or a label. It is unreliable on touch devices and not consistently announced. Use visible text where meaning matters.

---

# 3. lang and dir

`lang` declares the language of the element's content; `dir` sets text direction (`ltr`, `rtl`, `auto`). Both inherit, so set them at the right scope.

```html
<html lang="en">
  <body>
    <p lang="hi" dir="ltr">नमस्ते, मैं काकड़े हूँ।</p>
    <p dir="rtl" lang="ar">مرحبا</p>
  </body>
</html>
```

Output:

```text
Hindi text: correct pronunciation hints, left-to-right
Arabic text: right-to-left layout
```

`dir="auto"` lets the browser infer direction from the first strong character — useful for user-generated content.

---

# 4. hidden and tabindex

`hidden` removes an element from rendering (and from the accessibility tree). `tabindex` changes keyboard focusability: `0` adds natural order, `-1` makes it programmatically focusable, positive values force an order and should be avoided.

```html
<p hidden>This draft is not ready.</p>

<div tabindex="0" role="button">Kakade's custom button</div>
<div tabindex="-1" id="panel">Programmatically focusable panel</div>
```

Output:

```text
Draft paragraph: not rendered
tabindex="0": reachable by Tab
tabindex="-1": focusable via element.focus(), not by Tab
```

`hidden` is overridden by `display: block` in CSS, which surprises people. If you need hidden-but-focusable or animated hiding, use a utility class with `display:none`/`visibility` and manage focus deliberately.

---

# 5. contenteditable and draggable

`contenteditable` makes an element editable in the browser. `draggable` marks an element as draggable for the HTML drag-and-drop API. Both are powerful and easy to misuse.

```html
<div contenteditable="true" role="textbox" aria-label="Kakade's note">
  Edit this note.
</div>

<div draggable="true" id="card">Drag Kakade's card</div>
```

Output:

```text
Note text becomes editable on click.
Card can be dragged; drop targets still need JavaScript handlers.
```

Interview trap: `contenteditable` is not a form control — it has no `name`, no native validation, and no automatic submission. Sync its content to a hidden input or JS state.

---

# 6. data-*, accesskey, autofocus and inert

`data-*` stores custom data on an element, readable via `dataset`. `accesskey` defines a keyboard shortcut. `autofocus` focuses one element on load. `inert` makes a subtree non-interactive and hidden from assistive tech.

```html
<button data-user-id="42" data-role="admin">Kakade</button>

<input autofocus aria-label="Search notes" />

<div inert>
  <button>Unavailable while the modal is open</button>
</div>
```

Output:

```text
button.dataset.userId === "42"
Search input focused on load
inert subtree: not clickable, not focusable, hidden from screen readers
```

`autofocus` on more than one element is invalid, and focusing on load can disorient keyboard/screen-reader users — use it sparingly (search pages are the classic exception).

---

# 7. spellcheck, translate and Boolean Attributes

`spellcheck` toggles spellchecking, `translate` hints whether content should be translated. Boolean attributes are true when present and false when absent — their value is irrelevant.

```html
<textarea spellcheck="true">Kakade is revising HTML.</textarea>
<p translate="no">Kakade.dev</p>

<input required />
<input required="required" />
<input required="false" />
```

Output:

```text
Spellchecking on for the textarea.
Brand name not translated.
All three inputs are required — "false" still means true.
```

Interview trap: `required="false"`, `disabled="false"`, and `checked="false"` all enable the attribute. Remove the attribute to disable it.

---

# Real-World Uses

A profile card uses id, class, lang, data attributes, and a boolean `hidden` flag without any custom framework.

```html
<article
  id="profile-kakade"
  class="card card--profile"
  lang="en"
  data-user-id="42"
  data-plan="pro"
>
  <h2 class="card__title" title="Frontend developer">Kakade</h2>
  <p class="card__meta" translate="no">Kakade.dev</p>

  <button class="card__button" data-action="follow">Follow</button>
  <p class="card__note" hidden>Already following</p>
</article>
```

Output:

```text
Card with stable id, reusable classes, machine-readable dataset,
brand name excluded from translation, and a hidden note.
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **id** | Unique; anchors, labels, `getElementById` |
| **class** | Reusable styling/grouping hook |
| **style** | Inline CSS; high specificity, hard to maintain |
| **title** | Advisory tooltip, not a label/alt |
| **lang and dir** | Language and text direction; inherited |
| **hidden** | Removes from render and a11y tree |
| **tabindex** | `0` natural, `-1` programmatic, positive avoid |
| **contenteditable** | Editable, but not a form control |
| **draggable** | Enables drag API; drop needs JS |
| **data-*** | Custom data via `dataset` |
| **accesskey, autofocus, inert** | Shortcut, initial focus, disable subtree |
| **spellcheck, translate** | Spellcheck and translation hints |
| **Boolean attributes** | Presence = true; value ignored |

## The Mental Model

```text
global attributes
  ├── identity: id (unique), class (reusable)
  ├── presentation: style, title
  ├── language: lang, dir
  ├── visibility: hidden
  ├── focus: tabindex, autofocus, inert
  ├── interaction: contenteditable, draggable, accesskey
  └── data: data-* (+ spellcheck, translate hints)
```

### The 3 things to remember

**1. id unique, class reusable:**

```html
<h1 id="page-title" class="heading">Kakade's Notes</h1>
```

**2. Boolean means presence:**

```html
<input required />
```

**3. Custom data uses data-*:**

```html
<button data-user-id="42">Kakade</button>
```

> **Global attributes are identity, language, focus, and data — only `id` must be unique, and boolean attributes are true by presence.**
