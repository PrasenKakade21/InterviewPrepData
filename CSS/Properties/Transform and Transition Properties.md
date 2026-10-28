# Transform and Transition Properties

These properties move geometry and animate state changes. Transforms do not affect layout; transitions interpolate between values. This reference lists both families.

```text
transform & transition
  ├── transform → translate/scale/rotate/skew
  ├── origin/perspective → 3D reference
  ├── transition → property/duration/easing/delay
  └── performance → transform/opacity only
```

---

# transform

Applies one or more transform functions.

```css
transform: translateY(-4px);
transform: translateX(-50%) scale(1.02);
transform: rotate(45deg);
transform: skewX(-8deg);
transform: translate3d(0, 0, 0);
```

Gotchas: functions compose left to right; transforms create a stacking context and containing block.

---

# transform-origin

Sets the pivot for rotation/scaling.

```css
transform-origin: center;
transform-origin: top left;
transform-origin: 50% 100%;
```

Gotchas: changing origin changes the visual result, not the element's layout position.

---

# translate, rotate, scale (individual)

Independent transform properties applied in a fixed order (translate, rotate, scale).

```css
translate: 0 -4px;
rotate: 45deg;
scale: 1.02;
```

Gotchas: they compose with `transform`; order is not the same as writing them in one `transform` list.

---

# perspective and perspective-origin

Define 3D viewing depth for children.

```css
perspective: 800px;
perspective-origin: center;
```

```css
.flip-card {
  perspective: 800px;
}
.flip-card__inner {
  transform-style: preserve-3d;
  transition: transform 500ms ease;
}
.flip-card:hover .flip-card__inner {
  transform: rotateY(180deg);
}
```

Gotchas: `perspective` belongs on the parent; without it 3D rotations look flat.

---

# transform-style and backface-visibility

Keep children in 3D space and hide the back face.

```css
transform-style: preserve-3d;
backface-visibility: hidden;
```

Gotchas: `preserve-3d` must be on the element whose children are 3D; `backface-visibility` needs a 3D context.

---

# transition-property, duration, timing-function, delay

Controls which properties animate, how long, easing, and delay.

```css
transition-property: transform, opacity;
transition-duration: 180ms;
transition-timing-function: ease-out;
transition-delay: 40ms;
```

```css
.card {
  transition: transform 180ms ease, box-shadow 180ms ease;
}
```

Gotchas: `all` animates unexpected properties; not every property is animatable.

---

# transition shorthand

Combines the four longhands; multiple transitions are comma-separated.

```css
transition: transform 180ms ease, opacity 120ms linear 40ms;
```

Gotchas: shorthand resets omitted longhands; keep durations/delays aligned with property lists.

---

# Real-World Uses

A lift-on-hover card and a 3D flip.

```css
.card {
  transition: transform 180ms ease, box-shadow 180ms ease;
}

.card:hover,
.card:focus-visible {
  transform: translateY(-4px) scale(1.02);
  box-shadow: 0 12px 24px rgb(15 23 42 / 0.12);
}

.flip-card { perspective: 800px; }
.flip-card__inner {
  transform-style: preserve-3d;
  transition: transform 500ms ease;
}
.flip-card:hover .flip-card__inner {
  transform: rotateY(180deg);
}

@media (prefers-reduced-motion: reduce) {
  .card, .flip-card__inner { transition: none; }
}
```

---

# Quick Revision

| Property | Purpose |
| -------- | ------- |
| **transform** | Move/scale/rotate/skew |
| **transform-origin** | Pivot point |
| **translate/rotate/scale** | Individual transforms |
| **perspective** | 3D depth |
| **transform-style** | Preserve 3D children |
| **backface-visibility** | Hide back face |
| **transition longhands** | Property/duration/easing/delay |
| **transition shorthand** | Combined, comma-separated |

## The Mental Model

```text
transform
  ├── 2D → translate/scale/rotate/skew
  ├── 3D → perspective + preserve-3d
  └── origin → pivot

transition
  ├── property
  ├── duration
  ├── timing function
  └── delay
```

### The 3 things to remember

**1. Explicit transitions:**

```css
transition: transform 180ms ease, opacity 120ms ease;
```

**2. Perspective on parent:**

```css
.flip-card { perspective: 800px; }
```

**3. Reduced motion:**

```css
@media (prefers-reduced-motion: reduce) {
  .card { transition: none; }
}
```

> **Transforms change geometry without layout; transitions interpolate state — keep both explicit and compositor-friendly.**
