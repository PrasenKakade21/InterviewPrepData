# Control Flow and Loops

Control flow is how a program **decides** which line runs next and **repeats** work. The chain is **condition → branch → loop → jump out early**. Beginners often memorise the syntax but miss the rules: `switch` uses strict equality and falls through, `for...in` walks keys (including inherited ones), and `let` vs `var` changes what a loop closure sees.

```text
Control flow
     ↓
condition   if / else if / else / switch / ternary
     ↓
repeat      for / while / do...while / for...of / for...in
     ↓
steer       break / continue / labels
     ↓
choose      the simplest construct that reads clearly
```

---

# 1. Conditionals

### if, else and else if

`if` runs a block when its condition is truthy. `else if` adds another test, and `else` is the fallback. JavaScript checks the chain **top to bottom and stops at the first truthy branch**, so order matters when ranges overlap.

```javascript
const marks = 72;

if (marks >= 90) {
  console.log("Grade A");
} else if (marks >= 60) {
  console.log("Grade B");
} else {
  console.log("Grade C");
}
```

Output:

```text
Grade B
```

Notice: `marks >= 90` was checked first. If the `>= 60` branch had come first, a score of 95 would also have matched it and printed `B`. Always order range checks from most specific to least.

Bad:

```javascript
let score = 72;
if (score = 90) {        // assignment, not comparison
  console.log("A");
}
```

Good:

```javascript
if (score === 90) {
  console.log("A");
}
```

The assignment version overwrites `score` and evaluates to `90`, which is truthy, so the branch always runs. Strict mode does not save you here. Use braces even for one line — it prevents the "added a second statement outside the `if`" bug.

### switch

`switch` compares the expression against each `case` using **strict equality (`===`)**. Without `break` or `return`, execution **falls through** into the next case. Grouping empty cases is the one time fallthrough is intentional.

```javascript
function priceOf(size) {
  switch (size) {
    case "S":
      return 40;
    case "M":
      return 60;
    case "L":
    case "XL":
      return 80;
    default:
      return 0;
  }
}

console.log(priceOf("M"), priceOf("XL"), priceOf("XXL"));

switch (2) {
  case 1:
    console.log("one");
  case 2:
    console.log("two");
  case 3:
    console.log("three");
}
```

Output:

```text
60 80 0
two
three
```

`priceOf("M")` returns immediately, so no fallthrough. The second switch has no `break`, so after matching `case 2` it also runs `case 3`. `switch` does **not** coerce: `switch ("1")` will not match `case 1`. Prefer `return` inside a function; it ends the switch and the function in one step.

### Ternary

The conditional operator `condition ? whenTrue : whenFalse` is an **expression**, so it produces a value. Use it for a short either/or assignment or argument, not for multi-step logic.

```javascript
const age = 20;
const type = age >= 18 ? "adult" : "minor";

console.log(type);
console.log(age >= 18 ? "can vote" : "cannot vote");
```

Output:

```text
adult
can vote
```

Bad — nested ternaries become unreadable:

```javascript
const label = age < 13 ? "child" : age < 18 ? "teen" : age < 60 ? "adult" : "senior";
```

Good — a table or `if` chain:

```javascript
function labelFor(age) {
  if (age < 13) return "child";
  if (age < 18) return "teen";
  if (age < 60) return "adult";
  return "senior";
}
console.log(labelFor(20));
```

Output:

```text
adult
```

Avoid ternaries for side effects (`ok ? log("yes") : log("no")`) — an `if` states the intent more clearly.

---

# 2. Loops

### for loop

The classic `for (init; condition; update)` is best when you know **how many times** to repeat or need the index. Initialisation runs once, the condition is checked before every pass, and the update runs after every pass.

```javascript
for (let i = 1; i <= 3; i++) {
  console.log(`Round ${i}`);
}

const fns = [];
for (var i = 0; i < 3; i++) fns.push(() => i);
console.log(fns.map((f) => f()));

const fns2 = [];
for (let j = 0; j < 3; j++) fns2.push(() => j);
console.log(fns2.map((f) => f()));
```

Output:

```text
Round 1
Round 2
Round 3
[ 3, 3, 3 ]
[ 0, 1, 2 ]
```

Notice the closure difference: `var` is **function-scoped**, so all three callbacks share one `i`, which is `3` after the loop. `let` creates a **new binding per iteration**, so each callback keeps its own `j`.

Bad — off-by-one:

```javascript
const names = ["Kakade", "Asha"];
for (let i = 0; i <= names.length; i++) console.log(names[i]);
```

Output:

```text
Kakade
Asha
undefined
```

Good — use `<`, or `for...of` when the index is not needed. `i <= length` visits one past the end.

### while loop

`while (condition)` checks **before** each pass, so it may run zero times. Use it when the number of iterations is unknown — polling, reading until empty, retrying until success.

```javascript
let attempts = 0;

while (attempts < 3) {
  attempts++;
  console.log(`Attempt ${attempts}`);
}
```

Output:

```text
Attempt 1
Attempt 2
Attempt 3
```

The loop variable must change **inside** the body. Forgetting the update is the classic infinite loop:

```javascript
let n = 3;
while (n > 0) {
  console.log(n);   // n never changes → runs forever
}
```

A `while (true)` with an explicit `break` is fine for "loop until found".

### do...while

`do...while` runs the body **once before** checking the condition. Use it when the body must execute at least once, such as showing a menu before asking whether to continue.

```javascript
let tries = 0;

do {
  tries++;
  console.log(`try ${tries}`);
} while (tries < 0);

console.log("done", tries);
```

Output:

```text
try 1
done 1
```

Even though `tries < 0` is false from the start, the body ran once — the only behavioural difference from `while`.

---

# 3. Iteration

### for...of

`for...of` walks the **values** of any iterable: arrays, strings, `Map`, `Set`, generators. It is the default choice for arrays because it cannot go out of bounds and supports `break` and `continue`.

```javascript
const names = ["Kakade", "Asha", "Rohan"];

for (const name of names) {
  console.log(name);
}

for (const [index, name] of names.entries()) {
  console.log(index, name);
}
```

Output:

```text
Kakade
Asha
Rohan
0 Kakade
1 Asha
2 Rohan
```

Bad — a plain object is **not iterable**:

```javascript
const person = { name: "Kakade" };
for (const value of person) console.log(value); // TypeError
```

Output:

```text
TypeError: person is not iterable
```

Good — iterate `Object.values(person)` / `Object.entries(person)` instead. Strings iterate by code point, so `for (const ch of "Kakade")` gives one character at a time.

### for...in

`for...in` walks **enumerable property keys**, including inherited ones, as strings. It was designed for objects, not arrays, and it lists integer-like keys first.

```javascript
const person = { name: "Rohan", age: 21 };

for (const key in person) {
  console.log(key, person[key]);
}

const child = Object.create({ inherited: true });
child.own = 1;
for (const key in child) console.log("all:", key);
console.log("own:", Object.keys(child));
```

Output:

```text
name Rohan
age 21
all: own
all: inherited
own: [ 'own' ]
```

Notice: `for...in` visited `inherited` because it lives on the prototype. Guard with `Object.hasOwn(child, key)`, or use `Object.keys` when you only want own keys.

Bad — using `for...in` on an array:

```javascript
const arr = ["a", "b"];
arr.extra = "x";
for (const k in arr) console.log(typeof k, k);
```

Output:

```text
string 0
string 1
string extra
```

Keys are strings, and custom properties appear too. Use `for...of` for values or `forEach` for index plus value. As a rule: **`for...of` for values, `for...in` for object keys — and even then prefer `Object.keys`.**

---

# 4. Flow Control

### break and continue

`break` leaves the **nearest enclosing loop or switch** immediately. `continue` skips the rest of the current iteration and jumps to the next one. Both affect only one level unless you use a label.

```javascript
for (let n = 1; n <= 6; n++) {
  if (n === 2) continue;
  if (n === 5) break;
  console.log(n);
}
```

Output:

```text
1
3
4
```

`2` was skipped, and the loop stopped before printing `5`. Inside a `switch`, `break` exits the switch, not an outer loop — a common source of bugs when a switch sits inside a `for`.

### Labels

A label names a loop so `break label` or `continue label` can target it. This is the clean way to exit nested loops without a flag variable.

```javascript
outer: for (let row = 0; row < 3; row++) {
  for (let col = 0; col < 3; col++) {
    if (row === 1 && col === 1) break outer;
    console.log(row, col);
  }
}

rows: for (let row = 0; row < 2; row++) {
  for (let col = 0; col < 3; col++) {
    if (col === 1) continue rows;
    console.log("rc", row, col);
  }
}
```

Output:

```text
0 0
0 1
0 2
1 0
rc 0 0
rc 1 0
```

`break outer` left both loops. `continue rows` skipped the rest of the inner loop and started the next outer iteration. Labels are rare and can hurt readability — first consider extracting the inner loop into a function that returns early.

---

# 5. Choosing a Loop

### Choosing a loop

Pick the construct that matches the shape of the data, not the one you used last time.

| Situation | Use | Why |
| --- | --- | --- |
| Known number of passes | `for` | Index and update in one line |
| Unknown count, test first | `while` | May run zero times |
| Must run at least once | `do...while` | Condition checked after the body |
| Values of an array/iterable | `for...of` | No index math, supports `break` |
| Keys of an object | `Object.keys` / `entries` | Own keys only, predictable |
| Transform every item | `map` / `filter` / `reduce` | Declarative, returns a value |
| Stop when found | `for...of` + `break`, or `.find()` | Early exit |

```javascript
const prices = [199, 499, 99];

let total = 0;
for (const price of prices) total += price;
console.log(total);

const firstOver = prices.find((price) => price > 200);
console.log(firstOver);
```

Output:

```text
797
499
```

`for...of` is the readable default for arrays. Reach for `for` only when you need the index or a custom step, and for `map`/`filter` when you are building a new array. `forEach` cannot `break` — if early exit matters, it is the wrong tool.

Interview traps:

- `switch` uses `===`, so `"1"` never matches `case 1`.
- Missing `break` falls through, sometimes intentionally.
- `for...in` includes inherited enumerable keys and gives strings.
- `for...of` throws on plain objects.
- `var` in a loop shares one binding; `let` creates one per iteration.
- `i <= arr.length` reads one element past the end.
- `continue` in a `forEach` callback is a syntax error; use `return` (which only skips the callback).

---

# Real-World Uses

- **Validation** — `if` chains and early returns keep guard clauses flat.
- **Rendering lists** — `for...of` to build DOM nodes, `map` to produce React-style arrays.
- **Pagination and retries** — `while` loops with an explicit exit condition.
- **Searching grids** — labelled `break` when a match is found in nested loops.

---

# Quick Revision

| Concept | Remember |
| --- | --- |
| **if, else and else if** | First truthy branch wins; order ranges carefully; `=` is not `===` |
| **switch** | Strict comparison; falls through without `break`/`return`; `default` optional |
| **Ternary** | Expression producing a value; avoid nesting and side effects |
| **for loop** | Known count; `let` gives one binding per iteration |
| **while loop** | Tests first, may run zero times; update the condition variable |
| **do...while** | Runs once before testing |
| **for...of** | Iterable values; arrays, strings, Map, Set; supports `break` |
| **for...in** | Enumerable keys incl. inherited; strings; avoid on arrays |
| **break and continue** | `break` exits nearest loop; `continue` skips one iteration |
| **Labels** | `break outer` / `continue outer` for nested loops |
| **Choosing a loop** | Use the shape of the data; `for...of` for arrays, methods for transforms |

## The Mental Model

```text
                    CONTROL FLOW
                         │
        ┌────────────────┼────────────────┐
        ↓                ↓                ↓
     decide           repeat            steer
   if / switch      for / while       break / continue
   ternary          do...while        labels
        │           for...of/in           │
        └────────────────┼────────────────┘
                         ↓
                 next statement
```

### The 3 things to remember

**1. `switch` falls through without `break`.**

```javascript
switch (2) {
  case 2: console.log("two");
  case 3: console.log("three"); // also runs
}
```

**2. `let` in a `for` header creates a fresh binding per iteration.**

```javascript
const fns = [];
for (let i = 0; i < 3; i++) fns.push(() => i);
fns.map((f) => f()); // [ 0, 1, 2 ]
```

**3. `for...of` reads values; `for...in` reads keys.**

```javascript
for (const v of ["a", "b"]) console.log(v);   // a b
for (const k in ["a", "b"]) console.log(k);   // 0 1
```

> **Decide with `if`/`switch`, repeat with the loop that matches the data, and use `break`/`continue` sparingly and explicitly.**
