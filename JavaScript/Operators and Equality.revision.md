# Operators and Equality — Revision Notes (Intermediate)
> One-line mental model: operators combine values, coercion converts mismatched types, and `===`/`??` are the two habits that prevent most surprises.
## Core recall
- Arithmetic: `+ - * / % **`; `/` allows floats, `%` keeps the dividend's sign (`-5 % 2` → `-1`), `2 ** 3 ** 2` → `512`.
- `x++` returns the old value, `++x` returns the new one.
- Assignment: `+= -= *= /= %= **=`; logical assignment `??=`, `||=`, `&&=`. Chains run right-to-left (`a = b = 5`).
- Comparison: two strings compare lexicographically (`"10" < "9"` true); mixed types convert to numbers (`"10" < 9` false). Never chain (`3 > 2 > 1` false).
- `&&` returns the first falsy operand or the last; `||` returns the first truthy or the last. Both return **operands**, not booleans.
- Short-circuit: skipped operands never run — use it for guards and lazy work.
- `||` falls back on any falsy value (0, `""`, false, NaN); `??` only on `null`/`undefined`.
- `?.` short-circuits a property, index, or call to `undefined` instead of throwing.
- Ternary `cond ? a : b` is an expression; right-associative.
- Precedence: `**` > `* / %` > `+ -` > comparison > `&&` > `||` > `??` (not mixable) > ternary > assignment. `typeof 1 + 2` is `"number2"`.
- `==` coerces; `===` does not. `null == undefined` true, `null == 0` false, `NaN == NaN` false, `[] == false` true.
- `Object.is` = `===` except `NaN` equals itself and `0 !== -0`.
## Must-remember code
```javascript
console.log(1 == "1");    // true
console.log(1 === "1");   // false
console.log(null == 0);   // false
console.log([] == false); // true
```
```javascript
const config = { retries: 0 };
console.log(config.retries || 3); // 3 — wrong
console.log(config.retries ?? 3); // 0 — right
```
```javascript
const city = response?.user?.address?.city ?? "unknown";
console.log(city);
```
```javascript
console.log(Object.is(NaN, NaN)); // true
console.log(Object.is(0, -0));    // false
console.log(0 === -0);            // true
```
## Interview traps
- `"10" < "9"` is true (string comparison), but `"10" < 9` is false.
- `3 > 2 > 1` is false — left-associative, not a math chain.
- `null >= 0` is true while `null == 0` is false.
- `[] == false` and `[0] == false` are both true; `Boolean([])` is true.
- `a || b ?? c` is a SyntaxError; parenthesize.
- `??` is not "better ||" for booleans — `false ?? true` stays false.
- `Object.is` is not deep equality.
- `typeof 1 + 2` parses as `(typeof 1) + 2`.
- `?.` does not protect an undeclared variable — `undeclared?.x` still throws.
## 3 things to remember
1. **Use `===`:** `1 === "1"` is false; no hidden coercion.
2. **Use `??` for defaults:** `value ?? fallback` keeps `0` and `""`.
3. **Use `?.` for uncertain paths:** `user?.address?.city` never throws on null.
