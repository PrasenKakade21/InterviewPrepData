# Box Model Properties

These properties size and space boxes. They are the first place to look when a layout overflows or spacing behaves oddly.

```text
box model
  ├── size → width/height/min/max
  ├── inside → padding
  ├── edge → border
  ├── outside → margin
  ├── sizing mode → box-sizing
  └── ratio → aspect-ratio
```

---

# width, height, min-width, max-width, min-height, max-height

Set the content box (or border box with `border-box`). Min/max constrain flexible layouts.

```css
width: 100%;
max-width: 60ch;
min-height: 100dvh;
min-width: 0;   /* allow flex/grid shrinking */
```

```css
.card {
  width: 100%;
  max-width: 60ch;
  min-height: 120px;
}
```

Gotchas: `width`/`height` are ignored on inline elements; `min-width:auto` on flex items prevents shrinking below content.

---

# padding and padding-*

Space inside the border. Shorthand: `top right bottom left`; two values = vertical/horizontal.

```css
padding: 1rem 1.5rem;
padding-block: 1rem;
padding-inline: 1.5rem;
```

```css
.card {
  padding: 1.25rem;
  padding-block-start: 0;
}
```

Gotchas: padding takes the background, cannot be negative, and adds to `content-box` size.

---

# border and border-*

Width, style, and color of the edge. Longhands: `border-width`, `border-style`, `border-color`, plus per-side variants.

```css
border: 1px solid #e2e8f0;
border-inline-start: 4px solid #0f172a;
border-radius: 12px;
```

```css
.card {
  border: 1px solid #e2e8f0;
  border-radius: 12px;
}
```

Gotchas: `border: none` clears width/style; radius clips backgrounds but not overflowing children unless `overflow:hidden`.

---

# margin and margin-*

Space outside the border. Can be negative; vertical margins collapse in block layout.

```css
margin: 0 auto;
margin-block: 1rem;
margin-inline: auto;
```

```css
.container {
  max-width: 72ch;
  margin-inline: auto;
}
```

Gotchas: margin does not take the background; flex/grid prevent collapsing; use `gap` for item spacing.

---

# box-sizing

Chooses whether width/height include padding and border.

```css
box-sizing: content-box | border-box;
```

```css
*,
*::before,
*::after {
  box-sizing: border-box;
}
```

Gotchas: content-box is the default and the usual cause of overflow; set border-box globally.

---

# aspect-ratio

Locks width-to-height ratio. Accepts `auto` or a ratio.

```css
aspect-ratio: 16 / 9;
aspect-ratio: 1;
aspect-ratio: auto 4 / 3;
```

```css
.media {
  width: 100%;
  aspect-ratio: 16 / 9;
  object-fit: cover;
}
```

Gotchas: a fixed width and height override the ratio; `auto 4/3` uses the ratio only when one dimension is auto.

---

# Real-World Uses

A predictable card and responsive media box.

```css
*,
*::before,
*::after { box-sizing: border-box; }

.card {
  width: 100%;
  max-width: 60ch;
  padding: 1.25rem;
  border: 1px solid #e2e8f0;
  border-radius: 12px;
  margin-inline: auto;
}

.media {
  aspect-ratio: 16 / 9;
  overflow: hidden;
}
```

---

# Quick Revision

| Property | Purpose |
| -------- | ------- |
| **width/height** | Set size (content or border box) |
| **min/max** | Constrain flexible sizing |
| **padding** | Inside spacing, takes background |
| **border** | Edge width/style/color |
| **margin** | Outside spacing, can collapse |
| **box-sizing** | Include padding/border in size |
| **aspect-ratio** | Lock proportions |

## The Mental Model

```text
size
  ├── width/height
  ├── min/max constraints
  ├── padding (inside)
  ├── border (edge)
  ├── margin (outside)
  └── box-sizing decides what width counts
```

### The 3 things to remember

**1. Global border-box:**

```css
*, *::before, *::after { box-sizing: border-box; }
```

**2. Center with margin-inline:**

```css
.container { max-width: 72ch; margin-inline: auto; }
```

**3. Ratio without fixed height:**

```css
.media { aspect-ratio: 16 / 9; }
```

> **Box model properties size, pad, edge, and space a box — border-box plus min/max makes the numbers predictable.**
