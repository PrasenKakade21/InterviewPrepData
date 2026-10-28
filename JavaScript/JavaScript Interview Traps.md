# JavaScript Interview Traps

Most "gotcha" questions test one thing: **JavaScript decides behaviour from rules, not from what looks reasonable**. The chain is **declaration → binding → coercion → call site → prototype → task queue**; once you can name which rule is running, the output stops being magic.

```text
Source code
   ↓
parse        hoisting · TDZ · syntax
   ↓
evaluate     coercion · equality · floating point
   ↓
call         this · closures · arrows
   ↓
lookup       prototype chain · copies
   ↓
schedule     microtask → macrotask → render
```

---

# 1. Declarations and Types

### var vs let vs const

`var` is function-scoped and hoisted as `undefined`; `let` and `const` are block-scoped and stay in the temporal dead zone until initialised. `const` blocks reassignment, not mutation.

| | `var` | `let` | `const` |
| --- | --- | --- | --- |
| Scope | function | block | block |
| Reassign | yes | yes | no |
| Hoisted value | `undefined` | TDZ | TDZ |

```javascript
var a = 1;
let b = 2;
const c = 3;

{
  var a = 10;  // same binding
  let b = 20;  // new block binding
}

console.log(a, b, c);
```

Output:

```text
10 2 3
```

`const` still allows mutation: `const user = {}; user.name = "Asha"` works; only reassignment throws. Bad: `for (var i = 0; ...)` when callbacks need each value. Good: `for (let i = 0; ...)`.

### null vs undefined

`undefined` means no value was provided; `null` is an intentional empty value. They are loosely but not strictly equal, and differ in arithmetic, JSON and defaults.

```javascript
console.log(typeof null, typeof undefined);
console.log(null == undefined, null === undefined);
console.log(null + 1, undefined + 1);
console.log(JSON.stringify({ a: undefined, b: null }));

function greet(name = "Guest") { return name; }
console.log(greet(undefined), greet(null));
```

Output:

```text
object undefined
true false
1 NaN
{"b":null}
Guest null
```

`JSON.stringify` drops `undefined` properties but keeps `null`. Defaults apply only for `undefined`, so passing `null` suppresses them — a frequent API bug.

### typeof null

`typeof null` is `"object"` from a legacy tag; changing it would break the web.

```javascript
console.log(typeof null, typeof undefined);
console.log(typeof [], typeof function () {}, typeof NaN);
```

Output:

```text
object undefined
object function number
```

Bad: `typeof value === "object"` is true for `null`. Good: `value !== null && typeof value === "object"`.

---

# 2. Equality and Numbers

### == vs ===

`===` compares type and value. `==` runs the abstract equality algorithm, converting operands until types match — specified, but rarely what a reader expects.

```javascript
console.log(0 == "", 0 == "0", "" == "0");
console.log(null == undefined, null === undefined);
console.log([] == false, [] == ![]);
console.log(NaN == NaN);
```

Output:

```text
true true false
true false
true true
false
```

`[] == false` is true because `[]` → `""` → `0`, and `false` → `0`. `[] == ![]` is true because `![]` is `false`. Bad: `if (count == 0)` — `""` and `[]` pass. Good: `if (count === 0)`.

Notice: `value == null` is the useful `==` idiom — true for `null` and `undefined` only.

### NaN checks

`NaN` is the only value not equal to itself. `Number.isNaN` checks without converting; global `isNaN` coerces first, so strings can produce false positives.

```javascript
console.log(NaN === NaN, Number.isNaN(NaN));
console.log(isNaN("abc"), Number.isNaN("abc"));
console.log(Object.is(NaN, NaN), [NaN].includes(NaN));
```

Output:

```text
false true
true false
true true
```

Bad: `value === NaN` — always false. Good: `Number.isNaN(value)`. `Object.is` and `includes` also treat `NaN` as equal to itself.

### Floating point

Numbers are IEEE 754 binary64. Values such as `0.1` and `0.2` have no exact binary representation, so their sum is not exactly `0.3`.

```javascript
console.log(0.1 + 0.2);
console.log(0.1 + 0.2 === 0.3);
console.log(Math.abs(0.1 + 0.2 - 0.3) < Number.EPSILON);
console.log((0.1 + 0.2).toFixed(2));
console.log(9007199254740993);
```

Output:

```text
0.30000000000000004
false
true
0.30
9007199254740992
```

Never compare floats with `===` after arithmetic. Compare integers (store paise), round explicitly, or use `Math.abs(a - b) < Number.EPSILON`. `toFixed` returns a string; integers above `Number.MAX_SAFE_INTEGER` lose precision — use `BigInt` for IDs.

### Coercion quick hits

`+` prefers string concatenation if either operand is a string; `-`, `*`, `/` convert to number. Arrays become `""`, objects become `"[object Object]"`.

```javascript
console.log([] + {});
console.log(JSON.stringify([] + []));
console.log("5" - 2);
console.log("5" + 2);
console.log(true + 1, null + 1, undefined + 1);
```

Output:

```text
[object Object]
""
3
52
2 1 NaN
```

`"5" - 2` is `3` because `-` is numeric; `"5" + 2` is `"52"` because `+` sees a string. `true + 1` is `2`, `null + 1` is `1`, `undefined + 1` is `NaN`. Never rely on this in application code.

---

# 3. Declaration Timing

### Hoisting and TDZ

Hoisting moves declarations to the top of their scope before execution. `var` is initialised to `undefined`; `function` declarations are fully available; `let`, `const` and `class` are hoisted but **uninitialised** until their declaration runs — that gap is the TDZ.

```javascript
console.log(typeof hoisted);   // undefined
var hoisted = 1;

try {
  console.log(tdz);
  let tdz = 2;
} catch (error) {
  console.log(error.name);
}
```

Output:

```text
undefined
ReferenceError
```

Think:

```text
Scope starts
   │
   ├── var hoisted → undefined        (usable)
   ├── function hoisted → callable    (usable)
   └── let/const hoisted → <TDZ>      (ReferenceError if touched)
            │
            ▼
      declaration line runs
            │
            ▼
        initialised
```

`typeof undeclared` is `"undefined"`, but `typeof tdz` throws while `tdz` is in the TDZ — a favourite follow-up.

---

# 4. Closures and `this`

### Closure loop bug

A closure captures the **binding**, not the value. `var` creates one binding for the whole loop, so every callback sees the final value. `let` creates a fresh binding per iteration.

```javascript
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log("var", i), 0);
}

for (let j = 0; j < 3; j++) {
  setTimeout(() => console.log("let", j), 0);
}
```

Output:

```text
var 3
var 3
var 3
let 0
let 1
let 2
```

Bad: `var` plus a timer. Good: `let`, or an IIFE that passes the current value. The same rule explains closures inside a `while` loop sharing one variable.

### this binding

`this` is decided at **call time** (except for arrows). A method call sets it to the object before the dot; a plain call sets `undefined` in strict mode or `globalThis` in sloppy mode.

```javascript
"use strict";
const user = { name: "Kakade", greet() { return this.name; } };
const loose = user.greet;

console.log(user.greet());
try { loose(); } catch (error) { console.log(error.name); }
console.log(user.greet.call({ name: "Rohan" }));
```

Output:

```text
Kakade
TypeError
Rohan
```

Bad: passing `user.greet` as a callback and expecting `this` to stay `user`. Good: `user.greet.bind(user)`, an arrow wrapper, or `call`/`apply`.

### Arrow vs function

Arrow functions have no own `this`, no `arguments`, and cannot be constructors. They inherit `this` lexically from the enclosing scope, which is why they are ideal for callbacks and wrong for object methods.

```javascript
const team = {
  name: "Kakade",
  members: ["Asha"],
  listArrow() {
    return this.members.map((m) => `${this.name}:${m}`);
  },
  listFunction() {
    return this.members.map(function (m) {
      return `${this?.name ?? "no this"}:${m}`;
    });
  },
};

console.log(team.listArrow());
console.log(team.listFunction());
```

Output:

```text
[ 'Kakade:Asha' ]
[ 'no this:Asha' ]
```

The arrow sees `team`; the plain callback has its own `this` (undefined in strict mode, `globalThis` otherwise). Arrow trap: `arguments` inside an arrow refers to the outer function's arguments, not the arrow's.

---

# 5. Objects and Copies

### Prototype chain

Every object has an internal prototype link. Property lookup checks the object's own properties, then its prototype, then that prototype's prototype, until `null`.

```javascript
const person = { greet() { return `Hi ${this.name}`; } };
const kakade = Object.create(person);
kakade.name = "Kakade";

console.log(kakade.greet());
console.log(Object.getPrototypeOf(kakade) === person);
console.log(kakade.hasOwnProperty("name"), kakade.hasOwnProperty("greet"));
console.log("greet" in kakade);
```

Output:

```text
Hi Kakade
true
true false
true
```

`hasOwnProperty` checks only the object; `in` walks the chain. Bad: `if (user.isAdmin)` when it may come from `Object.prototype` via pollution. Good: `Object.hasOwn(user, "isAdmin")`.

### Shallow vs deep copy

Spread and `Object.assign` copy the **top level**; nested objects are still shared. `structuredClone` performs a real deep clone for supported values.

```javascript
const original = { name: "Kakade", skills: ["js", "css"] };
const shallow = { ...original };
shallow.name = "Asha";
shallow.skills.push("html");

const deep = structuredClone(original);
deep.skills.push("node");

console.log(original);
console.log(shallow);
console.log(deep);
```

Output:

```text
{ name: 'Kakade', skills: [ 'js', 'css', 'html' ] }
{ name: 'Asha', skills: [ 'js', 'css', 'html' ] }
{ name: 'Kakade', skills: [ 'js', 'css', 'html', 'node' ] }
```

Bad: `{ ...original }` then mutating `copy.skills` — it edits the original too. Good: `structuredClone(original)`. The JSON trick loses `undefined`, functions and `Date` types, and throws on cycles.

### Map vs object

Object keys are strings or symbols; anything else is coerced. `Map` keeps the original key type, preserves insertion order, has `size`, and is safe for untrusted keys.

```javascript
const obj = {};
obj[{}] = "x";

const map = new Map();
const key = {};
map.set(key, "x");

console.log(Object.keys(obj), obj["[object Object]"]);
console.log(map.size, map.get(key), map.get({}));
```

Output:

```text
[ '[object Object]' ] x
1 x undefined
```

Both object keys stringify to `"[object Object]"`, so the second overwrites the first. `map.get({})` is `undefined` because a new object has a different identity. Use `Map` for object-keyed caches; use an object for JSON records.

---

# 6. Loops and Task Order

### for...in vs for...of

`for...in` enumerates **enumerable property keys**, including inherited ones; on arrays those keys are strings. `for...of` reads **values** from an iterable's iterator.

```javascript
const arr = ["a", "b"];
arr.extra = "x";

for (const key in arr) console.log("in", key, typeof key);
for (const value of arr) console.log("of", value);
```

Output:

```text
in 0 string
in 1 string
in extra string
of a
of b
```

Bad: `for...in` over an array — it can pick up added or inherited properties. Good: `for...of` for values. For plain objects `for...of` throws; use `Object.keys`, `values` or `entries`.

### Event loop order

Synchronous code runs first, then the microtask queue drains completely (promises, `queueMicrotask`, `await` continuations), then one macrotask (timer, I/O), with rendering between tasks.

```javascript
console.log("A");
setTimeout(() => console.log("B"), 0);
Promise.resolve().then(() => console.log("C"));
queueMicrotask(() => console.log("D"));
console.log("E");
```

Output:

```text
A
E
C
D
B
```

An `await` continuation is also a microtask, so async code resumes before the next timer. `setTimeout(..., 0)` means "next task", not "now"; long synchronous work blocks both.

---

# Real-World Uses

- **Code review** — spotting `==`, `var` in loops and lost `this`.
- **Debugging** — why a cloned object still changes the original.

---

# Quick Revision

| Concept | Remember |
| --- | --- |
| **var vs let vs const** | Function vs block; const no reassign |
| **== vs ===** | `==` coerces; `x == null` checks both |
| **typeof null** | `"object"` legacy; test `=== null` |
| **NaN checks** | `NaN !== NaN`; use `Number.isNaN` |
| **Hoisting and TDZ** | var → undefined; let/const throw |
| **Closure loop bug** | var shares one binding; let per-iteration |
| **this binding** | Call-site; method → object |
| **Arrow vs function** | Lexical this, no arguments, no `new` |
| **Prototype chain** | Own → prototype → null |
| **Event loop order** | Sync → microtasks → macrotask |
| **Shallow vs deep copy** | Spread shallow; `structuredClone` deep |
| **Map vs object** | Map keeps key type; object stringifies |
| **for...in vs for...of** | in = keys; of = values |
| **null vs undefined** | Loose-equal; JSON drops undefined |
| **Floating point** | `0.1 + 0.2 !== 0.3` |

## The Mental Model

```text
             SOURCE
                │
   ┌────────────┼────────────┐
   ↓            ↓            ↓
hoisting     coercion     call site
var / TDZ    ==, +, -     this / arrow
   └────────────┼────────────┘
                ↓
    prototype → schedule → OUTPUT
```

### The 3 things to remember

**1. `===` unless you mean `value == null`.**

```javascript
0 == "";        // true
0 === "";       // false
value == null;  // null or undefined only
```

**2. `let` in loops, arrow for callbacks, function for methods.**

```javascript
for (let i = 0; i < 3; i++) setTimeout(() => console.log(i), 0); // 0 1 2
```

**3. Microtasks drain before the next timer.**

```javascript
Promise.resolve().then(() => console.log("micro"));
setTimeout(() => console.log("macro"), 0); // micro, then macro
```

> **Each trap is a rule: declarations hoist, `let`/`const` wait in the TDZ, `==` coerces, `this` depends on the call, prototypes chain, and microtasks run first.**
