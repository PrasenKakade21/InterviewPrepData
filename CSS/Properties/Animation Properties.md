# Animation Properties

Animation properties run named keyframes with timing, repetition, direction, and fill behavior. This reference covers `@keyframes` and every `animation-*` longhand.

```text
animation
  ├── @keyframes → steps
  ├── name/duration → what and how long
  ├── timing/delay → shape and start
  ├── iteration/direction → repetition
  ├── fill-mode → before/after state
  └── play-state → run/pause
```

---

# @keyframes

Defines named steps with `from`/`to` or percentages.

```css
@keyframes fade-up {
  from { opacity: 0; transform: translateY(12px); }
  to   { opacity: 1; transform: translateY(0); }
}

@keyframes pulse {
  0%, 100% { opacity: 1; }
  50% { opacity: 0.5; }
}
```

Gotchas: only animatable properties interpolate; duplicate percentages override.

---

# animation-name and animation-duration

Select the keyframes and set one cycle length.

```css
animation-name: fade-up;
animation-duration: 400ms;
```

Gotchas: a missing keyframes name means no animation; duration must be non-zero to see it.

---

# animation-timing-function and animation-delay

Easing curve and start delay.

```css
animation-timing-function: ease-out;
animation-timing-function: cubic-bezier(0.2, 0.8, 0.2, 1);
animation-delay: 120ms;
```

Gotchas: negative delays start partway through the animation; easing applies per cycle.

---

# animation-iteration-count

Number of cycles or `infinite`.

```css
animation-iteration-count: 3;
animation-iteration-count: infinite;
```

Gotchas: infinite animations on large areas cost battery/CPU.

---

# animation-direction

Forward, backward, or alternating cycles.

```css
animation-direction: normal | reverse | alternate | alternate-reverse;
```

```css
.breathe {
  animation: breathe 4s ease-in-out infinite alternate;
}
```

Gotchas: `alternate` counts each forward/backward pass as an iteration.

---

# animation-fill-mode

Applies keyframe values before/after the animation.

```css
animation-fill-mode: none | forwards | backwards | both;
```

| Value | Before | After |
| ----- | ------ | ----- |
| `none` | Base | Base |
| `forwards` | Base | Last keyframe |
| `backwards` | First keyframe | Base |
| `both` | First | Last |

Gotchas: without `forwards`/`both`, entrance animations snap back.

---

# animation-play-state and shorthand

Pause/resume and combine all longhands.

```css
animation-play-state: running | paused;
animation: fade-up 400ms ease-out 100ms 1 normal both;
```

Gotchas: the shorthand resets omitted longhands; comma-separate multiple animations.

---

# Scroll-driven animations (progressive)

`animation-timeline` ties progress to scroll instead of time.

```css
@keyframes reveal {
  from { opacity: 0; transform: translateY(24px); }
  to   { opacity: 1; transform: translateY(0); }
}

.reveal {
  animation: reveal linear both;
  animation-timeline: view();
  animation-range: entry 0% cover 40%;
}
```

Gotchas: check support; always provide a readable non-animated fallback.

---

# Real-World Uses

An entrance, a pulse, and a reduced-motion override.

```css
@keyframes toast-in {
  from { opacity: 0; transform: translateY(8px); }
  to   { opacity: 1; transform: translateY(0); }
}

.toast {
  animation: toast-in 250ms ease-out both;
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

---

# Quick Revision

| Property | Purpose |
| -------- | ------- |
| **@keyframes** | Named steps |
| **animation-name** | Which keyframes |
| **animation-duration** | Cycle length |
| **timing-function/delay** | Easing/start |
| **iteration-count** | Cycles or infinite |
| **direction** | normal/reverse/alternate |
| **fill-mode** | Before/after state |
| **play-state** | Running/paused |
| **shorthand** | All longhands |
| **animation-timeline** | Scroll-driven progress |

## The Mental Model

```text
animation
  ├── what → name + @keyframes
  ├── when → duration, delay, timeline
  ├── shape → timing function, direction
  ├── how often → iteration count
  └── persistence → fill-mode
```

### The 3 things to remember

**1. Keep the end state:**

```css
animation: fade-up 400ms ease-out both;
```

**2. Alternate for loops:**

```css
animation: breathe 4s ease-in-out infinite alternate;
```

**3. Reduced motion:**

```css
@media (prefers-reduced-motion: reduce) {
  .toast { animation: none; }
}
```

> **Animations are keyframes plus timing, repetition, and fill — control the end state and always offer a reduced-motion path.**
