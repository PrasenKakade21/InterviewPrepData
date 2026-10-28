# Units, Values and Functions

CSS values are numbers with units, keywords, functions, or colors. Choosing the right unit is the difference between a layout that scales and one that breaks at the next breakpoint.

The core idea:

```text
value
    ├── absolute → px, cm, in (fixed)
    ├── relative → em, rem, %, vw, ch (context)
    ├── function → calc(), clamp(), min(), max()
    └── color → hex, rgb(), hsl(), oklch()
```

---

# 1. Absolute Units

Absolute units do not depend on the parent or viewport. `px` is the practical one; the rest are mostly print-oriented.

```css
.card {
  width: 320px;
  border-width: 1px;
  padding: 16px;
}
```

Output:

```text
Same rendered size regardless of surrounding context.
```

| Unit | Equivalent |
| ---- | ---------- |
| `px` | CSS pixel |
| `in` | 96px |
| `cm` | 37.8px |
| `mm` | 3.78px |
| `pt` | 1.33px |
| `pc` | 16px |

Interview trap: px is not device pixels — it is a CSS reference pixel that scales with zoom and device pixel ratio.

---

# 2. Relative Units, em vs rem

Relative units scale with context. `em` is relative to the element's own font size (or parent for font-size), `rem` is relative to the root font size, and `%` depends on the property.

```css
:root {
  font-size: 16px;
}

.card {
  font-size: 1.25rem;   /* 20px */
  padding: 1em;         /* 20px, based on .card font-size */
  margin-bottom: 2rem;  /* 32px, based on root */
}

.card__title {
  font-size: 1.5em;     /* 30px, based on .card */
}
```

Output:

```text
Root 16px → card 20px → title 30px
Padding 20px; bottom margin 32px.
```

| Unit | Relative to |
| ---- | ----------- |
| `em` | Current/parent font size |
| `rem` | Root font size |
| `%` | Parent or property-specific base |
| `ch` | Width of "0" in the current font |
| `ex` | x-height |

Interview trap: nested `em` compounds (`1.2em` inside `1.2em` inside `1.2em`), which is why `rem` is safer for spacing and type scales.

---

# 3. Viewport Units and Percentages

Viewport units scale with the browser window. `vw`/`vh` are the large viewport; `dvh`/`svh` handle mobile browser chrome; `%` resolves against the containing block.

```css
.hero {
  min-height: 100vh;   /* classic, can overflow on mobile */
  min-height: 100dvh;  /* dynamic viewport */
  padding: 5vw;
}

.half {
  width: 50%;          /* half the containing block */
}
```

Output:

```text
Hero fills the visible viewport; 5vw padding scales with width.
.half is half its parent's content width.
```

Interview trap: `100vh` on mobile can include area hidden by the address bar. Prefer `100dvh` (dynamic) or `100svh` (smallest).

---

# 4. calc(), clamp(), min() and max()

Functions combine values and units. `calc()` does arithmetic, `clamp(min, preferred, max)` constrains a fluid value, and `min()`/`max()` pick a bound.

```css
.card {
  width: calc(100% - 2rem);
  padding: clamp(1rem, 2vw + 0.5rem, 2.5rem);
  font-size: clamp(1rem, 1rem + 1vw, 1.5rem);
  max-width: min(60ch, 90vw);
}
```

Output:

```text
Card leaves 2rem total horizontal space.
Padding scales between 1rem and 2.5rem.
Type scales between 1rem and 1.5rem.
Width never exceeds 60ch or 90vw.
```

Rules:

- `calc()` requires spaces around `+` and `-`.
- You can mix units (`100% - 2rem`).
- `clamp()` is `max(min, min(preferred, max))` conceptually.
- `min()`/`max()` accept multiple arguments.

---

# 5. Color Formats

CSS accepts named colors, hex, `rgb()`, `hsl()`, and modern spaces like `oklch()`. Modern syntax allows alpha and better perceptual uniformity.

```css
.card {
  color: #0f172a;                 /* hex */
  background: rgb(255 255 255);   /* modern rgb */
  border-color: hsl(215 28% 17%);
  accent-color: oklch(0.4 0.1 250);
  outline-color: rgb(15 23 42 / 0.4);
}
```

Output:

```text
Dark text on white, hsl border, oklch accent,
40% opacity outline.
```

| Format | Example | Notes |
| ------ | ------- | ----- |
| Named | `rebeccapurple` | Limited set |
| Hex | `#0f172a`, `#0f172a80` | Alpha in last 2 digits |
| `rgb()` | `rgb(15 23 42 / 0.8)` | Modern space syntax |
| `hsl()` | `hsl(215 28% 17%)` | Hue/saturation/lightness |
| `oklch()` | `oklch(0.4 0.1 250)` | Perceptually uniform |

Interview trap: `#0f172a80` alpha is hex, not opacity of the whole element; use `opacity` for the whole box, alpha for the color only.

---

# 6. Global Keywords and Value Types

Global keywords work on any property: `inherit`, `initial`, `unset`, `revert`. Values can also be keywords (`auto`, `none`, `fit-content`), functions, or custom properties.

```css
.card {
  width: fit-content;
  margin-inline: auto;
  color: inherit;
  border: initial;
}
```

Output:

```text
Card shrinks to content and centers horizontally;
text uses parent color; border returns to initial.
```

`auto` behavior depends on the property: for margins it absorbs free space (centering); for width it fills available space; for height it grows with content.

---

# Real-World Uses

A fluid section uses clamp for spacing and type, min() for width, and dvh for the hero.

```css
.hero {
  min-height: 100dvh;
  padding: clamp(2rem, 6vw, 6rem);
}

.hero__title {
  font-size: clamp(2rem, 5vw + 1rem, 4.5rem);
  max-width: min(30ch, 90%);
  margin-inline: auto;
}
```

Output:

```text
Hero fills the dynamic viewport; padding and title scale smoothly;
text line length stays readable.
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **Absolute units** | px (practical), in/cm/mm/pt |
| **Relative units** | em, rem, %, ch, ex |
| **em vs rem** | em compounds with context; rem uses root |
| **Viewport units** | vw/vh, dvh/svh for mobile chrome |
| **Percentages** | Resolve against containing block/property base |
| **calc()** | Arithmetic with mixed units |
| **clamp()** | Fluid value with min/max |
| **min()/max()** | Pick a bound |
| **Color formats** | hex, rgb, hsl, oklch |
| **Global keywords** | inherit, initial, unset, revert |

## The Mental Model

```text
value choice
  ├── fixed → px
  ├── scalable type/spacing → rem
  ├── component-relative → em
  ├── viewport-relative → vw/vh/dvh
  ├── container-relative → %
  └── responsive range → clamp()/min()/max()
```

### The 3 things to remember

**1. rem for global scale:**

```css
:root { font-size: 100%; }
.card { padding: 1.5rem; }
```

**2. clamp for fluid values:**

```css
font-size: clamp(1rem, 1rem + 1vw, 1.5rem);
```

**3. dvh for mobile viewports:**

```css
.hero { min-height: 100dvh; }
```

> **Units carry context — px for fixed edges, rem for scale, em for components, clamp() for fluid ranges, and dvh for honest viewport height.**
