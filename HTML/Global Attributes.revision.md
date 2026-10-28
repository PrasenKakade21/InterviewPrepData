# Global Attributes — Revision Notes (Beginner)
> One-line mental model: global attributes cover identity, language, focus, visibility, and data — only `id` is unique, and booleans are true by presence.

## Core recall
- `id` is unique per document; used by fragment links, `label for`, ARIA references, `getElementById`.
- `class` is reusable for styling and JS grouping; duplicate ids break references.
- `style` is inline CSS with high specificity — avoid for production styling.
- `title` is advisory tooltip text, not a substitute for `alt` or a label.
- `lang` and `dir` (`ltr`/`rtl`/`auto`) inherit; set at the correct scope.
- `hidden` removes from render and accessibility tree but can be overridden by `display` CSS.
- `tabindex`: `0` natural order, `-1` programmatic focus, positive values avoid.
- `contenteditable` is editable but not a form control (no name, no validation, no submit).
- `draggable` enables the drag API; drop targets and data transfer need JS.
- `data-*` stores custom data, read via `element.dataset`.
- `inert` freezes a subtree (non-interactive, hidden from AT) without removing layout.
- `autofocus`, `accesskey` use sparingly; multiple autofocus elements are invalid.
- Boolean attributes (`required`, `disabled`, `checked`, `hidden`) are true by presence — value is ignored.

## Must-remember code
```html
<h1 id="page-title" class="heading heading--xl">Kakade's Notes</h1>
```
```html
<button data-user-id="42" data-role="admin">Kakade</button>
<script>
  document.querySelector("button").dataset.userId; // "42"
</script>
```
```html
<div inert>
  <button>Unavailable while the modal is open</button>
</div>
```
```html
<div tabindex="0" role="button">Kakade's custom button</div>
```
```html
<input required />
<!-- not: required="false" -->
```

## Interview traps
- Duplicate `id`s — anchors, labels, and ARIA references break.
- `required="false"` / `disabled="false"` — still enabled because present.
- Expecting `hidden` to beat author CSS.
- `contenteditable` treated as a submittable field.
- Positive `tabindex` breaking natural focus order.
- Multiple `autofocus` attributes.
- Using `title` as the only accessible name.

## 3 things to remember
1. `id` unique, `class` reusable.
2. Boolean attributes are true by presence.
3. `data-*` for custom data, `dataset` to read it.
