# Border, Outline and Effects

These properties draw edges, corners, shadows, and visual effects. They do not affect layout except border width (unless `border-box`). This reference covers borders, outlines, shadows, filters, clipping, and masks.

```text
edge & effects
  ├── border → width/style/color + radius
  ├── outline → focus/selection ring
  ├── shadow → box/text
  ├── filter → blur/color effects
  ├── clip-path → shape clipping
  └── mask → alpha-based masking
```

---

# border, border-width/style/color

Sets the edge of the box.

```css
border: 1px solid #e2e8f0;
border-width: 2px;
border-style: dashed;
border-color: currentColor;
border-inline-start: 4px solid #0f172a;
```

Gotchas: `border: none` clears width/style; border adds to `content-box` size.

---

# border-radius

Rounds corners; accepts one to four radii or elliptical pairs.

```css
border-radius: 12px;
border-radius: 999px;
border-radius: 50%;
border-radius: 12px 12px 0 0;
border-radius: 50% / 20%;
```

Gotchas: radius clips backgrounds but not overflowing children unless `overflow:hidden`. Large radius creates pills/circles.

---

# outline, outline-offset

Draws a ring outside the border without affecting layout. Essential for focus visibility.

```css
outline: 3px solid #0f172a;
outline-offset: 2px;
```

```css
:focus-visible {
  outline: 3px solid #0f172a;
  outline-offset: 2px;
}
```

Gotchas: never remove outlines without a visible replacement; outlines can be clipped by `overflow:hidden` ancestors.

---

# box-shadow

Draws one or more shadows outside (or inset) the box.

```css
box-shadow: 0 1px 2px rgb(15 23 42 / 0.08);
box-shadow: 0 12px 24px rgb(15 23 42 / 0.12);
box-shadow: inset 0 1px 0 rgb(255 255 255 / 0.6);
box-shadow: 0 1px 2px rgb(0 0 0 / 0.1), 0 8px 24px rgb(0 0 0 / 0.08);
```

Gotchas: shadows do not affect layout; heavy shadows on scroll can hurt paint performance.

---

# text-shadow

Draws a shadow behind text.

```css
text-shadow: 0 1px 2px rgb(0 0 0 / 0.6);
text-shadow: 1px 0 0 #fff, -1px 0 0 #fff;
```

Gotchas: not a substitute for sufficient contrast; avoid complex shadows on body text.

---

# filter and backdrop-filter

`filter` applies effects to the element and children; `backdrop-filter` applies to what is behind it.

```css
filter: blur(4px) brightness(0.9) saturate(1.2);
backdrop-filter: blur(12px) saturate(1.2);
```

```css
.glass {
  background: rgb(255 255 255 / 0.6);
  backdrop-filter: blur(12px);
}
```

Gotchas: filters create containing blocks and stacking contexts; `backdrop-filter` support/performance varies.

---

# clip-path and mask

`clip-path` clips to a shape; `mask` uses alpha/luminance to hide parts.

```css
clip-path: circle(50%);
clip-path: polygon(0 0, 100% 0, 100% 80%, 0 100%);
mask-image: linear-gradient(to bottom, #000, transparent);
```

```css
.avatar {
  clip-path: circle(50%);
}
```

Gotchas: clipped content is still focusable/clickable outside the visible shape unless also hidden from interaction.

---

# mix-blend-mode and isolation

Blends an element with its backdrop; `isolation` scopes blending.

```css
.overlay {
  mix-blend-mode: multiply;
}

.card {
  isolation: isolate;
}
```

Gotchas: blending depends on the stacking context; `isolation: isolate` prevents bleed into the page.

---

# Real-World Uses

A focus ring, card shadow, glass panel, and circular avatar.

```css
.card {
  border: 1px solid #e2e8f0;
  border-radius: 12px;
  box-shadow: 0 12px 24px rgb(15 23 42 / 0.08);
}

.glass {
  background: rgb(255 255 255 / 0.6);
  backdrop-filter: blur(12px);
}

.avatar {
  width: 96px;
  height: 96px;
  border-radius: 50%;
  object-fit: cover;
}

:focus-visible {
  outline: 3px solid #0f172a;
  outline-offset: 2px;
}
```

---

# Quick Revision

| Property | Purpose |
| -------- | ------- |
| **border** | Edge width/style/color |
| **border-radius** | Rounded/pill/circle corners |
| **outline** | Non-layout ring, focus |
| **box-shadow** | Box depth/inset |
| **text-shadow** | Text depth |
| **filter** | Element effects |
| **backdrop-filter** | Behind-element effects |
| **clip-path** | Shape clipping |
| **mask** | Alpha/luminance masking |
| **mix-blend-mode** | Blending with backdrop |

## The Mental Model

```text
edge
  ├── border → box edge
  ├── outline → outside ring
  ├── radius → corners
  ├── shadow → depth
  └── effects → filter/clip/mask/blend
```

### The 3 things to remember

**1. Focus ring:**

```css
:focus-visible {
  outline: 3px solid #0f172a;
  outline-offset: 2px;
}
```

**2. Depth:**

```css
box-shadow: 0 12px 24px rgb(15 23 42 / 0.12);
```

**3. Shape:**

```css
clip-path: circle(50%);
```

> **Borders, outlines, shadows, and filters decorate the box — use them for hierarchy and focus, not layout.**
