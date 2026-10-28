# Flexbox — Revision Notes (Intermediate)
> One-line mental model: Flexbox is one-dimensional — direction sets the axis, justify/align place items, flex controls sizing.

## Core recall
- `display: flex` makes direct children flex items along the main axis; `inline-flex` is inline-level.
- `flex-direction`: `row` (default), `row-reverse`, `column`, `column-reverse`.
- `justify-content` works on the main axis; `align-items` on the cross axis; `align-content` distributes wrapped lines only.
- `flex-wrap: wrap` allows multiple lines; `align-content` then matters.
- `flex: grow shrink basis`; `flex:1` = `1 1 0%`, `flex:auto` = `1 1 auto`, `flex:none` = `0 0 auto`.
- Default `min-width:auto` blocks shrinking below content; set `min-width:0` for truncation.
- `gap` spaces items; flex containers prevent margin collapse.
- `order` is visual only; DOM/keyboard/screen-reader order is unchanged.
- Flex is one-dimensional; use Grid for two-dimensional placement.
- Percentage heights inside flex can be unreliable; prefer stretch/explicit sizes.

## Must-remember code
```css
.layout {
  display: flex;
  gap: 1.5rem;
  align-items: flex-start;
}
.sidebar { flex: 0 0 240px; }
.content { flex: 1 1 auto; min-width: 0; }
```
```css
.center {
  display: flex;
  align-items: center;
  justify-content: center;
  min-height: 100dvh;
}
```
```css
.card__title {
  flex: 1;
  min-width: 0;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}
```

## Interview traps
- Confusing `justify-content` (main) with `align-items` (cross).
- Expecting `align-content` to work on a single line.
- `flex:1` vs `flex:auto` basis difference.
- Text refusing to truncate because `min-width:auto`.
- Using flex for a two-dimensional grid.
- Reordering meaningful content with `order`.
- Relying on margin collapse inside flex.

## 3 things to remember
1. Main axis = flex-direction; justify vs align.
2. `flex: grow shrink basis`; `min-width:0` to shrink.
3. One axis per flex container; Grid for two.
