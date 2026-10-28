# Typography

Typography is most of a page's perceived quality. CSS controls the font family, size, weight, line height, spacing, alignment, and how text wraps or truncates. Interviewers test `rem` vs `em`, line-height units, and web-font loading.

The core idea:

```text
text
    ├── family → font-family + @font-face
    ├── scale  → font-size + rem/clamp
    ├── weight/style
    ├── rhythm → line-height + spacing
    └── behavior → alignment, wrapping, overflow
```

---

# 1. font-family Stacks and @font-face

`font-family` takes a prioritized list. A stack ends with a generic family. `@font-face` declares a custom font file and its descriptors.

```css
body {
  font-family: Inter, "Segoe UI", system-ui, -apple-system, sans-serif;
}

@font-face {
  font-family: "Kakade Sans";
  src: url("/fonts/kakade-sans.woff2") format("woff2");
  font-weight: 400 700;
  font-display: swap;
}
```

Output:

```text
Inter if available, then Segoe UI/system font, then any sans-serif.
Custom font loads with weight range 400–700.
```

Interview trap: unquoted multi-word family names are invalid; quote `"Segoe UI"`. Always end with a generic family.

---

# 2. font-size, font-weight and font-style

`font-size` sets the type scale; `font-weight` sets thickness (100–900 or keywords); `font-style` sets italic/oblique.

```css
h1 {
  font-size: clamp(2rem, 5vw + 1rem, 4rem);
  font-weight: 700;
  font-style: normal;
}

.emphasis {
  font-weight: 600;
  font-style: italic;
}
```

Output:

```text
H1 scales from 2rem to 4rem; emphasis is semibold italic.
```

Gotchas: a weight the font does not provide is synthesized or ignored; `font-style: oblique 10deg` is supported in modern browsers.

---

# 3. line-height

`line-height` controls leading between lines. Use a unitless number for scalable rhythm (e.g. `1.5`), not a fixed px value.

```css
body {
  font-size: 1rem;
  line-height: 1.5;
}

h1 {
  line-height: 1.1;
}
```

Output:

```text
Body lines are 1.5 × font size; headings are tighter at 1.1.
```

| Value | Behavior |
| ----- | -------- |
| `normal` | Font/browser default (~1.2) |
| unitless `1.5` | Multiplies each element's font size |
| `1.5em` | Computed once, can compound oddly |
| `24px` | Fixed, does not scale |

Interview trap: unitless is preferred because children recalculate against their own font size; `em`/`px` line-heights can inherit a value that no longer fits.

---

# 4. text-align, text-decoration and text-transform

`text-align` aligns inline content; `text-decoration` adds/removes lines; `text-transform` changes letter case visually.

```css
.hero__title {
  text-align: center;
  text-transform: uppercase;
  letter-spacing: 0.04em;
}

.link {
  text-decoration: underline;
  text-decoration-thickness: 2px;
  text-underline-offset: 3px;
}
```

Output:

```text
Centered uppercase title with slight tracking;
link underline 2px thick, offset from text.
```

Gotchas: `text-transform` changes appearance, not the underlying text or accessibility name; screen readers may read the original casing.

---

# 5. letter-spacing and word-spacing

`letter-spacing` adds space between characters; `word-spacing` between words. Use sparingly — too much tracking hurts readability.

```css
.label {
  letter-spacing: 0.08em;
  text-transform: uppercase;
}

.prose {
  word-spacing: 0.05em;
}
```

Output:

```text
Uppercase label with wide tracking; prose with slightly looser words.
```

Interview trap: negative letter-spacing on body text is a common "designer" mistake that harms accessibility; reserve tracking for short labels.

---

# 6. Font Shorthand and font-display

`font` is a shorthand for style, weight, size/line-height, and family. `font-display` controls how a web font behaves while loading.

```css
.card__title {
  font: 600 1.25rem/1.3 Inter, sans-serif;
}

@font-face {
  font-family: "Kakade Sans";
  src: url("/fonts/kakade-sans.woff2") format("woff2");
  font-display: swap;
}
```

Output:

```text
Title: weight 600, 1.25rem size, 1.3 line-height.
Font swaps immediately with fallback, then upgrades.
```

| `font-display` | Behavior |
| -------------- | -------- |
| `swap` | Show fallback immediately, swap when ready |
| `block` | Invisible text briefly, then font |
| `fallback` | Short block, then fallback |
| `optional` | Use font only if it loads fast |

Interview trap: the `font` shorthand resets omitted properties to initial — mixing it with longhands can wipe values.

---

# 7. Text Overflow, Wrapping and Balance

Long words and single-line labels need explicit wrapping rules. `overflow-wrap`, `text-overflow`, and `text-wrap` control what happens.

```css
.card__title {
  overflow-wrap: anywhere;
}

.ellipsis {
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.hero__title {
  text-wrap: balance;
}
```

Output:

```text
Long titles break instead of overflowing;
single-line labels end with …;
hero title balances line lengths.
```

Gotchas: `text-overflow: ellipsis` requires `white-space: nowrap` and a constrained width; `text-wrap: balance` is best for headings, not long paragraphs.

---

# Real-World Uses

A type system uses rem sizes, a unitless line-height, system-font fallback, and balanced headings.

```css
:root {
  --font-body: Inter, "Segoe UI", system-ui, sans-serif;
  --text-base: 1rem;
  --leading: 1.6;
}

body {
  font-family: var(--font-body);
  font-size: var(--text-base);
  line-height: var(--leading);
}

h1, h2, h3 {
  line-height: 1.15;
  text-wrap: balance;
}

.prose p + p {
  margin-top: 1em;
}
```

Output:

```text
Consistent type scale, readable leading, balanced headings,
and predictable paragraph rhythm.
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **font-family stacks** | Prioritized list, quote multi-word names, end with generic |
| **@font-face** | Declares custom font + descriptors |
| **font-size** | rem/clamp for scalable type |
| **font-weight/style** | 100–900, normal/italic |
| **line-height** | Unitless for scalable rhythm |
| **text-align** | Inline alignment |
| **text-decoration** | Underline/overline/line-through |
| **text-transform** | Visual casing only |
| **letter/word spacing** | Tracking, use sparingly |
| **font shorthand** | Resets omitted font longhands |
| **font-display** | swap/block/fallback/optional |
| **Text overflow** | wrap, ellipsis, balance |

## The Mental Model

```text
typography
  ├── family → stack + @font-face
  ├── size → rem/clamp scale
  ├── weight/style
  ├── leading → unitless line-height
  ├── spacing → letter/word
  └── behavior → align, transform, wrap, ellipsis
```

### The 3 things to remember

**1. Unitless line-height:**

```css
body { line-height: 1.5; }
```

**2. Scalable sizes:**

```css
h1 { font-size: clamp(2rem, 5vw + 1rem, 4rem); }
```

**3. Web fonts swap:**

```css
@font-face {
  font-family: "Kakade Sans";
  src: url("/fonts/kakade-sans.woff2") format("woff2");
  font-display: swap;
}
```

> **Typography is scale + rhythm + loading — rem sizes, unitless leading, and font-display: swap cover most of it.**
