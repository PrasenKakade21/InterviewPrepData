# Selectors — Revision Notes (Beginner)
> One-line mental model: selectors match DOM nodes; keep them flat, specific enough, and easy to override.

## Core recall
- Specificity tuple: (ids, classes/attributes/pseudo-classes, elements/pseudo-elements).
- Type `p` = 0,0,1; class `.card` = 0,1,0; id `#hero` = 1,0,0; `*` = 0,0,0.
- Attribute operators: `[attr]`, `=`, `~=`, `|=`, `^=`, `$=`, `*=`.
- Combinators: space = descendant, `>` = child, `+` = next sibling, `~` = later siblings.
- Comma groups selectors; each is evaluated independently.
- Pseudo-classes (`:hover`, `:nth-child`) select state/position; pseudo-elements (`::before`) select generated parts.
- `:is()` takes the highest specificity in its list; `:where()` contributes zero specificity; `:not()` excludes.
- `::before`/`::after` require `content`, are not in the DOM, and are unreliable for essential content.
- Deep descendant chains over-match and complicate specificity; prefer flat classes.
- `:nth-child` counts all siblings; `:nth-of-type` counts same-type siblings.

## Must-remember code
```css
.card { padding: 1rem; }
.card > p { margin-top: 0; }
h2 + p { margin-top: 0.25rem; }
a[href$=".pdf"]::after { content: " (PDF)"; }
```
```css
.card::before {
  content: "";
  display: block;
  height: 3px;
  background: #0f172a;
}
.card :where(h2, p) { margin: 0; }
```
```css
:is(h1, h2, h3) { line-height: 1.2; }
:not(.card__title) { color: #475569; }
```

## Interview traps
- Using ids for styling and then fighting specificity.
- Confusing descendant and child combinators.
- `:nth-child` vs `:nth-of-type` with mixed siblings.
- Expecting `:where()` to add specificity (it adds none).
- Putting meaningful text in `::before`/`::after`.
- Forgetting `content` on pseudo-elements.
- Overly deep selectors that match unintended nodes.

## 3 things to remember
1. Flat classes + low specificity win long term.
2. Combinators express structure; use `>` when you mean direct child.
3. `:where()` for defaults, `:is()` for grouping, `:not()` for exclusions.
