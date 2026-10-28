# Data Types and Coercion

Every value in JavaScript has a **type**, and every operation has to reconcile the types of its operands. The chain is: `a value has a type → the engine either copies it (primitive) or shares it (object) → mixed types get converted, explicitly or implicitly`. Most "JavaScript is weird" moments come from the third step, so this file separates what the types are from how they get converted.

The core idea:

```text
value
  │
  ├── primitive → copied by value
  │      string, number, boolean, null,
  │      undefined, symbol, bigint
  │
  └── object → copied by reference
         { }, [ ], functions

mixed operation
  ↓
explicit conversion  or  implicit coercion
  ↓
result (sometimes NaN)
```

---

# 1. Primitives vs Objects

### The two families

JavaScript has **seven primitive types** and one object type. Primitives are immutable and copied by value; objects are mutable and copied by reference.

```javascript
const values = ["Kakade", 42, true, null, undefined, Symbol("id"), 10n];
console.log(values.map((v) => typeof v).join(", "));
console.log(typeof {}, Array.isArray([]));
```

Output:

```text
string, number, boolean, object, undefined, symbol, bigint
object true
```

Arrays are objects; functions report `"function"`.

### Why the split matters

```javascript
const first = { name: "Kakade" };
const second = first;
second.name = "Asha";
console.log(first.name);
```

Output:

```text
Asha
```

Both names point to the **same object**.

---

# 2. string, number, boolean

### string

Text inside `"..."`, `'...'`, or backticks. Strings are immutable: methods return a new string.

```javascript
const name = "Kakade";
console.log(name.toUpperCase(), name);
console.log(name.length);
```

Output:

```text
KAKADE Kakade
6
```

### number

One numeric type for integers and decimals, stored as a 64-bit float. It also holds `Infinity`, `-Infinity`, and `NaN`.

```javascript
const age = 25;
const price = 99.5;
console.log(typeof age, typeof price);
console.log(10 / 0, 0 / 0);
```

Output:

```text
number number
Infinity NaN
```

### boolean

Only `true` and `false`. Comparisons produce booleans; the `if` statement accepts any value and coerces it.

```javascript
const active = true;
console.log(typeof active, 10 > 3, !active);
```

Output:

```text
boolean true false
```

---

# 3. null and undefined

### Two ways of saying "nothing"

`undefined` is the engine's default for a missing value; `null` is an intentional "no value" you assign yourself.

```javascript
let notAssigned;
const empty = null;

console.log(notAssigned, empty);
console.log(typeof notAssigned, typeof empty);
console.log(null == undefined, null === undefined);
```

Output:

```text
undefined null
undefined object
true false
```

### Where each shows up

| Situation | Value |
| --- | --- |
| `let x;` | `undefined` |
| Missing function argument | `undefined` |
| Missing object property | `undefined` |
| `JSON.stringify` drops it | `undefined` |
| Explicit "empty" from you or an API | `null` |
| `typeof null` | `"object"` (historic bug) |

**Notice:** `null == undefined` is `true`, but `null === undefined` is `false`. `null` only loosely equals `undefined` — not `0`, not `""`, not `false`.

---

# 4. symbol and bigint

### symbol

A `Symbol` is a guaranteed-unique primitive, often used as a hidden object key. Two symbols with the same description are still different values.

```javascript
const id1 = Symbol("id");
const id2 = Symbol("id");

console.log(id1 === id2);
console.log(typeof id1, id1.description);
```

Output:

```text
false
symbol id
```

### bigint

`BigInt` represents integers larger than `Number.MAX_SAFE_INTEGER`. Write it with an `n` suffix.

```javascript
const big = 9007199254740993n;
console.log(big + 1n);

try {
  console.log(big + 1);
} catch (error) {
  console.log(error.name);
}
```

Output:

```text
9007199254740994n
TypeError
```

**Notice:** BigInt and Number cannot be mixed in arithmetic; convert explicitly with `BigInt(10) + 5n` or `Number(10n)`.

---

# 5. typeof

### The type-check operator

`typeof` returns a lowercase string and is the quickest runtime check.

```javascript
console.log(typeof "Kakade");
console.log(typeof 42);
console.log(typeof true);
console.log(typeof undefined);
console.log(typeof null);
console.log(typeof []);
console.log(typeof function () {});
```

Output:

```text
string
number
boolean
undefined
object
object
function
```

### The full table

| Expression | `typeof` result |
| --- | --- |
| `"Kakade"` | `"string"` |
| `42`, `NaN`, `Infinity` | `"number"` |
| `10n` | `"bigint"` |
| `true` | `"boolean"` |
| `undefined` | `"undefined"` |
| `Symbol()` | `"symbol"` |
| `null` | `"object"` ← bug |
| `{}`, `[]`, `new Date()` | `"object"` |
| `function () {}` | `"function"` |

**Notice:** To detect an array use `Array.isArray(value)`. To detect `null` use `value === null`. Never rely on `typeof` for either.

---

# 6. Truthy and Falsy

### The eight falsy values

Everything is truthy except: `false`, `0`, `-0`, `0n`, `""`, `null`, `undefined`, and `NaN`.

```javascript
const falsy = [false, 0, -0, 0n, "", null, undefined, NaN];
console.log(falsy.map((v) => Boolean(v)).join(", "));
```

Output:

```text
false, false, false, false, false, false, false, false
```

### Truthy surprises

```javascript
console.log(Boolean("0"), Boolean(" "), Boolean([]), Boolean({}));
console.log(Boolean([]) === true, [] == false);
```

Output:

```text
true true true true
true true
```

**Notice:** `[]` is truthy, yet `[] == false` is also `true` — because `==` converts the array to `""`, then to `0`. Truthiness and loose equality use different rules.

---

# 7. Explicit Conversion

### Convert on purpose

Use `String()`, `Number()`, `Boolean()`, `parseInt()`, and `parseFloat()` when you need a conversion.

```javascript
console.log(Number("42"), Number(""), Number("12px"));
console.log(parseInt("42px", 10), parseFloat("3.14kg"));
console.log(String(42), String(null), Boolean("0"));
```

Output:

```text
42 0 NaN
42 3.14
42 null true
```

### `Number` vs `parseInt`

| Input | `Number(input)` | `parseInt(input, 10)` |
| --- | --- | --- |
| `"42"` | `42` | `42` |
| `"42px"` | `NaN` | `42` |
| `""` | `0` | `NaN` |
| `"3.9"` | `3.9` | `3` |
| `" 8 "` | `8` | `8` |

**Notice:** Always pass radix `10` to `parseInt`. `Number("")` being `0` is a common source of form bugs.

---

# 8. Implicit Coercion

### The `+` operator prefers strings

If either operand is a string, `+` concatenates. Every other arithmetic operator converts to number.

```javascript
console.log(1 + "2");
console.log("3" - 1);
console.log(true + 1);
console.log(null + 1);
console.log([1] + 1);
```

Output:

```text
12
2
2
1
11
```

### The conversion rules

| Operation | Rule | Example |
| --- | --- | --- |
| `+` with a string | concatenate | `1 + "2"` → `"12"` |
| `-`, `*`, `/`, `%` | convert both to number | `"6" / "2"` → `3` |
| `==` | complex coercion | `"0" == false` → `true` |
| `if (value)` | convert to boolean | `if ("")` → false |
| Template literal | convert to string | `` `x${1}` `` → `"x1"` |

**Think:**

```text
"5" + 1  → "51"   (string wins)
"5" - 1  → 4      (number wins)
```

**Notice:** Objects convert through `toString()` / `valueOf()`. That is why `[] + []` is `""` and `[] + {}` is `"[object Object]"`.

---

# 9. NaN

### "Not a Number" is still a number

`NaN` is the result of an invalid numeric operation. Its type is `"number"`, and it is the only value that is not equal to itself.

```javascript
const result = Number("abc");
console.log(result, typeof result);
console.log(result === result);
console.log(Number.isNaN(result), isNaN("abc"));
```

Output:

```text
NaN number
false
true true
```

### `Number.isNaN` vs `isNaN`

| Check | Coerces first? | `isNaN("abc")` | `isNaN(undefined)` |
| --- | --- | --- | --- |
| `isNaN(x)` | Yes | `true` | `true` |
| `Number.isNaN(x)` | No | `false` | `false` |

```javascript
console.log(isNaN("abc"), Number.isNaN("abc"));
console.log(Number.isNaN(NaN), Number.isNaN(undefined));
```

Output:

```text
true false
true false
```

**Notice:** Use `Number.isNaN(x)` to ask "is this value actually NaN". Use the global `isNaN` only when you deliberately want "would this convert to NaN".

---

# 10. Value vs Reference

### Primitives copy the value

```javascript
let score = 10;
let bonus = score;
bonus++;
console.log(score, bonus);
```

Output:

```text
10 11
```

### Objects copy the reference

```javascript
const a = { name: "Rohan" };
const b = a;
b.name = "Kakade";
console.log(a.name);

const c = { name: "Kakade" };
const d = { name: "Kakade" };
console.log(c === d);
```

Output:

```text
Kakade
false
```

Objects with identical contents have different references.

### Shallow copy

```javascript
const original = { name: "Kakade", address: { city: "Pune" } };
const copy = { ...original };
copy.name = "Asha";
copy.address.city = "Mumbai";
console.log(original.name, original.address.city);
```

Output:

```text
Kakade Mumbai
```

**Notice:** Spread copies the top level only. Nested objects are still shared — use `structuredClone()` for a deep copy.

---

# 11. Object Wrappers

### Boxing

Primitives have no methods, but JavaScript temporarily wraps them in an object so `"Kakade".toUpperCase()` works.

```javascript
const name = "Kakade";
console.log(name.length, name.toUpperCase());

const boxed = new String("Kakade");
console.log(typeof boxed, boxed === "Kakade", boxed == "Kakade");
```

Output:

```text
6 KAKADE
object false true
```

### Bad vs Good: object wrappers

```javascript
// Bad — an object wrapper is always truthy
const boxed = new String("");
console.log(Boolean(boxed));

// Good — convert to a primitive
const plain = String("");
console.log(Boolean(plain));
```

Output:

```text
true
false
```

**Notice:** An empty boxed string is an object, and every object is truthy. Never use `new String`, `new Number`, or `new Boolean`; call `String(value)` instead.

---

# Real-World Uses

### Form values arrive as strings

```javascript
const raw = "25";
const age = Number(raw);
console.log(age + 5);
console.log(raw + 5);
```

Output:

```text
30
255
```

### API payloads and `null`

```javascript
const payload = { name: "Kakade", middle: undefined, nickname: null };
console.log(JSON.stringify(payload));
```

Output:

```text
{"name":"Kakade","nickname":null}
```

`JSON.stringify` drops `undefined` properties but keeps `null` — a backend may treat them differently.

---

# Quick Revision

| Concept | Remember |
| --- | --- |
| **Primitives vs objects** | 7 primitives are copied by value; objects by reference |
| **string, number, boolean** | One `number` type for ints and floats; strings immutable |
| **null and undefined** | `undefined` = missing, `null` = intentional empty |
| **symbol and bigint** | Symbols are unique; BigInt uses `n` and cannot mix with Number |
| **typeof** | `typeof null === "object"`, arrays are `"object"` |
| **Truthy and falsy** | 8 falsy values; `[]` and `{}` are truthy |
| **Explicit conversion** | `Number("")` is `0`; `parseInt("12px")` is `12` |
| **Implicit coercion** | `+` prefers strings, other math prefers numbers |
| **NaN** | `NaN !== NaN`; use `Number.isNaN` |
| **Value vs reference** | Objects compare by reference; spread is shallow |
| **Object wrappers** | Boxing is temporary; never use `new String()` |

## The Mental Model

```text
                 VALUE
                   │
        ┌──────────┴──────────┐
        ↓                     ↓
   primitive               object
   (immutable)            (mutable)
        │                     │
   copy by value         copy by reference
        │                     │
        └──────────┬──────────┘
                   ↓
             mixed operation
                   ↓
        ┌──────────┴──────────┐
        ↓                     ↓
   explicit               implicit
   Number()               +, -, ==, if
        │                     │
        └──────────┬──────────┘
                   ↓
          number / string / boolean
                   ↓
             NaN if invalid
```

### The 3 things to remember

**1. `typeof null` lies:**

```javascript
console.log(typeof null);          // "object"
console.log(Array.isArray([]));    // true
```

**2. Convert explicitly at the boundary:**

```javascript
const age = Number(input.value); // not input.value + 1
```

**3. `NaN` needs a special check:**

```javascript
console.log(NaN === NaN);            // false
console.log(Number.isNaN(NaN));      // true
```

> **Types decide how a value is copied; coercion decides what happens when two different types meet — and `NaN` is the receipt for a conversion that failed.**
