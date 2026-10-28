# Operators and Equality

Operators are the verbs of JavaScript: they combine values into new values. The chain is: `an operator reads its operands → if the types differ it coerces them → it produces a result`, and equality is the place where that coercion causes the most bugs. This file covers arithmetic through optional chaining, then spends real time on `==` vs `===` and `Object.is()`.

The core idea:

```text
operand A   operator   operand B
    │           │          │
    └───────────┼──────────┘
                ↓
        same types? ── no ──→ coerce
                │               │
               yes              │
                └───────┬───────┘
                        ↓
                     result
                        │
          === stops coercion, == allows it
```

---

# 1. Arithmetic Operators

### The basics

`+` adds, `-` subtracts, `*` multiplies, `/` divides, `%` gives the remainder, and `**` raises to a power. `/` always produces a float when the result is not exact.

```javascript
console.log(7 + 3, 7 - 3, 7 * 3, 7 / 2, 7 % 2, 2 ** 3);
console.log(-5 % 2);
```

Output:

```text
10 4 21 3.5 1 8
-1
```

### Increment and decrement

`x++` returns the old value then increments; `++x` increments first. The same applies to `--`.

```javascript
let count = 5;
console.log(count++, count, ++count);
```

Output:

```text
5 6 7
```

**Notice:** `%` is a remainder, not a modulo. The sign follows the left operand, so `-5 % 2` is `-1`, not `1`.

### Unary plus and minus

Unary `+` converts to number; unary `-` converts and negates.

```javascript
console.log(+"42", +"", +"Kakade", -"5");
```

Output:

```text
42 0 NaN -5
```

---

# 2. Assignment Operators

### Shorthand assignment

`=` assigns. Compound operators apply an operation and assign the result back.

```javascript
let total = 100;
total += 20;
total *= 2;
total ??= 500;
console.log(total);
```

Output:

```text
240
```

| Operator | Meaning | Example |
| --- | --- | --- |
| `+=` | add and assign | `x += 2` |
| `-=` | subtract and assign | `x -= 2` |
| `*=` | multiply and assign | `x *= 2` |
| `/=` | divide and assign | `x /= 2` |
| `%=` | remainder and assign | `x %= 2` |
| `??=` | assign only if null/undefined | `x ??= 5` |
| `||=` | assign only if falsy | `x ||= 5` |
| `&&=` | assign only if truthy | `x &&= 5` |

### Chained assignment

```javascript
let a, b;
a = b = 5;
console.log(a, b);
```

Output:

```text
5 5
```

`b = 5` runs first, then its value is assigned to `a`. Avoid chains with `const`, which cannot be assigned after declaration.

---

# 3. Comparison Operators

### Greater and less than

`>`, `<`, `>=`, and `<=` return booleans. When both operands are strings, they compare **lexicographically** by UTF-16 code units, not by numeric value.

```javascript
console.log(10 > 3, 10 >= 10, "apple" < "banana");
console.log("10" < "9", "10" < 9);
```

Output:

```text
true true true
true false
```

`"10" < "9"` is `true` because `"1"` comes before `"9"` as text. When one side is a number, both become numbers, so `"10" < 9` is `false`.

### The chaining trap

```javascript
console.log(1 < 2 < 3);
console.log(3 > 2 > 1);
```

Output:

```text
true
false
```

`3 > 2` is `true`, then `true > 1` converts `true` to `1`, and `1 > 1` is `false`. Comparison operators are left-associative and cannot be chained like math notation.

---

# 4. Logical Operators

### `&&`, `||`, `!`

`&&` returns the first falsy operand or the last operand. `||` returns the first truthy operand or the last operand. `!` converts to boolean and negates.

```javascript
console.log(true && false, true || false, !true);
console.log("Kakade" && "Asha");
console.log(0 || "Rohan");
console.log(!"", !!"Kakade");
```

Output:

```text
false true false
Asha
Rohan
true true
```

### They return values, not booleans

This is the key difference from languages like Java. `&&` and `||` return one of their **operands**, which may be a string, number, or object — so `1 && 2` is `2` and `0 || "x"` is `"x"`.

---

# 5. Short-Circuiting

### Evaluation stops early

`&&` stops at the first falsy operand; `||` stops at the first truthy one. The skipped expression never runs — useful for guards and expensive calls.

```javascript
function expensive() {
  console.log("ran");
  return true;
}

console.log(false && expensive());
console.log(true || expensive());
console.log(true && expensive());
```

Output:

```text
false
true
ran
true
```

`expensive()` only ran for the last line.

### The default-value pattern

Before ES2020, `||` was the standard way to provide a fallback. It is wrong when `0`, `""`, or `false` are valid values.

```javascript
const config = { retries: 0 };

// Bad — 0 is valid, but treated as missing
console.log(config.retries || 3);

// Good — only null/undefined trigger the fallback
console.log(config.retries ?? 3);
```

Output:

```text
3
0
```

**Notice:** The modern guard pattern is `user?.name`.

---

# 6. Nullish Coalescing

### `??` only reacts to null and undefined

`??` returns the right side when the left is `null` or `undefined`. Every other falsy value (`0`, `""`, `false`, `NaN`) is kept.

```javascript
console.log(0 ?? 5);
console.log("" ?? "guest");
console.log(null ?? "guest");
console.log(undefined ?? "guest");
console.log(false ?? true);
```

Output:

```text
0

guest
guest
false
```

### `??` vs `||`

`??` differs from `||` only for `0`, `""`, `false`, and `NaN`: `||` replaces them, while `??` keeps them.

**Think:**

```text
a ?? b
   │
   ├── a is null / undefined → b
   └── anything else         → a
```

**Notice:** You cannot mix `??` with `||` or `&&` without parentheses — `a || b ?? c` is a `SyntaxError`. Write `(a || b) ?? c`.

---

# 7. Optional Chaining

### `?.` stops instead of throwing

`?.` reads a property only if the value is not `null`/`undefined`; otherwise the whole chain evaluates to `undefined`.

```javascript
const user = {
  name: "Kakade",
  greet() {
    return `Hi ${this.name}`;
  },
};

console.log(user.address?.city);
console.log(user.greet?.());
console.log(user.missing?.());
```

Output:

```text
undefined
Hi Kakade
undefined
```

### Three forms

| Form | Use |
| --- | --- |
| `obj?.prop` | optional property |
| `obj?.[key]` | optional computed access |
| `fn?.()` | optional call |

```javascript
const users = [{ name: "Asha" }];
console.log(users?.[0]?.name);
console.log(users?.[5]?.name);
```

Output:

```text
Asha
undefined
```

**Notice:** `?.` short-circuits the rest of the chain: in `user?.address.city`, if `user` is null the whole expression is `undefined` and `.city` is never evaluated.

---

# 8. Ternary Operator

### One expression, two outcomes

`condition ? valueIfTrue : valueIfFalse` is an **expression**, so it can be assigned, returned, or passed as an argument.

```javascript
const score = 72;
const grade = score >= 90 ? "A" : score >= 60 ? "B" : "C";
console.log(grade);
```

Output:

```text
B
```

### When not to use it

```javascript
const role = "admin";
const labels = { admin: "Administrator", user: "Customer" };
const label = labels[role] ?? "Guest";
console.log(label);
```

Output:

```text
Administrator
```

**Notice:** Use a ternary for a simple either/or. For more than two branches, use `if`/`else` or a lookup object.

---

# 9. Operator Precedence

### Multiplication before addition

Precedence decides which operator runs first. `*`, `/`, `%` bind tighter than `+` and `-`; `**` is right-associative; assignment is right-associative and very low.

```javascript
console.log(2 + 3 * 4);
console.log((2 + 3) * 4);
console.log(2 ** 3 ** 2);
console.log(typeof 1 + 2);
```

Output:

```text
14
20
512
number2
```

### The precedence trap

| Expression | Parsed as | Result |
| --- | --- | --- |
| `2 + 3 * 4` | `2 + (3 * 4)` | `14` |
| `2 ** 3 ** 2` | `2 ** (3 ** 2)` | `512` |
| `typeof 1 + 2` | `(typeof 1) + 2` | `"number2"` |
| `!true === false` | `(!true) === false` | `true` |

**Notice:** When in doubt, add parentheses. They remove a class of mistakes.

---

# 10. == vs ===

### Strict equality does not coerce

`===` returns `true` only when the types are the same and the values are equal. `==` converts the operands first.

```javascript
console.log(1 == "1", 1 === "1");
console.log(null == undefined, null === undefined);
console.log("" == 0, "0" == false);
console.log([] == false, null == 0);
```

Output:

```text
true false
true false
true true
true false
```

### How `==` decides

1. Same type → compare like `===`.
2. `null` and `undefined` → equal to each other, and nothing else.
3. Number vs string → convert the string to a number.
4. Boolean → convert to a number (`true` → `1`).
5. Object vs primitive → convert the object with `ToPrimitive`.

### The coercion table

| Expression | Result | Reason |
| --- | --- | --- |
| `"1" == 1` | `true` | string → number |
| `0 == ""` | `true` | `""` → `0` |
| `"0" == false` | `true` | both → `0` |
| `null == 0` | `false` | null only equals undefined |
| `null == undefined` | `true` | special rule |
| `NaN == NaN` | `false` | NaN is never equal |
| `[] == false` | `true` | `[]` → `""` → `0` |
| `[1] == 1` | `true` | `[1]` → `"1"` → `1` |

### Bad vs Good

```javascript
const count = "0";

// Bad — the string "0" passes, and so does ""
if (count == 0) {
  console.log("empty (loose)");
}

// Good — type must match
if (count === 0) {
  console.log("empty (strict)");
}
```

Output:

```text
empty (loose)
```

**Notice:** The only common exception is `value == null`, which conveniently matches both `null` and `undefined`. Prefer `value === null || value === undefined` or `value ?? fallback` for clarity.

---

# 11. Object.is()

### Almost `===`, with two fixes

`Object.is(a, b)` behaves like `===` except that `NaN` equals itself and `+0` / `-0` are different.

```javascript
console.log(Object.is(NaN, NaN), NaN === NaN);
console.log(Object.is(0, -0), 0 === -0);
console.log(Object.is("Kakade", "Kakade"));
```

Output:

```text
true false
false true
true
```

### When to use it

| Check | Use |
| --- | --- |
| Normal value equality | `===` |
| Detect `NaN` | `Number.isNaN(x)` or `Object.is(x, NaN)` |
| Distinguish `0` from `-0` | `Object.is(x, -0)` |
| Same-value semantics (e.g. React deps) | `Object.is` |

**Notice:** `Object.is` still compares objects by reference. It is not a deep-equality function.

---

# Real-World Uses

### Reading API data safely

```javascript
const response = { user: { name: "Kakade" } };
const city = response.user?.address?.city ?? "unknown";
console.log(city);
```

Output:

```text
unknown
```

### Form input comparison

```javascript
const input = "25";
console.log(input == 25);
console.log(Number(input) === 25);
```

Output:

```text
true
true
```

The second version makes the conversion explicit and keeps the comparison type-safe.

---

# Quick Revision

| Concept | Remember |
| --- | --- |
| **Arithmetic** | `%` keeps the dividend's sign; `2 ** 3 ** 2` is `512` |
| **Assignment** | `x += 1`, `x ??= 5`; chains assign right-to-left |
| **Comparison** | Two strings compare lexicographically; `"10" < "9"` is true |
| **Logical** | `&&`/`\|\|` return an operand, not a boolean |
| **Short-circuiting** | Skipped operands never evaluate |
| **Nullish coalescing** | `??` only falls back on `null`/`undefined` |
| **Optional chaining** | `?.` returns `undefined` instead of throwing |
| **Ternary** | `cond ? a : b` is an expression |
| **Precedence** | `*` before `+`; parenthesize when unsure |
| **== vs ===** | `==` coerces, `===` does not |
| **Object.is()** | Like `===`, but `NaN` equals itself and `0 !== -0` |

## The Mental Model

```text
                 OPERATOR
                    │
       ┌────────────┴────────────┐
       ↓                         ↓
   arithmetic               comparison
   + - * / % **             == vs ===
       │                         │
       ↓                    ┌────┴────┐
   numbers               coerces   strict
       │                    │         │
       └──────────┬─────────┘         │
                  ↓                   ↓
            logical / ternary     Object.is
             && || ?? ?:          (NaN, -0)
                  │
                  ↓
          short-circuit: skip work
```

### The 3 things to remember

**1. Default to `===`:**

```javascript
console.log(1 === "1"); // false, no surprise coercion
```

**2. Use `??` for defaults:**

```javascript
const retries = config.retries ?? 3; // keeps 0
```

**3. Parenthesize precedence:**

```javascript
console.log((2 + 3) * 4); // 20, not 14
```

> **Operators are simple until types differ — `===` and `??` remove most of the surprises, and parentheses remove the rest.**
