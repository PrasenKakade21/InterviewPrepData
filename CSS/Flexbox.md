# Flexbox

Flexbox lays out items along one axis at a time. It is the default choice for rows, nav bars, toolbars, card internals, and centering. Interviewers test the difference between main/cross axis, `justify-content` vs `align-items`, and how `flex-grow`/`shrink`/`basis` interact.

The core idea:

```text
display: flex
    ↓
main axis (flex-direction)
    ↓
items grow/shrink/align on the cross axis
```

Flexbox is one-dimensional: one row **or** one column, with wrapping creating new lines.

---

# 1. Flex Container and Items

Set `display: flex` on the parent. Its direct children become flex items, laid out along the main axis. Inline-level flex containers use `inline-flex`.

```css
.toolbar {
  display: flex;
  align-items: center;
  gap: 0.75rem;
}
```

```html
<div class="toolbar">
  <button>Save</button>
  <button>Cancel</button>
  <span>Kakade</span>
</div>
```

Output:

```text
Buttons and label sit in a row, vertically centered, 0.75rem apart.
```

Only direct children are flex items. Text nodes become anonymous flex items; nested elements keep their own layout.

---

# 2. flex-direction and flex-wrap

`flex-direction` sets the main axis: `row` (default), `row-reverse`, `column`, `column-reverse`. `flex-wrap` allows items to move to new lines.

```css
.stack {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.wrap {
  display: flex;
  flex-wrap: wrap;
  gap: 1rem;
}
```

Output:

```text
.stack: vertical list
.wrap: row that wraps when space runs out
```

Interview trap: with `flex-wrap: wrap`, `align-content` controls the spacing between lines, while `align-items` controls items within each line.

---

# 3. justify-content

`justify-content` distributes items along the **main axis**: start, end, center, space-between, space-around, space-evenly.

```css
.nav {
  display: flex;
  justify-content: space-between;
}

.center {
  display: flex;
  justify-content: center;
}
```

Output:

```text
.nav: first and last items pushed to the edges
.center: items clustered in the middle
```

| Value | Distribution |
| ----- | ------------ |
| `flex-start` | Packed at start |
| `flex-end` | Packed at end |
| `center` | Centered |
| `space-between` | First/last at edges, equal gaps |
| `space-around` | Equal space around each item |
| `space-evenly` | Equal space between and at edges |

---

# 4. align-items and align-content

`align-items` aligns items on the **cross axis**. `align-content` distributes wrapped lines on the cross axis.

```css
.card-row {
  display: flex;
  align-items: stretch;   /* equal height cards */
}

.wrapped {
  display: flex;
  flex-wrap: wrap;
  align-content: space-between;
}
```

Output:

```text
Cards stretch to the tallest card;
wrapped lines spread across the container height.
```

Values: `stretch` (default), `flex-start`, `flex-end`, `center`, `baseline`.

Interview trap: `align-content` has no effect on a single-line flex container.

---

# 5. flex-grow, flex-shrink and flex-basis

These control how items use free space (`grow`), give it up (`shrink`), and what their starting size is (`basis`). The `flex` shorthand is `grow shrink basis`.

```css
.sidebar {
  flex: 0 0 240px;   /* fixed width */
}

.content {
  flex: 1 1 auto;    /* take remaining space */
}

.fixed {
  flex: none;        /* 0 0 auto */
}
```

Output:

```text
Sidebar stays 240px; content fills the rest; fixed keeps natural size.
```

Common shorthands:

| Shorthand | Meaning |
| --------- | ------- |
| `flex: 1` | `1 1 0%` — equal flexible items |
| `flex: auto` | `1 1 auto` — grow from content size |
| `flex: none` | `0 0 auto` — no grow/shrink |
| `flex: 0 0 240px` | Fixed 240px |

Interview trap: `flex: 1` uses `basis: 0`, so content size is ignored for distribution; `flex: auto` starts from content size.

---

# 6. gap and order

`gap` spaces items without margins. `order` changes visual order without changing DOM order.

```css
.nav {
  display: flex;
  gap: 0.5rem;
}

.nav__cta {
  order: 2;
  margin-inline-start: auto;
}
```

Output:

```text
Nav items spaced by gap; CTA moves to the end visually.
```

Interview trap: `order` changes visual order only — keyboard and screen-reader order stay in the DOM. Do not reorder meaningful content.

---

# 7. Flexbox Layouts and Gotchas

Flexbox handles most one-dimensional layouts: nav bars, media objects, card rows, centered content.

```css
/* Center anything */
.center {
  display: flex;
  align-items: center;
  justify-content: center;
  min-height: 100dvh;
}

/* Media object */
.media {
  display: flex;
  gap: 1rem;
  align-items: flex-start;
}

.media__body { flex: 1; min-width: 0; }
```

Output:

```text
Perfect centering; media object with text taking remaining width.
```

Gotchas:

- `min-width: auto` prevents flex items from shrinking below content; set `min-width: 0` for truncation.
- Margins do not collapse in flex containers.
- Percentage heights inside flex can be tricky; use `align-items: stretch` or explicit heights.
- Flexbox is not a grid — use it for one axis, not two-dimensional placement.

---

# Real-World Uses

A card header uses flex for title/meta alignment and an action pushed to the right.

```css
.card__header {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  padding: 1rem 1.25rem;
  border-bottom: 1px solid #e2e8f0;
}

.card__title {
  margin: 0;
  flex: 1;
  min-width: 0;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.card__action {
  flex: none;
}
```

Output:

```text
Title truncates with an ellipsis; action stays visible on the right.
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **Container and items** | Only direct children are flex items |
| **flex-direction** | Main axis: row/column (+reverse) |
| **justify-content** | Main-axis distribution |
| **align-items/align-content** | Cross-axis items / wrapped lines |
| **flex-wrap** | Allow multiple lines |
| **flex-grow/shrink/basis** | Grow, shrink, starting size |
| **gap** | Spacing without margins |
| **order** | Visual reorder only |
| **Layouts** | Nav, media object, centering |
| **Gotchas** | min-width:auto, no margin collapse |

## The Mental Model

```text
flex container
  ├── main axis → flex-direction → justify-content
  ├── cross axis → align-items (single line) / align-content (wrapped)
  └── items
        ├── flex: grow shrink basis
        ├── min-width: 0 for shrinking
        └── order = visual only
```

### The 3 things to remember

**1. Main vs cross:**

```css
display: flex;
justify-content: space-between; /* main */
align-items: center;            /* cross */
```

**2. Shorthand:**

```css
.content { flex: 1 1 auto; }
.sidebar { flex: 0 0 240px; }
```

**3. Allow shrinking:**

```css
.media__body { flex: 1; min-width: 0; }
```

> **Flexbox is one-dimensional: direction sets the axis, justify/align place items, and flex controls how they grow or shrink.**
