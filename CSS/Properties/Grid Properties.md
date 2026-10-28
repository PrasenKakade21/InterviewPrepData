# Grid Properties

Grid properties define tracks, place items, and align content in two dimensions. This reference separates container properties from item properties.

```text
grid container
  ├── template → columns/rows/areas
  ├── auto → rows/columns/flow
  ├── alignment → justify/align/place
  └── gap

grid item
  ├── grid-column / grid-row
  ├── grid-area
  └── justify-self / align-self
```

---

# grid-template-columns and grid-template-rows

Define explicit track sizes.

```css
grid-template-columns: 240px 1fr;
grid-template-columns: repeat(3, minmax(0, 1fr));
grid-template-rows: auto 1fr auto;
```

```css
.layout {
  display: grid;
  grid-template-columns: 240px 1fr;
  grid-template-rows: auto 1fr auto;
}
```

Gotchas: `1fr` distributes free space, not percentage; `minmax(auto,1fr)` can prevent shrinking.

---

# grid-template-areas

Names regions in a row-by-row string map.

```css
grid-template-areas:
  "header header"
  "sidebar main"
  "footer footer";
```

```css
.layout {
  display: grid;
  grid-template-areas:
    "header header"
    "sidebar main";
}
.site-header { grid-area: header; }
.sidebar { grid-area: sidebar; }
.main { grid-area: main; }
```

Gotchas: repeated names create spanning areas; every row must have the same number of columns.

---

# grid-auto-columns, grid-auto-rows, grid-auto-flow

Size and place implicit tracks/items.

```css
grid-auto-rows: minmax(120px, auto);
grid-auto-flow: row | column | row dense | column dense;
```

```css
.grid {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  grid-auto-rows: minmax(120px, auto);
  grid-auto-flow: dense;
}
```

Gotchas: `dense` fills gaps but can reorder visual placement away from DOM order.

---

# repeat(), minmax(), fr

Track helpers used inside `grid-template-*`.

```css
repeat(auto-fit, minmax(240px, 1fr));
repeat(3, 1fr);
minmax(200px, 1fr);
```

```css
.gallery {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(240px, 1fr));
  gap: 1.5rem;
}
```

Gotchas: `auto-fit` collapses empty tracks; `auto-fill` keeps them.

---

# grid-column, grid-row, grid-area

Place items by line, span, or named area.

```css
grid-column: 1 / 3;
grid-row: 2 / span 2;
grid-area: header;
```

```css
.feature {
  grid-column: 1 / -1; /* full width */
  grid-row: 2;
}
```

Gotchas: placement uses grid lines; `-1` is the last line. `span n` counts tracks.

---

# justify-items, align-items, justify-content, align-content, place-*

Alignment on both axes.

```css
justify-items: stretch | start | end | center;
align-items: stretch | start | end | center;
justify-content: start | end | center | space-between | space-around | space-evenly;
align-content: start | end | center | space-between | space-around | space-evenly;
place-items: center;
```

```css
.grid {
  display: grid;
  justify-items: stretch;
  align-items: center;
  place-content: center;
}
```

Gotchas: `justify-*` is the inline axis; `align-*` is the block axis.

---

# Real-World Uses

A dashboard and a gallery.

```css
.dashboard {
  display: grid;
  grid-template-columns: 220px 1fr;
  grid-template-rows: auto 1fr;
  grid-template-areas:
    "sidebar header"
    "sidebar main";
  min-height: 100dvh;
  gap: 1rem;
}

.gallery {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(240px, 1fr));
  gap: 1.5rem;
  align-content: start;
}
```

---

# Quick Revision

| Property | Purpose |
| -------- | ------- |
| **template columns/rows** | Explicit tracks |
| **template areas** | Named layout regions |
| **auto rows/columns/flow** | Implicit tracks and placement |
| **repeat/minmax/fr** | Compact responsive tracks |
| **grid-column/row/area** | Item placement |
| **justify/align/place** | Two-axis alignment |

## The Mental Model

```text
container
  ├── tracks → template + repeat/minmax/fr
  ├── implicit → auto rows/columns/flow
  ├── areas → names
  └── alignment → justify (inline) / align (block)

item
  ├── placement → column/row/area
  └── self alignment → justify-self/align-self
```

### The 3 things to remember

**1. Responsive gallery:**

```css
grid-template-columns: repeat(auto-fit, minmax(240px, 1fr));
```

**2. Named areas:**

```css
grid-template-areas: "sidebar main";
```

**3. Lines, not tracks:**

```css
grid-column: 1 / 3;
```

> **Grid properties define tracks, place items by line/area, and align on both axes — know the difference between tracks and lines.**
