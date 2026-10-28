# Variables, Scope and Hoisting — Revision Notes (Beginner)
> One-line mental model: scope decides *where* a name is visible, hoisting decides *when* it exists, and the TDZ blocks early reads.
## Core recall
- `const` by default; `let` only when reassigning; `var` never in new code.
- `var` is **function-scoped**; `let`/`const` are **block-scoped** (block = `{ }`, `if`, `for`).
- `var` hoists as `undefined`; `let`/`const` hoist but sit in the **TDZ** until the declaration line runs.
- Reading a TDZ name throws `ReferenceError`; `typeof` does **not** protect you there, only for undeclared names.
- Function **declarations** are fully hoisted and callable early; function **expressions** only hoist the `var` binding (calling early → `TypeError`).
- `var` can be redeclared; `let`/`const` cannot — that is a **SyntaxError**, so `try/catch` will not save the file.
- Shadowing: inner binding hides outer. `let` inside a block can shadow; `var` inside a block cannot shadow an outer `let` (illegal redeclaration).
- `const` locks the **binding**, not the object: mutation is fine, reassignment throws.
- Global object: `window` (browser), `global` (Node), `globalThis` (standard). Top-level `var` becomes a `window` property in classic browser scripts; `let`/`const` do not.
- Implicit globals (`x = 1` without declaration) work in sloppy mode and throw `ReferenceError` in strict mode or modules.
- `let` in a `for` loop creates a fresh binding per iteration — the fix for the classic `3 3 3` closure bug.
## Must-remember code
```javascript
const user = { name: "Kakade" };
user.name = "Asha";   // allowed
// user = {};         // TypeError
```
```javascript
// Hoisting: undefined, then value
console.log(city); // undefined
var city = "Pune";
console.log(city); // "Pune"
```
```javascript
// TDZ
try {
  console.log(user);
  let user = "Kakade";
} catch (e) {
  console.log(e.name); // ReferenceError
}
```
```javascript
// Per-iteration binding
for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 0); // 0 1 2
}
```
## Interview traps
- Saying `const` makes objects immutable — it only prevents rebinding.
- Thinking hoisting moves the assignment — it only registers the declaration.
- Using `typeof x` to "safely" check a shadowed `let` in the same block → still `ReferenceError`.
- Believing `let` is not hoisted at all — it is hoisted but uninitialized.
- Expecting `try/catch` to catch a redeclaration — it is a parse-time `SyntaxError`.
- Top-level `var` is **not** global in a Node module, even though it is in a classic browser script.
- Forgetting that `var` in a block leaks to the enclosing function.
## 3 things to remember
1. **`const` → `let` → never `var`:** `const user = "Kakade";`
2. **Blocks contain `let`:** `{ let x = 1; }` — `x` is gone after `}`.
3. **No reads before init:** `let x = 1;` must run before `console.log(x)`.
