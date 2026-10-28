# JavaScript Interview Traps — Revision Notes (Beginner)

> One-line mental model: every trap is a rule — hoisting, TDZ, coercion, call-site `this`, prototype lookup, and microtasks before macrotasks.

## Core recall

- **var vs let vs const**: var is function-scoped and hoisted to `undefined`; let/const are block-scoped with a TDZ; const blocks reassignment, not mutation.
- **== vs ===**: `==` coerces (`0 == ""`); `===` does not. `x == null` is the useful exception.
- **typeof null**: `"object"` from a legacy tag; test `x === null`.
- **NaN checks**: `NaN !== NaN`; use `Number.isNaN`, not global `isNaN`.
- **Hoisting and TDZ**: var → undefined; functions fully hoisted; let/const/class throw before init.
- **Closure loop bug**: var shares one binding (3,3,3); let is per-iteration (0,1,2).
- **this binding**: call-site — `obj.fn()` → obj; `fn()` → undefined/global; `call/apply/bind` explicit.
- **Arrow vs function**: lexical `this`, no `arguments`, not constructible; use functions for methods.
- **Prototype chain**: own → prototype → null; `in` walks it, `Object.hasOwn` does not.
- **Event loop order**: sync → drain microtasks (promises, await) → one macrotask → render.
- **Shallow vs deep copy**: spread copies one level; `structuredClone` deep; JSON loses types.
- **Map vs object**: object keys stringify; Map keeps type/identity, has `size` and order.
- **for...in vs for...of**: in = keys (incl. inherited); of = iterable values.
- **null vs undefined**: loosely equal, not strict; JSON drops undefined.
- **Floating point**: `0.1 + 0.2 !== 0.3`; compare epsilon or integers; `toFixed` returns a string.

## Must-remember code

```javascript
// one binding vs one per iteration
for (var i = 0; i < 3; i++) setTimeout(() => console.log(i), 0); // 3 3 3
for (let j = 0; j < 3; j++) setTimeout(() => console.log(j), 0); // 0 1 2
```

```javascript
// lost this, and the three fixes
const loose = user.greet;
loose();                       // TypeError in strict mode
user.greet.call(user);         // Kakade
const bound = user.greet.bind(user);
const wrapper = () => user.greet();
```

```javascript
// equality and numbers
0 == "";                 // true  (trap)
0 === "";                // false
0.1 + 0.2;               // 0.30000000000000004
Number.isNaN(NaN);       // true
typeof null;             // "object"
```

```javascript
// copy and order
const deep = structuredClone(original);
console.log("A");
setTimeout(() => console.log("B"), 0);
Promise.resolve().then(() => console.log("C"));
console.log("D");        // A D C B
```

## Interview traps

- `typeof null` is `"object"`, but `null` is not an object; guard with `!== null`.
- `NaN !== NaN`; `Object.is(NaN, NaN)` and `[NaN].includes(NaN)` are true.
- `0.1 + 0.2 !== 0.3`; `(0.1 + 0.2).toFixed(2)` is the string `"0.30"`.
- `[] + {}` is `"[object Object]"`; `"5" - 2` is `3` but `"5" + 2` is `"52"`.
- `typeof tdz` throws `ReferenceError`; `typeof undeclared` returns `"undefined"`.
- `var` in a loop shares one binding; callbacks all see the final value.
- A method passed as a callback loses `this`; bind it or wrap it in an arrow.
- `{ ...obj }` is shallow — nested objects are still shared.
- `JSON.parse(JSON.stringify(obj))` drops functions/undefined and turns Dates into strings.
- Object keys are strings or symbols; `obj[{}]` becomes `obj["[object Object]"]`.
- `for...in` on an array yields string indexes plus custom/inherited keys.
- `setTimeout(fn, 0)` runs after all current microtasks, not immediately.

## 3 things to remember

1. **`===` unless you mean `x == null`.**
   ```javascript
   0 == "";       // true  (trap)
   0 === "";      // false
   x == null;     // null or undefined only
   ```

2. **`let` in loops, arrows for callbacks, functions for methods.**
   ```javascript
   for (let i = 0; i < 3; i++) setTimeout(() => console.log(i), 0);
   ```

3. **Microtasks drain before the next timer.**
   ```javascript
   Promise.resolve().then(() => console.log("micro"));
   setTimeout(() => console.log("macro"), 0); // micro, then macro
   ```
