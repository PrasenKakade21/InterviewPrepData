# Regular Expressions

A regular expression is a **pattern for matching text**, created with a literal or the `RegExp` constructor. The chain is **pattern → flags → match → capture → replace**, and the interview themes are `g` statefulness, greedy vs lazy, named groups, and patterns that freeze the browser.

```text
/pattern/flags
     ↓
  test()  → true / false
  exec()  → match + index + groups
     ↓
  match / matchAll → all matches
     ↓
  replace() → new string
     ↓
  anchored, bounded, safe
```

---

# 1. Creating Patterns

### Regex literals

A literal is written between slashes. Use it when the pattern is fixed in the source.

```javascript
const re = /hello/i;

console.log(re.test("Hello Kakade"));
console.log(re.source, re.flags);
```

Output:

```text
true
hello i
```

A literal creates a new `RegExp` per evaluation; a module-level `const` keeps its state.

### RegExp constructor

`new RegExp(pattern, flags)` builds a pattern from a string at runtime. This is for dynamic input such as a search box, not for static patterns.

```javascript
const word = "Kakade";
const re = new RegExp(`\\b${word}\\b`, "i");

console.log(re.test("Hi Kakade"), re.source);
```

Output:

```text
true Kakade
```

Notice the **double backslash**: `"\\b"` becomes the two characters `\b` in the pattern. Writing `"\b"` is a backspace character. If `word` comes from a user, escape it first:

```javascript
const escape = (s) => s.replace(/[.*+?^${}()|[\]\\]/g, "\\$&");
console.log(new RegExp(escape("a.b")).test("axb"));
```

Output:

```text
false
```

### Flags

Flags change how the engine reads the pattern.

| Flag | Meaning |
| --- | --- |
| `g` | Find all matches; `test`/`exec` move `lastIndex` |
| `i` | Case-insensitive |
| `m` | `^` and `$` match at line breaks |
| `s` | `.` also matches newline |
| `u` | Unicode mode; enables `\p{...}` |
| `y` | Sticky: match only at `lastIndex` |
| `d` | Include `.indices` in match results |

```javascript
console.log(/kakade/i.test("KAKADE"));
console.log(/^b$/m.test("a\nb"));
console.log(/./s.test("\n"));
console.log("a1b2".match(/\d/g));
```

Output:

```text
true
true
true
[ '1', '2' ]
```

### Escaping

These characters have special meaning and must be escaped to match literally: `. * + ? ^ $ { } [ ] ( ) | \ /`. The slash needs escaping only in a literal, because it ends the pattern.

```javascript
const price = "Total: $4.99";

console.log(/\$4\.99/.test(price));
console.log(new RegExp("\\$4\\.99").test(price));
```

Output:

```text
true
true
```

`new RegExp("$4.99")` fails because `$` means end-of-string and `.` means any character; escape both.

---

# 2. Matching

### test and exec

`test` returns a boolean. `exec` returns an array-like match with `index`, `input` and `groups`, or `null`. Without `g`, both always start at index 0.

```javascript
const re = /(\d{4})-(\d{2})/;
const result = re.exec("Due 2026-03");

console.log(result[0], result[1], result.index);
console.log(re.exec("none"));
```

Output:

```text
2026-03 2026 4
null
```

**The `/g` trap:** with `g` (or `y`), `test` and `exec` resume from `lastIndex`, so repeated calls on the same regex object change results.

```javascript
const re = /a/g;

console.log(re.test("banana"), re.lastIndex);
console.log(re.test("banana"), re.lastIndex);
console.log(re.test("banana"), re.lastIndex);
console.log(re.test("banana"), re.lastIndex);
```

Output:

```text
true 2
true 4
true 6
false 0
```

Bad: a module-level `/x/g` reused in a validation function — the second call can return `false` for the same input. Good: use a non-global regex for `test`, reset `re.lastIndex = 0`, or create it inside the function.

### match and matchAll

`String.prototype.match` behaves differently with `g`: without it you get an `exec`-style result with groups; with it you get only the full matched strings, with no groups or indices.

```javascript
const text = "Kakade: 10, Asha: 20, Rohan: 30";

console.log(text.match(/\d+/g));

const all = [...text.matchAll(/(?<name>\w+): (?<score>\d+)/g)];
console.log(all.map((m) => `${m.groups.name}=${m.groups.score}`).join(", "));
```

Output:

```text
[ '10', '20', '30' ]
Kakade=10, Asha=20, Rohan=30
```

`matchAll` **requires** the `g` flag (otherwise `TypeError`) and yields full match objects with `groups` and `index`. Spread it for an array; for a large input, loop with `for...of` to avoid building one.

---

# 3. Groups and Replacement

### Groups

Parentheses **capture** a sub-match. `m[0]` is the whole match, `m[1]` the first group, and an optional group that did not participate is `undefined`.

```javascript
const m = "asha@example.com".match(/(\w+)@(\w+)\.com/);
console.log(m[0], m[1], m[2]);

console.log(/(\w+)\s+\1/.test("Kakade Kakade"));
```

Output:

```text
asha@example.com asha example
true
```

`\1` is a **backreference** to group 1 — here it requires the same word twice. Use `(?:...)` for a non-capturing group when you only need grouping, so the numbering stays clean.

### Named groups

`(?<name>...)` labels a group. The result exposes `groups.name`, and replacement strings can use `$<name>`. This is far more readable than counting parentheses.

```javascript
const re = /(?<user>\w+)@(?<domain>[\w.]+)/;
const { groups } = re.exec("kakade@dev.io");

console.log(groups.user, groups.domain);
console.log("2026-03-01".replace(/(?<y>\d{4})-(?<m>\d{2})-(?<d>\d{2})/, "$<d>/$<m>/$<y>"));
```

Output:

```text
kakade dev.io
01/03/2026
```

Named groups still consume capture numbers, so `\1` and `\k<user>` can refer to the same group. An absent optional group gives `undefined`, not `""`.

### replace with functions

The replacement can be a function, called for each match with `(match, p1, p2, ..., offset, input, groups)`. Use it when the replacement depends on the match.

```javascript
const line = "Kakade scored 92, Asha scored 88";
const masked = line.replace(/\d+/g, (n) => "#".repeat(n.length));

console.log(masked);
console.log("2026-03-01".replace(/(\d{4})-(\d{2})-(\d{2})/, "$3/$2/$1"));
```

Output:

```text
Kakade scored ##, Asha scored ##
01/03/2026
```

With named groups, the `groups` object is the **last** argument, after `offset` and `input`. Return the original match to leave it unchanged. In string replacements, `$&` is the whole match, `$$` a literal `$`, and `$1` a numbered group.

---

# 4. Pattern Building Blocks

### Character classes

A class matches **one** character from a set. `\d` digit, `\w` word character (`A–Z`, `a–z`, `0–9`, `_`), `\s` whitespace, `.` any character except line terminators (unless `s`).

```javascript
console.log(/\d{3}/.test("PIN 412"));
console.log(/[^a-z]/i.test("Kakade"));
console.log("a-b_c".split(/[^a-z]+/i));
console.log(/\p{L}+/u.test("Ångström"));
```

Output:

```text
true
false
[ 'a', 'b', 'c' ]
true
```

`[^...]` negates, `[a-z]` is a range, and `-` at the start or end is literal. `\p{L}` needs the `u` flag and matches any Unicode letter, so `Å` works. `\w` stays ASCII-only even with `u`; use `\p{L}\p{N}_` for Unicode words.

### Quantifiers

A quantifier repeats the previous token: `?` zero or one, `*` zero or more, `+` one or more, `{3}` exactly three, `{2,}` at least two, `{2,5}` between two and five.

```javascript
console.log(/^\d{3}-\d{4}$/.test("555-1234"));
console.log(/^ab*c$/.test("abbbc"), /^ab+c$/.test("ac"));
```

Output:

```text
true
true false
```

Notice: `/^\d+$/` on `"123px"` is `false` because the anchors force the whole string; without anchors, `/\d+/` matches the `123` inside. Quantifiers apply to **one token**, so `ab+` means `a` then one-or-more `b` — use `(?:ab)+` for the pair.

### Anchors

Anchors match a **position**, not a character: `^` start, `$` end, `\b` word boundary, `\B` non-boundary. With `m`, `^`/`$` also match at line breaks.

```javascript
console.log(/^\d+$/.test("123"), /^\d+$/.test("123px"));
console.log(/\bcat\b/.test("concatenate"), /\bcat\b/.test("a cat."));
```

Output:

```text
true false
false true
```

`\b` is a boundary between a word character and a non-word character, so it finds `cat` in `"a cat."` but not inside `"concatenate"`. Always anchor validation patterns; without `^` and `$`, a match anywhere in the string counts.

### Greedy vs lazy

Quantifiers are **greedy** by default: they take as much as possible, then backtrack. Add `?` to make them **lazy**: take as little as possible.

```javascript
const html = "<b>Kakade</b>";

console.log(html.match(/<.+>/)[0]);
console.log(html.match(/<.+?>/)[0]);
```

Output:

```text
<b>Kakade</b>
<b>
```

Greedy is usually right for validation; lazy is useful for extracting one tag or quoted field. Neither parses nested HTML — use the DOM.

---

# 5. Lookahead and Lookbehind

### Lookahead and lookbehind

Lookaround checks what comes **next** or **before** without consuming it. `(?=...)` positive lookahead, `(?!...)` negative, `(?<=...)` positive lookbehind, `(?<!...)` negative.

```javascript
const prices = "tea 20, coffee 40, cake 60";

console.log(prices.match(/\d+(?=,)/g));
console.log(prices.match(/\d+(?!,)/g));
console.log(prices.match(/\d+\b(?!,)/g));

const log = "price: 20, tax: 5";
console.log(log.match(/(?<=price: )\d+/g));
console.log(log.match(/(?<!price: )\b\d+\b/g));
```

Output:

```text
[ '20', '40' ]
[ '2', '4', '60' ]
[ '60' ]
[ '20' ]
[ '5' ]
```

Notice line 2: `\d+(?!,)` looks correct, but the engine **backtracks** from `20` to `2` so the lookahead can succeed. Adding `\b` stops that. Lookbehind works in current Chrome, Firefox, Node and Safari 16.4+; older Safari throws at **parse time**. Lookaround is zero-width — it conditions the match without consuming characters.

---

# 6. Safety

### ReDoS

Regular-expression denial of service happens when nested or ambiguous quantifiers let the engine try exponentially many ways to match a failing input. `/^(a+)+$/` on `"aaaa…!"` is the classic example: the browser tab can freeze for seconds.

```javascript
const EMAIL = /^[^\s@]+@[^\s@]+\.[^\s@]{2,}$/;

console.log(EMAIL.test("kakade@dev.io"), EMAIL.test("kakade@dev"));
```

Output:

```text
true false
```

Rules that keep you safe:

- Avoid nested quantifiers like `(a+)+` and overlapping alternatives like `(a|aa)+`.
- Anchor the pattern and cap the input length before testing.
- Never run a user-supplied regex on untrusted input on the main thread; use a Worker or a linear-time engine.

---

# Real-World Uses

- **Form validation** — anchored email, phone and postal-code checks with a length cap.
- **Log parsing** — named groups turn a line into `{ level, time, message }`.

---

# Quick Revision

| Concept | Remember |
| --- | --- |
| **Regex literals** | `/pattern/flags`; new object per evaluation; no dynamic values |
| **RegExp constructor** | `new RegExp(str, flags)`; double-escape `\\d`; escape user input |
| **test and exec** | `test` → boolean; `exec` → match array or `null`; `/g` moves `lastIndex` |
| **match and matchAll** | `match` with `g` loses groups; `matchAll` needs `g` and yields match objects |
| **replace with functions** | `(match, p1…, offset, input, groups)`; return the replacement |
| **Groups** | `(...)` captures; `m[1]`; `(?:...)` does not capture; `\1` backreference |
| **Named groups** | `(?<name>...)`, `m.groups.name`, `$<name>` in replacements |
| **Flags** | `g` all, `i` case, `m` multiline, `s` dotAll, `u` Unicode, `y` sticky, `d` indices |
| **Character classes** | `\d \w \s .`, `[a-z]`, `[^...]`; `\p{L}` with `u` |
| **Quantifiers** | `? * + {n} {n,} {n,m}`; apply to one token |
| **Anchors** | `^ $ \b \B`; `m` changes `^`/`$` to line anchors |
| **Lookahead and lookbehind** | `(?=) (?!) (?<=) (?<!)`; zero-width; check backtracking |
| **Greedy vs lazy** | Greedy takes the most then backtracks; `?` makes it lazy |
| **Escaping** | Escape `. * + ? ^ $ { } [ ] ( ) \| \ /`; double in constructor |
| **ReDoS** | Nested quantifiers explode; anchor, bound length, avoid user regex |

## The Mental Model

Think:

```text
                 /pattern/flags
                        │
          ┌─────────────┼─────────────┐
          ↓             ↓             ↓
       test()        exec()       matchAll()
      true/false    index+groups   all matches
          │             │             │
          └─────────────┼─────────────┘
                        ↓
                    capture
              ( )  (?<name>)  \1
                        │
                        ↓
                    replace
              "$1"  "$<name>"  fn(match)
                        │
                        ↓
              anchored + bounded
                 (safe from ReDoS)
```

### The 3 things to remember

**1. `g` makes `test` stateful.**

```javascript
const re = /a/g;
re.test("banana"); // true, lastIndex 2
re.test("banana"); // true, lastIndex 4 — not a fresh check
```

**2. Greedy vs lazy is one character.**

```javascript
"<b>Kakade</b>".match(/<.+>/)[0];  // <b>Kakade</b>
"<b>Kakade</b>".match(/<.+?>/)[0]; // <b>
```

**3. Groups make replacement readable.**

```javascript
"2026-03-01".replace(/(?<y>\d{4})-(?<m>\d{2})-(?<d>\d{2})/, "$<d>/$<m>/$<y>");
// 01/03/2026
```

> **A regex is a tiny program: anchor it, keep quantifiers unambiguous, capture only what you need, and never let untrusted input run an exponential pattern.**
