# Transitions and Transforms

Transitions animate property changes between two states. Transforms move, scale, rotate, and skew elements without affecting layout. Together they cover most hover effects, drawers, and micro-interactions.

The core idea:

```text
state change
    ↓
transition interpolates properties
    ↓
transform changes geometry (composited)
```

Transitions need a start value, an end value, a duration, and a trigger (usually `:hover`, `:focus`, or a class).

---

# 1. transition-property, duration and delay

`transition` is shorthand for property, duration, timing function, and delay. Only animatable properties transition; `display` and `height: auto` are special cases.

```css
.button {
  background-color: #0f172a;
  transition: background-color 150ms ease, transform 150ms ease;
}

.button:hover {
  background-color: #334155;
  transform: translateY(-1px);
}
```

Output:

```text
Background and lift animate over 150ms on hover.
```

| Longhand | Purpose |
| -------- | ------- |
| `transition-property` | Which properties |
| `transition-duration` | How long |
| `transition-timing-function` | Easing |
| `transition-delay` | Wait before starting |

Gotchas: `transition: all` is convenient but expensive and can animate unexpected properties; list properties explicitly.

---

# 2. Timing Functions

Easing controls the speed curve: `ease`, `linear`, `ease-in`, `ease-out`, `ease-in-out`, `cubic-bezier()`, `steps()`.

```css
.panel {
  transition: transform 250ms cubic-bezier(0.2, 0.8, 0.2, 1);
}

.steps {
  transition: opacity 400ms steps(4, end);
}
```

Output:

```text
Panel slides with a fast-start, soft-landing ease;
steps animation changes in four discrete jumps.
```

Interview trap: `ease-in` starts slow (good for exits), `ease-out` starts fast (good for entrances). `linear` feels mechanical for UI motion except for spinners/progress.

---

# 3. 2D Transforms

`transform` applies translate, scale, rotate, and skew. Multiple functions compose left to right.

```css
.card:hover {
  transform: translateY(-4px) scale(1.02);
}

.icon {
  transform: rotate(45deg);
}

.skewed {
  transform: skewX(-8deg);
}
```

Output:

```text
Card lifts and grows slightly; icon rotated 45°; box slanted.
```

| Function | Effect |
| -------- | ------ |
| `translate(x, y)` | Move |
| `scale(x, y)` | Resize |
| `rotate(angle)` | Rotate |
| `skew(x, y)` | Slant |
| `matrix(...)` | Combined 2D matrix |

Gotchas: transforms do not affect layout — surrounding elements stay in place. `scale` can blur text at fractional values.

---

# 4. 3D Transforms and transform-origin

`translate3d`, `rotateX/Y/Z`, `perspective`, and `transform-style: preserve-3d` enable 3D effects. `transform-origin` sets the pivot point.

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

.scale-from-corner {
  transform-origin: top left;
}
```

Output:

```text
Card flips in 3D around its vertical axis; scaling pivots from top-left.
```

Interview trap: `perspective` belongs on the parent; `transform-style: preserve-3d` must be on the element whose children are 3D.

---

# 5. Performance, Hover/Focus and Reduced Motion

Animate `transform` and `opacity` — they can be composited without layout. Avoid animating width, height, top, or left. Respect `prefers-reduced-motion`.

```css
.drawer {
  transition: transform 250ms ease;
  transform: translateX(-100%);
}

.drawer[data-open="true"] {
  transform: translateX(0);
}

@media (prefers-reduced-motion: reduce) {
  .drawer,
  .button {
    transition: none;
  }
}
```

Output:

```text
Drawer slides in; with reduced motion it appears instantly.
```

Gotchas: `transition: all` on a transformed element can create stacking contexts mid-animation; focus styles must remain visible during transitions.

---

# Real-World Uses

A card hover and a slide-in drawer use transforms, transitions, and a reduced-motion fallback.

```css
.card {
  transition: transform 180ms ease, box-shadow 180ms ease;
}

.card:hover,
.card:focus-visible {
  transform: translateY(-4px);
  box-shadow: 0 12px 24px rgb(15 23 42 / 0.12);
}

.drawer {
  position: fixed;
  inset: 0 auto 0 0;
  width: min(320px, 85vw);
  transform: translateX(-100%);
  transition: transform 250ms ease;
}

.drawer[data-open="true"] {
  transform: translateX(0);
}

@media (prefers-reduced-motion: reduce) {
  .card,
  .drawer {
    transition: none;
  }
}
```

Output:

```text
Cards lift on hover/focus; drawer slides in; reduced motion removes animation.
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **transition-property** | Which properties animate |
| **duration and delay** | How long, when to start |
| **Timing functions** | Easing curve |
| **2D transforms** | translate, scale, rotate, skew |
| **3D transforms** | perspective, rotateX/Y/Z, preserve-3d |
| **transform-origin** | Pivot point |
| **Performance** | Animate transform/opacity |
| **Hover and focus** | Trigger states, keep focus visible |
| **Reduced motion** | Disable/limit animation |

## The Mental Model

```text
state change
  ├── transition → property, duration, easing, delay
  ├── transform → translate/scale/rotate/skew
  └── constraints → transform/opacity only + reduced motion
```

### The 3 things to remember

**1. Explicit properties:**

```css
transition: transform 180ms ease, box-shadow 180ms ease;
```

**2. Transform for motion:**

```css
.card:hover { transform: translateY(-4px); }
```

**3. Respect reduced motion:**

```css
@media (prefers-reduced-motion: reduce) {
  .card { transition: none; }
}
```

> **Transitions interpolate state changes; transforms move without layout — animate transform/opacity and honor reduced-motion.**
