# Box Model — Revision Notes (Beginner)
> One-line mental model: content + padding + border + margin — set `border-box` and the numbers behave.

## Core recall
- Box layers inside out: content → padding → border → margin.
- Default `box-sizing: content-box`: `width` is content only; padding/border add to rendered size.
- `border-box`: `width` includes content + padding + border; standard global reset.
- Padding takes the background and cannot be negative; margin is transparent and can be negative.
- Margin shorthand: 1 value = all; 2 = vertical/horizontal; 3 = top/horizontal/bottom; 4 = top/right/bottom/left.
- Vertical margins collapse to the larger value between adjacent blocks; flex/grid gaps do not collapse.
- `overflow: hidden` clips content and creates a BFC (can stop margin collapse).
- `min-width`/`max-width` constrain flexible layouts better than fixed widths.
- `aspect-ratio` locks width-to-height for media and cards.
- Watch sizing: 100% width + padding overflows under content-box.

## Must-remember code
```css
*,
*::before,
*::after {
  box-sizing: border-box;
}

.card {
  width: 300px;
  padding: 20px;
  border: 2px solid #0f172a;
}
```
```css
.grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(240px, 1fr));
  gap: 1.5rem;
}
```
```css
.media {
  width: 100%;
  max-width: 800px;
  aspect-ratio: 16 / 9;
  overflow: hidden;
}
```

## Interview traps
- Forgetting `box-sizing` and overflowing a 100% element.
- Thinking padding and margin are interchangeable.
- Expecting margins to add rather than collapse.
- Using margins for grid/flex spacing instead of `gap`.
- `overflow: hidden` accidentally clipping focus rings/tooltips.
- Confusing `max-width` with fixed `width` in responsive layouts.

## 3 things to remember
1. Border-box globally.
2. Vertical margins collapse; gap does not.
3. Constrain with min/max + aspect-ratio, not fixed pixels.
