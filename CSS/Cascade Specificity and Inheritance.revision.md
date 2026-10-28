# Cascade, Specificity and Inheritance — Revision Notes (Intermediate)
> One-line mental model: the cascade picks a winner by origin, layers, specificity, then order — fix that cause, not the symptom.

## Core recall
- Cascade order: user-agent normal < author normal < author !important < user !important; animations/transitions can also outrank.
- Specificity tuple: (ids, classes/attributes/pseudo-classes, elements/pseudo-elements), compared left to right.
- `!important` only for external/uncontrollable CSS; otherwise fix specificity or use layers.
- Inherited by default: `color`, `font-*`, `line-height`, `visibility`, `cursor`, `letter-spacing`.
- Not inherited: `margin`, `padding`, `border`, `width`, `height`, `display`, `background`.
- Global keywords: `inherit`, `initial`, `unset`, `revert`, `revert-layer`.
- `@layer` order declaration sets priority; later layers beat earlier ones regardless of specificity.
- Unlayered normal styles generally beat layered normal styles.
- Shorthand properties reset unspecified longhands (e.g. `margin: 0 auto` zeroes top/bottom).
- Debug with Styles (struck-through losers), Computed (final value), and `:hov` state.

## Must-remember code
```css
@layer reset, base, components, utilities;

@layer components {
  .card { padding: 1rem; }
}
@layer utilities {
  .p-0 { padding: 0; } /* wins */
}
```
```css
.card { color: #0f172a; }
.card p { color: inherit; margin: initial; }
.reset { all: revert; }
```
```css
#hero .title { color: #b91c1c; } /* 1,1,0 */
```

## Interview traps
- Thinking specificity is a single added number.
- Reaching for `!important` against your own CSS.
- Forgetting layer order and blaming specificity.
- Expecting margin/padding to inherit.
- `all: unset` wiping typography.
- Shorthand resetting a longhand set earlier.
- Ignoring inline styles when debugging.

## 3 things to remember
1. Origin/layers → specificity → order.
2. Keep specificity flat; layers replace wars.
3. Inherit/initial/unset/revert are explicit controls, not magic.
