# Functions — Revision Notes (Beginner)

> One-line mental model: A function is a reusable block **and** a value — define it, pass it, return it.

## Core recall

- **Declaration**: `function g() {}` — hoisted with body, callable before its line.
- **Expression**: `const g = function () {}` — TDZ until assignment; `var` version is `undefined` and throws `TypeError` if called early.
- **Arrow**: `(a) => a * 2`; no own `this`, no `arguments`, no `new`, no `prototype`; object return needs `() => ({ a: 1 })`; single param may drop parens.
- **Defaults**: fire only for **`undefined`** (`null`, `0`, `""` skip them), evaluated left-to-right at call time; later params can reference earlier ones.
- **Rest**: `...args` → **real array**, must be last parameter; replaces `arguments`.
- **Return**: no `return` → `undefined`; `return` stops immediately; a newline after `return` triggers ASI and returns `undefined`.
- **First-class**: store in variables/objects/arrays, pass as arguments, return from functions. Pass `fn`, not `fn()`.
- **Higher-order**: takes a function, returns a function, or both (`map`, `debounce`, `withLogging`).
- **Callbacks**: run later (timers, events) or per item (`map`); callbacks receive `(element, index, array)`.
- **IIFE**: `(() => { ... })()` — runs once, creates private scope; wrapping parens force an expression.
- **Pure**: same input → same output, no side effects (no mutation, I/O, `random`, `Date.now`).
- **Recursion**: base case + smaller step; every call is a stack frame; deep recursion → `RangeError`; no tail-call optimisation.
- **arguments**: array-like, `length` + indexes, no array methods, absent in arrows, not linked to params in strict mode.

## Must-remember code

```javascript
// declaration vs expression
greet();                       // works (hoisted)
function greet() { return "Hi Kakade"; }
```

```javascript
// defaults + rest
const sum = (start = 0, ...nums) => nums.reduce((a, n) => a + n, start);
sum(undefined, 1, 2); // 3
```

```javascript
// higher-order / closure
const once = (fn) => {
  let done = false, result;
  return (...args) => (done ? result : ((done = true), (result = fn(...args))));
};
```

## Interview traps

- Default params do not apply to `null` → `f(null)` keeps `null`; `5 + null` is `5`.
- `return` followed by a newline returns `undefined`.
- Arrow returning an object without parens: `() => { a: 1 }` → `undefined`.
- `setTimeout(fn(), 0)` calls now and schedules `undefined` → `setTimeout(fn, 0)`.
- `[1,2,3].map(console.log)` passes `(value, index, array)` — wrap the callback.
- `arguments` is not an array (`Array.isArray(arguments)` is `false`) and does not exist in arrows.
- Arrows cannot be constructors (`new (() => {})` throws) and `call`/`bind` cannot rebind their `this`.
- A function with `{}` body and no `return` returns `undefined`.
- Pure-looking helpers that `push` into their argument are impure.
- Deep recursion overflows the stack; convert to a loop for linear depth.

## 3 things to remember

1. **Pass the function, don't call it.**
   ```javascript
   setTimeout(() => greet("Kakade"), 1000); // later
   setTimeout(greet("Kakade"), 1000);       // now
   ```

2. **Defaults are for `undefined` only.**
   ```javascript
   const f = (n = 10) => n;
   f(undefined); // 10
   f(null);      // null
   ```

3. **Arrow vs function.**
   ```javascript
   const obj = { n: 1, a: () => this?.n, m() { return this.n; } };
   obj.m(); // 1 — arrow has no own this
   ```
