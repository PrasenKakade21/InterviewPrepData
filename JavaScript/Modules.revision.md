# Modules — Revision Notes (Beginner)

> One-line mental model: a module is a file-scoped unit with explicit exports, imported as a live binding, evaluated once and cached.

## Core recall

- **Script vs module**: classic scripts are global, non-strict, and blocking; modules are file-scoped, strict, deferred, and evaluated once. Module top-level `this` is `undefined`.
- **Module scope**: top-level declarations are private unless exported. Two modules can reuse the same names.
- **import / export**: static, top-level, hoisted. Imports are **live read-only bindings**; assigning to one throws.
- **Named exports**: many per module, fixed names, rename with `as`, tree-shakeable.
- **Default exports**: one per module, importer chooses the name, not tree-shakeable. Rename on re-export: `default as greet`.
- **Re-exports**: `export { x } from "./m.js"`; `export *` skips the default.
- **Dynamic import**: `import()` returns a **promise**, works conditionally, enables code splitting.
- **Bundlers**: follow the graph, resolve bare imports, tree-shake, split chunks, hash assets.
- **CommonJS**: `require`, `module.exports` / `exports`, synchronous, cached by filename, copies values.
- **ESM vs CommonJS**: live bindings vs snapshots; async graph vs sync require; top-level await only in ESM.
- **Circular dependencies**: ESM can hit the TDZ; CJS returns a partial `module.exports`. Extract shared code to break cycles.
- **package.json type**: `"type": "module"` makes `.js` ESM; `"commonjs"` or absent makes it CJS. `.mjs` / `.cjs` always override.

## Must-remember code

```javascript
// named + default export
export const add = (a, b) => a + b;
export default function greet(name) {
  return `Hello ${name}`;
}
```

```javascript
// live binding: count updates after increment
import { count, increment } from "./counter.js";
increment();
console.log(count); // 1
```

```javascript
// lazy load, with fallback
const mod = await import("./greet.js");
const greet = mod.default ?? mod.greet;
```

```javascript
// CommonJS: cached, same object every time
const a = require("./config.cjs");
const b = require("./config.cjs");
a === b; // true
```

## Interview traps

- ESM imports are **live**; `const snapshot = count` is a copy.
- CommonJS destructuring copies the value at `require` time.
- `require` runs a module once and caches it; the cache is keyed by resolved filename.
- `export *` does not re-export the default.
- `import { default as x }` or `export { default as x }` is required for defaults.
- `require` is not available in ESM; `module` / `__dirname` are `undefined`.
- Top-level `await` works only in ESM (and only at module top level).
- Circular ESM imports can throw `ReferenceError: Cannot access 'x' before initialization`.
- Circular CJS imports can silently give `undefined` from partial exports.
- Dynamic `import()` is an expression returning a promise — it can be conditional and lazy.
- A `.js` file's module type depends on the nearest `package.json` `"type"` field.

## 3 things to remember

1. **ESM shares live bindings.**
   ```javascript
   import { count, increment } from "./counter.js";
   increment();
   console.log(count); // updated
   ```

2. **CommonJS caches the module.**
   ```javascript
   require("./config.cjs") === require("./config.cjs"); // true
   ```

3. **Dynamic import is a promise.**
   ```javascript
   const { add } = await import("./math.js");
   ```

> **Modules are file-scoped, explicit, evaluated once, and cached — ESM gives live bindings, CommonJS gives snapshots.**
