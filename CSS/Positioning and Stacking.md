# Positioning and Stacking

`position` takes an element out of normal flow or shifts it relative to a reference box. `z-index` decides who paints on top — but only inside the same stacking context. Most overlay bugs are containing-block or stacking-context bugs.

The core idea:

```text
position
    ├── static   → normal flow (default)
    ├── relative → shifted from its normal spot
    ├── absolute → removed, positioned to nearest positioned ancestor
    ├── fixed    → viewport-relative
    └── sticky   → relative until a scroll threshold
```

---

# 1. position: static and relative

`static` is the default: the element stays in normal flow and `inset`/`z-index` have no effect. `relative` keeps its space but can be shifted with `top/right/bottom/left`.

```css
.card {
  position: relative;
  top: 4px;
  left: 8px;
}
```

Output:

```text
Card stays in flow but paints 4px lower and 8px right;
the original space is still reserved.
```

Interview trap: `relative` is often used just to create a containing block for an absolutely positioned child, even with no offsets.

---

# 2. position: absolute and the Containing Block

`absolute` removes the element from flow and positions it against the nearest ancestor with a `position` other than `static` (or a transform/filter/contain). Without one, it uses the initial containing block.

```css
.card {
  position: relative;
}

.card__badge {
  position: absolute;
  top: 8px;
  right: 8px;
}
```

Output:

```text
Badge sits 8px from the card's top-right corner,
out of flow, not affecting layout.
```

Think:

```text
nearest positioned ancestor
    ↓
padding box becomes the containing block
    ↓
top/right/bottom/left resolve against it
```

Interview trap: `absolute` is not "relative to the page" — it is relative to the nearest positioned ancestor. Forgetting `position: relative` on the parent sends it to the viewport.

---

# 3. position: fixed and sticky

`fixed` positions against the viewport and stays put while scrolling. `sticky` behaves like `relative` until a threshold, then sticks within its scrolling container.

```css
.header {
  position: fixed;
  inset: 0 0 auto 0;
  height: 64px;
}

.toc {
  position: sticky;
  top: 1rem;
  align-self: start;
}
```

Output:

```text
Header pinned to the top of the viewport;
TOC sticks 1rem from the top while its column scrolls.
```

Sticky requirements:

- A `top`/`bottom`/`left`/`right` threshold.
- A scrolling ancestor.
- No `overflow: hidden` ancestor that clips it.

Interview trap: `sticky` fails silently if an ancestor has `overflow: hidden/auto` or if the element has no room to move.

---

# 4. inset Properties

`inset` is shorthand for `top/right/bottom/left`. `inset-inline`/`inset-block` are logical equivalents that adapt to writing direction.

```css
.overlay {
  position: fixed;
  inset: 0;
}

.badge {
  position: absolute;
  inset-block-start: 8px;
  inset-inline-end: 8px;
}
```

Output:

```text
Overlay covers the viewport; badge sits top-right
(logical properties flip in RTL).
```

`inset: 0` plus `position: fixed` is the standard full-screen overlay. `inset: 0` plus `margin: auto` with fixed width/height centers an absolute element.

---

# 5. z-index and Stacking Contexts

`z-index` orders elements within the same stacking context. A new stacking context is created by positioned elements with `z-index`, `opacity < 1`, `transform`, `filter`, `will-change`, `isolation: isolate`, and more.

```css
.modal {
  position: fixed;
  z-index: 100;
}

.header {
  position: sticky;
  z-index: 10;
  transform: translateZ(0); /* creates a stacking context */
}
```

Output:

```text
Modal above header if both are in the same context;
a transformed ancestor can trap the modal below other elements.
```

Think:

```text
stacking context
  ├── child z-index 1
  ├── child z-index 2
  └── child z-index 3
        │
        └── cannot escape above a sibling context with higher z-index
```

Interview trap: `z-index: 9999` does nothing outside its stacking context. Fix the context, not the number.

---

# 6. Positioning Patterns

Common patterns: full-screen overlay, centered modal, badge on a card, sticky header, and a fixed footer.

```css
/* Centered modal */
.modal {
  position: fixed;
  inset: 0;
  margin: auto;
  width: min(90vw, 480px);
  height: fit-content;
  z-index: 100;
}

/* Overlay behind it */
.backdrop {
  position: fixed;
  inset: 0;
  background: rgb(15 23 42 / 0.5);
  z-index: 90;
}

/* Badge */
.card { position: relative; }
.card__badge { position: absolute; top: 8px; right: 8px; }
```

Output:

```text
Backdrop covers the page; modal centers on top;
badge anchors to its card.
```

Prefer `dialog` + `showModal()` for real modals — it handles focus trapping, backdrop, and Escape natively.

---

# Real-World Uses

A sticky header, relative card, absolute badge, and fixed action bar combine without layout surprises.

```css
.site-header {
  position: sticky;
  top: 0;
  z-index: 20;
  background: #fff;
  border-bottom: 1px solid #e2e8f0;
}

.card {
  position: relative;
}

.card__badge {
  position: absolute;
  inset-block-start: 8px;
  inset-inline-end: 8px;
}

.action-bar {
  position: fixed;
  inset-inline: 0;
  inset-block-end: 0;
  padding: 1rem;
}
```

Output:

```text
Header sticks, badges anchor, action bar pins to the bottom.
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **static** | Default normal flow |
| **relative** | Keeps space, shifts visually |
| **absolute** | Out of flow, nearest positioned ancestor |
| **fixed** | Viewport-relative |
| **sticky** | Relative until threshold, then sticks |
| **inset** | Shorthand for top/right/bottom/left |
| **Containing block** | Positioned ancestor (or transform/filter/contain) |
| **z-index** | Order within a stacking context |
| **Stacking contexts** | Transform/opacity/filter/z-index create them |
| **Patterns** | Overlay, centered modal, badge, sticky header |

## The Mental Model

```text
position
  ├── static → flow
  ├── relative → flow + offsets + containing block
  ├── absolute → nearest positioned ancestor
  ├── fixed → viewport
  └── sticky → flow until threshold
        │
        └── z-index only compares within the same stacking context
```

### The 3 things to remember

**1. Absolute needs a positioned parent:**

```css
.card { position: relative; }
.card__badge { position: absolute; top: 8px; right: 8px; }
```

**2. Fix contexts, not z-index numbers:**

```css
.modal { position: fixed; z-index: 100; }
```

**3. inset:0 for overlays:**

```css
.overlay { position: fixed; inset: 0; }
```

> **Positioning is relative to a containing block, and z-index is relative to a stacking context — fix those two references and overlays behave.**
