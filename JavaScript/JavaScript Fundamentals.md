# JavaScript Fundamentals

JavaScript is the programming language that makes web pages interactive, and it also runs servers, command-line tools, and tests through Node.js. Learning it is a chain: you write **source code → the engine parses and executes it → you see a result** in the browser console, on the page, or in a terminal. This file covers what the language is, where it runs, how it evolved, and the syntax rules that trip people up.

The core idea:

```text
JavaScript source (.js)
        ↓
engine parses → compiles → executes
        ↓
     needs a host
        ↓
┌───────────────┴───────────────┐
↓                               ↓
browser                     Node.js
(window, DOM)               (files, process)
↓                               ↓
user sees a page            terminal sees output
```

---

# 1. What is JavaScript

### A programming language, not a framework

JavaScript is the **language**. React, Vue, and Express are libraries or frameworks *written in* JavaScript. The language is defined by a standard called **ECMAScript**; "ES2020" and "ES6" are versions of that standard, while "JavaScript" is the implementation you run.

```javascript
const name = "Kakade";
const role = "developer";
console.log(`${name} is a ${role}.`);
```

Output:

```text
Kakade is a developer.
```

### What the language gives you

Values, logic, functions, arrays, objects, `JSON`, promises, and modules. The DOM, `fetch`, and the file system come from the **host environment**.

---

# 2. Browser vs Node.js

### The same language, different environments

The engine (V8 in Chrome and Node) understands the core language. The **host** adds APIs on top, which is why `document` exists in a browser but not in Node, and `require` exists in Node but not in a browser.

```javascript
const scores = [70, 85, 92];
const average = scores.reduce((sum, n) => sum + n, 0) / scores.length;
console.log(average);
```

Output:

```text
82.33333333333333
```

The same code runs in both hosts because it only uses the language.

### What each environment adds

| Feature | Browser | Node.js |
| --- | --- | --- |
| Global object | `window` | `globalThis` (alias `global`) |
| DOM (`document`) | Yes | No |
| File system (`fs`) | No | Yes |
| `process`, `Buffer` | No | Yes |

### A browser-only example

```html
<div id="greeting">Hello</div>
<script>
  document.querySelector("#greeting").textContent = "Hello Kakade";
  console.log(typeof window, typeof document);
</script>
```

Output:

```text
object object
```

---

# 3. ECMAScript Versions

### Spec vs implementation

**ECMAScript** is the written specification; **JavaScript** is what browsers and Node implement. Engines may ship features slightly before or after the spec is finalized.

### The versions that matter

| Version | Year | Headline features |
| --- | --- | --- |
| ES5 | 2009 | `"use strict"`, `JSON`, `map`/`filter` |
| ES6 / ES2015 | 2015 | `let`/`const`, arrows, classes, promises, modules |
| ES2017 | 2017 | `async` / `await`, `Object.entries` |
| ES2020 | 2020 | `?.`, `??`, `BigInt`, `Promise.allSettled` |
| ES2021 | 2021 | `replaceAll`, `Promise.any` |
| ES2022 | 2022 | class fields, top-level `await`, `.at()` |

### Modern syntax in one snippet

```javascript
const user = { name: "Rohan" };
console.log(user.name.toUpperCase());
console.log(user.address?.city ?? "no city");
```

Output:

```text
ROHAN
no city
```

**Notice:** `?.` and `??` are ES2020. Asked to avoid them, write `user.address && user.address.city`.

---

# 4. Statements and Expressions

### Statements do something

A **statement** is an instruction. It performs an action and does not produce a usable value: `let x = 5;`, `if (...) {}`, `for (...) {}`, `return`.

### Expressions produce a value

An **expression** evaluates to a value: `5 + 3`, `"Kakade".toUpperCase()`, `user.name`, `x > 10`. Expressions can go anywhere a value is expected — inside a call, on the right of `=`, or in a template literal.

```javascript
const price = 200;
const taxPercent = 9;

// "price * taxPercent / 100" is an expression inside a statement
const total = price + (price * taxPercent) / 100;
console.log(total);
```

Output:

```text
218
```

### Why the difference matters

An `if` statement cannot be assigned to a variable, but a ternary **expression** can: `const label = age >= 18 ? "adult" : "minor";`.

---

# 5. Semicolons

### Automatic Semicolon Insertion (ASI)

JavaScript does not strictly require semicolons. The parser inserts them at line breaks when the next token cannot continue the current statement, which is why both styles exist.

```javascript
const numbers = [1, 2, 3]
const doubled = numbers.map((n) => n * 2)
console.log(doubled)
```

Output:

```text
[ 2, 4, 6 ]
```

### The classic ASI trap

`return` followed by a newline gets a semicolon inserted immediately, so the value on the next line is never returned.

```javascript
function getTotal() {
  return
  42;
}

console.log(getTotal());
```

Output:

```text
undefined
```

### Bad vs Good

```javascript
// Bad — parsed as 10(...)(), so it throws
const total = 10
(function () { console.log(total) })()
```

Output:

```text
TypeError: 10 is not a function
```

```javascript
// Good — explicit semicolon
const safeTotal = 10;
(function () { console.log(safeTotal); })();
```

Output:

```text
10
```

**Notice:** Pick one style and stay consistent. If you omit semicolons, lead lines starting with `(`, `[`, or `` ` `` with `;`.

---

# 6. Comments

### Line and block comments

```javascript
// Line comment: explains one line.

/*
  Block comment: explains a larger idea.
*/
const price = 200;
console.log(price);
```

Output:

```text
200
```

Comments are ignored by the engine, so they never affect output. Comment the **why**, not the **what**: `// add 1 to count` is noise, while `// First retry is free, so start at 1` is useful.

### JSDoc comments

JSDoc is a comment convention editors understand. It gives autocomplete and type hints without TypeScript.

```javascript
/**
 * @param {number} amount - value in rupees
 * @returns {string} formatted price
 */
const formatPrice = (amount) => `Rs. ${amount.toFixed(2)}`;

console.log(formatPrice(1499));
```

Output:

```text
Rs. 1499.00
```

**Notice:** Block comments cannot be nested and cannot contain `*/` inside — both are syntax errors.

---

# 7. Strict Mode

### Turning it on

Add `"use strict";` as the **first statement** of a file or function. Comments may come before it; anything else (even a variable declaration) disables it.

```javascript
"use strict";

function showThis() {
  return this;
}

console.log(showThis());

try {
  undeclared = 5;
} catch (error) {
  console.log(error.name);
}
```

Output:

```text
undefined
ReferenceError
```

### What strict mode changes

| Behaviour | Sloppy mode | Strict mode |
| --- | --- | --- |
| Assign to undeclared variable | Creates a global | `ReferenceError` |
| `this` in a plain function call | Global object | `undefined` |
| Delete a variable | Silent | `SyntaxError` |

### Where strict mode is automatic

ES modules and class bodies are always strict.

**Think:** Strict mode does not change normal behaviour. It turns silent mistakes into loud errors — exactly what you want while debugging.

---

# 8. Console and Debugging

### The console is your first debugger

`console.log` prints values; `console.warn` and `console.error` print to the warning/error stream, which browser devtools colour differently.

```javascript
const order = { id: 101, customer: "Kakade", total: 1499 };

console.log("order:", order);
console.warn("Stock is low");
console.error("Payment declined");
console.log(`Total: ₹${order.total}`);
```

Output:

```text
order: { id: 101, customer: 'Kakade', total: 1499 }
Stock is low
Payment declined
Total: ₹1499
```

### Useful console methods

`console.table` renders arrays of objects as a table, `console.dir` inspects properties, `console.time`/`timeEnd` measures duration, and `console.trace` prints the call stack.

### The `debugger` statement

`debugger;` pauses execution in devtools, as if you had set a breakpoint. It does nothing when devtools is closed.

```javascript
function applyDiscount(price, percent) {
  const discount = (price * percent) / 100;
  debugger; // inspect values here
  return price - discount;
}
console.log(applyDiscount(2000, 10));
```

Output:

```text
1800
```

---

# 9. Script Loading

### Four ways to load a script

```html
<!-- 1. Blocking: HTML parsing stops while the file downloads and runs -->
<script src="app.js"></script>

<!-- 2. Deferred: download in parallel, run after parsing, in order -->
<script src="app.js" defer></script>

<!-- 3. Async: download in parallel, run as soon as ready -->
<script src="app.js" async></script>

<!-- 4. Module: deferred by default, strict mode, own scope -->
<script type="module" src="app.js"></script>
```

Output:

```text
classic → immediate, blocks parsing
defer   → after parsing, keeps order
async   → when ready, order can change
module  → deferred, strict, top-level await
```

### When to use which

Use `defer` for most page scripts, `async` for independent widgets like analytics, and `type="module"` for modern app code.

### The DOM timing trap

A script placed before the HTML runs before the elements exist. Move it to the end of `<body>` or load it with `defer`.

```html
<script src="app.js" defer></script>
<button id="save">Save</button>
```

Output:

```text
app.js runs after parsing, when #save exists
```

**Notice:** `defer` only works on external scripts (`src`), not on inline `<script>` blocks.

---

# 10. Developer Tools

### The panels you will actually use

| Panel | What it answers |
| --- | --- |
| **Elements** | What is the DOM right now? |
| **Console** | What does this expression evaluate to? |
| **Sources** | Where is this code, and how do I pause it? |
| **Network** | Which requests ran, and how long did they take? |
| **Application** | What is in localStorage, cookies, and the cache? |

### Inspecting the page live

Type `document.title` in the Console to read it, or `document.title = "Kakade"` to change it.

**Notice:** Read the **first** red error — later errors are usually consequences. Never ship `debugger;` or noisy logs.

---

# Real-World Uses

### Validating a form in the browser

```javascript
const email = "kakade@example.com";
const isValid = email.includes("@") && email.includes(".");
console.log(`Valid: ${isValid}`);
```

Output:

```text
Valid: true
```

### A Node.js script

```javascript
const args = process.argv.slice(2);
console.log(`Hello ${args[0] ?? "Kakade"}`);
```

Output (run as `node app.js Asha`):

```text
Hello Asha
```

---

# Quick Revision

| Concept | Remember |
| --- | --- |
| **What is JavaScript** | A language defined by ECMAScript |
| **Browser vs Node.js** | Same language, different host APIs |
| **ECMAScript versions** | ES6 (2015) is the big one; `?.`/`??` are ES2020 |
| **Statements vs expressions** | Statements act; expressions produce a value |
| **Semicolons** | ASI inserts them; `return` + newline is `undefined` |
| **Comments** | `//`, `/* */`, JSDoc; never executed |
| **Strict mode** | First line only; no implicit globals |
| **Console and debugging** | `log`, `warn`, `error`, `time`, `debugger` |
| **Script loading** | `defer` = ordered after parse; `async` = whenever ready |
| **Developer tools** | Elements, Console, Sources, Network, Application |

## The Mental Model

```text
                 JAVASCRIPT
                     │
        ┌────────────┴────────────┐
        ↓                         ↓
   the language              the host
   (ECMAScript)          ┌───────┴────────┐
        │                 ↓                ↓
        ↓              browser           Node.js
  syntax + types      window, DOM       process, fs
        │                 │                │
        └────────┬────────┘                │
                 ↓                         ↓
          statements & expressions   terminal output
                 │
                 ↓
          console / debugger / devtools
```

### The 3 things to remember

**1. Choose the host deliberately:**

```javascript
console.log(typeof window); // "undefined" in Node, "object" in a browser
```

**2. Statements act, expressions return:**

```javascript
const label = 18 >= 18 ? "adult" : "minor"; // expression
```

**3. Strict mode turns silence into errors:**

```javascript
"use strict";
// undeclared = 1; // ReferenceError instead of a hidden global
```

> **JavaScript is one language with two homes — the browser gives it the DOM, Node gives it the filesystem, and the syntax is identical in both.**
