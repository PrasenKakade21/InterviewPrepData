# Layout Properties

Layout properties decide how boxes are generated, positioned, floated, stacked, and clipped. This reference groups the core layout properties with syntax, common values, and gotchas.

```text
layout property groups
    ├── display → box type / formatting context
    ├── position + inset → placement
    ├── float/clear → legacy text wrapping
    ├── overflow → clipping/scrolling
    ├── z-index/isolation → stacking
    └── columns → multi-column flow
```

---

# display

Sets the element's outer box type and inner layout mode.

```css
display: block | inline | inline-block | flex | inline-flex | grid | inline-grid | contents | none;
```

```css
.page { display: grid; }
.tag { display: inline-block; }
.sr-only { display: none; }
```

Gotchas: `none` removes the box and accessibility; `contents` removes the wrapper box but keeps children (can harm semantics).

---

# position

Places an element in normal flow or relative to a reference box.

```css
position: static | relative | absolute | fixed | sticky;
```

```css
.card { position: relative; }
.badge {
  position: absolute;
  inset: 8px 8px auto auto;
}
.header { position: sticky; top: 0; }
```

Gotchas: `absolute` uses the nearest positioned ancestor; `sticky` needs a threshold and scroll room.

---

# top, right, bottom, left and inset

Offsets for positioned elements. `inset` is the shorthand; logical `inset-block`/`inset-inline` adapt to writing direction.

```css
inset: 0;                 /* all sides */
inset: 1rem auto auto 0;  /* top right bottom left */
inset-block: 1rem;
inset-inline: 2rem;
```

```css
.overlay {
  position: fixed;
  inset: 0;
}
```

Gotchas: offsets do nothing on `static`; over-constrained absolute boxes resolve per direction rules.

---

# float and clear

Float pulls an element to a side and lets inline content wrap. `clear` prevents wrapping past a float.

```css
.figure {
  float: right;
  width: 240px;
  margin: 0 0 1rem 1rem;
}

.clear {
  clear: both;
}
```

Gotchas: floats are legacy for layout; use flex/grid. A parent with only floated children collapses unless it establishes a BFC (`display: flow-root`).

---

# overflow and overflow-*

Controls content that exceeds the box. Axis longhands: `overflow-x`, `overflow-y`.

```css
overflow: visible | hidden | clip | scroll | auto;
```

```css
.card {
  max-height: 240px;
  overflow: auto;
}

.row {
  overflow-x: auto;
  overflow-y: hidden;
}
```

Gotchas: non-visible overflow creates a BFC and can clip focus rings; one axis set to non-visible may force the other.

---

# z-index and isolation

`z-index` orders positioned/flex/grid items within a stacking context. `isolation: isolate` creates a new context without other side effects.

```css
.modal {
  position: fixed;
  z-index: 100;
}

.card {
  isolation: isolate;
}
```

Gotchas: z-index has no effect on `static` elements; numbers only compare within the same context. `isolation` is the clean way to scope blend modes/z-index.

---

# columns and column-*

Multi-column layout flows content into columns.

```css
columns: 3 240px;
column-gap: 2rem;
column-rule: 1px solid #e2e8f0;
```

```css
.article {
  columns: 3 240px;
  column-gap: 2rem;
}
```

Gotchas: column balancing is automatic; `break-inside: avoid` prevents awkward splits. Multi-column is for text, not general layout.

---

# Real-World Uses

A page shell, sticky header, clipped card, and isolated stacking.

```css
.shell {
  display: grid;
  grid-template-columns: 240px 1fr;
  min-height: 100dvh;
}

.site-header {
  position: sticky;
  top: 0;
  z-index: 20;
}

.card {
  position: relative;
  overflow: hidden;
  isolation: isolate;
}

.card__badge {
  position: absolute;
  inset-block-start: 8px;
  inset-inline-end: 8px;
}
```

---

# Quick Revision

| Property | Values / purpose |
| -------- | ---------------- |
| **display** | block, inline, inline-block, flex, grid, contents, none |
| **position** | static, relative, absolute, fixed, sticky |
| **inset** | top/right/bottom/left shorthand |
| **float/clear** | Legacy wrapping and clearing |
| **overflow** | visible, hidden, clip, scroll, auto |
| **z-index** | Stacking order within a context |
| **isolation** | Create a stacking context |
| **columns** | Multi-column text flow |

## The Mental Model

```text
layout
  ├── box type → display
  ├── placement → position + inset
  ├── clipping → overflow
  ├── stacking → z-index + isolation
  └── text flow → columns / float
```

### The 3 things to remember

**1. Position needs a reference:**

```css
.card { position: relative; }
.badge { position: absolute; inset: 8px; }
```

**2. Overflow clips:**

```css
.card { overflow: hidden; }
```

**3. z-index is contextual:**

```css
.modal { position: fixed; z-index: 100; }
```

> **Layout properties control box generation, placement, clipping, and stacking — always know the reference box and context.**
