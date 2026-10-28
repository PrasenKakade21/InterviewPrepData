# Regular Expressions — Revision Notes (Intermediate)

> One-line mental model: a regex is a tiny program — anchor it, capture only what you need, and keep every quantifier unambiguous.

## Core recall

- **Literals**: `/pattern/flags`, new object per evaluation, cannot use variables. **Constructor**: `new RegExp(str, flags)`, double-escape (`"\\d"`), escape user input.
- **test/exec**: `test` → boolean; `exec` → `[full, ...groups]` with `index`, `input`, `groups`, or `null`.
- **`g` statefulness**: `test`/`exec` resume from `lastIndex`; after a failed match it resets to 0.
- **match/matchAll**: `match` with `g` returns only full strings; `matchAll` needs `g` and yields full match objects with groups.
- **Groups**: `(...)` captures, `(?:...)` groups only, `\1` backreference. **Named**: `(?<name>...)`, `m.groups.name`, `\k<name>`.
- **replace function**: `(match, p1…, offset, input, groups)`; return the replacement. String tokens: `$1`, `$<name>`, `$&`, `$$`.
- **Flags**: `g` all, `i` case, `m` multiline anchors, `s` dotAll, `u` Unicode, `y` sticky, `d` indices.
- **Classes**: `\d \w \s .`, `[a-z]`, `[^...]`, `\p{L}` with `u`; `\w` stays ASCII.
- **Quantifiers**: `? * + {n} {n,} {n,m}`, applied to one token; `(?:ab)+` for a pair.
- **Anchors**: `^ $ \b \B`; with `m`, `^`/`$` match line boundaries.
- **Lookaround**: `(?=) (?!) (?<=) (?<!)`, zero-width; lookbehind needs Safari 16.4+.
- **Greedy/lazy**: greedy takes the most then backtracks; `?` makes it lazy.
- **Escaping**: `. * + ? ^ $ { } [ ] ( ) | \ /`; slash only in literals.
- **ReDoS**: nested/ambiguous quantifiers explode on failing input; anchor, cap length, avoid user regex.

## Must-remember code

```javascript
// all matches with named groups
const all = [...text.matchAll(/(?<name>\w+): (?<score>\d+)/g)];
all.map((m) => `${m.groups.name}=${m.groups.score}`);
```

```javascript
// replace with a callback
text.replace(/\d+/g, (n) => "#".repeat(n.length));
```

```javascript
// named replacement tokens
"2026-03-01".replace(
  /(?<y>\d{4})-(?<m>\d{2})-(?<d>\d{2})/,
  "$<d>/$<m>/$<y>"
); // 01/03/2026
```

```javascript
// safe validation shape
const EMAIL = /^[^\s@]+@[^\s@]+\.[^\s@]{2,}$/;
```

## Interview traps

- Reusing a `/g` regex for `test` gives state-dependent results; reset `lastIndex` or drop `g`.
- `match` with `g` silently loses capture groups — use `matchAll`.
- `matchAll` without `g` throws `TypeError`.
- `\d+(?!,)` can match `"2"` inside `"20,"` because `\d+` backtracks; add `\b`.
- `new RegExp("\\d")` needs two backslashes; `new RegExp("\\b")`, not `"\b"`.
- Greedy `<.+>` spans the whole string; lazy `<.+?>` stops at the first `>`.
- Lookbehind is a parse-time syntax error in old Safari, even if the branch is never run.
- `^`/`$` without `m` anchor the whole string; validation needs both.
- `\w` is ASCII-only; use `\p{L}` with `u` for Unicode letters.
- A catastrophic pattern cannot be interrupted with `setTimeout` on the same thread.

## 3 things to remember

1. **`g` changes state.**
   ```javascript
   /a/g.test("banana"); // true; lastIndex 2
   ```

2. **Greedy vs lazy.**
   ```javascript
   "<b>Kakade</b>".match(/<.+?>/)[0]; // <b>
   ```

3. **Anchor and bound.**
   ```javascript
   const re = /^[^\s@]{1,64}@[^\s@]{1,190}$/;
   ```
