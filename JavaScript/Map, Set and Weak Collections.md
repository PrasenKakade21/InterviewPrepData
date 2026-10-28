# Map, Set and Weak Collections

A plain object stores values under **string** keys; a `Map` stores values under **any** key; a `Set` stores **unique values** with no key at all. The chain is **keyed lookup → unique membership → weak references that the garbage collector may clean up**.

```text
Keyed collections
      ↓
Map        key → value, any key type, insertion order
Set        value only, duplicates ignored
      ↓
WeakMap / WeakSet / WeakRef
      ↓
keys are objects, held weakly → GC can collect them
```

---

# 1. Map vs Object

### Map vs object

An object is a record whose keys are strings or symbols. A `Map` is a general key→value table: keys can be numbers, booleans, objects, even `NaN`. Both keep keys in a defined order, but `Map` is strictly insertion-ordered, has `size`, is directly iterable, and has no prototype keys.

```javascript
const scores = {};
const a = { id: 1 };
const b = { id: 2 };

scores[a] = 90;
scores[b] = 85;
console.log(Object.keys(scores));

const map = new Map();
map.set(a, 90).set(b, 85);
console.log(map.size);
```

Output:

```text
[ '[object Object]' ]
2
```

Notice: `scores[a]` and `scores[b]` both used the string `"[object Object]"`, so the second write overwrote the first. The `Map` kept two separate keys because it compares the object references themselves.

| Need | Object | Map |
| --- | --- | --- |
| Key types | string / symbol only | any value |
| Size | `Object.keys(o).length` | `map.size` |
| Iteration | `for...in`, `Object.entries` | `for...of`, built-in |
| Key order | integer-like keys first | strictly insertion order |
| Prototype keys | `in` / `for...in` can see them | none |
| Best for | fixed records, config, JSON | dynamic lookup, counters, non-string keys |

Use an object for a known shape (`user.name`). Use a `Map` when keys arrive at runtime, are objects, or entries are added and deleted often.

### Key equality

`Map` and `Set` compare keys with **SameValueZero**: like `===`, except `NaN` equals `NaN`. Objects are compared by **reference**, never by shape, so `{ id: 1 }` is a different key from another `{ id: 1 }`.

```javascript
const m = new Map();
m.set(NaN, "found");
console.log(m.get(NaN), m.get(Number("x")));

const kakade = { id: 1 };
m.set(kakade, "admin");
console.log(m.get(kakade), m.get({ id: 1 }));
```

Output:

```text
found found
admin undefined
```

`Number("x")` is `NaN`, and SameValueZero makes it match the stored `NaN`. A fresh `{ id: 1 }` is a different reference, so `get` is `undefined`.

Interview trap: `map.get({ id: 1 })` after `map.set({ id: 1 }, ...)` is always `undefined` — you must keep the original object reference.

---

# 2. Map Methods

### Map methods

The API is tiny: `set`, `get`, `has`, `delete`, `clear`, plus `size`. `set` returns the map (so it chains), `delete` returns a boolean, and a missing key gives `undefined` — not an error.

```javascript
const menu = new Map([
  ["idli", 40],
  ["dosa", 60],
]);

menu.set("tea", 15);
console.log(menu.size, menu.get("dosa"), menu.has("pizza"));
console.log(menu.delete("tea"), menu.size);
console.log(menu.get("pizza"));
```

Output:

```text
3 60 false
true 2
undefined
```

Bad: `menu["dosa"]`, `menu.length`, or `menu.dosa`. Good: `menu.get("dosa")`, `menu.size`. Bracket access sets a property on the Map instance, outside its entries, so `menu["idli"] = 40` leaves `size` at `0`. `new Map(Object.entries(obj))` converts a plain object, and `Object.fromEntries(map)` converts back.

### Iteration

`Map` is iterable in **insertion order**. `for...of` yields `[key, value]` pairs; `keys()`, `values()` and `entries()` return iterators, and spreading them gives arrays. `forEach` exists but its callback order is `(value, key, map)` — the opposite of what most people expect.

```javascript
const menu = new Map([
  ["idli", 40],
  ["dosa", 60],
]);

for (const [item, price] of menu) {
  console.log(`${item} = ${price}`);
}

console.log([...menu.keys()], [...menu.values()]);
menu.forEach((value, key) => console.log(key, value));
```

Output:

```text
idli = 40
dosa = 60
[ 'idli', 'dosa' ] [ 40, 60 ]
idli 40
dosa 60
```

Notice: `forEach` gives **value first**. Prefer `for...of` with destructuring — it is clearer and lets you `break`. `map.size` is a property, not `size()`. Updating an existing key keeps its original position; deleting and re-adding moves it to the end.

---

# 3. Set Methods

### Set methods

A `Set` is a collection of unique values. `add` returns the set (chainable), `has` answers membership, `delete` returns a boolean, and `size` counts the entries. Duplicate `add` calls are silently ignored.

```javascript
const tags = new Set(["js", "node", "js"]);

tags.add("css").add("js");
console.log(tags.size, tags.has("node"), tags.has("Node"));
console.log([...tags]);

tags.delete("node");
console.log(tags.size, [...tags]);
```

Output:

```text
3 true false
[ 'js', 'node', 'css' ]
2 [ 'js', 'css' ]
```

`has` is case-sensitive and uses SameValueZero, so `"Node"` and `"node"` are different. This makes `Set` the standard dedupe tool: `[...new Set(items)]` is O(n) and keeps first-seen order.

```javascript
const a = { id: 1 };
const b = { id: 1 };

console.log(new Set([a, b]).size);
console.log([...new Set([NaN, NaN, 1, "1"])]);
```

Output:

```text
2
[ NaN, 1, '1' ]
```

Two structurally identical objects are still two values because references differ. To dedupe objects by a field, use a `Map` keyed by that field: `new Map(users.map((u) => [u.id, u]))` keeps one entry per ID, with the last write winning.

Modern engines also provide set algebra — `union`, `intersection`, `difference`, `symmetricDifference`, `isSubsetOf`, `isSupersetOf`, `isDisjointFrom` (ES2025, Node 22+). For older runtimes the fallback is `[...a].filter((x) => b.has(x))` for intersection and `[...a].filter((x) => !b.has(x))` for difference.

---

# 4. WeakMap, WeakSet and WeakRef

### WeakMap

A `WeakMap` maps an **object key** to a value without preventing that object from being garbage-collected. It has only `get`, `set`, `has`, `delete` — no `size`, no `clear`, no iteration. That is deliberate: if you could list the keys, the engine could not free them safely.

```javascript
const visits = new WeakMap();
let asha = { name: "Asha" };

visits.set(asha, 3);
console.log(visits.get(asha), visits.has(asha));

asha = null;
console.log(visits.has(asha));
console.log(Object.keys(visits).length, visits.size, typeof visits[Symbol.iterator]);
```

Output:

```text
3 true
false
0 undefined undefined
```

After `asha = null`, nothing else references that object, so the entry becomes eligible for collection. `has(null)` is `false`; `WeakMap` rejects strings and numbers with `TypeError`.

The classic use is **per-object metadata** — data that belongs to an object but should not appear on it. Use the object itself as the key; when the object is dropped, the metadata goes with it.

```javascript
const state = new WeakMap();
const asha = { name: "Asha" };

state.set(asha, { visits: 1 });
console.log(state.get(asha), Object.keys(asha));
```

Output:

```text
{ visits: 1 } [ 'name' ]
```

Notice: `asha` keeps its own `name` key; the data lives in the WeakMap, invisible to `Object.keys` and `JSON.stringify`.

### WeakSet

A `WeakSet` is membership-only: object values, no duplicates, no size, no iteration. It is perfect for marking objects you have already processed without leaking them.

```javascript
const seen = new WeakSet();
const rohan = { id: 7 };

seen.add(rohan);
console.log(seen.has(rohan), seen.has({ id: 7 }));
console.log(typeof seen[Symbol.iterator], seen.size);
```

Output:

```text
true false
undefined undefined
```

Bad: `seen.add("rohan")` — a `WeakSet` accepts objects (and non-registered symbols) only. Good: wrap the value in an object, or use a regular `Set` when the values are primitives.

### WeakRef overview

`WeakRef` wraps a single object without keeping it alive. `ref.deref()` returns the object, or `undefined` **if it has been collected**. Timing is non-deterministic, so use it for caches and diagnostics, never correctness.

```javascript
const target = { id: 1, name: "Kakade" };
const cache = new WeakRef(target);
console.log(cache.deref()?.name ?? "collected");
```

Output:

```text
Kakade
```

`FinalizationRegistry` lets you register a callback after an object is collected, but again the callback may run late or never before the process exits. Think:

```text
Strong reference  → object stays alive
Weak reference    → object may be collected
                     ↓
             deref() → object | undefined
```

---

# 5. Use Cases and Memory

### Use cases

Reach for each collection for a different reason:

| Collection | Use it for |
| --- | --- |
| `Map` | lookup tables, counters, caches, grouping, non-string keys |
| `Set` | dedupe, membership tests, tags, visited tracking |
| `WeakMap` | metadata attached to objects, private state, memoisation keyed by object |
| `WeakSet` | "already handled" marks on objects, cycle detection |

A word-frequency counter is the canonical `Map` example — keys are added dynamically and order matters:

```javascript
const text = "js css js node css js";
const counts = new Map();

for (const word of text.split(" ")) {
  counts.set(word, (counts.get(word) ?? 0) + 1);
}

console.log([...counts]);
```

Output:

```text
[ [ 'js', 3 ], [ 'css', 2 ], [ 'node', 1 ] ]
```

`counts.get(word) ?? 0` handles the first occurrence. An object works here too, but a `Map` guarantees order and accepts non-string keys without coercion.

### Memory behavior

A regular `Map` or `Set` holds **strong** references: every key and value stays alive while the collection does. Weak collections hold **weak** references: an entry disappears once nothing else references its key. That difference is the whole point.

```javascript
const strong = new Map();
const weak = new WeakMap();
let user = { name: "Rohan" };

strong.set("user", user);
weak.set(user, "metadata");

user = null;
console.log(strong.size, weak.has({ name: "Rohan" }));
console.log(JSON.stringify(new Map([["a", 1]])));
console.log([...new Set([1, 2])]);
```

Output:

```text
1 false
{}
[ 1, 2 ]
```

The strong map still reports size `1` because it keeps the object alive. The weak map's entry is no longer reachable by identity, so `has` is `false`. Also notice `JSON.stringify` ignores `Map`/`Set` contents — convert with `Object.fromEntries` / `[...set]` first.

Think:

```text
Map/Set        → strong → object cannot be GC'd
WeakMap/WeakSet → weak   → object can be GC'd
```

Interview trap: "WeakMap prevents memory leaks" is imprecise. It prevents **the cache itself** from being the reason an object stays alive; if your app still holds the object elsewhere, nothing is collected.

---

# Real-World Uses

- **Dedupe** — `Set` for tag lists, selected IDs, and `[...new Set(arr)]` in UI code.
- **Grouping** — `Map` keyed by user ID or status, then `Object.fromEntries` for JSON.
- **Caches** — `WeakMap` for per-object metadata, so removed objects are collectable.

---

# Quick Revision

| Concept | Remember |
| --- | --- |
| **Map vs object** | Map keys can be any type, has `size`, is iterable, no prototype keys |
| **Map methods** | `set` chains, `get` → value/`undefined`, `delete` → boolean, `size` property |
| **Set methods** | `add` chains, `has`, `delete`, `size`; duplicates ignored |
| **WeakMap** | Object keys only; no size/iteration; entry freed with key |
| **WeakSet** | Object membership only; no size/iteration |
| **WeakRef overview** | `deref()` → object or `undefined`; timing is non-deterministic |
| **Iteration** | Insertion order; Map yields `[key, value]`; `forEach` is `(value, key)` |
| **Key equality** | SameValueZero: `NaN` matches `NaN`, objects by reference |
| **Use cases** | Map = lookup/count; Set = dedupe; Weak = metadata |
| **Memory behavior** | Map/Set strong; WeakMap/WeakSet weak; JSON gives `{}` |

## The Mental Model

```text
                  KEYED COLLECTIONS
                          │
        ┌─────────────────┼─────────────────┐
        ↓                 ↓                 ↓
       Map               Set           Weak versions
   key → value        value only       object keys only
   any key type       unique           GC may collect
        │                 │                 │
        ↓                 ↓                 ├── WeakMap  metadata
   get/set/has       add/has/delete         ├── WeakSet  seen marks
   size, iterate     size, iterate          └── WeakRef  optional cache
        └─────────────────┴─────────────────┘
                          ↓
                 insertion order kept
```

### The 3 things to remember

**1. Map keys are compared by identity, not shape.**

```javascript
const m = new Map();
m.set({ id: 1 }, "yes");
m.get({ id: 1 }); // undefined
```

**2. `Set` dedupes primitives and references in O(n).**

```javascript
[...new Set([1, 1, 2, NaN, NaN])]; // [ 1, 2, NaN ]
```

**3. Weak collections trade features for memory.**

```javascript
const wm = new WeakMap();
wm.set({}, 1);
wm.size; // undefined — no size, no iteration
```

> **Use `Map` for dynamic keys, `Set` for uniqueness, and weak collections when the cache must not keep objects alive.**
