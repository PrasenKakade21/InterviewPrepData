# Destructuring, Rest and Spread

Three related syntaxes do three different jobs: **destructuring unpacks**, **rest collects**, **spread expands**. They share `...` and `[]`/`{}` shapes, which is why beginners mix them up. The chain is: **read a value → unpack what you need → collect the remainder or spread it somewhere new**.

```text
const { name, ...rest } = user;
        │            │
     unpack       collect leftovers
        │
   [...items]  →  expand into a new array
```

---

# 1. Array Destructuring

Array destructuring assigns by **position**. The variable names are yours; the order is the array's.

### Basic positional binding

```javascript
const members = ['Kakade', 'Asha', 'Rohan'];
const [first, second] = members;

console.log(first, second);
console.log(members.length);
```

Output:

```text
Kakade Asha
3
```

The array is not modified — destructuring only reads.

### Skipping holes

```javascript
const [, , third] = ['Kakade', 'Asha', 'Rohan'];
const [lead, ...others] = ['Kakade', 'Asha', 'Rohan'];

console.log(third);
console.log(lead, others);
```

Output:

```text
Rohan
Kakade [ 'Asha', 'Rohan' ]
```

An empty slot skips a position. `...others` must be **last** and collects everything remaining into a real array.

> **Trap:** destructuring does not require an array — any **iterable** works, including strings and `Set`s.

---

# 2. Object Destructuring

Object destructuring assigns by **key name**, not position. Order in the pattern is irrelevant.

### By key, not order

```javascript
const kakade = { name: 'Kakade', role: 'admin', city: 'Pune' };
const { role, name } = kakade;

console.log(name, role);
```

Output:

```text
Kakade admin
```

### Defaults only apply to `undefined`

```javascript
const asha = { name: 'Asha', age: null };

const { age = 30, city = 'Mumbai' } = asha;

console.log(age, city);
```

Output:

```text
null Mumbai
```

`age` is `null`, so the default is **not** used. Defaults trigger only when the value is `undefined` (including a missing key). This is the most common destructuring bug.

### Renaming

```javascript
const rohan = { name: 'Rohan', role: 'tester' };
const { name: userName, role: jobTitle } = rohan;

console.log(userName, jobTitle);
console.log(typeof name);
```

Output:

```text
Rohan tester
undefined
```

`name: userName` means "read key `name`, store in `userName`". The original key name is not declared as a variable.

### Defaults plus renaming

```javascript
const { name: member = 'Guest', role = 'member' } = { role: 'admin' };
console.log(member, role);
```

Output:

```text
Guest admin
```

Read as: key `name` → variable `member`, fallback `'Guest'`.

---

# 3. Nested Destructuring

Patterns can nest to match nested data. Each level needs its own pattern.

### Arrays inside objects

```javascript
const team = {
  name: 'Platform',
  members: ['Kakade', 'Asha'],
};

const { name, members: [lead, second] } = team;

console.log(name, lead, second);
```

Output:

```text
Platform Kakade Asha
```

### Guard nested objects with defaults

Bad — `user` is missing, so reading `.address` throws:

```javascript
try {
  const { user: { city } } = {};
  console.log(city);
} catch (error) {
  console.log(error.constructor.name);
}
```

Output:

```text
TypeError
```

Good — default the intermediate object:

```javascript
const { user: { city = 'Pune' } = {} } = {};
console.log(city);
```

Output:

```text
Pune
```

The `= {}` applies when `user` is `undefined`; `city = 'Pune'` applies when `city` is `undefined`. Notice that a leaf default cannot protect a missing parent.

---

# 4. Rest in Arrays and Objects

`...` on the **left** side of an assignment is **rest**: it collects what is left. It must be the last element and produces a new array or object.

> **Notice:** rest and spread use the same dots but sit on opposite sides — left collects, right expands.

| | Rest (`...` left) | Spread (`...` right) |
| --- | --- | --- |
| Job | collect the remainder | expand into values |
| Position | last only | anywhere, repeatable |
| Result | new array/object | new array/object or call args |

### Rest in arrays

```javascript
const scores = [90, 80, 70, 60];
const [top, ...remaining] = scores;

console.log(top);
console.log(remaining);
console.log(Array.isArray(remaining));
```

Output:

```text
90
[ 80, 70, 60 ]
true
```

### Rest in objects

```javascript
const kakade = { name: 'Kakade', age: 30, city: 'Pune', role: 'admin' };
const { name, ...details } = kakade;

console.log(name);
console.log(details);
```

Output:

```text
Kakade
{ age: 30, city: 'Pune', role: 'admin' }
```

Object rest copies **own enumerable** properties and drops the extracted key. It is commonly used to strip fields before sending data to an API.

> **Trap:** rest is shallow. Nested objects in the rest result point to the same objects as the source.

---

# 5. Spread in Arrays

`...` on the **right** side is **spread**: it expands an iterable into individual values.

### Copy and combine

```javascript
const weekdays = ['Mon', 'Tue'];
const weekend = ['Sat', 'Sun'];

const week = [...weekdays, 'Wed', ...weekend];

console.log(week);
console.log(week.length);
```

Output:

```text
[ 'Mon', 'Tue', 'Wed', 'Sat', 'Sun' ]
5
```

### Copying is shallow

```javascript
const original = [{ name: 'Kakade' }];
const copy = [...original];

copy.push({ name: 'Asha' });
copy[0].name = 'Rohan';

console.log(original.length, copy.length);
console.log(original[0].name);
```

Output:

```text
1 2
Rohan
```

The array is new, but the objects inside are shared. Spread is a **shallow** copy.

### Any iterable expands

```javascript
console.log([...'Kakade']);
console.log([...new Set([1, 1, 2, 3])]);
console.log(Math.max(...[10, 25, 7]));
```

Output:

```text
[ 'K', 'a', 'k', 'a', 'd', 'e' ]
[ 1, 2, 3 ]
25
```

Spread works on strings, `Set`, `Map` (as entry pairs), and array-likes with an iterator. `Math.max(...arr)` is a common idiom, but not for very large arrays (argument limit).

---

# 6. Spread in Objects

Object spread copies **own enumerable** properties into a new object. Later properties overwrite earlier ones.

### Shallow merge and override

```javascript
const defaults = { theme: 'light', lang: 'en', fontSize: 14 };
const userPrefs = { theme: 'dark' };

const settings = { ...defaults, ...userPrefs, fontSize: 16 };

console.log(settings);
```

Output:

```text
{ theme: 'dark', lang: 'en', fontSize: 16 }
```

Order matters: `...defaults` first, then overrides, then explicit values last.

### Prototype and getters are not copied

```javascript
const base = { inherited: true };
const child = Object.create(base);
child.own = 1;

const spread = { ...child };

console.log(spread);
console.log(spread.inherited);
```

Output:

```text
{ own: 1 }
undefined
```

Only **own enumerable** keys are copied — the prototype is lost. Getter values are evaluated and copied as plain data, not as accessors.

---

# 7. Function Parameters

Destructuring and rest are heavily used in function signatures to make options readable.

### Destructured options object

```javascript
function connect({ host = 'localhost', port = 80, secure = false } = {}) {
  return `${secure ? 'https' : 'http'}://${host}:${port}`;
}

console.log(connect());
console.log(connect({ host: 'api.dev', port: 443, secure: true }));
```

Output:

```text
http://localhost:80
https://api.dev:443
```

The trailing `= {}` is essential: without it, `connect()` throws because the whole argument is `undefined`.

### Rest parameters

```javascript
function sum(first, ...rest) {
  return rest.reduce((total, n) => total + n, first);
}

console.log(sum(1, 2, 3, 4));
console.log(sum(10));
```

Output:

```text
10
10
```

Rest parameters collect extra arguments into a **real array** (unlike the old `arguments` object) and must be last.

### Bad vs Good: rest vs spread at the call site

```javascript
const nums = [1, 2, 3];

function add(a, b, c) {
  return a + b + c;
}

console.log(add(...nums)); // spread at the call
console.log(add(nums));    // Bad: passes the whole array as `a`
```

Output:

```text
6
1,2,3undefinedundefined
```

`add(nums)` puts the array in `a` and leaves `b` and `c` undefined. `+` then concatenates strings, so the result is not even `NaN` — a silent wrong answer.

---

# 8. Copying vs Referencing

Destructuring and spread create **new containers**, never deep clones. Nested values stay shared.

### Shallow vs deep

```javascript
const kakade = { name: 'Kakade', address: { city: 'Pune' } };
const shallow = { ...kakade };

shallow.name = 'Asha';
shallow.address.city = 'Mumbai';

console.log(kakade.name);
console.log(kakade.address.city);
```

Output:

```text
Kakade
Mumbai
```

Changing a top-level property is isolated; changing a nested object leaks back.

### Deep copy with structuredClone

```javascript
const original = { name: 'Rohan', tags: ['js', 'node'] };
const deep = structuredClone(original);

deep.tags.push('react');
console.log(original.tags, deep.tags);
```

Output:

```text
[ 'js', 'node' ] [ 'js', 'node', 'react' ]
```

Use `structuredClone` for data. It cannot clone functions, DOM nodes, or class prototypes.

> **Trap:** `JSON.parse(JSON.stringify(obj))` drops `undefined` and functions, turns `Date` into strings, and loses `Map`/`Set` — it is not a general deep clone.

---

# 9. Swap Variables and Assignment Destructuring

### Swap without a temp

```javascript
let a = 'Kakade';
let b = 'Asha';

[a, b] = [b, a];

console.log(a, b);
```

Output:

```text
Asha Kakade
```

The right side builds a new array before assignment, so no temporary variable is needed.

### Destructuring assignment needs parentheses

Bad — the parser sees a block:

```javascript
let name;
try {
  eval('{ name } = { name: "Rohan" }');
} catch (error) {
  console.log(error.constructor.name);
}
```

Output:

```text
SyntaxError
```

Good — wrap the pattern in parentheses:

```javascript
let member;
({ name: member } = { name: 'Rohan' });
console.log(member);
```

Output:

```text
Rohan
```

### Defaults with assignment

```javascript
let role = 'guest';
({ role = 'member' } = {});
console.log(role);
```

Output:

```text
member
```

The default applies because the property is missing, not because `role` already had a value.

---

# Real-World Uses

- **API payloads**: `const { data, error } = await response.json();`
- **React props**: `function Card({ title, children }) {}`.
- **Config merging**: `{ ...defaults, ...overrides }`.
- **Stripping secrets**: `const { password, ...safeUser } = user;`.
- **Function arguments**: `fn(...args)` forwarding.
- **Immutable updates**: `{ ...state, count: state.count + 1 }`.

```javascript
const user = { name: 'Kakade', password: 'secret', role: 'admin' };
const { password, ...safeUser } = user;

console.log(safeUser);
```

Output:

```text
{ name: 'Kakade', role: 'admin' }
```

---

# Quick Revision

| Concept | Remember |
| --- | --- |
| Array destructuring | Binds by position; works on any iterable |
| Object destructuring | Binds by key name; order irrelevant |
| Default values | Apply only when the value is `undefined`, not `null` |
| Renaming | `{ name: userName }` reads `name`, declares `userName` |
| Nested destructuring | Each level needs a pattern; default intermediates with `= {}` |
| Rest in arrays | `[first, ...rest]` — last position, new array |
| Rest in objects | `{ a, ...rest }` — own enumerable leftovers |
| Spread in arrays | `[...a, ...b]` — shallow copy/combine, any iterable |
| Spread in objects | `{ ...a, ...b }` — own enumerable only, later wins |
| Function parameters | `fn({ a = 1 } = {})` — trailing `= {}` prevents a throw |
| Copying vs referencing | New container, nested values still shared |
| Swap variables | `[a, b] = [b, a]`; assignment patterns need `(...)` |

## The Mental Model

```text
                 ...
        ┌─────────┴─────────┐
        ↓                   ↓
   left side            right side
     (rest)              (spread)
        │                   │
   collect leftovers     expand values
        │                   │
   [a, ...rest]        [...arr], {...obj}
        │                   │
        └─────────┬─────────┘
                  ↓
        new array / new object
        (shallow — nested refs shared)
```

### The 3 things to remember

**1. Defaults are for `undefined`, not `null`:**

```javascript
const { age = 30 } = { age: null }; // age is null
```

**2. Rest collects, spread expands:**

```javascript
const [first, ...rest] = [1, 2, 3]; // rest = [2, 3]
const copy = [...rest];             // copy = [2, 3]
```

**3. Copies are shallow:**

```javascript
const clone = { ...original }; // nested objects are still shared
```

> **Destructuring unpacks by shape, rest collects the remainder, and spread expands into a new container — all shallow, with defaults applying only to `undefined`.**
