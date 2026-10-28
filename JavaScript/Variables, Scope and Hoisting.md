# Variables, Scope and Hoisting

A variable is a **name that points to a value**. Before you can use that name, the engine must decide two things: **where the name is visible** (scope) and **when it becomes safe to read** (hoisting and the temporal dead zone). Once you see declarations as `declare → engine registers the binding → the line actually initializes it`, most "weird" JavaScript behaviour becomes predictable.

The core idea:

```text
let user = "Kakade";
     │
     ├── declaration   → which scope can see it?
     └── initialization → when does it get a value?
                ↓
        scope + hoisting
                ↓
      "can this line read it?"
```

---

# 1. var, let and const

### The three declarations

`var` is the old keyword, `let` is a reassignable block-scoped binding, and `const` is a block-scoped binding that cannot be reassigned.

```javascript
var city = "Pune";
let user = "Kakade";
const role = "developer";

user = "Asha";
console.log(city, user, role);
```

Output:

```text
Pune Asha developer
```

### The rules side by side

| Rule | `var` | `let` | `const` |
| --- | --- | --- | --- |
| Scope | function | block | block |
| Hoisted | Yes, as `undefined` | Yes, but TDZ | Yes, but TDZ |
| Reassign | Yes | Yes | No |
| Redeclare in same scope | Yes | No | No |
| Must initialize | No | No | Yes |

### `const` locks the binding, not the value

```javascript
const user = { name: "Kakade" };
user.name = "Asha";
console.log(user.name);

// user = {}; // TypeError: Assignment to constant variable.
```

Output:

```text
Asha
```

**Notice:** The object is still mutable. `const` only stops you from pointing the name at a different value.

### Bad vs Good

```javascript
// Bad — var ignores the block and leaks into the function
for (var i = 0; i < 3; i++) {}
console.log(i);

// Good — let stays inside the loop block
for (let j = 0; j < 3; j++) {}
console.log(typeof j);
```

Output:

```text
3
undefined
```

---

# 2. Scope

### Block scope

A **block** is anything between `{` and `}` — an `if`, a `for`, or even a bare pair of braces. `let` and `const` are visible only inside that block. `var` ignores blocks entirely.

```javascript
{
  let message = "inside";
  const count = 1;
  var leaked = "outside";
  console.log(message, count);
}

console.log(leaked);
console.log(typeof message);
```

Output:

```text
inside 1
outside
undefined
```

### Function scope

Every function creates a scope. Variables declared with `var`, `let`, or `const` inside it disappear when the function returns.

```javascript
function greet() {
  var local = "Kakade";
  let alsoLocal = "Asha";
  console.log(local, alsoLocal);
}

greet();
console.log(typeof local);
```

Output:

```text
Kakade Asha
undefined
```

### Global scope

A binding declared at the top level of a script is visible everywhere below it in that script. Lookups walk outward: current scope → outer scope → global scope.

```javascript
const appName = "InterviewPrep";

function show() {
  console.log(appName);
}

show();
```

Output:

```text
InterviewPrep
```

### Scope types compared

| Scope | Created by | Visible in |
| --- | --- | --- |
| Block | `{ }`, `if`, `for` | that block only (`let`/`const`) |
| Function | `function`, arrow | that function and inner functions |
| Global | top level of a script | the whole script |

**Think:**

```text
global
  └── function greet
        └── block { }
              ↓
   lookup walks upward only
```

---

# 3. Hoisting

### What hoisting really means

Before running a script, the engine scans it and **registers declarations**. Function declarations are fully hoisted, `var` is hoisted but initialized to `undefined`, and `let`/`const` are hoisted but left uninitialized (the TDZ). Hoisting moves the *declaration*, never the assignment.

### `var` hoisting

```javascript
console.log(city);
var city = "Pune";
console.log(city);
```

Output:

```text
undefined
Pune
```

The first line does not throw because `city` already exists — it just has no value yet.

### Function declaration hoisting

```javascript
console.log(greet("Kakade"));

function greet(name) {
  return `Hello ${name}`;
}
```

Output:

```text
Hello Kakade
```

### Function expressions are different

Only the `var` binding is hoisted, not the function value.

```javascript
try {
  console.log(sayHi());
} catch (error) {
  console.log(error.name);
}

var sayHi = () => "Hi";
```

Output:

```text
TypeError
```

**Notice:** `sayHi` exists as `undefined`, so calling it raises `TypeError: sayHi is not a function`. A `function` declaration would have worked.

---

# 4. Temporal Dead Zone

### Reading before declaration

The **temporal dead zone (TDZ)** is the time between entering a scope and the moment a `let`/`const` declaration is initialized. Reading the name in that window throws.

```javascript
try {
  console.log(user);
  let user = "Kakade";
} catch (error) {
  console.log(error.name);
  console.log(error.message);
}
```

Output:

```text
ReferenceError
Cannot access 'user' before initialization
```

### The TDZ is about time, not position

```javascript
let status = "ready";

try {
  {
    console.log(typeof status); // TDZ: inner `status` shadows the outer one
    let status = "loading";
  }
} catch (error) {
  console.log(error.name);
}
```

Output:

```text
ReferenceError
```

**Notice:** `typeof` is safe for a name that was never declared, but it **throws** for a `let`/`const` name that is still in its TDZ. That difference is a favourite interview question.

### Why the TDZ exists

It turns "I used it too early" into an immediate error instead of a silent `undefined`. That matters most for `const`, which must be initialized exactly once.

---

# 5. Redeclaration

### `var` allows it

```javascript
var count = 1;
var count = 2;
console.log(count);
```

Output:

```text
2
```

The second declaration is silently ignored and the assignment overwrites the value. This is a common source of bugs in large files.

### `let` and `const` do not

```javascript
// This file fails before any line runs:
let count = 1;
let count = 2;
```

Output:

```text
SyntaxError: Identifier 'count' has already been declared
```

### The mixed cases

```javascript
var user = "Kakade";
let user = "Asha"; // SyntaxError
```

Output:

```text
SyntaxError: Identifier 'user' has already been declared
```

```javascript
let role = "admin";
{
  var role = "guest"; // SyntaxError: var tries to claim the outer binding
}
```

Output:

```text
SyntaxError: Identifier 'role' has already been declared
```

**Notice:** Redeclaration errors are **syntax errors**, so the whole script fails to parse. You cannot catch them with `try/catch`.

---

# 6. Shadowing

### The inner scope wins

```javascript
let user = "Kakade";

{
  let user = "Rohan";
  console.log(user);
}

console.log(user);
```

Output:

```text
Rohan
Kakade
```

The inner `user` is a **different binding** that hides the outer one. The outer value is untouched.

### Shadowing with `var`

```javascript
var role = "admin";

function demo() {
  var role = "guest";
  console.log(role);
}

demo();
console.log(role);
```

Output:

```text
guest
admin
```

### Illegal shadowing

You cannot shadow a `let` with a `var` in a nested block, because `var` is not block-scoped and would collide with the outer binding.

```javascript
let name = "Kakade";
{
  var name = "Asha"; // SyntaxError
}
```

Output:

```text
SyntaxError: Identifier 'name' has already been declared
```

**Notice:** Shadowing is legal but often harmful. If a reviewer has to scroll up to understand which `user` you mean, rename one of them.

---

# 7. The Global Object

### One object, different names

The global object holds globals and host APIs. It is `window` in browsers, `global` in Node, and `globalThis` in modern JavaScript everywhere.

```javascript
globalThis.site = "InterviewPrep";
console.log(globalThis.site);
```

Output:

```text
InterviewPrep
```

### `var` vs `let` at the top level (browser)

In a classic browser script, top-level `var` becomes a property of `window`, but `let` and `const` do not.

```html
<script>
  var oldWay = "var";
  let newWay = "let";
  console.log(window.oldWay, window.newWay);
</script>
```

Output:

```text
var undefined
```

### Implicit globals

Assigning without declaring creates a global in sloppy mode and throws in strict mode.

```javascript
function leak() {
  accidental = "oops"; // no declaration
}

leak();
console.log(globalThis.accidental);
```

Output:

```text
oops
```

**Notice:** In a module or under `"use strict"`, this same line throws `ReferenceError: accidental is not defined`. Implicit globals are exactly what strict mode was designed to stop.

---

# 8. Best Practices

### Rules worth memorizing

- Use `const` by default; switch to `let` only when reassigning.
- Never use `var` in new code.
- Declare close to first use and initialize immediately, so the TDZ never bites.
- Avoid globals; share state through modules or function arguments.
- Avoid shadowing, especially across a long function.
- Prefer block-scoped loop variables; closures capture them per iteration.

### Bad vs Good

```javascript
// Bad — same var redeclared, outer value destroyed
var user = "Kakade";
if (true) {
  var user = "Asha";
}
console.log(user);

// Good — separate block-scoped bindings
const userName = "Kakade";
if (true) {
  const userName = "Asha";
  console.log(userName);
}
console.log(userName);
```

Output:

```text
Asha
Asha
Kakade
```

**Notice:** The first block prints only `Asha`; the second prints `Asha` then `Kakade`, because the inner `const` never touches the outer binding.

---

# Real-World Uses

### Configuration constants

```javascript
const API_URL = "https://jsonplaceholder.typicode.com";
const MAX_RETRIES = 3;
console.log(`${API_URL} / retries: ${MAX_RETRIES}`);
```

Output:

```text
https://jsonplaceholder.typicode.com / retries: 3
```

### Loop variables that behave

```javascript
const timers = [];
for (let i = 0; i < 3; i++) {
  timers.push(() => i);
}
console.log(timers.map((fn) => fn()));
```

Output:

```text
[ 0, 1, 2 ]
```

With `var`, all three functions would return `3`. This is the classic closure + scope bug — `let` fixes it by creating a fresh binding per iteration.

---

# Quick Revision

| Concept | Remember |
| --- | --- |
| **var, let and const** | `const` default, `let` for reassignment, never `var` |
| **Block scope** | `let`/`const` die at `}`; `var` ignores blocks |
| **Function scope** | `var` lives for the whole function |
| **Global scope** | Top-level bindings visible throughout the script |
| **Hoisting** | Declarations registered first; `var` starts as `undefined` |
| **Temporal dead zone** | `let`/`const` throw if read before initialization |
| **Redeclaration** | `var` allows it; `let`/`const` throw `SyntaxError` |
| **Shadowing** | Inner binding hides outer; `let` outer + `var` inner is illegal |
| **Global object** | `window` / `global` / `globalThis`; implicit globals are bugs |
| **Best practices** | `const` first, declare near use, avoid globals and shadowing |

## The Mental Model

```text
            DECLARATION
                 │
      ┌──────────┴──────────┐
      ↓                     ↓
   var                   let / const
      │                     │
 function scope         block scope
      │                     │
 hoisted as undefined   hoisted but TDZ
      │                     │
      └──────────┬──────────┘
                 ↓
        assignment initializes
                 ↓
        readable from here on
```

### The 3 things to remember

**1. Default to `const`:**

```javascript
const user = "Kakade"; // reassign → TypeError
```

**2. `let` is block-scoped:**

```javascript
{ let x = 1; }
// x is gone here
```

**3. Never read before initialization:**

```javascript
// console.log(x); let x = 1; → ReferenceError
```

> **Scope decides *where* a name is visible, hoisting decides *when* it exists, and the TDZ stops you from reading it too early.**
