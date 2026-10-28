# Closures and the Scope Chain — Revision Notes (Intermediate)

> One-line mental model: A closure is a function that carries its birth environment — lexical scope decides what it can see, and it keeps that scope alive.

## Core recall

- **Lexical scope**: resolution depends on **where the function was written**, not where it is called. Lookup walks inner → outer → module → global; a missing name is `ReferenceError`, not `undefined`.
- **Nested functions**: inner sees outer variables, outer never sees inner. Each call of the outer function creates a **fresh environment**.
- **Closure**: function + captured environment. The environment survives after the outer function returns, for as long as the function is reachable.
- **Counter**: `let count` in the factory, returned methods increment it. Separate factory calls → separate state.
- **Private state**: captured variables are not properties — no external read/write, not enumerable, not serialized. Classes do the same with `#field`.
- **Module pattern**: IIFE creates state, returns only the public API. Return copies (`[...items]`) so callers can't mutate internals.
- **Loop bug**: `var` = one shared binding → all callbacks see the final value (`3 3 3`). `let` = per-iteration binding (`0 1 2`). IIFE or `forEach` fix the old style.
- **Event handlers**: each listener closes over its own element/iteration value; clean up or the element is retained.
- **Memoization**: `Map` cache in a closure; check `cache.has(k)` (a cached `0` is falsy); recurse through the **memoized** reference.
- **Memory**: captured environments stay alive while the function is reachable. Long-lived listeners, uncleared timers and unbounded caches leak; `WeakMap`/`WeakSet` let keys be collected.
- **`this`**: closures do not capture `this`; arrows inherit it lexically, normal functions get their own (undefined in strict/module code).

## Must-remember code

```javascript
function createCounter() {
  let count = 0;
  return () => ++count;
}
const next = createCounter(); // 1, 2, 3 — other() starts at 1
```

```javascript
const memoize = (fn) => {
  const cache = new Map();
  return (n) => {
    if (cache.has(n)) return cache.get(n);
    const out = fn(n);
    cache.set(n, out);
    return out;
  };
};
```

```javascript
const debounce = (fn, delay) => {
  let timer;
  return (...args) => {
    clearTimeout(timer);
    timer = setTimeout(() => fn(...args), delay);
  };
};
```

```javascript
for (let i = 0; i < 3; i++) setTimeout(() => console.log(i)); // 0 1 2
// with var: 3 3 3 — one shared binding
```

## Interview traps

- "Closure captures the value" is false — it captures the **variable**; later writes are visible (`n = 5` before returning `get` gives 5).
- `var` in a loop shares one binding; `let` does not.
- A closure does not capture `this`; use an arrow to inherit it or `bind` explicitly.
- `cache.get(k)` truthiness check breaks for cached `0`, `""`, `false` — use `has`.
- Memoized recursion that calls the original function by name bypasses the cache.
- Returning the internal array directly exposes private state — return a copy.
- A closure retaining a DOM node after removal is a leak, not just a reference.
- `setInterval` without `clearInterval` keeps its closure (and everything in it) alive forever.
- Module-pattern variables are private by scope, not by naming convention (`_x` is still public).

## 3 things to remember

1. **Scope is decided at definition.**
   ```javascript
   const x = "outer";
   function f() { return x; }
   f(); // "outer"
   ```

2. **`let` fixes the loop bug.**
   ```javascript
   for (var i = 0; i < 3; i++) setTimeout(() => console.log(i)); // 3 3 3
   for (let j = 0; j < 3; j++) setTimeout(() => console.log(j)); // 0 1 2
   ```

3. **Private state is just a captured variable.**
   ```javascript
   const counter = (() => { let n = 0; return () => ++n; })();
   ```
