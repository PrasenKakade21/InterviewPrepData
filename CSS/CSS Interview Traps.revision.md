# CSS Interview Traps — Revision Notes (Advanced)
> One-line mental model: traps are reference-frame mistakes — know what width, specificity, z-index, and vh are relative to.

## Core recall
- content-box adds padding/border outside width; border-box includes them.
- Adjacent vertical margins collapse to the larger value; flex/grid `gap` does not.
- Specificity tuple (ids, classes, elements); inline and `!important` sit outside.
- z-index only compares within one stacking context; transforms/opacity/filter create contexts.
- Sticky needs an inset threshold, a scrolling ancestor, and room; `overflow:hidden` ancestors break it.
- Fixed is viewport-relative and out of flow; add spacing for it.
- em compounds with nested font sizes; rem is root-stable.
- Fixed `width` cannot shrink; use `width:100%` + `max-width` or `minmax(0,1fr)`.
- `display:none` removes layout; `visibility:hidden` keeps space.
- Inline elements ignore width/height; inline-block accepts both.
- Flex = one axis; Grid = two axes.
- `100vh` includes hidden mobile chrome; use `100dvh`.

## Must-remember code
```css
*, *::before, *::after { box-sizing: border-box; }
```
```css
.header { position: sticky; top: 0; z-index: 20; }
```
```css
.card { width: 100%; max-width: 60ch; }
.hero { min-height: 100dvh; }
```
```css
.grid { display: grid; grid-template-columns: minmax(0, 1fr) 240px; }
```

## Interview traps
- Adding specificity instead of finding the winning rule.
- `!important` as a fix.
- Raising z-index without fixing the context.
- Expecting sticky without `top`.
- Using em for global spacing.
- Fixed widths in responsive layouts.
- Treating `visibility:hidden` as no layout.
- Setting width on inline elements.
- Using flex for two-dimensional grids.
- `100vh` on mobile.

## 3 things to remember
1. Sizing reference: border-box + min/max.
2. Stacking reference: context, not z-index number.
3. Viewport reference: dvh, not vh.
