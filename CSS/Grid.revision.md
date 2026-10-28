# Grid — Revision Notes (Intermediate)
> One-line mental model: Grid places items in two dimensions — define tracks, place by area/line, and auto-fit for responsive galleries.

## Core recall
- `display: grid` makes direct children grid items; `inline-grid` is inline-level.
- `grid-template-columns`/`rows` define explicit tracks; other items create implicit tracks.
- `fr` distributes remaining free space, not a percentage.
- `repeat(n, size)` and `minmax(min, max)` keep track definitions compact and clamped.
- `auto-fit` collapses empty tracks; `auto-fill` keeps them.
- Placement uses grid **lines**, not track numbers; `span n` spans tracks.
- `grid-template-areas` + `grid-area` = readable named layout.
- Alignment: `justify-*` = inline axis; `align-*` = block axis; `place-*` shorthand.
- `grid-auto-flow: dense` fills gaps but can break visual/DOM order.
- `subgrid` inherits parent tracks for nested alignment.
- `minmax(auto, 1fr)` can block shrinking; use `minmax(0, 1fr)` for truncation.

## Must-remember code
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
.dashboard__main { grid-area: main; }
```
```css
.gallery {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(240px, 1fr));
  gap: 1.5rem;
}
```
```css
.feature {
  grid-column: 1 / 3;
  grid-row: 2 / span 2;
}
```

## Interview traps
- Counting tracks instead of lines for placement.
- Thinking `1fr` equals 25%.
- Using `auto-fill` when cards should stretch.
- `dense` reordering content away from DOM order.
- Expecting a nested grid to align without `subgrid`.
- `minmax(auto,1fr)` preventing truncation.
- Using Grid for simple one-axis alignment where Flex fits better.

## 3 things to remember
1. Tracks + lines + areas.
2. `auto-fit` + `minmax` for responsive galleries.
3. Grid for two axes, Flex for one.
