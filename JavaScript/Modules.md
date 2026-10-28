# Modules

A module is a file with its **own scope** and an explicit list of what it shares. The chain is **one file → explicit exports → imports elsewhere → one shared instance**. Modules make dependencies visible, avoid name collisions, and let tools drop unused code.

```text
Module system
     ↓
file scope          nothing leaks to global by default
     ↓
export              named / default / re-export
     ↓
import              static at top, or dynamic import()
     ↓
one instance        evaluated once, then cached
```

---

# 1. Scripts vs Modules

### Script vs module

A classic `<script>` runs in the **global scope**, shares variables with other scripts, and executes as soon as it is parsed. `<script type="module">` runs in **strict mode**, gets its own scope, is **deferred** until the HTML is parsed, and can use `import` / `export`.

```html
<script src="analytics.js"></script>
<script type="module" src="app.js"></script>

<script type="module">
  import { add } from "./math.js";
  console.log(add(2, 3));
</script>
```

Output:

```text
5
```

Differences worth memorising:

| | Classic script | Module |
| --- | --- | --- |
| Scope | global | file-local |
| Strict mode | optional | always |
| `this` at top level | `window` | `undefined` |
| `import` / `export` | no | yes |

Modules are deferred and need a JavaScript MIME type; `file://` module pages are blocked by CORS in most browsers.

### Module scope

Top-level variables in a module are **private to that file** unless exported. Two modules can both declare `const count` without conflict, and neither becomes a property of `window`.

```javascript
// counter.js
let count = 0;

export function increment() {
  count++;
  return count;
}
```

```javascript
// app.js
import { increment } from "./counter.js";

console.log(increment(), increment());
console.log(typeof count);
```

Output:

```text
1 2
undefined
```

`count` lives in `counter.js`'s module environment; `app.js` reaches it only through the exported function — module-level encapsulation.

---

# 2. ESM Syntax

### import and export

ES modules use **static** `import` and `export` declarations. They must be at the top level of the file, are hoisted before any other code runs, and the imported bindings are **live** — they reflect later changes in the exporting module.

```javascript
// math.js
export const add = (a, b) => a + b;
export function double(n) {
  return n * 2;
}
```

```javascript
// app.js
import { add, double } from "./math.js";
import * as math from "./math.js";
import "./side-effect.js";

console.log(add(2, 3), math.double(4));
```

Output:

```text
5 8
```

`import * as math` collects all named exports into a module namespace object. A bare `import "./side-effect.js"` runs the module for its side effects without binding anything. Imports are read-only views; assigning to `add` throws. Paths need the `./` prefix and, in Node ESM, the `.js` extension.

### Named exports

Named exports share specific bindings under fixed names. A module can have as many as it likes, and the importer must use the exported names (or rename them with `as`).

```javascript
// format.js
export const currency = (n) => `₹${n}`;
export function title(text) {
  return text.toUpperCase();
}
```

```javascript
// app.js
import { currency as money, title } from "./format.js";

console.log(money(499), title("kakade"));
```

Output:

```text
₹499 KAKADE
```

Named exports are preferred for libraries: renaming is explicit, editors can auto-import, and bundlers can tree-shake unused names. `export { currency, title };` at the bottom is equivalent to marking the declarations inline.

### Default exports

A module can have **one** default export. The importer chooses the local name, which is convenient but makes refactors and auto-import less predictable.

```javascript
// greet.js
export default function greet(name) {
  return `Hello ${name}`;
}
```

```javascript
// app.js
import greet from "./greet.js";
import hello from "./greet.js";

console.log(greet("Kakade"), hello("Asha"));
```

Output:

```text
Hello Kakade Hello Asha
```

Both imports name the same function. A default is sugar for `export { greet as default }` and can coexist with named exports. Bad: `export default { add, subtract }` — no tree-shaking. Good: export each function by name.

### Re-exports

A re-export forwards a binding from another module without importing it locally. This is how `index.js` barrel files expose a folder's public API. Assuming the `math.js` and `greet.js` modules above:

```javascript
// index.js
export { add } from "./math.js";
export { default as greet } from "./greet.js";
```

```javascript
// app.js
import { add, greet } from "./index.js";

console.log(add(1, 2), greet("Rohan"));
```

Output:

```text
3 Hello Rohan
```

`export * from "./math.js"` forwards every **named** export but **not** the default. Re-exports keep the public surface small, but deep barrels can slow builds and create cycles.

---

# 3. Loading Modules

### Dynamic import

`import()` is an **expression** that returns a promise for the module namespace. It can run conditionally, inside a function, with a computed path — unlike static `import`.

```javascript
// app.js
const loadMath = () => import("./math.js");

loadMath()
  .then(({ add }) => console.log(add(2, 3)))
  .catch((err) => console.log("load failed:", err.message));
```

Output:

```text
5
```

Use dynamic import for route-level code splitting, optional features, and polyfills. The path resolves relative to the current module, and the module is evaluated once no matter how many times it is imported dynamically. `await import(...)` works in an `async` function or at the top level of an ES module.

### Bundlers

A bundler follows the import graph, resolves paths, combines modules into output files, removes unused exports (**tree-shaking**), and rewrites asset URLs. Vite serves native ESM in development and uses Rollup for production; webpack and esbuild solve the same problem differently.

```javascript
// entry.js — the graph a bundler walks
import { add } from "./math.js";
import greet from "./greet.js";

console.log(add(20, 22), greet("Kakade"));
```

Output:

```text
42 Hello Kakade
```

The bundler emits `dist/assets/index-8f3a1.js`: three files became one and the unused `double` export was dropped. Bundlers earn their keep with many files, bare package imports, TypeScript/JSX, and hashed caching, and they split dynamic imports into separate chunks.

---

# 4. CommonJS

### CommonJS

CommonJS is Node's original module system: synchronous `require`, a `module.exports` object, and `exports` as a shortcut. It is still everywhere in older packages and config files.

```javascript
// counter.cjs
let hits = 0;

function track(name) {
  hits++;
  return `${name}:${hits}`;
}

module.exports = { track, getHits: () => hits };
console.log("counter loaded");
```

```javascript
// app.cjs
const a = require("./counter.cjs");
const b = require("./counter.cjs");

console.log(a === b, a.track("Kakade"), b.getHits());
```

Output:

```text
counter loaded
true Kakade:1 1
```

Notice: the module body ran **once**. `require` caches by resolved filename, so the second call returns the same object, which is why `counter loaded` appears once and `a === b` is `true`. `module.exports = {...}` replaces the exported object; `exports.track = track` adds a property to the default object. Never mix the two — reassigning `exports` breaks the link to `module.exports`.

### ESM vs CommonJS

The two systems differ in syntax, timing, and what importers receive. ESM is the standard for new code; CommonJS remains common in Node tooling.

| | ESM | CommonJS |
| --- | --- | --- |
| Syntax | `import` / `export` | `require` / `module.exports` |
| Loading | asynchronous graph, static | synchronous, on demand |
| Bindings | **live**, read-only views | value copied at require time |
| Top-level await | yes | no |
| Tree-shaking | yes | limited |
| File hints | `.mjs` or `"type": "module"` | `.cjs` or default |

```javascript
// counter.js
export let count = 0;
export function increment() {
  count++;
}
```

```javascript
// app.js
import { count, increment } from "./counter.js";

console.log(count);
increment();
console.log(count);
```

Output:

```text
0
1
```

The imported `count` is a **live binding**, not a snapshot: it shows the updated value after `increment()`. With CommonJS, `const { count } = require("./counter")` copies the number at require time and stays `0`. This is the single most important semantic difference.

### package.json type

Node decides how to treat a `.js` file from the nearest `package.json`. `"type": "module"` makes `.js` ESM; `"type": "commonjs"` (or no field) makes it CommonJS. `.mjs` is always ESM, `.cjs` is always CommonJS.

```javascript
// In a package with "type": "module", this .js file is ESM:
console.log(typeof module, typeof require);
console.log(import.meta.url.endsWith("app.js"));
```

Output:

```text
undefined undefined
true
```

`module` and `require` do not exist in ESM, and `import.meta.url` gives the file's URL. Use `.cjs` for CommonJS inside an ESM package, or `.mjs` / `"type": "module"` for ESM.

---

# 5. Circular Dependencies

### Circular dependencies

A cycle happens when `a.js` imports `b.js` and `b.js` imports `a.js`. ESM handles cycles with **live bindings and hoisting**, but the binding is in the **temporal dead zone** until the exporting module evaluates. Reading it too early throws.

```javascript
// a.js
import { bName } from "./b.js";

export const aName = "A";
export function describeA() {
  return `a sees ${bName}`;
}
console.log("a evaluated");
```

```javascript
// b.js (entry)
import { aName, describeA } from "./a.js";

export const bName = "B";
console.log("b sees", aName);
console.log(describeA());
```

Output:

```text
a evaluated
b sees A
a sees B
```

Evaluation starts at `b.js`, which imports `a.js`; `a.js` imports `b.js` (already in progress), evaluates its own body, and only reads `bName` later inside `describeA`. By then `bName` is initialized. If `a.js` logged `bName` at the top level instead, you would get:

```text
ReferenceError: Cannot access 'bName' before initialization
```

Think:

```text
b.js → imports a.js → a.js imports b.js (in progress)
     → a.js finishes → b.js finishes → describeA() reads bName
```

In CommonJS the cycle is tolerated differently: `require` returns a **partial `module.exports`** rather than throwing, so you can silently get `undefined` for something not yet assigned. Either way, cycles usually signal a design problem — extract shared code into a third module, inject the dependency, or use a dynamic import.

---

# Real-World Uses

- **Feature folders** — `index.js` barrels expose a small public API while internals stay private.
- **Lazy routes** — `import("./pages/Settings.js")` loads a screen only when visited.
- **Shared packages** — named exports give consumers stable, tree-shakeable entry points.

---

# Quick Revision

| Concept | Remember |
| --- | --- |
| **Script vs module** | Modules are strict, file-scoped, deferred, and evaluated once |
| **Module scope** | Top-level declarations stay private unless exported |
| **import and export** | Static, top-level, hoisted; imports are live read-only bindings |
| **Named exports** | Many per file, fixed names, rename with `as`, tree-shakeable |
| **Default exports** | One per file, importer picks the name, not tree-shakeable |
| **Re-exports** | `export { x } from` / `export *`; default must be named explicitly |
| **Dynamic import** | `import()` returns a promise; conditional and lazy |
| **Bundlers** | Follow the graph, tree-shake, split chunks, hash assets |
| **CommonJS** | `require`, `module.exports`, synchronous, cached, copies values |
| **ESM vs CommonJS** | Live bindings vs snapshots; async graph vs sync require |
| **Circular dependencies** | ESM bindings can be in TDZ; CJS returns partial exports |
| **package.json type** | `"module"` → `.js` is ESM; `.mjs` / `.cjs` always override |

## The Mental Model

```text
                     MODULES
                        │
          ┌─────────────┴─────────────┐
          ↓                           ↓
         ESM                         CJS
   import / export              require / module.exports
   live bindings                value snapshots
   async graph                  synchronous cache
          │                           │
          ├── named / default           ├── .cjs
          ├── re-export                 └── package type
          ├── dynamic import()
          └── cycles → TDZ
                        │
                        ↓
                 one instance, cached
```

### The 3 things to remember

**1. ESM imports are live bindings.**

```javascript
import { count, increment } from "./counter.js";
increment();
console.log(count); // updated value
```

**2. CommonJS caches by filename.**

```javascript
require("./config.cjs") === require("./config.cjs"); // true
```

**3. Dynamic import returns a promise.**

```javascript
const { add } = await import("./math.js");
```

> **Modules are file-scoped, explicitly connected, evaluated once, and cached — ESM shares live bindings, CommonJS shares snapshots.**
