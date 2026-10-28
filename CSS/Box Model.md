# Box Model

Every element is a box made of four layers: content, padding, border, and margin. Most "why is this wider than I asked?" questions are box-model questions, and `box-sizing` changes the answer entirely.

The core idea:

```text
┌──────────────────────────────┐  margin
│  ┌────────────────────────┐  │
│  │  border                │  │
│  │  ┌──────────────────┐  │  │
│  │  │  padding         │  │  │
│  │  │  ┌────────────┐  │  │  │
│  │  │  │  content   │  │  │  │
│  │  │  └────────────┘  │  │  │
│  │  └──────────────────┘  │  │
│  └────────────────────────┘  │
└──────────────────────────────┘
```

---

# 1. Content Box, Width and Height

The content box is where text and children live. By default (`box-sizing: content-box`), `width` and `height` set only the content area — padding and border add to the rendered size.

```css
.card {
  width: 300px;
  padding: 20px;
  border: 2px solid #0f172a;
}
```

Output:

```text
Content: 300px
Rendered width: 300 + 40 padding + 4 border = 344px
Rendered height: content height + padding + border
```

Think:

```text
content-box: width = content only
border-box:  width = content + padding + border
```

Interview trap: a 100% wide element with padding overflows its parent under `content-box`. Use `border-box` globally.

---

# 2. Padding

Padding is space inside the border, between content and border. It takes the element's background and cannot be negative.

```css
.card {
  padding: 1rem 1.5rem;      /* 1rem top/bottom, 1.5rem left/right */
  padding-top: 2rem;
}
```

Output:

```text
Content inset 1rem vertically, 1.5rem horizontally,
then top padding overridden to 2rem.
```

Shorthand order: `top right bottom left`; with two values it is `vertical horizontal`; with three it is `top horizontal bottom`.

Interview trap: padding adds to the click target and background but does not collapse like margin.

---

# 3. Border

A border sits between padding and margin and takes width, style, and color. It always adds to the box size unless `box-sizing: border-box` is set.

```css
.card {
  border: 1px solid #e2e8f0;
  border-radius: 12px;
}

.card--accent {
  border-left: 4px solid #0f172a;
}
```

Output:

```text
1px light border with 12px rounded corners;
accent card has a 4px dark left edge.
```

Gotchas: `border: none` removes width/style; `border-radius` clips backgrounds but not child overflow unless combined with `overflow: hidden`.

---

# 4. Margin and Margin Collapse

Margin is space outside the border. Vertical margins between adjacent blocks collapse to the larger value (not the sum), which surprises almost everyone.

```css
h2 {
  margin-bottom: 24px;
}

p {
  margin-top: 16px;
}
```

Output:

```text
Gap between h2 and p: 24px (the larger margin), not 40px.
```

Where collapse happens:

- Adjacent siblings.
- Parent and first/last child when no padding/border/overflow separates them.
- Empty blocks.

Where it does not:

- Flex and grid items.
- Floated or absolutely positioned elements.
- Elements with `overflow` other than `visible`.

Fixes: use `gap` in flex/grid, or add padding/border/`display: flow-root` to the parent.

---

# 5. box-sizing

`box-sizing: border-box` makes `width` include padding and border, so the element renders at exactly the width you set.

```css
*,
*::before,
*::after {
  box-sizing: border-box;
}

.card {
  width: 300px;
  padding: 20px;
  border: 2px solid #0f172a;
}
```

Output:

```text
Rendered width: 300px total
Content width: 300 - 40 - 4 = 256px
```

This is the standard reset. Without it, percentage widths plus padding overflow their containers.

---

# 6. Overflow, Sizing Limits and aspect-ratio

`overflow` controls what happens to content that does not fit. `min-width`/`max-width` constrain flexible layouts. `aspect-ratio` locks width-to-height without hardcoding both.

```css
.card {
  max-width: 60ch;
  min-height: 120px;
  overflow: hidden;
  aspect-ratio: 16 / 9;
}

.scroll {
  overflow-x: auto;
}
```

Output:

```text
Card never wider than 60 characters, at least 120px tall,
16:9 ratio, clipped children; .scroll scrolls horizontally.
```

| Value | Behavior |
| ----- | -------- |
| `visible` | Content spills out |
| `hidden` | Clipped, no scroll |
| `scroll` | Scrollbars always |
| `auto` | Scrollbars when needed |
| `clip` | Clipped, cannot scroll |

Interview trap: `overflow: hidden` creates a new block formatting context, which stops margin collapse and can change layout unexpectedly.

---

# Real-World Uses

A card grid uses border-box, padding, max-width, aspect-ratio, and gap instead of margins.

```css
*,
*::before,
*::after { box-sizing: border-box; }

.grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(240px, 1fr));
  gap: 1.5rem;
}

.card {
  padding: 1.25rem;
  border: 1px solid #e2e8f0;
  border-radius: 12px;
  max-width: 100%;
  aspect-ratio: 4 / 3;
  overflow: hidden;
}
```

Output:

```text
Cards size predictably, never overflow, and space evenly via gap.
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **Content box** | Area for content |
| **Width and height** | Content-only by default |
| **Padding** | Inside border, background applies |
| **Border** | Between padding and margin |
| **Margin** | Outside border, can collapse |
| **Margin collapse** | Adjacent vertical margins take the larger value |
| **box-sizing** | `border-box` includes padding/border in width |
| **Overflow** | visible/hidden/scroll/auto/clip |
| **min/max width** | Constrain flexible elements |
| **aspect-ratio** | Lock width-to-height |

## The Mental Model

```text
box
  ├── content
  ├── padding  → inside, background
  ├── border   → edge
  └── margin   → outside, collapses vertically
        │
        └── box-sizing decides whether width counts padding+border
```

### The 3 things to remember

**1. Global border-box:**

```css
*, *::before, *::after { box-sizing: border-box; }
```

**2. Margin collapses; gap does not:**

```css
.grid { display: grid; gap: 1rem; }
```

**3. Constraints beat fixed widths:**

```css
.card { width: 100%; max-width: 60ch; }
```

> **Every element is content + padding + border + margin — set border-box and use gap/constraints to keep sizes predictable.**
