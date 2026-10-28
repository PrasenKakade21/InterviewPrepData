# Functions

A function is a **reusable block of code you can call by name**, and in JavaScript it is also a value you can store, pass and return. That one fact — functions are first-class — is what enables callbacks, higher-order functions and closures. The chain is **define → take inputs → return a value → pass it around**, and most interview questions test the small differences: hoisting, defaults, `this`, and what a function returns when you forget `return`.

```text
Function
   ↓
define    declaration / expression / arrow
   ↓
call      arguments → parameters → body
   ↓
return    one value (or undefined)
   ↓
pass it   callback → higher-order function → closure
```

---

# 1. Creating Functions

### Declarations vs expressions

A **declaration** is hoisted with its body, so you can call it before the line where it appears. An **expression** is assigned to a variable, so it only exists after that assignment.

```javascript
console.log(greet());          // works — hoisted

function greet() {
  return "Hello Kakade";
}

const hi = function () {
  return "Hi Asha";
};

console.log(hi());
```

Output:

```text
Hello Kakade
Hi Asha
```

Bad: `console.log(hi())` **before** the `const` line → `ReferenceError` (temporal dead zone). With `var fn = function () {}` you would instead get `TypeError: fn is not a function`, because `var` hoists as `undefined`. Declarations are best for top-level helpers; expressions keep the name scoped to one place.

### Arrow functions

Arrow functions are a shorter syntax with **no own `this`, no `arguments`, and no `new`**. The expression after `=>` is returned implicitly.

```javascript
const double = (n) => n * 2;
const add = (a, b) => a + b;
const makeUser = (name) => ({ name, active: true });

console.log(double(4), add(2, 3));
console.log(makeUser("Rohan"));
```

Output:

```text
8 5
{ name: 'Rohan', active: true }
```

Two syntax rules trip people up:

- Returning an **object literal** needs parentheses: `() => ({ a: 1 })`. Without them the braces are a block and you get `undefined`.
- A single parameter may drop its parentheses, but zero or two-plus need them: `x => x`, `() => 1`, `(a, b) => a + b`.

Use arrows for callbacks and short transforms. Use `function` when you need `this`, `arguments`, or a constructor.

---

# 2. Inputs and Outputs

### Parameters and defaults

Default values apply only when the argument is **`undefined`** — not when it is `null`, `0`, `""` or `false`. Defaults are evaluated **at call time**, left to right, so a later default can use an earlier parameter.

```javascript
function add(a, b = 10) {
  return a + b;
}

console.log(add(5), add(5, 1), add(5, undefined), add(5, null));
```

Output:

```text
15 6 15 5
```

`add(5, null)` is `5` because `null` skips the default and then coerces to `0` in `+`. That double surprise — "default didn't apply" plus "null became zero" — is a real interview trap. For a `null`-safe default use `b ?? 10` in the body (nullish, not falsy).

```javascript
function range(start = 1, end = start + 2) {
  return [start, end];
}

console.log(range(), range(5));
```

Output:

```text
[ 1, 3 ]
[ 5, 7 ]
```

### Rest parameters

`...nums` collects **all remaining arguments** into a real array. It must be the last parameter, and it replaces `arguments`.

```javascript
function total(...nums) {
  return nums.reduce((sum, n) => sum + n, 0);
}

function firstRest(a, ...rest) {
  return [a, rest];
}

console.log(total(1, 2, 3), total());
console.log(firstRest(1, 2, 3));
```

Output:

```text
6 0
[ 1, [ 2, 3 ] ]
```

`rest` is a genuine array (`Array.isArray(rest)` is `true`), so `map`, `filter` and `reduce` just work — unlike `arguments`.

### Return values

A function without `return` returns **`undefined`**. `return` also stops execution immediately, and it must be on the **same line** as its value because of automatic semicolon insertion.

```javascript
function returnsNothing() {}
function bad() {
  return
  5;
}

console.log(returnsNothing());
console.log(bad());
```

Output:

```text
undefined
undefined
```

In `bad`, the newline after `return` inserts a semicolon, so the function returns `undefined` and `5` is dead code. Nothing warns you — keep the value on the same line.

### arguments object

Every **normal** function has an `arguments` object: array-like, with `length` and indexes, but no array methods. Arrow functions do not have it.

```javascript
function count() {
  return [arguments.length, Array.isArray(arguments)];
}

const arrowCount = (...args) => args.length;

console.log(count(1, 2, 3));
console.log(arrowCount(1, 2));
```

Output:

```text
[ 3, false ]
2
```

Prefer rest parameters. `arguments` is not a real array, does not exist in arrows, and in strict mode is not linked to the named parameters.

---

# 3. Functions as Values

### First-class functions

Functions can be stored in variables, put in objects and arrays, passed as arguments and returned from other functions — exactly like numbers or strings.

```javascript
const sayHello = () => "Hello Kakade";

const actions = {
  greet: sayHello,
  farewell: () => "Bye Asha",
};

console.log(actions.greet());
console.log(typeof sayHello);
```

Output:

```text
Hello Kakade
function
```

Notice: `sayHello` is passed **without parentheses**. `sayHello()` would pass the string it returns instead of the function itself — the classic callback mistake.

### Higher-order functions

A **higher-order function** either takes a function as an argument, returns a function, or both. `map`, `filter`, `setTimeout` and `addEventListener` are all higher-order.

```javascript
const withLogging = (fn) => (value) => {
  console.log("calling with", value);
  return fn(value);
};

const square = withLogging((n) => n * n);
console.log(square(4));
```

Output:

```text
calling with 4
16
```

`withLogging` returns a **new function** that closes over `fn`. Currying and decorators are built from this same shape.

### Callbacks

A **callback** is a function you hand to another function to be called later. It can run synchronously (`map`) or asynchronously (`setTimeout`, event handlers).

```javascript
const names = ["Kakade", "Asha"];

const shout = names.map((name) => name.toUpperCase());
console.log(shout);

setTimeout(() => console.log("later: Rohan"), 0);
console.log("now: Kakade");
```

Output:

```text
[ 'KAKADE', 'ASHA' ]
now: Kakade
later: Rohan
```

The `setTimeout` callback runs after the current code, because timers go to the task queue. Passing `console.log("x")` instead of `() => console.log("x")` calls it immediately and passes `undefined` — the number-one callback bug.

---

# 4. Patterns

### IIFE

An **IIFE** (immediately invoked function expression) runs the moment it is defined. It was the pre-`let` way to create a private scope; today it still appears in one-off setup code and libraries.

```javascript
const counter = (() => {
  let count = 0;
  return () => ++count;
})();

console.log(counter(), counter());
console.log(typeof count);
```

Output:

```text
1 2
undefined
```

The outer arrow runs once, `count` lives in its scope, and the returned function closes over it. The wrapping parentheses are needed so the parser treats `function () {}` as an **expression**, not a declaration. `count` is unreachable from outside — `typeof count` is `"undefined"`.

### Pure functions

A function is **pure** if it returns the same output for the same input and has **no side effects** — no mutation of arguments, no I/O, no `Math.random`.

```javascript
// pure
const addItem = (list, item) => [...list, item];

// impure: mutates the caller's array
const pushItem = (list, item) => {
  list.push(item);
  return list;
};

const base = [1, 2];
console.log(addItem(base, 3), base);
console.log(pushItem(base, 9), base);
```

Output:

```text
[ 1, 2, 3 ] [ 1, 2 ]
[ 1, 2, 9 ] [ 1, 2, 9 ]
```

Both "work", but only `addItem` leaves `base` untouched. Pure functions are easier to test, cache and parallelise — the basis of `map`/`filter`/`reduce` code.

### Recursion

A recursive function calls itself and needs a **base case** to stop. Each call adds a stack frame, so deep recursion throws `RangeError: Maximum call stack size exceeded`.

```javascript
function factorial(n) {
  if (n <= 1) return 1;        // base case
  return n * factorial(n - 1); // recursive step
}

console.log(factorial(5));
```

Output:

```text
120
```

Think:

```text
factorial(3)
  └─ 3 * factorial(2)
           └─ 2 * factorial(1)
                    └─ 1   ← base case unwinds the stack
```

Recursion is the natural fit for trees and nested data (folders, comments, the DOM). For deep linear work prefer a loop, because JavaScript engines do **not** optimise tail calls.

---

# Real-World Uses

- **Event handlers** — arrow callbacks that capture the surrounding `this`.
- **Array pipelines** — `map`, `filter`, `reduce` take a function per step.
- **Debounce/throttle** — higher-order functions returning a wrapped callback.
- **Validation** — small pure predicates composed into a form validator.
- **Config factories** — `createClient({ baseUrl })` returning a configured function.
- **Tree walking** — recursion over nested menus and comment threads.

---

# Quick Revision

| Concept | Remember |
| --- | --- |
| **Declarations vs expressions** | Declarations hoist with body; `const` expressions hit TDZ |
| **Arrow functions** | No own `this`/`arguments`/`new`; object return needs `()` |
| **Parameters and defaults** | Defaults apply only to `undefined`, evaluated at call time |
| **Rest parameters** | `...args` → real array, must be last |
| **Return values** | No `return` → `undefined`; `return` + newline breaks |
| **First-class functions** | Store, pass, return; pass `fn` not `fn()` |
| **Higher-order functions** | Take and/or return a function |
| **Callbacks** | Called later; sync or async; wrap when you need to delay |
| **IIFE** | Runs immediately; creates private scope |
| **Pure functions** | Same input → same output, no side effects |
| **Recursion** | Base case + smaller step; stack limit, no TCO |
| **arguments object** | Array-like, normal functions only; prefer rest |

## The Mental Model

```text
                 FUNCTION
                    │
     ┌──────────────┼──────────────┐
     ↓              ↓              ↓
   define          call          return
 declaration    args → params    one value
 expression                      or undefined
 arrow              │
     │              ↓
     └────────→ is a VALUE
                    │
        ┌───────────┼───────────┐
        ↓           ↓           ↓
     stored      passed      returned
     in obj     to HOF      from HOF
                    │
                    ↓
                callback / closure
```

### The 3 things to remember

**1. Pass the function, don't call it.**

```javascript
setTimeout(() => console.log("Kakade"), 0); // correct
setTimeout(console.log("Kakade"), 0);       // calls now, passes undefined
```

**2. Defaults only fire on `undefined`.**

```javascript
const f = (n = 10) => n;
f(undefined); // 10
f(null);      // null
```

**3. Every path needs a return.**

```javascript
const area = (r) => { Math.PI * r * r; }; // undefined
const ok = (r) => Math.PI * r * r;        // value
```

> **Functions are reusable blocks that are also values: define them, feed them arguments, return one result, and pass them around as callbacks.**
