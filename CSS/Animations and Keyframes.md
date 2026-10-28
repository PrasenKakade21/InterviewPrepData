# Animations and Keyframes

Transitions need a state change. `@keyframes` lets you define multi-step motion that runs on its own: spinners, pulses, entrance effects, and attention cues. The interview focus is fill-mode, iteration, direction, and respecting reduced motion.

The core idea:

```text
@keyframes name
    ↓
animation: name duration timing ...
    ↓
browser interpolates between keyframe steps
```

---

# 1. @keyframes

`@keyframes` declares named steps with percentages (or `from`/`to`). Each step lists the properties to animate.

```css
@keyframes fade-up {
  from {
    opacity: 0;
    transform: translateY(12px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.hero {
  animation: fade-up 400ms ease-out both;
}
```

Output:

```text
Hero fades in and moves up 12px over 400ms.
```

Keyframes can be reused; only animatable properties interpolate. `transform` and `opacity` are the safe performance choices.

---

# 2. animation-name, duration and iteration-count

The `animation` shorthand combines name, duration, timing function, delay, iteration count, direction, fill mode, and play state.

```css
.spinner {
  animation: spin 800ms linear infinite;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}

.pulse {
  animation: pulse 2s ease-in-out 3;
}

@keyframes pulse {
  50% { opacity: 0.5; }
}
```

Output:

```text
Spinner rotates forever; pulse fades three times.
```

| Longhand | Purpose |
| -------- | ------- |
| `animation-name` | Keyframes name |
| `animation-duration` | One cycle length |
| `animation-timing-function` | Easing |
| `animation-delay` | Start delay |
| `animation-iteration-count` | Number or `infinite` |
| `animation-direction` | normal/reverse/alternate/alternate-reverse |
| `animation-fill-mode` | before/after state |
| `animation-play-state` | running/paused |

---

# 3. animation-direction and fill-mode

`direction` controls whether cycles play forward, backward, or alternate. `fill-mode` decides which values apply before/after the animation.

```css
@keyframes slide-in {
  from { transform: translateX(-100%); }
  to { transform: translateX(0); }
}

.drawer {
  animation: slide-in 300ms ease-out both;
}

.breathe {
  animation: breathe 4s ease-in-out infinite alternate;
}
```

Output:

```text
Drawer stays at translateX(0) after finishing (both).
Breathe alternates forward/backward forever.
```

| fill-mode | Before | After |
| --------- | ------ | ----- |
| `none` | Normal styles | Normal styles |
| `forwards` | Normal | Last keyframe |
| `backwards` | First keyframe (during delay) | Normal |
| `both` | First | Last |

Interview trap: without `forwards`/`both`, an entrance animation snaps back to its pre-animation state when it ends.

---

# 4. Multiple Animations and play-state

Comma-separate animations to run several at once. `animation-play-state: paused` stops without resetting.

```css
.card {
  animation:
    fade-up 400ms ease-out both,
    glow 2s ease-in-out infinite alternate;
}

.paused {
  animation-play-state: paused;
}
```

Output:

```text
Card fades in while glowing; .paused freezes the glow at its current frame.
```

`animation-play-state` is useful for hover-to-pause carousels; JavaScript can toggle it.

---

# 5. prefers-reduced-motion and Performance

Large or infinite motion can cause dizziness and seizures. Respect `prefers-reduced-motion`; keep essential state changes but remove decoration.

```css
@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
}
```

Output:

```text
Animations effectively finish instantly; auto-scroll becomes instant.
```

Performance rules:

- Animate `transform`/`opacity`, not layout properties.
- Avoid infinite animations on large areas.
- Pause offscreen animations when possible.
- Use `will-change` sparingly and remove it after.

---

# Real-World Uses

A toast enters with fade-up, pulses subtly, and respects reduced motion.

```css
@keyframes toast-in {
  from { opacity: 0; transform: translateY(8px); }
  to   { opacity: 1; transform: translateY(0); }
}

.toast {
  animation: toast-in 250ms ease-out both;
  padding: 0.75rem 1rem;
  border-radius: 0.5rem;
  background: #0f172a;
  color: #fff;
}

.toast--attention {
  animation:
    toast-in 250ms ease-out both,
    pulse 1.5s ease-in-out 2;
}

@media (prefers-reduced-motion: reduce) {
  .toast,
  .toast--attention {
    animation: none;
  }
}
```

Output:

```text
Toast slides in, optionally pulses twice,
and appears instantly with reduced motion.
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **@keyframes** | Named multi-step animation |
| **name and duration** | Which animation, how long |
| **iteration-count** | Number or infinite |
| **direction** | normal/reverse/alternate |
| **fill-mode** | before/after state (`both` keeps end) |
| **Multiple animations** | Comma-separated |
| **play-state** | running/paused |
| **Reduced motion** | Disable decorative animation |
| **Performance** | transform/opacity only |

## The Mental Model

```text
animation
  ├── @keyframes → steps
  ├── timing → duration, easing, delay
  ├── repetition → iteration, direction
  ├── persistence → fill-mode
  └── control → play-state + reduced motion
```

### The 3 things to remember

**1. Keep the end state:**

```css
animation: fade-up 400ms ease-out both;
```

**2. Alternate for breathing:**

```css
animation: breathe 4s ease-in-out infinite alternate;
```

**3. Respect reduced motion:**

```css
@media (prefers-reduced-motion: reduce) {
  .toast { animation: none; }
}
```

> **Keyframes define multi-step motion — control repetition and fill-mode, animate transform/opacity, and always offer a reduced-motion path.**
