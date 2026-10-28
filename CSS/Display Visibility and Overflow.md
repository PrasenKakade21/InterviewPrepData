# Display, Visibility and Overflow

`display` decides whether an element generates a block, inline, flex, grid, or no box at all. `visibility` and `opacity` hide without removing layout. `overflow` decides what happens when content does not fit. These are the first properties to check when a layout "does nothing".

The core idea:

```text
element
    ├── display → box type / layout mode
    ├── visibility → paint or not (layout kept)
    ├── opacity → transparency (layout kept)
    └── overflow → clip or scroll
```

---

# 1. Display Values

`display` sets the outer box type and often an inner layout mode.

```css
.block      { display: block; }
.inline     { display: inline; }
.inline-block { display: inline-block; }
.flex       { display: flex; }
.grid       { display: grid; }
.contents   { display: contents; }
.hidden     { display: none; }
```

Output:

```text
block: full width, new line
inline: flows with text
inline-block: flows inline but accepts width/height
flex/grid: lays out children
contents: box removed, children participate in parent layout
none: no box at all
```

| Value | Width/height | New line | Children layout |
| ----- | ------------ | -------- | --------------- |
| `block` | Yes | Yes | Normal flow |
| `inline` | No (ignored) | No | Inline flow |
| `inline-block` | Yes | No | Normal flow inside |
| `flex` | Yes | Yes | Flex formatting |
| `grid` | Yes | Yes | Grid formatting |
| `none` | — | — | No box |

Interview trap: width/height do not apply to `inline` elements; margins top/bottom also behave differently. Use `inline-block` or flex when you need both inline flow and sizing.

---

# 2. block vs inline vs inline-block

Block elements stack and fill available width. Inline elements sit in text and ignore width/height. `inline-block` combines inline flow with block sizing.

```css
.tag {
  display: inline-block;
  padding: 0.25rem 0.6rem;
  border-radius: 999px;
  background: #f1f5f9;
}

.title {
  display: block;
  margin-bottom: 0.5rem;
}
```

Output:

```text
Tags sit next to each other and accept padding;
title occupies its own line.
```

Notice: inline elements have vertical padding that paints but does not affect line height — a classic overlap bug.

---

# 3. display:none vs visibility:hidden vs opacity:0

All three hide content, but they affect layout, paint, accessibility, and events differently.

```css
.a { display: none; }
.b { visibility: hidden; }
.c { opacity: 0; }
```

Output:

```text
.a: no box, no space, removed from a11y tree
.b: box and space kept, not painted, hidden from a11y tree
.c: box and space kept, transparent, still in a11y tree and clickable
```

| Property | Layout space | Painted | Accessible | Clickable |
| -------- | ------------ | ------- | ---------- | --------- |
| `display:none` | No | No | No | No |
| `visibility:hidden` | Yes | No | No | No |
| `opacity:0` | Yes | No | Yes | Yes |

Interview trap: `opacity: 0` is not "hidden" for assistive tech or clicks — use `visibility: hidden` or `display: none` when you mean hidden.

---

# 4. Overflow

`overflow` controls content that exceeds the box. It applies to block containers and creates a block formatting context when not `visible`.

```css
.card {
  max-height: 240px;
  overflow: auto;        /* scroll only if needed */
}

.scroller {
  overflow-x: auto;
  overflow-y: hidden;
}

.clip {
  overflow: clip;        /* no scrolling possible */
}
```

Output:

```text
.card scrolls when content exceeds 240px;
.scroller scrolls sideways only;
.clip cuts off overflow with no scrollbar.
```

Gotchas: setting only `overflow-x` can force `overflow-y` to `auto` due to spec coupling; `overflow: hidden` clips focus rings and tooltips.

---

# 5. object-fit, object-position and display:contents

`object-fit` controls how replaced elements (images/video) fill their box. `object-position` places the content. `display: contents` removes the element's box while keeping children in the layout.

```css
.avatar {
  width: 96px;
  height: 96px;
  object-fit: cover;
  object-position: center;
  border-radius: 50%;
}

.wrapper {
  display: contents;
}
```

Output:

```text
Avatar crops to a centered circle;
wrapper disappears from layout, children lay out in its parent.
```

Interview trap: `display: contents` on an element with its own background/border removes those visuals; on some elements (like buttons) it can harm accessibility.

---

# 6. Hiding Content Accessibly

Choose the technique based on intent: remove from everyone, hide visually but keep for screen readers, or hide from screen readers but keep visible.

```css
/* Visually hidden but available to screen readers */
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

/* Hidden until focused (skip link) */
.skip-link:focus {
  position: static;
  width: auto;
  height: auto;
  clip: auto;
}
```

Output:

```text
.sr-only text is read by assistive tech but invisible;
skip link becomes visible on keyboard focus.
```

Interview trap: `display: none` on a skip link means keyboard users can never focus it. Use the visually-hidden pattern, then reveal on `:focus`.

---

# Real-World Uses

A card grid uses `display: grid`, clamps text with overflow, and hides metadata for screen readers only.

```css
.grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(240px, 1fr));
  gap: 1.5rem;
}

.card__excerpt {
  display: -webkit-box;
  -webkit-line-clamp: 3;
  -webkit-box-orient: vertical;
  overflow: hidden;
}

.sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  overflow: hidden;
  clip: rect(0 0 0 0);
  white-space: nowrap;
}
```

Output:

```text
Responsive cards; excerpts clamp to three lines;
extra context available only to screen readers.
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **display values** | block, inline, inline-block, flex, grid, contents, none |
| **block vs inline** | Stack/full width vs flows in text |
| **inline-block** | Inline flow + width/height |
| **none vs hidden vs opacity** | Remove layout / keep space / transparent but interactive |
| **overflow** | visible, hidden, scroll, auto, clip |
| **object-fit** | cover/contain/fill for replaced elements |
| **display: contents** | Remove wrapper box, keep children |
| **Hiding accessibly** | sr-only pattern, reveal on focus |

## The Mental Model

```text
visibility decision
  ├── remove everything → display:none
  ├── keep space, remove paint/a11y → visibility:hidden
  ├── transparent, still interactive → opacity:0
  └── visual only hidden → sr-only pattern
        │
        └── overflow controls what happens outside the box
```

### The 3 things to remember

**1. Inline ignores size:**

```css
.tag { display: inline-block; padding: 0.25rem 0.6rem; }
```

**2. opacity is not hidden:**

```css
.overlay { opacity: 0; } /* still clickable and announced */
```

**3. sr-only for accessible hiding:**

```css
.sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  overflow: hidden;
  clip: rect(0 0 0 0);
  white-space: nowrap;
}
```

> **display chooses the box, visibility/opacity choose the paint, and overflow chooses what happens when content escapes — pick by intent.**
