# Transitions and Transforms — Revision Notes (Intermediate)
> One-line mental model: transitions interpolate state changes; transforms move geometry — animate transform/opacity and honor reduced motion.

## Core recall
- Transition needs start/end values, an animatable property, duration, and a trigger.
- `transition` shorthand = property, duration, timing function, delay.
- Prefer explicit properties over `transition: all`.
- Timing functions: `ease`, `linear`, `ease-in`, `ease-out`, `ease-in-out`, `cubic-bezier()`, `steps()`.
- 2D transforms: `translate`, `scale`, `rotate`, `skew`; they do not affect layout.
- 3D: `perspective` on parent, `transform-style: preserve-3d` on the inner element.
- `transform-origin` sets the pivot for scale/rotate.
- Animate `transform` and `opacity` for compositor-friendly motion.
- Transforms create a stacking context and containing block.
- Wrap non-essential motion in `@media (prefers-reduced-motion: reduce)`.

## Must-remember code
```css
.card {
  transition: transform 180ms ease, box-shadow 180ms ease;
}
.card:hover,
.card:focus-visible {
  transform: translateY(-4px) scale(1.02);
  box-shadow: 0 12px 24px rgb(15 23 42 / 0.12);
}
```
```css
.flip-card { perspective: 800px; }
.flip-card__inner {
  transform-style: preserve-3d;
  transition: transform 500ms ease;
}
.flip-card:hover .flip-card__inner {
  transform: rotateY(180deg);
}
```
```css
@media (prefers-reduced-motion: reduce) {
  .card, .drawer { transition: none; }
}
```

## Interview traps
- `transition: all` causing jank and surprises.
- Animating width/height/top/left instead of transform.
- Wrong placement of `perspective`/`preserve-3d`.
- Forgetting that transforms create stacking contexts.
- Removing focus styles during transitions.
- Ignoring reduced-motion preferences.
- Scale blur on text at fractional values.

## 3 things to remember
1. Explicit transition properties.
2. Transform/opacity for smooth motion.
3. Reduced motion and visible focus always.
