# Control Flow and Loops — Revision Notes (Beginner)

> One-line mental model: decide with conditions, repeat with the loop that matches the data, and steer with `break` / `continue`.

## Core recall

- **if / else if / else**: first truthy branch wins; order ranges from specific to general; `=` assigns, `===` compares.
- **switch**: strict comparison (`===`), so `"1"` never matches `case 1`; without `break` or `return` it **falls through**; `default` is optional.
- **Ternary**: `cond ? a : b` is an expression producing a value; avoid nesting and side effects.
- **for**: known count, index, custom step. `(init; condition; update)`; condition checked before each pass.
- **while**: condition checked **before** the body, may run zero times; update the variable inside.
- **do...while**: body runs **once before** the condition is tested.
- **for...of**: iterable **values** (arrays, strings, Map, Set); supports `break`; plain objects throw `TypeError`.
- **for...in**: enumerable **keys** as strings, including inherited ones; do not use on arrays.
- **break / continue**: `break` exits the nearest loop or switch; `continue` skips one iteration.
- **Labels**: `outer: for (...)` plus `break outer` / `continue outer` targets a named outer loop.
- **Choosing a loop**: known count → `for`; unknown → `while`; at least once → `do...while`; values → `for...of`; transform → `map`/`filter`/`reduce`.

## Must-remember code

```javascript
// classic counted loop
for (let i = 1; i <= 3; i++) {
  console.log(i); // 1 2 3
}
```

```javascript
// values, with index when needed
for (const name of names) console.log(name);
for (const [i, name] of names.entries()) console.log(i, name);
```

```javascript
// object keys, own only
for (const [key, value] of Object.entries(person)) {
  console.log(key, value);
}
```

```javascript
// early exit from nested loops
outer: for (let row = 0; row < grid.length; row++) {
  for (let col = 0; col < grid[row].length; col++) {
    if (grid[row][col] === target) break outer;
  }
}
```

## Interview traps

- Missing `break` in `switch` falls through into the next case (sometimes deliberately, for grouped cases).
- `switch` does not coerce: `case 1` will not match `"1"`.
- `for...in` on an array yields string indexes **and** custom properties.
- `for...in` also visits inherited enumerable keys; guard with `Object.hasOwn`.
- `for...of` on `{ a: 1 }` throws `TypeError: not iterable`.
- `var` in a `for` header shares one binding → all closures see the final value.
- `let` in a `for` header creates a fresh binding per iteration.
- `i <= arr.length` visits one past the end (`undefined`).
- `continue` inside a `forEach` callback is a syntax error; `return` only skips that callback.
- `do...while` runs at least once even when the condition is false.

## 3 things to remember

1. **Switch is strict and falls through.**
   ```javascript
   switch ("1") {
     case 1: console.log("number"); break;
     case "1": console.log("string"); break;
   }
   // string
   ```

2. **let fixes the loop closure bug.**
   ```javascript
   const fns = [];
   for (let i = 0; i < 3; i++) fns.push(() => i);
   fns.map((f) => f()); // [ 0, 1, 2 ]
   ```

3. **Values vs keys.**
   ```javascript
   for (const v of ["a", "b"]) console.log(v); // a b
   for (const k in ["a", "b"]) console.log(k); // 0 1
   ```

> **Use `for...of` for values, `Object.entries` for object keys, and always give `switch` cases a `break` or `return`.**
