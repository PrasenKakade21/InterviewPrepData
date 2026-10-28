# Grid

CSS Grid is two-dimensional layout: rows and columns at the same time. Use it for page shells, dashboards, galleries, and any design where alignment across both axes matters. Flexbox distributes along one axis; Grid places on a track system.

The core idea:

```text
display: grid
    ↓
define columns + rows (tracks)
    ↓
place items into cells/areas
    ↓
align on both axes
```

---

# 1. Grid Container and Items, template columns/rows

`display: grid` makes direct children grid items. `grid-template-columns` and `grid-template-rows` define the track sizes.

```css
.grid {
  display: grid;
  grid-template-columns: 240px 1fr;
  grid-template-rows: auto 1fr auto;
  gap: 1rem;
}
```

Output:

```text
Two columns: fixed 240px + remaining space
Three rows: header-sized, flexible, footer-sized
1rem gaps between tracks.
```

Only direct children are items; nested elements follow their own layout. Explicit tracks define the grid; items can also create implicit tracks if they are placed outside it.

---

# 2. fr, repeat() and minmax()

`fr` distributes free space. `repeat()` avoids repetition. `minmax(min, max)` clamps a track size.

```css
.grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
}

.auto {
  grid-template-columns: repeat(auto-fit, minmax(240px, 1fr));
}

.clamped {
  grid-template-columns: minmax(200px, 1fr) 3fr;
}
```

Output:

```text
Three equal columns; responsive auto-fit columns minimum 240px;
first column never below 200px and takes 1/4 of free space.
```

Interview trap: `1fr` is not `25%` — it distributes remaining space after fixed tracks and gaps, and `minmax(auto, 1fr)` can prevent shrinking below content.

---

# 3. auto-fit vs auto-fill

Both repeat as many tracks as fit. `auto-fill` keeps empty tracks; `auto-fit` collapses empty tracks so items stretch.

```css
.fill {
  grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
}

.fit {
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
}
```

Output:

```text
3 items in a 900px container:
auto-fill → 4 tracks (one empty)
auto-fit  → 4 tracks, empty ones collapse, 3 items stretch
```

Use `auto-fit` for galleries/cards; `auto-fill` when you want consistent track widths and empty slots.

---

# 4. Placement by Lines and Grid Areas

Items can be placed by line numbers, spans, or named areas. `grid-template-areas` is the most readable for page layouts.

```css
.layout {
  display: grid;
  grid-template-columns: 240px 1fr;
  grid-template-rows: auto 1fr auto;
  grid-template-areas:
    "header header"
    "sidebar main"
    "footer footer";
  min-height: 100dvh;
}

.site-header { grid-area: header; }
.sidebar { grid-area: sidebar; }
.main { grid-area: main; }
.site-footer { grid-area: footer; }
```

Output:

```text
Header spans both columns; sidebar and main share the middle row;
footer spans the bottom.
```

Line-based placement:

```css
.feature {
  grid-column: 1 / 3;   /* start line 1, end line 3 */
  grid-row: 2 / span 2;
}
```

Interview trap: line numbers count grid lines, not tracks. A 3-column grid has 4 vertical lines.

---

# 5. Alignment in Grid

`justify-*` aligns along the inline (column) axis; `align-*` aligns along the block (row) axis. `place-*` is the shorthand.

```css
.grid {
  display: grid;
  justify-items: stretch;
  align-items: center;
  place-content: center;
}
```

Output:

```text
Items stretch horizontally, center vertically,
and the whole grid content is centered in its container.
```

| Property | Axis | Applies to |
| -------- | ---- | ---------- |
| `justify-items` | Inline | Items in cells |
| `align-items` | Block | Items in cells |
| `justify-content` | Inline | Whole track grid |
| `align-content` | Block | Whole track grid |
| `place-items` | Both | Items |
| `place-content` | Both | Track grid |

---

# 6. grid-auto-flow, Implicit Tracks and Subgrid

`grid-auto-flow` controls auto-placement. `grid-auto-rows`/`grid-auto-columns` size implicit tracks. `subgrid` lets a nested grid inherit parent tracks.

```css
.grid {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  grid-auto-flow: dense;       /* fill gaps */
  grid-auto-rows: minmax(120px, auto);
}

.card {
  display: grid;
  grid-template-rows: subgrid;
  grid-row: span 2;
}
```

Output:

```text
Items auto-place across 4 columns, dense fills gaps,
implicit rows are at least 120px, nested card shares parent rows.
```

Interview trap: `dense` can reorder visual placement away from DOM order; avoid it when reading order matters.

---

# 7. Grid vs Flexbox and Common Layouts

Grid is for two-dimensional placement and page structure; Flexbox is for one-dimensional distribution and component internals. They compose: a grid cell can be a flex container.

```css
/* Page shell: Grid */
.shell {
  display: grid;
  grid-template-columns: 240px 1fr;
  grid-template-rows: auto 1fr;
  min-height: 100dvh;
}

/* Toolbar inside a cell: Flex */
.toolbar {
  display: flex;
  align-items: center;
  gap: 0.75rem;
}

/* Card gallery: Grid */
.gallery {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(240px, 1fr));
  gap: 1.5rem;
}
```

Output:

```text
Stable page shell, flexible toolbar, responsive gallery.
```

Interview answer: "Grid for layout in two dimensions, Flexbox for alignment/distribution in one."

---

# Real-World Uses

A dashboard uses named areas for the shell and auto-fit for the widget grid.

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

.dashboard__header { grid-area: header; }
.dashboard__sidebar { grid-area: sidebar; }
.dashboard__main {
  grid-area: main;
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(220px, 1fr));
  gap: 1rem;
  align-content: start;
}
```

Output:

```text
Fixed sidebar, header, and a responsive widget grid
that fills available space.
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **Container and items** | Direct children become items |
| **template columns/rows** | Define explicit tracks |
| **fr unit** | Distributes free space |
| **repeat()/minmax()** | Compact, clamped tracks |
| **auto-fit vs auto-fill** | Collapse empty tracks vs keep them |
| **auto-flow** | Auto-placement; dense fills gaps |
| **Grid areas** | Named layout regions |
| **Alignment** | justify = inline, align = block |
| **Subgrid** | Inherit parent tracks |
| **Grid vs Flexbox** | Two axes vs one axis |

## The Mental Model

```text
grid
  ├── tracks → columns/rows (fr, repeat, minmax)
  ├── placement → lines, spans, areas
  ├── alignment → justify (inline) / align (block)
  ├── auto flow → implicit tracks
  └── subgrid → inherit parent tracks
```

### The 3 things to remember

**1. Responsive gallery:**

```css
grid-template-columns: repeat(auto-fit, minmax(240px, 1fr));
```

**2. Named areas:**

```css
grid-template-areas:
  "header header"
  "sidebar main";
```

**3. Grid + flex compose:**

```css
.main { display: grid; }
.toolbar { display: flex; }
```

> **Grid places items in two dimensions — define tracks, place by area/line, and use auto-fit for responsive galleries.**
