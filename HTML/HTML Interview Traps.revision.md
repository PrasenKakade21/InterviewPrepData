# HTML Interview Traps — Revision Notes (Intermediate)
> One-line mental model: trap questions come in pairs — answer with the meaning or behavior that separates them.

## Core recall
- `div` generic; `section` named thematic region (with heading); `article` self-contained and syndicatable.
- `strong`/`em` carry meaning; `b`/`i` are visual only.
- `id` unique (anchors, labels, ARIA); `class` reusable (styling/grouping); duplicates break references.
- `alt` = accessible name, required; `title` = advisory tooltip, not a substitute.
- GET = retrieval in URL, bookmarkable; POST = change in body; neither encrypts — use HTTPS.
- `label for` must match the input's `id`, not its `name`.
- Void elements (`img`, `br`, `hr`, `input`, `meta`, `link`, `source`, `track`, `wbr`, `area`, `base`, `col`, `embed`) have no children/closing tag.
- Block vs inline is default display, changeable with CSS.
- Boolean attributes (`required`, `disabled`, `checked`, `hidden`, `readonly`, `autofocus`) are true by presence.
- Missing doctype → quirks mode with legacy box sizing.
- Tables are for data; CSS grid/flexbox for layout.

## Must-remember code
```html
<article>
  <h2>Kakade's Post</h2>
  <p><strong>Warning:</strong> <em>today</em>.</p>
</article>
```
```html
<h1 id="page-title" class="title title--xl">Kakade</h1>
```
```html
<label for="email">Email</label>
<input id="email" name="email" type="email" required />
```
```html
<img src="/kakade.png" alt="Kakade presenting HTML notes" title="DevConf 2026" />
```
```html
<!DOCTYPE html>
<html lang="en">...</html>
```

## Interview traps
- Choosing `div` for a named region or `section` for pure layout.
- Using `b`/`i` where meaning matters.
- Duplicate ids — fragment/label/ARIA breakage.
- Treating `title` as `alt`.
- Saying POST is secure.
- `label for` pointing at a `name`.
- Closing void elements or giving them children.
- Omitting the doctype and blaming CSS for quirks-mode sizing.
- Using tables for layout.

## 3 things to remember
1. Answer pairs by meaning: generic/named/standalone, semantic/visual.
2. `id` unique, `for` = `id`, boolean = presence.
3. Doctype + HTTPS + CSS layout are non-negotiables.
