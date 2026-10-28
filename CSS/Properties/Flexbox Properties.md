# Flexbox Properties

Flexbox properties split into container properties (axis, distribution, wrapping) and item properties (grow, shrink, basis, order). This reference lists both with values and gotchas.

```text
flex container
  ├── flex-direction → main axis
  ├── flex-wrap → lines
  ├── justify-content → main-axis distribution
  ├── align-items/content → cross-axis alignment
  └── gap → spacing

flex item
  ├── flex → grow shrink basis
  ├── order
  ├── align-self
  └── min-width: 0 for shrinking
```

---

# display: flex / inline-flex

Creates a flex formatting context; direct children become items.

```css
display: flex | inline-flex;
```

```css
.toolbar {
  display: flex;
  align-items: center;
  gap: 0.75rem;
}
```

Gotchas: only direct children are items; anonymous text becomes an item.

---

# flex-direction and flex-wrap, flex-flow

`flex-direction` sets the main axis; `flex-wrap` allows new lines; `flex-flow` is the shorthand.

```css
flex-direction: row | row-reverse | column | column-reverse;
flex-wrap: nowrap | wrap | wrap-reverse;
flex-flow: row wrap;
```

```css
.stack {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}
```

---

# justify-content

Distributes items along the main axis.

```css
justify-content: flex-start | flex-end | center | space-between | space-around | space-evenly;
```

```css
.nav {
  display: flex;
  justify-content: space-between;
}
```

Gotchas: it does not align the cross axis; use `align-items` for that.

---

# align-items, align-content, align-self

Cross-axis alignment. `align-items` applies to items in a line; `align-content` to wrapped lines; `align-self` overrides one item.

```css
align-items: stretch | flex-start | flex-end | center | baseline;
align-content: stretch | flex-start | flex-end | center | space-between | space-around;
align-self: auto | flex-start | flex-end | center | stretch;
```

```css
.card-row {
  display: flex;
  align-items: stretch;
}
```

Gotchas: `align-content` has no effect on a single-line container.

---

# gap, row-gap, column-gap

Spaces flex/grid items without margins.

```css
gap: 1rem;
gap: 1rem 2rem; /* row column */
```

```css
.grid {
  display: flex;
  flex-wrap: wrap;
  gap: 1rem;
}
```

Gotchas: gap is not a replacement for padding inside items; margins still work but do not collapse.

---

# flex, flex-grow, flex-shrink, flex-basis

Controls how items grow, shrink, and start.

```css
flex: 1;             /* 1 1 0% */
flex: auto;          /* 1 1 auto */
flex: none;          /* 0 0 auto */
flex: 0 0 240px;     /* fixed */
```

```css
.sidebar { flex: 0 0 240px; }
.content { flex: 1 1 auto; min-width: 0; }
```

Gotchas: `flex:1` ignores content size for distribution; `min-width:auto` prevents shrinking.

---

# order

Changes visual order without changing DOM order.

```css
order: 0;   /* default */
order: 2;
order: -1;
```

```css
.nav__cta {
  order: 2;
  margin-inline-start: auto;
}
```

Gotchas: keyboard and screen-reader order stay DOM-based; do not reorder meaningful content.

---

# Real-World Uses

A media object and a responsive toolbar.

```css
.media {
  display: flex;
  gap: 1rem;
  align-items: flex-start;
}

.media__body {
  flex: 1;
  min-width: 0;
}

.toolbar {
  display: flex;
  flex-wrap: wrap;
  align-items: center;
  justify-content: space-between;
  gap: 0.75rem;
}
```

---

# Quick Revision

| Property | Purpose |
| -------- | ------- |
| **flex-direction** | Main axis |
| **flex-wrap** | Multiple lines |
| **justify-content** | Main-axis distribution |
| **align-items/content** | Cross-axis items/lines |
| **gap** | Item spacing |
| **flex** | Grow/shrink/basis |
| **order** | Visual reorder only |
| **align-self** | Per-item cross alignment |

## The Mental Model

```text
main axis → justify-content
cross axis → align-items / align-content
item sizing → flex: grow shrink basis
```

### The 3 things to remember

**1. Axis first:**

```css
flex-direction: row;
justify-content: space-between;
align-items: center;
```

**2. Fixed + fluid:**

```css
.sidebar { flex: 0 0 240px; }
.content { flex: 1 1 auto; min-width: 0; }
```

**3. Wrap with gap:**

```css
flex-wrap: wrap;
gap: 1rem;
```

> **Flexbox is axis + distribution + sizing — set direction, justify/align, and flex shorthand deliberately.**
