# Colors, Backgrounds and Gradients — Revision Notes (Beginner)
> One-line mental model: backgrounds are layers — color, image/gradient, placement — and text must stay readable on top.

## Core recall
- `color` is inherited; `background-color` is not (children are transparent).
- Gradients are images: `linear-gradient(angle, stops)`, `radial-gradient()`, `conic-gradient()`.
- Multiple backgrounds are comma-separated; the first layer paints on top.
- `background-size`: `cover` fills/crops, `contain` fits.
- `background-position`: `center`, `top right`, percentages.
- `background-repeat`: `no-repeat`, `repeat-x`, `space`, etc.
- `opacity` fades the whole element and creates a stacking context; alpha colors fade only the color.
- `color-mix(in oklab, colorA 20%, white)` blends colors.
- `currentColor` reuses the element's text color.
- Contrast AA: 4.5:1 normal text, 3:1 large text; gradients/images need overlays.

## Must-remember code
```css
.hero {
  min-height: 60dvh;
  color: #fff;
  background-image:
    linear-gradient(to bottom, rgb(15 23 42 / 0.55), rgb(15 23 42 / 0.9)),
    url("/img/kakade-hero.jpg");
  background-size: cover;
  background-position: center;
  background-repeat: no-repeat;
}
```
```css
.button {
  color: #0f172a;
  border: 1px solid currentColor;
  background: color-mix(in oklab, #0f172a 12%, white);
}
```
```css
.badge {
  background-image: radial-gradient(circle at 30% 30%, #fde68a, #f59e0b);
}
```

## Interview traps
- Expecting background-color to inherit.
- Using opacity for a background and fading the text too.
- Gradient overlay order reversed (photo covers the gradient).
- `background-attachment: fixed` jank on mobile.
- White-on-photo text without a consistent overlay.
- Forgetting `background-repeat: no-repeat` for hero images.

## 3 things to remember
1. Layers: first background paints on top.
2. cover fills, contain fits.
3. Overlay + contrast check for text over media.
