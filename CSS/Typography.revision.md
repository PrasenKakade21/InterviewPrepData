# Typography — Revision Notes (Beginner)
> One-line mental model: typography is scale + rhythm + loading — rem sizes, unitless leading, and font-display: swap.

## Core recall
- `font-family`: prioritized stack, quote multi-word names, end with a generic family.
- `@font-face` declares a custom font with `src`, `font-weight`, `font-style`, `font-display`.
- `font-size`: use `rem`/`clamp()` for scalable type; avoid fixed px for body text.
- `font-weight` 100–900 or keywords; unavailable weights are synthesized or ignored.
- `line-height` should be unitless (`1.5`) so children recompute from their own size.
- `text-align` aligns inline content; `text-transform` changes visual casing only.
- `text-decoration` supports thickness/offset/color; don't remove underlines without a replacement.
- `letter-spacing`/`word-spacing` are for short labels and fine-tuning, not body text.
- `font` shorthand resets omitted font longhands to initial.
- `font-display: swap` shows fallback immediately; `block` hides text briefly; `optional` loads only if fast.
- Single-line ellipsis needs `white-space: nowrap` + `overflow: hidden` + `text-overflow: ellipsis`.
- `text-wrap: balance` suits headings; `overflow-wrap: anywhere` handles long words.

## Must-remember code
```css
body {
  font-family: Inter, "Segoe UI", system-ui, sans-serif;
  font-size: 1rem;
  line-height: 1.6;
}
```
```css
@font-face {
  font-family: "Kakade Sans";
  src: url("/fonts/kakade-sans.woff2") format("woff2");
  font-weight: 400 700;
  font-display: swap;
}
```
```css
h1 {
  font-size: clamp(2rem, 5vw + 1rem, 4rem);
  line-height: 1.15;
  text-wrap: balance;
}
```
```css
.ellipsis {
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}
```

## Interview traps
- Unquoted `"Segoe UI"`.
- Fixed px/em line-height that does not scale.
- Believing `text-transform` changes the accessible name.
- Removing link underlines without another affordance.
- Using negative letter-spacing on body text.
- Forgetting `white-space: nowrap` with ellipsis.
- `font` shorthand wiping earlier longhands.

## 3 things to remember
1. rem + clamp for scale, unitless line-height for rhythm.
2. Always end font stacks with a generic family.
3. `font-display: swap` prevents invisible text.
