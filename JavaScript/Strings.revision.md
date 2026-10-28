# Strings — Revision Notes (Beginner)
> One-line mental model: strings are immutable UTF-16 sequences — every method returns a new string, never changes the original.
## Core recall
- Create with `'...'`, `"..."`, or backticks. `new String()` makes an object wrapper — never use it.
- Template literals support `${expr}`, multiline text, and `String.raw` for literal backslashes.
- Indexing is zero-based: `str[0]`, `str.at(-1)`, `str.charAt(i)`. Out of range: `[]`/`at` → `undefined`, `charAt` → `""`.
- `length` counts **UTF-16 code units**, so most emoji count as 2.
- Strings are **immutable**: `str[0] = "X"` is ignored (throws in strict mode); assign method results (`s = s.trim()`).
- `slice(start, end)` is the default: exclusive end, negative indexes from the end, `""` if start > end.
- `substring` treats negatives as 0 and **swaps** if start > end; `substr(start, length)` is deprecated.
- `includes`, `startsWith`, `endsWith` return booleans, are case-sensitive, accept a start/end position.
- `indexOf`/`lastIndexOf` return an index or `-1`; `-1` is truthy, so compare explicitly.
- `split(sep)` → array; `join(sep)` → string. `split("")` breaks surrogate pairs; use `[...str]`.
- `replace` changes the first match; `replaceAll` every match. `replaceAll(/a/, ...)` requires the `g` flag.
- `trim`/`trimStart`/`trimEnd` remove whitespace; `padStart`/`padEnd` fill to a target length.
- `toUpperCase`/`toLowerCase` return new strings; `toLocaleUpperCase("tr-TR")` handles language rules.
- `repeat(n)` duplicates; `n < 0` throws `RangeError`.
- `localeCompare` returns negative/0/positive; `{ numeric: true }` compares numbers naturally.
- Unicode: `codePointAt`/`String.fromCodePoint` for full characters, `normalize()` before comparing accents.
## Must-remember code
```javascript
// slice vs substring
console.log("Kakade".slice(4, 1));      // ""
console.log("Kakade".substring(4, 1));  // "aka"
console.log("Kakade".slice(-3));        // "ade"
```
```javascript
// Immutability: capture the result
let name = "  Kakade  ";
name = name.trim().toUpperCase();
console.log(name); // "KAKADE"
```
```javascript
// Unicode-safe operations
const emoji = "😀";
console.log(emoji.length, [...emoji].length); // 2 1
console.log([..."Kakade"].reverse().join("")); // "edakaK"
```
## Interview traps
- `str[0] = "X"` does nothing — strings are immutable.
- `substring(4, 1)` is `"aka"`, `slice(4, 1)` is `""`.
- `substr`'s second argument is a **length**, not an end index.
- `indexOf` returns `-1` for missing; `if (str.indexOf(x))` is wrong.
- `replace("a", "b")` replaces only the first match; `replaceAll(/a/, "b")` needs `g`.
- `"😀".length === 2`; `split("")` corrupts emoji.
- `"10".localeCompare("9")` is `-1`; `{ numeric: true }` gives `1`.
## 3 things to remember
1. **Strings never mutate:** `const clean = raw.trim();`
2. **`slice` over `substring`:** `"Kakade".slice(-3)` → `"ade"`.
3. **Count code points, not units:** `[...str].length`.
