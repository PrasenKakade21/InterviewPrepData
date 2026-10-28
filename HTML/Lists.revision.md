# Lists — Revision Notes (Beginner)
> One-line mental model: ul for sets, ol for sequences, dl for pairs — and nesting always inside an li.

## Core recall
- `ul` = unordered set (features, nav); `ol` = ordered sequence (steps, rankings).
- `li` must be a direct child of `ul`/`ol`; other elements go inside the `li`.
- `ol` attributes: `start` (first number), `reversed` (countdown), `type` (1, A, a, I, i).
- Nested lists go inside an `li`, never directly under the outer list.
- `dl` = description list; `dt` term, `dd` description; multiple `dd`s allowed per `dt`.
- `menu` is a list of commands and behaves like `ul`; rarely needed.
- Keep real list markup for screen readers (they announce list and item count).
- Style with CSS (`list-style`, `display: flex`); if removing markers, add `role="list"` to preserve semantics.
- Use `dl` for metadata/FAQ pairs, not tables or fake layouts.

## Must-remember code
```html
<ul>
  <li>HTML</li>
  <li>CSS</li>
  <li>JavaScript</li>
</ul>
```
```html
<ol start="4">
  <li>Build the layout</li>
  <li>Add styles</li>
  <li>Ship it</li>
</ol>
```
```html
<ul>
  <li>
    Frontend
    <ul>
      <li>HTML</li>
      <li>CSS</li>
    </ul>
  </li>
  <li>Backend</li>
</ul>
```
```html
<dl>
  <dt>Status</dt>
  <dd>Stable</dd>
  <dt>Version</dt>
  <dd>1.4.0</dd>
</dl>
```

## Interview traps
- Divs or anchors directly inside `ul`/`ol` instead of `li`.
- Nesting a list directly under another list instead of inside an `li`.
- Using `dl` as a layout or comparison table.
- Removing bullets without preserving list semantics (`role="list"`).
- Choosing `ol` for a set where order is meaningless (or `ul` for steps).

## 3 things to remember
1. `ul` set, `ol` sequence, `dl` pairs.
2. Only `li` directly inside `ul`/`ol`.
3. Nest inside an `li`; style with CSS, keep semantics.
