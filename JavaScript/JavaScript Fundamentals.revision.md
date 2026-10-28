# JavaScript Fundamentals — Revision Notes (Beginner)
> One-line mental model: JavaScript is one language that borrows its powers from a host — the browser gives it the DOM, Node gives it files and processes.
## Core recall
- **ECMAScript** = spec; **JavaScript** = implementation. React/Vue/Express are libraries written in JS, not the language.
- Browser host: `window`, `document`, `localStorage`. Node host: `globalThis`/`global`, `process`, `Buffer`, `fs`. Core syntax and built-ins are shared.
- `typeof window` is safe in Node (`"undefined"`); touching `document` throws `ReferenceError`.
- **Statement** = instruction (`let x = 1;`, `if`, `for`, `return`), produces no usable value. **Expression** = value (`5 + 3`, `n > 2`, ternary), usable in a call, assignment, or template literal.
- ES6/ES2015 brought `let`/`const`, arrows, classes, promises, modules. `?.` and `??` are ES2020; `replaceAll` is ES2021.
- **ASI** inserts semicolons at line breaks; `return` + newline returns `undefined`; lines starting with `(`, `[`, `` ` `` can continue the previous expression.
- Comments: `//`, `/* */`, JSDoc `/** */`. Block comments cannot nest or contain `*/`.
- `"use strict"` must be the **first statement**; it blocks implicit globals, sets plain-call `this` to `undefined`, and turns silent writes/deletes into errors. Modules and classes are strict automatically.
- Console: `log`, `warn`, `error`, `table`, `dir`, `time`/`timeEnd`, `trace`; `debugger;` pauses in devtools only.
- Script loading: classic blocks parsing; `defer` runs after parse in order; `async` runs when ready in any order; `type="module"` is deferred and strict.
- DevTools: Elements, Console, Sources, Network.
## Must-remember code
```javascript
// Host check that works everywhere
const host = typeof window !== "undefined" ? "browser" : "node";
console.log(host); // "browser" or "node"
```
```javascript
// ASI trap and the fix
function bad() {
  return
  42;            // returns undefined
}
const good = () => 42;
console.log(bad(), good()); // undefined 42
```
```javascript
// Expression vs statement
const score = 72;
const label = score >= 60 ? "pass" : "fail"; // expression
console.log(label); // pass
```
## Interview traps
- Saying "JavaScript and ECMAScript are the same" — the spec is ECMAScript, the runtime language is JavaScript.
- Forgetting the host split: `document` in Node, `require` in a browser.
- Writing `return` on its own line and expecting the next line's value.
- Thinking `"use strict"` works anywhere in the file — position matters.
- Believing `async` scripts preserve order, or that `defer` works on inline scripts.
- Calling `querySelector(...).addEventListener` before the DOM exists — `null` error.
## 3 things to remember
1. **Pick the host:** `typeof window` guards browser-only code; `process` is Node-only.
2. **Statements act, expressions return:** use a ternary when you need a value.
3. **Strict first line:** `"use strict";` at the very top — modules get it free.
