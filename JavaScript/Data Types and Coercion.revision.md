# Data Types and Coercion — Revision Notes (Beginner)
> One-line mental model: seven primitives copy by value, objects copy by reference, and mixed types get coerced — explicitly or not.
## Core recall
- Primitives: `string`, `number`, `boolean`, `null`, `undefined`, `symbol`, `bigint`. Everything else (arrays, functions, dates) is an object.
- Primitives are immutable and copied by value; objects are mutable and copied by reference. `{a:1} === {a:1}` is `false`.
- `number` is one 64-bit float type; it also holds `Infinity`, `-Infinity`, `NaN`.
- `undefined` = missing/default; `null` = intentional empty. `null == undefined` true, `===` false. `typeof null === "object"` (bug).
- `Symbol("id") !== Symbol("id")`; symbols are unique. `10n` is a BigInt; mixing BigInt and Number throws `TypeError`.
- `typeof`: `"string"`, `"number"`, `"bigint"`, `"boolean"`, `"undefined"`, `"symbol"`, `"object"` (null, arrays, objects), `"function"`.
- Falsy: `false`, `0`, `-0`, `0n`, `""`, `null`, `undefined`, `NaN`. Everything else is truthy — including `"0"`, `" "`, `[]`, `{}`.
- Explicit: `Number("")` → `0`, `Number("12px")` → `NaN`, `parseInt("12px", 10)` → `12`, `parseFloat("3.14kg")` → `3.14`.
- Implicit: `+` with a string concatenates (`1 + "2"` → `"12"`); `- * / %` convert to number (`"6" / "2"` → `3`); `==` coerces.
- `NaN` is a number, never equals itself. Use `Number.isNaN(x)`; global `isNaN` coerces first.
- Object wrappers: `"abc".length` borrows a temporary `String` object. Never use `new String()`.
## Must-remember code
```javascript
// typeof table
console.log(typeof null);        // "object"
console.log(typeof []);          // "object"
console.log(typeof 10n);         // "bigint"
console.log(Array.isArray([]));  // true
```
```javascript
// Explicit conversions
console.log(Number("42"), Number("42px"));     // 42 NaN
console.log(parseInt("42px", 10));             // 42
console.log(String(null), Boolean("0"));       // "null" true
```
```javascript
// Copy semantics
const a = { name: "Kakade" };
const b = a;              // same reference
const c = { ...a };       // shallow copy
b.name = "Asha";
console.log(a.name, c.name); // "Asha" "Kakade"
```
```javascript
// NaN
console.log(NaN === NaN);         // false
console.log(Number.isNaN(NaN));   // true
console.log(isNaN("abc"));        // true (coerces)
```
## Interview traps
- `typeof null === "object"` — never a null check. Use `value === null`.
- `typeof [] === "object"` — use `Array.isArray`.
- `Boolean([])` is `true` but `[] == false` is `true` — truthiness ≠ loose equality.
- `Number("")` and `Number("  ")` are `0`, not `NaN`.
- `parseInt("08")` — always pass radix `10`.
- Spread is shallow; nested objects remain shared.
## 3 things to remember
1. **Check null and arrays properly:** `x === null`, `Array.isArray(x)`.
2. **Convert at the boundary:** `Number(input.value)`, `parseInt(str, 10)`.
3. **Objects compare by reference:** `{a:1} !== {a:1}`; clone before mutating.
