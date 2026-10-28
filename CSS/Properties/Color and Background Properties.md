# Color and Background Properties

Color and background properties paint text, fills, images, and layers. This reference covers color formats, background longhands, gradients, and blending.

```text
color & background
  ├── text → color
  ├── fill → background-color
  ├── layers → background-image (gradients/images)
  ├── placement → size/position/repeat
  └── compositing → opacity/blend
```

---

# color

Sets text color and the `currentColor` value.

```css
color: #0f172a;
color: rgb(15 23 42 / 0.9);
color: oklch(0.4 0.1 250);
```

Gotchas: inherited; contrast must meet 4.5:1 for normal text.

---

# background-color

Fills the padding box behind content.

```css
background-color: #ffffff;
background-color: transparent;
```

Gotchas: not inherited; children are transparent by default.

---

# background-image and gradients

Accepts URLs and gradient functions; multiple layers are comma-separated.

```css
background-image: url("/img/hero.jpg");
background-image: linear-gradient(135deg, #0f172a, #334155);
background-image:
  linear-gradient(rgb(15 23 42 / 0.7), rgb(15 23 42 / 0.9)),
  url("/img/hero.jpg");
```

Gotchas: the first layer paints on top; gradients are images, not colors.

---

# background-size, background-position, background-repeat

Scale, place, and tile layers.

```css
background-size: cover | contain | 100% 100%;
background-position: center | top right | 20% 80%;
background-repeat: no-repeat | repeat | repeat-x | space;
```

```css
.hero {
  background-size: cover;
  background-position: center;
  background-repeat: no-repeat;
}
```

Gotchas: `cover` crops; `contain` may leave gaps; `fixed` attachment is janky on mobile.

---

# background-attachment, background-clip, background-origin

Scroll behavior, clipping, and positioning origin.

```css
background-attachment: scroll | fixed | local;
background-clip: border-box | padding-box | content-box | text;
background-origin: border-box | padding-box | content-box;
```

```css
.gradient-text {
  background: linear-gradient(90deg, #0f172a, #334155);
  -webkit-background-clip: text;
  background-clip: text;
  color: transparent;
}
```

Gotchas: `background-clip:text` needs a transparent text color and a fallback.

---

# background shorthand

Sets multiple background longhands and resets omitted ones.

```css
background: #0f172a url("/img/hero.jpg") center / cover no-repeat;
```

Gotchas: shorthand resets `background-size`, `position`, etc.; mixing shorthand and longhands can wipe values.

---

# opacity, mix-blend-mode and color-mix()

Whole-element transparency, blending, and color mixing.

```css
opacity: 0.6;
mix-blend-mode: multiply;
background: color-mix(in oklab, #0f172a 20%, white);
```

Gotchas: `opacity` creates a stacking context and fades children; prefer alpha colors for backgrounds.

---

# accent-color, caret-color, color-scheme

Native control accent, text caret, and color scheme hints.

```css
accent-color: #0f172a;
caret-color: #0f172a;
color-scheme: light dark;
```

```css
:root {
  color-scheme: light dark;
  accent-color: #0f172a;
}
```

Gotchas: `accent-color` affects checkboxes/radios/range/progress; `color-scheme` also styles scrollbars and form controls.

---

# Real-World Uses

A hero, a gradient-text heading, and themed controls.

```css
.hero {
  color: #fff;
  background-image:
    linear-gradient(rgb(15 23 42 / 0.6), rgb(15 23 42 / 0.9)),
    url("/img/hero.jpg");
  background-size: cover;
  background-position: center;
}

.gradient-text {
  background: linear-gradient(90deg, #0f172a, #475569);
  background-clip: text;
  color: transparent;
}

:root {
  color-scheme: light dark;
  accent-color: #0f172a;
}
```

---

# Quick Revision

| Property | Purpose |
| -------- | ------- |
| **color** | Text color, inherited |
| **background-color** | Fill |
| **background-image** | Images/gradients, layers |
| **background-size/position/repeat** | Placement |
| **background-clip** | Clip to box/text |
| **background shorthand** | Resets omitted longhands |
| **opacity** | Whole-element alpha |
| **mix-blend-mode** | Layer blending |
| **color-mix()** | Blend two colors |
| **accent-color** | Native control accent |
| **color-scheme** | Theme native UI |

## The Mental Model

```text
paint
  ├── text → color
  ├── fill → background-color
  ├── layers → background-image (+ size/position/repeat)
  ├── compositing → opacity/blend
  └── native UI → accent-color/color-scheme
```

### The 3 things to remember

**1. Overlay text over images:**

```css
background-image:
  linear-gradient(rgb(15 23 42 / 0.7), rgb(15 23 42 / 0.9)),
  url("/img/hero.jpg");
```

**2. Alpha, not opacity, for backgrounds:**

```css
background: rgb(15 23 42 / 0.5);
```

**3. Theme native controls:**

```css
:root {
  color-scheme: light dark;
  accent-color: #0f172a;
}
```

> **Color and background properties paint text, fills, layers, and native controls — keep layers ordered and contrast accessible.**
