# CSS Interview Traps

Most CSS interviews are won or lost on a handful of distinctions. These are the questions that separate "I use CSS" from "I understand CSS". Each trap includes the correct mental model and the answer to say out loud.

The core idea:

```text
trap
    ├── box model → sizing + collapse
    ├── cascade → specificity + !important
    ├── stacking → z-index contexts
    ├── layout → positioning + flex/grid
    └── units → em/rem, vh, min-width
```

---

# 1. box-sizing and Margin Collapse

**Trap:** "Why is my 100%-wide element overflowing?" Because `width` excludes padding/border under `content-box`. **Fix:** global `border-box`.

**Trap:** "Why is the gap smaller than the sum of margins?" Vertical margins collapse to the larger value.

```css
*,
*::before,
*::after {
  box-sizing: border-box;
}

h2 { margin-bottom: 24px; }
p  { margin-top: 16px; }
/* gap = 24px, not 40px */
```

Output:

```text
Element width includes padding/border;
adjacent margins collapse to the larger value.
```

Say: "Border-box makes sizing predictable; margin collapse is a block-layout feature that flex/grid gap avoids."

---

# 2. Specificity, !important and z-index

**Trap:** "Why doesn't my class override the style?" Higher specificity, inline style, later layer, or `!important` won. **Fix:** inspect the Styles pane; do not add more selectors blindly.

**Trap:** "Why doesn't z-index:9999 work?" The element is trapped in a stacking context. **Fix:** raise the ancestor context or restructure.

```css
.card .title { color: #475569; }   /* 0,2,0 */
#hero .title { color: #0f172a; }   /* 1,1,0 wins */
```

Output:

```text
The id selector wins regardless of source order.
```

Say: "Specificity is a three-part tuple; z-index only compares inside one stacking context."

---

# 3. position: fixed vs sticky

**Trap:** "Why doesn't sticky work?" No `top` threshold, no scrolling ancestor, or an ancestor with `overflow: hidden/auto`. **Fix:** add `top`, remove clipping, ensure room to move.

**Trap:** "Why does fixed cover content?" Fixed is removed from flow. **Fix:** add padding/margin equal to the fixed element's size, or use sticky.

```css
.header {
  position: sticky;
  top: 0;
  z-index: 20;
}

.toc {
  position: sticky;
  top: 1rem;
  align-self: start;
}
```

Output:

```text
Header sticks to the viewport edge;
TOC sticks inside its column, not the page.
```

---

# 4. em vs rem, min-width vs width

**Trap:** "Why did padding grow unexpectedly?" `em` compounds with nested font sizes. **Fix:** `rem` for spacing/type scales.

**Trap:** "Why does the layout break on mobile?" Fixed `width` cannot shrink. **Fix:** `width: 100%; max-width: ...` or `minmax(0, 1fr)`.

```css
.card {
  width: 100%;
  max-width: 60ch;
  padding: 1.5rem; /* rem, predictable */
}
```

Output:

```text
Card fills small screens, caps on large ones, spacing stays stable.
```

Say: "rem is root-relative and predictable; em is component-relative and compounds."

---

# 5. display:none vs visibility:hidden, inline vs inline-block

**Trap:** "Difference between display:none and visibility:hidden?" None removes layout and accessibility; hidden keeps space but removes paint and accessibility.

**Trap:** "Why can't I set width on a span?" Inline elements ignore width/height. **Fix:** `inline-block`, `block`, or flex/grid item.

```css
.a { display: none; }
.b { visibility: hidden; }
.tag { display: inline-block; padding: 0.25rem 0.6rem; }
```

Output:

```text
.a takes no space; .b reserves space; .tag accepts padding/sizing.
```

---

# 6. Flexbox vs Grid, and 100vh on Mobile

**Trap:** "When Flexbox vs Grid?" Flex = one axis/distribution; Grid = two axes/track alignment. They compose.

**Trap:** "Why is 100vh too tall on mobile?" It uses the large viewport including browser chrome. **Fix:** `100dvh` or `100svh`.

```css
.shell {
  display: grid;
  grid-template-columns: 240px 1fr;
}

.toolbar {
  display: flex;
  align-items: center;
  gap: 0.75rem;
}

.hero {
  min-height: 100dvh;
}
```

Output:

```text
Grid shell, flex toolbar, honest mobile viewport height.
```

---

# 7. Rapid-Fire Answers

| Question | Answer |
| -------- | ------ |
| Is CSS a programming language? | Declarative style language with functions/variables, not general-purpose |
| Does margin inherit? | No |
| Does `opacity:0` hide from screen readers? | No |
| Does `order` change tab order? | No |
| Is `1fr` a percentage? | No, it distributes free space |
| Does `display:contents` keep the box? | No, removes it, keeps children |
| Does `transition: all` perform well? | Usually no, be explicit |
| Is `!important` a good strategy? | No, fix specificity/layers |
| Does `position:absolute` need a positioned parent? | Yes, otherwise initial containing block |
| Does `100vh` equal visible height on mobile? | Not reliably; use `dvh` |

Output:

```text
Short, precise answers with the underlying rule.
```

---

# Real-World Uses

A code review catches the common traps before they ship.

```css
/* 1. Predictable sizing */
*, *::before, *::after { box-sizing: border-box; }

/* 2. Flat specificity */
.card__title { color: #0f172a; }

/* 3. Sticky header with threshold */
.site-header { position: sticky; top: 0; z-index: 20; }

/* 4. Responsive width */
.card { width: 100%; max-width: 60ch; }

/* 5. Honest viewport */
.hero { min-height: 100dvh; }
```

Output:

```text
Review passes: no overflow, no specificity war, sticky works,
responsive width, correct mobile height.
```

---

# Quick Revision

| Trap | Correct answer |
| ---- | -------------- |
| **box-sizing** | border-box includes padding/border |
| **Margin collapse** | Larger vertical margin wins |
| **Specificity wars** | Tuple; fix cause, not with more selectors |
| **!important** | Last resort only |
| **z-index** | Only within a stacking context |
| **fixed vs sticky** | Viewport vs scroll threshold |
| **em vs rem** | Contextual/compounding vs root-stable |
| **min-width vs width** | Constraint vs fixed size |
| **display none vs hidden** | Remove layout vs keep space |
| **inline vs inline-block** | No sizing vs sizing + inline flow |
| **Flex vs Grid** | One axis vs two axes |
| **100vh mobile** | Use dvh/svh |

## The Mental Model

```text
interview trap
  ├── sizing → box-sizing, margin collapse, min/max
  ├── cascade → specificity, layers, !important
  ├── stacking → contexts, z-index
  ├── layout → position, flex/grid, display
  └── units → em/rem, vh/dvh
```

### The 3 things to remember

**1. Predictable sizing:**

```css
*, *::before, *::after { box-sizing: border-box; }
```

**2. Stacking contexts:**

```css
.modal { position: fixed; z-index: 100; }
```

**3. Honest viewport:**

```css
.hero { min-height: 100dvh; }
```

> **CSS traps are mostly reference-frame mistakes — know what width, specificity, z-index, and vh are actually relative to.**
