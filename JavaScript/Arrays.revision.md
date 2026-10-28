# Arrays — Revision Notes (Beginner)

> One-line mental model: An ordered list you create, index, mutate with a few methods, and transform with `map` / `filter` / `reduce`.

## Core recall

- **Create**: literal wins. `Array(3)` = 3 **holes**, `Array(3, 4)` = two items, `Array.of(3)` = `[3]`. `typeof []` is `"object"` → use `Array.isArray`.
- **Index/length**: 0-based. `arr[-1]` is `undefined`; use `at(-1)`. Writing `length` truncates or extends. Reading past the end is `undefined`, not an error.
- **push/pop**: end. `push` → **new length**; `pop` → removed element. O(1).
- **shift/unshift**: front. Same returns as pop/push, but **O(n)** — every index moves.
- **splice(start, deleteCount, ...items)**: **mutates**, returns **removed** items; can insert/replace.
- **slice(start, end)**: copies, **end excluded**, negatives count from the end, never mutates.
- **concat**: new array, flattens one level of array arguments. **join**: string, default `","`.
- **includes**: boolean, finds `NaN` (SameValueZero). **indexOf**: index or `-1`, uses `===`, cannot find `NaN`.
- **find/findIndex**: first match; `undefined` / `-1` otherwise. Never assume a match — use `?.`.
- **filter**: keeps truthy results, new array (may be `[]`), never mutates.
- **map**: one output per input, same length. Callback gets `(element, index, array)` → `map(parseInt)` gives `[1, NaN, NaN]`.
- **forEach**: returns `undefined`, not chainable — it is for side effects.
- **reduce**: folds to one value; callback `(acc, cur, idx, arr)`. **Always pass an initial value**; `[]` without one throws `TypeError`.
- **some/every**: short-circuit. Empty array: `some` → `false`, `every` → `true` (vacuous).
- **sort**: mutates, returns the same array, **lexicographic by default** (`[10,9,1].sort()` → `[1,10,9]`). Numeric: `(a, b) => a - b`. Stable.
- **flat(depth = 1)** / **flatMap**: `flatMap` flattens exactly one level (`[[n]]` stays nested).
- **Array.from(iterable, mapper)**: strings, Sets, array-likes; `{ length: n }` builds filled ranges.
- **Copying**: `=` copies the reference; `[...a]` / `slice()` are **shallow**; `structuredClone` is deep.
- **Mutating**: push, pop, shift, unshift, splice, sort, reverse, fill, copyWithin. **Non-mutating**: slice, concat, map, filter, flat, flatMap, join, toSorted, toReversed, toSpliced, with.

## Must-remember code

```javascript
// numeric sort, without mutating
const sorted = [...nums].sort((a, b) => a - b);
// ES2023: const sorted = nums.toSorted((a, b) => a - b);
```

```javascript
// dedupe, order preserved
const unique = [...new Set(items)];
// filled range
const range = Array.from({ length: 5 }, (_, i) => i); // [0,1,2,3,4]
```

```javascript
// group with reduce + initial value
const byStatus = orders.reduce((acc, o) => {
  (acc[o.status] ??= []).push(o);
  return acc;
}, {});
```

## Interview traps

- `sort()` on numbers sorts as **strings** and mutates → always pass a comparator.
- `splice` vs `slice`: mutate + removed vs copy + exclusive end.
- `map(parseInt)` → `[1, NaN, NaN]` because the index becomes the radix.
- `forEach` returns `undefined` → `const x = arr.forEach(...)` is a bug.
- `reduce` without an initial value on `[]` throws `TypeError`.
- `new Array(3).map(...)` stays holes; use `Array.from({ length: 3 }, ...)`.
- `push` returns the length, not the array; `find` returns `undefined`, not `-1`.
- `[1, NaN].indexOf(NaN)` is `-1`, but `.includes(NaN)` is `true`.
- `[...a]` copies only one level; nested objects stay shared.
- `[].every(...)` is `true`, `[].some(...)` is `false`.

## 3 things to remember

1. **Default sort is lexicographic.**
   ```javascript
   [10, 9, 1].sort();                // [ 1, 10, 9 ]
   [10, 9, 1].sort((a, b) => a - b); // [ 1, 9, 10 ]
   ```

2. **map returns, forEach does not.**
   ```javascript
   [1, 2].map((n) => n * 2);     // [2, 4]
   [1, 2].forEach((n) => n * 2); // undefined
   ```

3. **Give reduce an initial value.**
   ```javascript
   [].reduce((a, b) => a + b, 0); // 0 (without the 0: TypeError)
   ```
