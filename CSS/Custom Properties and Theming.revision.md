# Custom Properties and Theming — Revision Notes (Intermediate)
> One-line mental model: custom properties are inherited tokens — declare semantic names at :root, override for themes, read with var().

## Core recall
- Custom properties `--name: value` work on any element and participate in cascade/inheritance.
- They are live at runtime, unlike Sass variables which compile away.
- `var(--name, fallback)` uses the fallback when missing/invalid.
- Names are case-sensitive; values are mostly untyped strings unless registered.
- Inheritance enables subtree theming: override tokens on `[data-theme]`.
- Dark mode: default tokens + `@media (prefers-color-scheme: dark)` + explicit `data-theme` override.
- Set `color-scheme` so native controls/scrollbars match the theme.
- `calc()` combines numeric/length tokens.
- `@property` registers syntax/inheritance/initial value and enables interpolation.
- Invalid values fail the declaration at computed-value time; fallbacks do not rescue invalid present values.
- Never store secrets in CSS — it is public.

## Must-remember code
```css
:root {
  --color-bg: #ffffff;
  --color-surface: #f8fafc;
  --color-text: #0f172a;
  --radius: 12px;
}
.card {
  background: var(--color-surface);
  color: var(--color-text);
  border-radius: var(--radius, 8px);
}
```
```css
[data-theme="dark"] {
  --color-bg: #0f172a;
  --color-surface: #1e293b;
  --color-text: #e2e8f0;
}
```
```css
@property --angle {
  syntax: "<angle>";
  inherits: false;
  initial-value: 0deg;
}
```
```css
.grid { gap: calc(var(--gap) * 2); }
```

## Interview traps
- Thinking custom properties are compile-time like Sass.
- Expecting var() fallback to fix a present-but-invalid value.
- Forgetting inheritance when theming.
- Not setting `color-scheme` for native controls.
- Trying to animate unregistered custom properties.
- Storing sensitive values in CSS.
- Raw color names instead of semantic tokens (`--color-text` not `--dark-blue`).

## 3 things to remember
1. Semantic tokens at `:root`, override in a subtree.
2. `var()` fallback + `@property` for typed values.
3. Theme via data attributes + `prefers-color-scheme`.
