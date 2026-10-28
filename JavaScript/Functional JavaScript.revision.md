# Functional JavaScript — Revision Notes (Intermediate)

> One-line mental model: pure functions over immutable data, composed into pipelines, with effects pushed to the edges.

## Core recall

- **Pure functions**: same input → same output; no external reads/writes. `add(a, b)` is pure; a function that mutates `total` is not.
- **Side effects**: logging, I/O, DOM, clock, random, mutation. Keep them at the edges; pure core is easy to test.
- **Immutability**: copy before changing. Spread/`toSorted`/`toReversed`/`with`; `Object.freeze` is shallow and silent outside strict mode.
- **map**: transform every element, same length. **filter**: keep matches. **reduce**: fold to one value; empty array without initial throws.
- **Higher-order functions**: take and/or return functions — `map`, `filter`, wrappers, `setTimeout`.
- **Composition**: `compose` right-to-left, `pipe` left-to-right; both are `reduce` over functions.
- **Pipelines**: named stages composed into a reusable flow; each stage unary.
- **Currying**: `f(a)(b)(c)`; generic version compares `args.length` with `fn.length`.
- **Partial application**: fix some arguments now (`bind`, closure), supply the rest later.
- **Declarative vs imperative**: say what, not how; chain transforms instead of manual loops.
- **Performance tradeoffs**: each chain link allocates; `reduce` is one pass; loops can be faster in hot paths.

## Must-remember code

```javascript
// pipe, left to right
const pipe = (...fns) => (x) => fns.reduce((acc, fn) => fn(acc), x);
const slugify = pipe((s) => s.trim(), (s) => s.toLowerCase(), (s) => s.replace(/\s+/g, "-"));
```

```javascript
// reduce grouping
const byRole = users.reduce((acc, u) => {
  (acc[u.role] ??= []).push(u.name);
  return acc;
}, {});
```

```javascript
// currying
const volume = (l) => (w) => (h) => l * w * h;
volume(2)(3)(4); // 24
```

```javascript
// partial application
const greet = (g, n, p) => `${g}, ${n}${p}`;
const hi = greet.bind(null, "Hi");
hi("Kakade", "!"); // Hi, Kakade!
```

## Interview traps

- `[...arr].sort()` copies the array but **not the objects**; nested values stay shared.
- `arr.sort()`, `push`, `splice` mutate the input — use `toSorted`, spread, or `toSpliced`.
- `[].reduce((a, b) => a + b)` throws without an initial value.
- `["1","2","3"].map(parseInt)` → `[1, NaN, NaN]`; map passes the index as radix.
- `Object.freeze` is shallow: nested objects remain editable.
- `compose` runs right-to-left while `pipe` runs left-to-right — mixing them reverses the pipeline.
- Currying with `fn.length` breaks when defaults or rest parameters are present.
- `bind` fixes `this` as well as arguments; a closure is often clearer.
- Chained `filter().map()` makes two passes; don't call it “more efficient” than a loop.

## 3 things to remember

1. **Pure + immutable.**
   ```javascript
   const add = (a, b) => a + b;          // pure
   const next = [...items, item];        // new array
   ```

2. **Compose small stages.**
   ```javascript
   const slugify = pipe(trim, lower, dash);
   ```

3. **Curry vs partial.**
   ```javascript
   const volume = (l) => (w) => (h) => l * w * h; // curried
   const hi = greet.bind(null, "Hi");             // partially applied
   ```
