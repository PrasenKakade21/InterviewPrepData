# Animations and Keyframes — Revision Notes (Intermediate)
> One-line mental model: keyframes define multi-step motion — control repetition/fill, animate transform/opacity, and always offer reduced motion.

## Core recall
- `@keyframes name { from {} to {} }` or percentage steps.
- `animation` shorthand = name duration timing delay iteration direction fill-mode play-state.
- Transitions need a trigger; animations can run automatically and loop.
- `animation-direction`: normal, reverse, alternate, alternate-reverse.
- `fill-mode: both` applies the first keyframe during delay and keeps the last after finishing.
- Comma-separate animations to run several at once.
- `animation-play-state: paused` freezes without resetting.
- Animate `transform`/`opacity`; avoid layout properties and huge infinite animations.
- `prefers-reduced-motion` should disable decorative motion and smooth scrolling.
- Without `forwards`/`both`, elements snap back to their base styles.

## Must-remember code
```css
@keyframes fade-up {
  from { opacity: 0; transform: translateY(12px); }
  to   { opacity: 1; transform: translateY(0); }
}
.hero { animation: fade-up 400ms ease-out both; }
```
```css
.spinner {
  animation: spin 800ms linear infinite;
}
@keyframes spin {
  to { transform: rotate(360deg); }
}
```
```css
.card {
  animation:
    fade-up 400ms ease-out both,
    glow 2s ease-in-out infinite alternate;
}
```
```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    scroll-behavior: auto !important;
  }
}
```

## Interview traps
- Forgetting `fill-mode` and getting a snap-back.
- Confusing `alternate` with `reverse`.
- Animating width/height/top/left.
- Infinite full-page animations.
- Ignoring reduced-motion.
- Multiple animations without matching timing lists.
- Using `will-change` permanently.

## 3 things to remember
1. Keyframes + duration + fill-mode.
2. transform/opacity for performance.
3. Reduced motion is required, not optional.
