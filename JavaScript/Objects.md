# Objects

An object is a **collection of key–value pairs** where keys are strings (or symbols) and values can be anything, including functions. You create it, read and write properties, then copy, loop over or lock it down. The chain is **literal → property access → inspect/copy → protect**, and the interview themes are reference vs value, shallow vs deep copy, and own vs inherited keys.

```text
Object
   ↓
create    { name: "Kakade" }
   ↓
read      obj.name / obj["name"] / obj?.name
   ↓
inspect   keys / values / entries
   ↓
copy      assign / spread / structuredClone
   ↓
protect   freeze / seal / descriptors
```

---

# 1. Creating Objects

### Object literals

The literal `{}` is the normal way to create an object. Keys can be identifiers or quoted strings, and values can be any expression.

```javascript
const user = {
  name: "Kakade",
  age: 30,
  "home city": "Pune",
};

console.log(user.name, user["home city"]);
console.log(typeof user, Array.isArray(user));
```

Output:

```text
Kakade Pune
object false
```

Keys are always **strings or symbols**, even when you write them without quotes. `Object.create(null)` makes an object with **no prototype** — useful as a dictionary, but it has no `hasOwnProperty`.

### Computed keys

Wrap an expression in `[]` inside the literal and its **value** becomes the key. This is how you build objects from variables.

```javascript
const field = "email";
const index = 2;

const form = {
  [field]: "kakade@example.com",
  [`field${index}`]: true,
};

console.log(form);
```

Output:

```text
{ email: 'kakade@example.com', field2: true }
```

Without the brackets, `{ field: ... }` creates a key literally named `"field"`. That one-character difference is a common bug when building dynamic payloads.

### Shorthand properties

If the variable name and the key are the same, write it once. This is used constantly in React props and API payloads.

```javascript
const name = "Kakade";
const role = "admin";

const user = { name, role };
console.log(user);
```

Output:

```text
{ name: 'Kakade', role: 'admin' }
```

Bad: `{ name: name, role: role }`. Good: `{ name, role }`. To rename, use `{ name: userName }`.

### Methods

A function stored on an object is a **method**. The shorthand `greet() {}` is equivalent to `greet: function () {}`, and `this` inside a method points at the object it was called on.

```javascript
const user = {
  name: "Asha",
  greet() {
    return `Hi ${this.name}`;
  },
};

console.log(user.greet());
const fn = user.greet;
console.log(fn === undefined);
```

Output:

```text
Hi Asha
false
```

Notice: `user.greet()` works because the call site is `user`. Extracting the function (`const fn = user.greet`) loses that binding — see `this Binding` for the full story. Arrow functions do **not** get their own `this`, so avoid them for methods that use `this`.

---

# 2. Reading and Changing

### Property access

Dot access needs a valid identifier; bracket access takes any expression. Missing properties return `undefined` — they do **not** throw, unless you then read through them.

```javascript
const user = { name: "Rohan", address: { city: "Mumbai" } };

console.log(user.name, user["name"]);
console.log(user.age);
console.log(user.address?.city, user.contact?.phone);
```

Output:

```text
Rohan Rohan
undefined
Mumbai undefined
```

Bad: `user.contact.phone` when `contact` is missing → `TypeError`. Good: `user.contact?.phone` → `undefined`. Use `in` to test a key: `"name" in user`.

### delete

`delete` removes an **own** property and returns `true` (even if the key did not exist). It cannot remove inherited properties or variables.

```javascript
const user = { name: "Kakade", temp: true };
const proto = { inherited: 1 };
const child = Object.create(proto);
child.own = 2;

console.log(delete user.temp, user);
console.log(delete child.inherited, child.inherited);
```

Output:

```text
true { name: 'Kakade' }
true 1
```

`delete child.inherited` returns `true` but the property is still readable — it only tried to remove it from `child`, where it never existed. On arrays, `delete arr[0]` leaves a **hole**; use `splice` instead.

---

# 3. Inspecting Objects

### Object.keys, values, entries

These three return arrays of the object's **own enumerable string keys**, their values, or `[key, value]` pairs. Symbols and non-enumerable properties are skipped.

```javascript
const scores = { b: 2, 1: "one", a: 1 };

console.log(Object.keys(scores));
console.log(Object.values(scores));
console.log(Object.entries(scores).length);
```

Output:

```text
[ '1', 'b', 'a' ]
[ 'one', 2, 1 ]
[ 3 ]
```

Notice the key order: **integer-like keys come first in ascending order**, then string keys in insertion order — a spec rule, not random. Sort explicitly if display order matters.

`Object.entries` + `Object.fromEntries` converts an object to a filtered/transformed object:

```javascript
const prices = { tea: 20, coffee: 40 };
const doubled = Object.fromEntries(
  Object.entries(prices).map(([k, v]) => [k, v * 2])
);
console.log(doubled);
```

Output:

```text
{ tea: 40, coffee: 80 }
```

### Own vs inherited properties

Objects inherit from a prototype. `in` and `for...in` see the **whole chain**; `Object.keys` and `Object.hasOwn` see only the object itself.

```javascript
const proto = { role: "admin" };
const user = Object.create(proto);
user.name = "Kakade";

console.log("role" in user, Object.hasOwn(user, "role"));
console.log(Object.keys(user));
for (const key in user) console.log(key);
```

Output:

```text
true false
[ 'name' ]
name
role
```

Bad: `for (const key in obj)` on data that may inherit enumerable properties — you iterate someone else's keys. Good: `Object.keys(obj)` or guard with `Object.hasOwn(obj, key)`. `obj.hasOwnProperty(key)` also works but breaks on `Object.create(null)` objects.

---

# 4. Copying Objects

### Object.assign

`Object.assign(target, ...sources)` copies **own enumerable** properties into `target`, left to right, and returns `target`. Later sources win.

```javascript
const defaults = { theme: "light", lang: "en" };
const userPrefs = { theme: "dark" };

const settings = Object.assign({}, defaults, userPrefs);
console.log(settings);
console.log(settings === defaults);
```

Output:

```text
{ theme: 'dark', lang: 'en' }
false
```

Bad: `Object.assign(defaults, userPrefs)` — you just mutated the defaults. Good: pass `{}` as the target. The copy is **shallow**.

### Spread

`{ ...source }` creates a new object with the same own enumerable properties. It is the modern, readable alternative to `Object.assign({}, src)`.

```javascript
const base = { name: "Kakade", role: "dev" };
const updated = { ...base, role: "lead" };

console.log(updated);
console.log(base.role);
```

Output:

```text
{ name: 'Kakade', role: 'lead' }
dev
```

Spread is the standard immutable-update pattern: copy, then override. It ignores inherited properties and non-enumerable ones.

### Shallow vs deep copy

Both `Object.assign` and spread copy only the **first level**. Nested objects are still shared by reference.

```javascript
const original = { name: "Kakade", address: { city: "Pune" } };

const shallow = { ...original };
shallow.address.city = "Mumbai";
console.log(original.address.city);

const deep = structuredClone(original);
deep.address.city = "Delhi";
console.log(original.address.city, deep.address.city);
```

Output:

```text
Mumbai
Mumbai Delhi
```

`structuredClone` handles nested objects, arrays, `Date`, `Map`, `Set` and circular references, but throws `DataCloneError` on functions and DOM nodes.

### JSON clone limits

The old trick `JSON.parse(JSON.stringify(obj))` is a **lossy** deep clone. It only understands what JSON understands.

```javascript
const original = {
  when: new Date("2026-01-15T00:00:00Z"),
  count: NaN,
  note: undefined,
  speak() {},
};

console.log(JSON.stringify(original));
console.log(JSON.parse(JSON.stringify(original)));
```

Output:

```text
{"when":"2026-01-15T00:00:00.000Z","count":null}
{ when: '2026-01-15T00:00:00.000Z', count: null }
```

What is lost:

| Value | After JSON clone |
| --- | --- |
| `Date` | ISO **string** |
| `undefined`, function, symbol | key **dropped** |
| `NaN`, `Infinity` | `null` |
| `Map`, `Set` | `{}` |
| `BigInt` | throws `TypeError` |
| Circular reference | throws `TypeError` |

Use `structuredClone` when available; JSON cloning is only safe for plain data.

---

# 5. Locking Objects Down

### Object.freeze and seal

`Object.freeze` makes an object **read-only** (no add, edit or delete); `Object.seal` allows editing existing values but blocks add and delete.

```javascript
const frozen = Object.freeze({ a: 1 });
frozen.a = 2;
frozen.b = 3;
console.log(frozen, Object.isFrozen(frozen));

const sealed = Object.seal({ a: 1 });
sealed.a = 5;
sealed.b = 2;
delete sealed.a;
console.log(sealed, Object.isSealed(sealed));
```

Output:

```text
{ a: 1 } true
{ a: 5 } true
```

In non-strict code the writes fail **silently**; in strict mode or a module they throw `TypeError`. Both are **shallow** — `Object.freeze(obj.nested)` is a separate call.

### Property descriptors

Every property has flags: `value`, `writable`, `enumerable`, `configurable`. `Object.defineProperty` lets you set them, and **omitted flags default to `false`** — unlike a literal, where all three are `true`.

```javascript
const account = {};
Object.defineProperty(account, "id", { value: 7 });

console.log(account.id);
console.log(Object.keys(account));
console.log(Object.getOwnPropertyDescriptor(account, "id"));
```

Output:

```text
7
[]
{ value: 7, writable: false, enumerable: false, configurable: false }
```

Because `enumerable` is `false`, the key disappears from `Object.keys`, spread and `JSON.stringify` — a hidden property. Descriptors also power getters:

```javascript
const temp = {
  _c: 25,
  get f() { return this._c * 9 / 5 + 32; },
  set f(value) { this._c = (value - 32) * 5 / 9; },
};

temp.f = 212;
console.log(temp._c.toFixed(0));
```

Output:

```text
100
```

---

# Real-World Uses

- **API payloads** — shorthand properties and spread to build request bodies.
- **Immutable state** — `{ ...state, user: nextUser }` instead of mutating.
- **Config merging** — `Object.assign({}, defaults, env, userPrefs)`.
- **Dictionaries** — `Object.create(null)` or `Map` when keys are untrusted.
- **Draft editing** — `structuredClone` before changing a nested object.

---

# Quick Revision

| Concept | Remember |
| --- | --- |
| **Object literals** | `{}`; keys are strings or symbols; `Object.create(null)` has no prototype |
| **Property access** | `obj.key` vs `obj["k"]`; missing → `undefined`; `?.` guards |
| **Computed keys** | `{ [expr]: value }` evaluates the key |
| **Shorthand properties** | `{ name }` = `{ name: name }` |
| **Methods** | `greet() {}` uses `this`; extracting the method loses `this` |
| **delete** | Removes own property, returns `true`; never touches inherited; arrays get holes |
| **keys/values/entries** | Own enumerable string keys; integers sort first |
| **Object.assign** | Mutates `target`, later sources win, shallow |
| **Spread** | New object, own enumerable only, shallow, later wins |
| **freeze and seal** | freeze = read-only, seal = edit-only; both shallow; silent fail vs strict throw |
| **Property descriptors** | `value/writable/enumerable/configurable`; `defineProperty` defaults them to `false` |
| **Own vs inherited** | `in` and `for...in` walk the chain; `Object.hasOwn` does not |
| **Shallow vs deep copy** | Spread/assign = one level; `structuredClone` = deep |
| **JSON clone limits** | Dates → strings, `undefined`/functions dropped, `NaN` → `null`, `Map` → `{}` |

## The Mental Model

```text
                    OBJECT
                       │
        ┌──────────────┼──────────────┐
        ↓              ↓              ↓
     create          read           copy
  {} / computed   dot / bracket   assign / spread
        │              │              │
        │              ↓              ├── shallow (one level)
        │           delete            └── deep  (structuredClone)
        │              │
        └──────────────┼──────────────┐
                       ↓              ↓
                    inspect        protect
              keys/values/entries  freeze / seal
                       │           descriptors
                       ↓
                own vs inherited
```

### The 3 things to remember

**1. `=` and spread are not the same.**

```javascript
const a = { n: 1 };
const b = a;        // same object
const c = { ...a }; // new object, shallow
```

**2. Frozen is shallow and silent (sloppy mode).**

```javascript
const cfg = Object.freeze({ a: 1, nested: { b: 1 } });
cfg.a = 2;        // ignored
cfg.nested.b = 2; // allowed — nested is not frozen
```

**3. JSON clone loses types.**

```javascript
JSON.parse(JSON.stringify({ d: new Date(), u: undefined }));
// { d: "2026-01-15T00:00:00.000Z" }  → u gone, d is a string
```

> **Objects are reference values: `=` aliases, spread shallow-copies, `structuredClone` deep-copies, and `freeze` only protects the top level.**
