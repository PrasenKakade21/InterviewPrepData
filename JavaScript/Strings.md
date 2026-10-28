# Strings

A string is an **immutable sequence of UTF-16 code units** used for every piece of text — names, URLs, JSON, DOM content. The chain is: `you create a string → you read or search it → every "change" returns a new string`. Once you accept that strings never mutate, methods like `slice`, `replace`, and `trim` become easy to predict.

The core idea:

```text
"Kakade"
   │
   ├── index 0 1 2 3 4 5
   │        K a k a d e
   │
   ├── read     → str[0], slice(), includes()
   └── change   → returns a NEW string
                  (the original never changes)
```

---

# 1. String Creation

### Three literal styles

Single and double quotes are equivalent. Backticks create a **template literal**, which supports interpolation and multiline text.

```javascript
const single = 'Kakade';
const double = "Kakade";
const built = String(42);

console.log(single === double, built, typeof built);
console.log(new String("Kakade") === "Kakade");
```

Output:

```text
true 42 string
false
```

**Notice:** `String(42)` converts a value. `new String("Kakade")` creates an **object wrapper**, which is a different type and never `===` a primitive. Never use `new String`.

### Escaping

```javascript
console.log("He said \"hi\"");
console.log("C:\\Users\\Kakade");
```

Output:

```text
He said "hi"
C:\Users\Kakade
```

---

# 2. Template Literals

### Interpolation

`${...}` evaluates any expression and inserts its string form. It is the modern replacement for `+` concatenation.

```javascript
const name = "Kakade";
const role = "developer";

console.log(`Hi ${name}, you are a ${role}.`);
console.log(`2 + 2 = ${2 + 2}`);
```

Output:

```text
Hi Kakade, you are a developer.
2 + 2 = 4
```

### Multiline and raw strings

```javascript
console.log(`Line 1
Line 2`);

console.log(String.raw`C:\Users\Kakade`);
```

Output:

```text
Line 1
Line 2
C:\Users\Kakade
```

**Notice:** Backticks preserve newlines, so indentation becomes part of the string. `String.raw` leaves backslashes untouched.

---

# 3. Indexing and Length

### Zero-based access

`str[0]` is the first character. `length` counts UTF-16 code units, not user-perceived characters.

```javascript
const name = "Kakade";
console.log(name.length, name[0], name.at(-1));
console.log(name.charAt(99) === "", name[99]);
```

Output:

```text
6 K e
true undefined
```

### `[]` vs `charAt` vs `at`

| Access | Out of range | Negative index |
| --- | --- | --- |
| `str[99]` | `undefined` | `undefined` |
| `str.charAt(99)` | `""` | `""` |
| `str.at(-1)` | `undefined` | counts from end |

**Notice:** `.at(-1)` is the clean way to get the last character; `str[str.length - 1]` is the older equivalent.

---

# 4. Immutability

### Strings never change

You cannot assign to an index. In sloppy mode the write is silently ignored; in strict mode it throws `TypeError`. Methods always return a **new** string.

```javascript
let name = "Kakade";
name[0] = "X";
console.log(name);
console.log(name.toUpperCase());
console.log(name);
```

Output:

```text
Kakade
KAKADE
Kakade
```

### Bad vs Good

```javascript
// Bad — the result is thrown away
let label = "kakade";
label.toUpperCase();
console.log(label);

// Good — capture the returned string
label = label.toUpperCase();
console.log(label);
```

Output:

```text
kakade
KAKADE
```

**Notice:** This is the number one string bug: `trim()`, `replace()`, and `toUpperCase()` do not modify the receiver. Assign the result or chain it.

---

# 5. slice, substring and substr

### Three ways to extract

All three take a start index; they differ in how they treat the end and negative values.

```javascript
const name = "Kakade";
console.log(name.slice(1, 4), name.substring(1, 4), name.substr(1, 3));
console.log(name.slice(-3), name.substring(-3), name.substr(-3));
console.log(name.substring(4, 1), `[${name.slice(4, 1)}]`);
```

Output:

```text
aka aka aka
ade Kakade ade
aka []
```

### The comparison

| Method | End argument | Negative start | Swaps if start > end |
| --- | --- | --- | --- |
| `slice(start, end)` | exclusive index | counts from end | No → `""` |
| `substring(start, end)` | exclusive index | treated as `0` | Yes |
| `substr(start, length)` | **length** | counts from end | No |

**Notice:** `substr` is deprecated — use `slice`. `substring` swapping arguments is a classic interview trap: `substring(4, 1)` is `"aka"`.

**Think:**

```text
"Kakade".slice(1, 4)
         │     │
       start   end (exclusive)
         ↓
       "aka"
```

---

# 6. includes, startsWith, endsWith

### Boolean searches

These return `true`/`false` and are case-sensitive. `includes` and `startsWith` accept a start position; `endsWith` accepts an end position.

```javascript
console.log("Kakade".includes("kad"));
console.log("Kakade".startsWith("Kak"));
console.log("Kakade".endsWith("ade"));
console.log("Kakade".includes("Kad", 1));
```

Output:

```text
true
true
true
false
```

**Notice:** `includes("Kad", 1)` searches from index 1, where the text is `"akade"` — no match. For case-insensitive checks, normalize both sides: `str.toLowerCase().includes(term.toLowerCase())`.

---

# 7. indexOf and lastIndexOf

### Position searches

`indexOf` returns the first index of a substring, `lastIndexOf` the last, and both return `-1` when absent. They accept a start position.

```javascript
console.log("Kakade".indexOf("a"), "Kakade".lastIndexOf("a"));
console.log("Kakade".indexOf("z"));
console.log("Kakade".indexOf("a", 2));
```

Output:

```text
1 3
-1
3
```

### Choosing a search method

| Need | Method |
| --- | --- |
| Does it contain this? | `includes` |
| Where is it? | `indexOf` |
| Where is the last one? | `lastIndexOf` |
| Does it start/end with this? | `startsWith` / `endsWith` |

**Notice:** `indexOf` is `-1` for "not found", which is truthy. `if (str.indexOf(x))` is a bug — compare `!== -1`.

---

# 8. split and join

### String ↔ array

`split(separator)` breaks a string into an array. `join(separator)` glues an array back into a string.

```javascript
console.log("a,b,c".split(","));
console.log("a b  c".split(" "));
console.log(["Kakade", "Rohan"].join(" & "));
```

Output:

```text
[ 'a', 'b', 'c' ]
[ 'a', 'b', '', 'c' ]
Kakade & Rohan
```

### Useful forms

```javascript
console.log("Kakade".split(""));
console.log("a-b-c".split("-", 2));
console.log("Kakade".split("").reverse().join(""));
```

Output:

```text
[ 'K', 'a', 'k', 'a', 'd', 'e' ]
[ 'a', 'b' ]
edakaK
```

**Notice:** `split("")` splits code units, not characters.

---

# 9. replace and replaceAll

### Replace the first or every match

`replace` swaps only the first match when given a string. `replaceAll` swaps every match.

```javascript
console.log("hello world".replace("o", "0"));
console.log("hello world".replaceAll("o", "0"));
console.log("Kakade".replace("Kak", "[$&]"));
```

Output:

```text
hell0 world
hell0 w0rld
[Kak]ade
```

### Regex and functions

A global regex also replaces every match. With `replaceAll`, a regex **must** have the `g` flag.

```javascript
console.log("Kakade".replace(/a/g, "A"));
console.log("price: 100".replace(/(\d+)/, (match) => `$${match}`));
```

Output:

```text
KAkAde
price: $100
```

**Notice:** `$&` inserts the whole match, `$1` the first group. `replaceAll(/a/, "A")` throws `TypeError` because the regex lacks `g`.

---

# 10. trim and pad

### Remove surrounding whitespace

`trim()` removes both ends; `trimStart()` and `trimEnd()` remove one side. Whitespace includes spaces, tabs, and newlines.

```javascript
const raw = "  Kakade  ";
console.log(`[${raw.trim()}]`);
console.log(raw.trim().length);
```

Output:

```text
[Kakade]
6
```

### Pad to a width

`padStart(targetLength, fill)` and `padEnd` add characters until the string reaches the target length.

```javascript
console.log("5".padStart(3, "0"), "5".padEnd(3, "0"));
console.log("Kakade".padEnd(10, "."));
```

Output:

```text
005 500
Kakade....
```

**Notice:** Padding does nothing if the string is already long enough, and the fill string is truncated to fit.

---

# 11. Case Conversion

### Upper and lower case

`toUpperCase()` and `toLowerCase()` return new strings. `toLocaleUpperCase()` / `toLocaleLowerCase()` respect language rules.

```javascript
console.log("Kakade".toUpperCase());
console.log("KAKADE".toLowerCase());
console.log("i".toLocaleUpperCase("tr-TR"));
```

Output:

```text
KAKADE
kakade
İ
```

**Notice:** For comparisons, lowercase both sides explicitly instead of mutating the original.

---

# 12. repeat

### Duplicate a string

`repeat(count)` returns the string repeated `count` times. `repeat(0)` is `""`; a negative or infinite count throws `RangeError`.

```javascript
console.log("ab".repeat(3));
console.log("[".repeat(5));
```

Output:

```text
ababab
[[[[[
```

**Notice:** `repeat` is handy; `padStart` is clearer when the target width matters.

---

# 13. localeCompare

### Language-aware ordering

`localeCompare` returns a negative number, zero, or a positive number — the shape `sort` expects.

```javascript
console.log("a".localeCompare("b"));
console.log("b".localeCompare("a"));
console.log("a".localeCompare("a"));
```

Output:

```text
-1
1
0
```

### Numeric and case options

```javascript
console.log("10".localeCompare("9"));
console.log("10".localeCompare("9", undefined, { numeric: true }));
```

Output:

```text
-1
1
```

**Notice:** Plain `<` and `localeCompare` disagree. Lexicographically `"10"` comes before `"9"`; with `{ numeric: true }` the numbers are compared, so `10 > 9`. Use `localeCompare` for human-facing sorting.

---

# 14. Unicode Basics

### UTF-16 code units

JavaScript strings are sequences of 16-bit code units. Characters outside the Basic Multilingual Plane (most emoji) need **two** units, so `.length` can be misleading.

```javascript
const emoji = "😀";
console.log(emoji.length, [...emoji].length);
console.log(emoji.codePointAt(0).toString(16));
console.log("é".normalize("NFD").length);
```

Output:

```text
2 1
1f600
2
```

### Working with code points

```javascript
console.log(String.fromCodePoint(0x1f600));
console.log([..."Kakade"].reverse().join(""));
```

Output:

```text
😀
edakaK
```

**Notice:** Use `[...str]` or `Array.from(str)` to iterate by code point, `codePointAt` instead of `charCodeAt` for full values, and `normalize()` before comparing accented text. `"é"` can be one code point or `e` + a combining accent — visually identical, not `===`.

---

# Real-World Uses

### Cleaning form input

```javascript
const raw = "  Kakade@Example.com  ";
const email = raw.trim().toLowerCase();
console.log(email);
```

Output:

```text
kakade@example.com
```

### Building a slug

```javascript
const title = "JavaScript Strings Guide";
const slug = title.toLowerCase().replaceAll(" ", "-");
console.log(slug);
```

Output:

```text
javascript-strings-guide
```

---

# Quick Revision

| Concept | Remember |
| --- | --- |
| **String creation** | Quotes or backticks; avoid `new String` |
| **Template literals** | `${expr}`, multiline, `String.raw` |
| **Indexing and length** | Zero-based; `length` counts UTF-16 units |
| **Immutability** | Every change returns a new string |
| **slice/substring/substr** | `slice` is the default; `substring` swaps, `substr` is deprecated |
| **includes/startsWith/endsWith** | Boolean, case-sensitive searches |
| **indexOf/lastIndexOf** | Return an index or `-1` |
| **split and join** | String ↔ array conversion |
| **replace/replaceAll** | First vs every match; `$&`, `$1` |
| **trim and pad** | Remove whitespace; pad to a width |
| **Case conversion** | `toUpperCase` / `toLowerCase` return new strings |
| **repeat** | Repeat n times; negative throws |
| **localeCompare** | Language-aware sort; `{ numeric: true }` |
| **Unicode basics** | Emoji use 2 code units; use `[...str]` |

## The Mental Model

```text
                  STRING
                    │
        ┌───────────┴───────────┐
        ↓                       ↓
      read                    change
   [i], length            returns NEW string
   at, charAt                   │
        │                 slice / replace /
        ↓                 trim / pad / repeat
     search                     │
   includes, indexOf            ↓
        │                 assign or chain it
        └───────────┬───────────┘
                    ↓
            still immutable
```

### The 3 things to remember

**1. Capture the result:**

```javascript
const clean = "  Kakade  ".trim(); // "Kakade"
```

**2. `slice` beats `substring`:**

```javascript
console.log("Kakade".slice(-3)); // "ade"
```

**3. Emoji are two code units:**

```javascript
console.log("😀".length);   // 2
console.log([..."😀"].length); // 1
```

> **Strings never change in place — every method returns a new string, and every index is a UTF-16 code unit.**
