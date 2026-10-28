# Display, Visibility and Overflow — Revision Notes (Beginner)
> One-line mental model: display chooses the box, visibility/opacity choose the paint, overflow handles escaped content.

## Core recall
- `display`: `block`, `inline`, `inline-block`, `flex`, `grid`, `contents`, `none`.
- Inline elements ignore width/height and stack vertically in line boxes; vertical padding paints but does not expand line height.
- `inline-block` = inline flow + explicit sizing.
- `display:none` removes box/space/a11y/clicks; `visibility:hidden` keeps space but removes paint/a11y/clicks; `opacity:0` keeps space, a11y, and clicks.
- `overflow`: `visible`, `hidden`, `scroll`, `auto`, `clip`; non-visible creates a BFC.
- `object-fit: cover` crops to fill; `contain` fits fully; `fill` stretches.
- `display:contents` removes the wrapper box but keeps children; can harm semantics on some elements.
- Visually-hidden pattern (`sr-only`) keeps content for screen readers; reveal on `:focus` for skip links.
- Setting only one overflow axis can affect the other.

## Must-remember code
```css
.sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0 0 0 0);
  white-space: nowrap;
  border: 0;
}
```
```css
.avatar {
  width: 96px;
  height: 96px;
  object-fit: cover;
  object-position: center;
  border-radius: 50%;
}
```
```css
.card__excerpt {
  display: -webkit-box;
  -webkit-line-clamp: 3;
  -webkit-box-orient: vertical;
  overflow: hidden;
}
```
```css
.scroller { overflow-x: auto; overflow-y: hidden; }
```

## Interview traps
- Trying to set width/height on inline elements.
- Using `opacity:0` and leaving invisible controls focusable.
- Confusing `visibility:hidden` with removing layout space.
- `display:none` on skip links.
- `display:contents` on semantic elements.
- Forgetting overflow clipping for focus rings.
- Assuming `overflow:clip` can scroll (it cannot).

## 3 things to remember
1. `display` controls box generation; `none` removes layout entirely.
2. `opacity:0` is transparent, not hidden.
3. Use sr-only for accessible hiding, not `display:none`.
