# Arrays

An array is an **ordered, index-based list**. You create it, read and change positions, then transform it with `map`, `filter` and `reduce`. The chain is **create → index → mutate or transform → return a new array**; the key interview theme is knowing which methods mutate.

```text
Array
  ↓
create   [1, 2, 3] / Array.from(...)
  ↓
read     arr[0], arr.length, arr.at(-1)
  ↓
change   push / pop / splice / sort   (mutates)
  ↓
transform  map / filter / reduce      (new array)
```

---

# 1. Creating and Reading Arrays

### Array creation

An array literal is almost always what you want. `new Array(n)` creates **empty slots** (holes), not `n` zeros — a classic bug source.

```javascript
const names = ["Kakade", "Asha", "Rohan"];
const scores = new Array(3);

console.log(names, names.length);
console.log(scores, scores.length);
```

Output:

```text
[ 'Kakade', 'Asha', 'Rohan' ] 3
[ <3 empty items> ] 3
```

`Array(3)` = three holes; `Array(3, 4)` = two elements; `Array.of(3)` = one element. Arrays are objects, so `typeof []` is `"object"` — use `Array.isArray()`.

### Indexing and length

Indexes start at `0`. `length` is one more than the highest index, and writing it **truncates** or extends the array.

```javascript
const list = ["a", "b", "c"];

console.log(list[0], list.at(-1), list[9]);
list.length = 2;
console.log(list);
```

Output:

```text
a c undefined
[ 'a', 'b' ]
```

Notice: `list[-1]` is `undefined` (a property named `"-1"`), which is why `.at(-1)` exists.

---

# 2. Adding and Removing at the Ends

### push and pop

`push` adds to the **end** and returns the **new length**. `pop` removes from the end and returns the **removed element**.

```javascript
const stack = ["Kakade"];

console.log(stack.push("Asha"), stack);
console.log(stack.pop(), stack);
```

Output:

```text
2 [ 'Kakade', 'Asha' ]
Asha [ 'Kakade' ]
```

These are O(1) and make arrays work as stacks. Trap: `arr.push(x)` returns the **new length**, not `arr`.

### shift and unshift

`shift` removes the **first** element; `unshift` adds to the **front**. Both return like `pop`/`push`, and both are **O(n)** because every other index must move.

```javascript
const queue = ["Asha", "Rohan"];

console.log(queue.shift(), queue);
console.log(queue.unshift("Kakade"), queue);
```

Output:

```text
Asha [ 'Rohan' ]
2 [ 'Kakade', 'Rohan' ]
```

For a real queue with thousands of items, keep an index pointer instead of repeated `shift()`.

---

# 3. Slicing and Combining

### splice and slice

Names look alike, behaviour is opposite. `splice` **mutates** and returns the **removed** items. `slice` **copies** a range; the end index is **excluded**.

```javascript
const days = ["Mon", "Tue", "Wed", "Thu"];

console.log(days.splice(1, 2));
console.log(days);

const week = ["Mon", "Tue", "Wed", "Thu"];
console.log(week.slice(1, 3), week.slice(-2));
console.log(week);
```

Output:

```text
[ 'Tue', 'Wed' ]
[ 'Mon', 'Thu' ]
[ 'Tue', 'Wed' ] [ 'Wed', 'Thu' ]
[ 'Mon', 'Tue', 'Wed', 'Thu' ]
```

`splice(start, deleteCount, ...items)` can also insert and replace: `days.splice(1, 0, "Sun")` inserts without deleting. Negative indexes count from the end.

### concat and join

`concat` merges arrays into a **new** array (it flattens one level of array arguments). `join` turns an array into a **string** using a separator, default `","`.

```javascript
const a = [1, 2];
const b = [3, 4];

console.log(a.concat(b, 5), a);
console.log(["Kakade", "Asha"].join(" & "));
```

Output:

```text
[ 1, 2, 3, 4, 5 ] [ 1, 2 ]
Kakade & Asha
```

`join` with no argument uses `","`. Prefer `[...a, ...b]` for readability; `concat` is still useful because it accepts non-array values.

---

# 4. Searching

### includes and indexOf

`includes` returns a boolean and finds `NaN`. `indexOf` returns the index (or `-1`) and uses strict equality, so it **cannot** find `NaN`.

```javascript
const marks = [45, 67, 89, 67];

console.log(marks.includes(67), marks.indexOf(67));
console.log([1, NaN].includes(NaN), [1, NaN].indexOf(NaN));
```

Output:

```text
true 1
true -1
```

`includes` uses SameValueZero (`NaN` matches); `indexOf` uses `===`. Neither compares objects by value: `[{ a: 1 }].includes({ a: 1 })` is `false`.

### find and findIndex

`find` returns the **first matching element**; `findIndex` returns its index. No match gives `undefined` and `-1`.

```javascript
const users = [
  { name: "Kakade", role: "admin" },
  { name: "Asha", role: "editor" },
];

console.log(users.find((u) => u.role === "editor").name);
console.log(users.findIndex((u) => u.name === "Asha"));
console.log(users.find((u) => u.role === "owner"));
```

Output:

```text
Asha
1
undefined
```

Bad: `users.find(...).name` without a guard — throws when nothing matches. Good: `users.find(...)?.name ?? "Guest"`.

---

# 5. Transforming Arrays

### filter

`filter` keeps the elements whose callback returns truthy. It always returns a **new array** (possibly empty) and never mutates.

```javascript
const ages = [12, 18, 25, 15, 30];

const adults = ages.filter((age) => age >= 18);
console.log(adults);
console.log(ages.filter((age) => age > 100));
```

Output:

```text
[ 18, 25, 30 ]
[]
```

### map

`map` transforms **every** element, one output per input. It returns a new array of the **same length**.

```javascript
const prices = [100, 250, 400];

console.log(prices.map((p) => p * 1.18));
console.log(prices.map((p, i) => `${i}:${p}`));
```

Output:

```text
[ 118, 295, 472 ]
[ '0:100', '1:250', '2:400' ]
```

Trap — `map` passes **three** arguments `(element, index, array)`, so handing it a function that takes a second parameter breaks:

```javascript
console.log([1, 2, 3].map(parseInt));         // index becomes the radix
console.log([1, 2, 3].forEach((n) => n * 2)); // forEach returns undefined
```

Output:

```text
[ 1, NaN, NaN ]
undefined
```

`parseInt(2, 1)` is `NaN` — index `1` is not a valid radix. Fix with `map((n) => parseInt(n, 10))`. And since `forEach` returns `undefined`, it cannot be chained.

### reduce

`reduce` folds the array into **one value** — number, string, object, or array. The callback gets `(accumulator, current, index, array)`.

```javascript
const cart = [199, 499, 99];
console.log(cart.reduce((sum, price) => sum + price, 0));

const byRole = ["admin", "editor", "admin"].reduce((acc, role) => {
  acc[role] = (acc[role] ?? 0) + 1;
  return acc;
}, {});
console.log(byRole);
```

Output:

```text
797
{ admin: 2, editor: 1 }
```

**Always pass an initial value.** Without it the first element becomes the accumulator, and `[].reduce((a, b) => a + b)` throws `TypeError` — there is nothing to start from.

```javascript
console.log([1, 2, 3, 4].reduce((acc, n) => acc + n, 100)); // 110
```

Output:

```text
110
```

### some and every

`some` asks "does **at least one** match?"; `every` asks "do **all** match?". Both short-circuit and return booleans.

```javascript
const words = ["apple", "avocado", "banana"];

console.log(words.some((w) => w.startsWith("a")));
console.log(words.every((w) => w.startsWith("a")));
console.log([].some((w) => w), [].every((w) => w));
```

Output:

```text
true
false
false true
```

Empty-array defaults: `some` is `false` (no witness), `every` is `true` (vacuously) — a favourite interview question.

---

# 6. Sorting and Flattening

### sort

`sort()` mutates and, **without a comparator, converts items to strings and compares them lexicographically**.

```javascript
const nums = [10, 9, 1, 100];

console.log(nums.sort());
console.log([10, 9, 1].sort((a, b) => a - b));
console.log(nums);
```

Output:

```text
[ 1, 10, 100, 9 ]
[ 1, 9, 10 ]
[ 1, 10, 100, 9 ]
```

Bad: `nums.sort()` on numbers — `"10" < "9"` because `"1" < "9"`. Good: `nums.sort((a, b) => a - b)`. For objects, compare a field: `people.sort((a, b) => a.age - b.age)`.

`sort` returns the same array and is stable; `toSorted()` returns a copy.

### flat and flatMap

`flat(depth)` removes nested arrays up to `depth` (default `1`). `flatMap` maps **and** flattens exactly **one** level.

```javascript
const nested = [1, [2, [3, [4]]]];

console.log(nested.flat());
console.log(nested.flat(Infinity));
console.log([1, 2, 3].flatMap((n) => [n, n * 2]));
console.log([1, 2].flatMap((n) => [[n]]));
```

Output:

```text
[ 1, 2, [ 3, [ 4 ] ] ]
[ 1, 2, 3, 4 ]
[ 1, 2, 2, 4, 3, 6 ]
[ [ 1 ], [ 2 ] ]
```

`flatMap` is not "map then deep flatten" — the last line stays nested because only one level is removed.

---

# 7. Static Helpers and Mutation Rules

### Array.from and Array.of

`Array.from` builds an array from any **iterable or array-like** value, with an optional mapper. `Array.of` avoids the `new Array(n)` hole trap.

```javascript
console.log(Array.from("Kakade"));
console.log(Array.from({ length: 3 }, (_, i) => i * 2));
console.log(Array.of(3), Array.of(3, 4));
```

Output:

```text
[ 'K', 'a', 'k', 'a', 'd', 'e' ]
[ 0, 2, 4 ]
[ 3 ] [ 3, 4 ]
```

`Array.from({ length: n })` is the standard way to make a **filled** range, because the mapper runs on holes — unlike `map` on `new Array(n)`, which skips them.

### Copying arrays

`=` copies the **reference**, not the contents. `[...arr]` and `arr.slice()` make a **shallow** copy — a new outer array, but nested objects are still shared.

```javascript
const orig = [1, 2, 3];
const copy = [...orig];
copy.push(4);
console.log(orig, copy);

const nested = [[1], [2]];
const shallow = [...nested];
shallow[0].push(99);      // nested object still shared
console.log(nested);

const deep = structuredClone(nested);
deep[1].push(42);
console.log(nested, deep);
```

Output:

```text
[ 1, 2, 3 ] [ 1, 2, 3, 4 ]
[ [ 1, 99 ], [ 2 ] ]
[ [ 1, 99 ], [ 2 ] ] [ [ 1 ], [ 2, 42 ] ]
```

Bad: `const copy = orig;` — `copy.push()` edits the original. Good: `[...orig]`. For nested data use `structuredClone`.

### Mutating vs non-mutating

This is the table to memorise:

| Mutates the original | Returns something new |
| --- | --- |
| `push`, `pop`, `shift`, `unshift` | `slice`, `concat`, `join` |
| `splice`, `sort`, `reverse` | `map`, `filter`, `flat`, `flatMap` |
| `fill`, `copyWithin` | `toSorted`, `toReversed`, `toSpliced`, `with` |

```javascript
const base = [3, 1, 2];
const sortedCopy = base.toSorted((a, b) => a - b);

console.log(base, sortedCopy);
console.log(base.with(0, 99), base);
```

Output:

```text
[ 3, 1, 2 ] [ 1, 2, 3 ]
[ 99, 1, 2 ] [ 3, 1, 2 ]
```

`toSorted`/`toReversed`/`toSpliced`/`with` are ES2023 and do not touch the original — handy in React state updates where mutation breaks re-renders.

---

# Real-World Uses

- **Lists and tables** — `map` for rows, `filter` for search, `sort` for columns.
- **Carts** — `reduce` for totals and grouping by category.
- **APIs** — `Array.from(new Set(...))` to dedupe, `flatMap` to expand orders.

---

# Quick Revision

| Concept | Remember |
| --- | --- |
| **Array creation** | Literal wins; `Array(3)` = holes, `Array(3, 4)` = 2 items |
| **Indexing and length** | 0-based; writable `length` truncates; `at(-1)` = last |
| **push and pop** | End; `push` → length, `pop` → element |
| **shift and unshift** | Front; **O(n)** |
| **splice and slice** | `splice` mutates + returns removed; `slice` copies |
| **concat and join** | `concat` → array; `join` → string |
| **includes and indexOf** | `includes` finds `NaN`; `indexOf` uses `===` |
| **find and findIndex** | First match; `undefined` / `-1` |
| **filter** | Keeps truthy results; new array |
| **map** | One output per input; `map(parseInt)` trap |
| **reduce** | Folds to one value; pass an initial value |
| **some and every** | Short-circuit; empty: `false` / `true` |
| **sort** | Lexicographic by default, mutates; `(a, b) => a - b` |
| **flat and flatMap** | `flat(depth)` default 1; `flatMap` one level |
| **Array.from and Array.of** | From iterables; `of` avoids holes |
| **Copying arrays** | `[...a]` shallow; `structuredClone` deep |
| **Mutating vs non-mutating** | push/pop/splice/sort mutate; map/filter/slice do not |

## The Mental Model

```text
                     ARRAY
         ┌────────────┼────────────┐
         ↓            ↓            ↓
      create        read        change
    [] / from    [i] / at     push/splice/sort
         └────────────┼────────────┘
                      ↓
                  transform
         ┌────────────┼────────────┐
         ↓            ↓            ↓
        map        filter       reduce
      (1 → 1)    (1 → 0|1)    (many → 1)
         └────────────┼────────────┘
                      ↓
                NEW array (unless mutating)
```

### The 3 things to remember

**1. `sort()` is lexicographic by default.**

```javascript
[10, 9, 1].sort();                 // [ 1, 10, 9 ]
[10, 9, 1].sort((a, b) => a - b);  // [ 1, 9, 10 ]
```

**2. `map` returns, `forEach` does not.**

```javascript
[1, 2].map((n) => n * 2);     // [2, 4]
[1, 2].forEach((n) => n * 2); // undefined
```

**3. Give `reduce` an initial value.**

```javascript
[].reduce((a, b) => a + b);    // TypeError
[].reduce((a, b) => a + b, 0); // 0
```

> **Arrays are ordered lists: know which methods mutate (push, splice, sort) and which return new arrays (map, filter, slice), and always give `reduce` an initial value.**
