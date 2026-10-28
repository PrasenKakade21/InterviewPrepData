# Typography Properties

Typography properties control font selection, size, weight, spacing, alignment, decoration, and text overflow. This reference covers the ones used in production UI.

```text
typography
  ├── family → font-family
  ├── scale → font-size
  ├── weight/style
  ├── rhythm → line-height, letter/word-spacing
  ├── behavior → align, transform, wrap, overflow
  └── loading → @font-face, font-display
```

---

# font-family

Prioritized font stack ending in a generic family.

```css
font-family: Inter, "Segoe UI", system-ui, sans-serif;
```

Gotchas: quote multi-word names; always provide a generic fallback.

---

# font-size

Sets the type size. Use rem/clamp for scalable type.

```css
font-size: 1rem;
font-size: clamp(2rem, 5vw + 1rem, 4.5rem);
```

Gotchas: px body text ignores user font-size preferences; rem respects root scaling.

---

# font-weight and font-style

Weight 100–900 (or keywords), style normal/italic/oblique.

```css
font-weight: 400 | 600 | 700 | normal | bold;
font-style: normal | italic | oblique 10deg;
```

Gotchas: weights not provided by the font are synthesized or ignored.

---

# line-height

Leading between lines. Unitless is preferred.

```css
line-height: 1.5;
line-height: 1.15; /* headings */
```

Gotchas: fixed px/em values do not scale with child font sizes.

---

# letter-spacing and word-spacing

Tracking between characters/words.

```css
letter-spacing: 0.04em;
word-spacing: 0.05em;
```

Gotchas: wide tracking suits short labels, not body text; negative tracking harms readability.

---

# text-align, text-indent, text-transform

Inline alignment, first-line indent, and visual casing.

```css
text-align: left | right | center | justify | start | end;
text-indent: 2rem;
text-transform: uppercase | lowercase | capitalize;
```

Gotchas: `text-transform` changes appearance only, not the accessible name.

---

# text-decoration and text-underline-offset

Lines on text and their placement.

```css
text-decoration: underline;
text-decoration-thickness: 2px;
text-underline-offset: 3px;
text-decoration-color: currentColor;
```

Gotchas: removing link underlines requires another affordance; keep focus/hover distinct.

---

# white-space, overflow-wrap, word-break, text-overflow

Control wrapping and truncation.

```css
white-space: nowrap;
overflow-wrap: anywhere;
word-break: break-word;
text-overflow: ellipsis;
```

```css
.ellipsis {
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}
```

Gotchas: ellipsis needs `nowrap` + constrained width + `overflow:hidden`; `word-break: break-all` is aggressive.

---

# text-wrap and text-shadow

Balanced wrapping and shadow.

```css
text-wrap: balance;   /* headings */
text-wrap: pretty;    /* paragraphs */
text-shadow: 0 1px 2px rgb(0 0 0 / 0.5);
```

Gotchas: `balance` is for short blocks; long paragraphs are expensive and may be ignored.

---

# font shorthand and @font-face

`font` combines style/weight/size/line-height/family. `@font-face` declares a font file.

```css
font: 600 1.25rem/1.3 Inter, sans-serif;
```

```css
@font-face {
  font-family: "Kakade Sans";
  src: url("/fonts/kakade-sans.woff2") format("woff2");
  font-weight: 400 700;
  font-display: swap;
}
```

Gotchas: `font` resets omitted longhands to initial; always set `font-display`.

---

# Real-World Uses

A type system with fluid scale and readable rhythm.

```css
:root {
  --font-body: Inter, "Segoe UI", system-ui, sans-serif;
  --leading: 1.6;
}

body {
  font-family: var(--font-body);
  font-size: 1rem;
  line-height: var(--leading);
}

h1, h2, h3 {
  line-height: 1.15;
  text-wrap: balance;
}

.ellipsis {
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}
```

---

# Quick Revision

| Property | Purpose |
| -------- | ------- |
| **font-family** | Font stack + fallback |
| **font-size** | Type scale |
| **font-weight/style** | Thickness/slant |
| **line-height** | Leading |
| **letter/word-spacing** | Tracking |
| **text-align/transform** | Alignment/casing |
| **text-decoration** | Underline/line-through |
| **white-space/wrap** | Wrapping behavior |
| **text-overflow** | Ellipsis |
| **text-wrap** | Balance/pretty |
| **font shorthand** | Resets omitted longhands |

## The Mental Model

```text
type
  ├── family → stack + @font-face
  ├── size → rem/clamp
  ├── weight/style
  ├── rhythm → unitless line-height + spacing
  └── behavior → align, transform, wrap, ellipsis
```

### The 3 things to remember

**1. Unitless leading:**

```css
body { line-height: 1.6; }
```

**2. Fluid headings:**

```css
h1 { font-size: clamp(2rem, 5vw + 1rem, 4.5rem); }
```

**3. Ellipsis trio:**

```css
white-space: nowrap;
overflow: hidden;
text-overflow: ellipsis;
```

> **Typography properties set family, scale, rhythm, and text behavior — rem/clamp plus unitless leading covers most needs.**
