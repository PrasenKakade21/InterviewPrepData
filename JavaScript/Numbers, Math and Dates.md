# Numbers, Math and Dates

JavaScript has only **one number type** — a 64-bit float — so `10` and `10.5` share a type, and `0.1 + 0.2` is not `0.3`. `Math` gives you rounding and randomness, while `Date` is just a millisecond count since 1970 that we format for humans. The chain is **Number → Math → Date → Intl**, and most "weird output" interview questions sit on it.

```text
Number (one float type)
   ↓
parse / round / format
   ↓
Math helpers + random
   ↓
Date = ms since 1970
   ↓
Intl.DateTimeFormat → text
```

---

# 1. Number Formats and Parsing

### Number formats

Literals can use other bases; `_` separators are ignored by the engine. All become `number`.

```javascript
const million = 1_000_000;
console.log(million, 0xff, 0b1010, 0o17, 1.5e3);
console.log((255).toString(16), (255).toString(2));
```

Output:

```text
1000000 255 10 15 1500
ff 11111111
```

`toString(radix)` converts **out** of decimal and always returns a **string**.

### parseInt and parseFloat

Both read a number from the **start** of a string and stop at the first invalid character. `Number()` is strict — the whole string must be numeric.

```javascript
console.log(parseInt("12px"));       // 12
console.log(parseFloat("3.14abc"));  // 3.14
console.log(Number("12px"));         // NaN
console.log(parseInt("0x10"), parseInt("10", 2));
```

Output:

```text
12
3.14
NaN
16 2
```

Think:

```text
parseInt("12px") → reads "12", hits "p", stops → 12   (junk ignored)
Number("12px")   → whole string is not numeric → NaN
```

Bad:

```javascript
const price = parseInt("19.99"); // 19 — decimals dropped
```

Good:

```javascript
const price = parseFloat("19.99");     // 19.99
const count = parseInt("19.99", 10);   // 19 — deliberate
```

Always pass radix `10`: without it `"0x10"` is hex, and legacy leading-zero strings were octal. Also note `parseInt` stringifies first, so `parseInt(0.0000005)` is `5` (`"5e-7"` → `5`).

### toFixed and toPrecision

`toFixed(n)` rounds to `n` decimal places; `toPrecision(n)` keeps `n` significant digits. Both return **strings**.

```javascript
const total = 1234.5678;
console.log(total.toFixed(2), typeof total.toFixed(2));
console.log(total.toPrecision(6), (0.000123456).toPrecision(2));
```

Output:

```text
1234.57 string
1234.57 0.00012
```

Trap — `toFixed` rounds the **binary** value, not the decimal you typed:

```javascript
console.log((1.005).toFixed(2)); // "1.00"
console.log((1.45).toFixed(1));  // "1.4"
```

Output:

```text
1.00
1.4
```

For money, store integer paise and `Math.round(amount * 100)`, then format once at display time.

---

# 2. Math and Randomness

### Math methods

`Math` is a plain namespace object — never `new Math()`.

| Method | Does | Example |
| --- | --- | --- |
| `Math.round(x)` | nearest; `.5` goes to `+∞` | `round(-2.5)` → `-2` |
| `Math.floor(x)` / `Math.ceil(x)` | down / up | `floor(-2.1)` → `-3` |
| `Math.trunc(x)` | drops decimals | `trunc(-2.7)` → `-2` |
| `Math.abs(x)`, `Math.sign(x)` | distance, sign | `abs(-5)` → `5` |
| `Math.max(...)` / `Math.min(...)` | extremes | `max(3, 9, 4)` → `9` |
| `Math.sqrt`, `**` | root, power | `sqrt(81)` → `9` |

```javascript
console.log(Math.round(2.5), Math.round(-2.5));
console.log(Math.floor(-2.1), Math.trunc(-2.7));
console.log(Math.max(3, 9, 4), Math.sqrt(81), Math.sign(-5));
console.log(Math.max(), Math.min());
```

Output:

```text
3 -2
-3 -2
9 9 -1
-Infinity Infinity
```

`Math.max()` with no arguments is `-Infinity` — hence `Math.max(...[])` breaks empty-array logic.

### Random numbers

`Math.random()` returns `0 ≤ x < 1`. It never returns `1`, so forget the `+ 1` and your range is short.

```javascript
const randomInt = (min, max) =>
  Math.floor(Math.random() * (max - min + 1)) + min;

console.log(randomInt(1, 6)); // dice: 1..6
console.log(Math.random() >= 0 && Math.random() < 1);
```

Output:

```text
4
true
```

Think:

```text
random()          → 0 .. 0.999999
× (max-min+1)     → 0 .. 5.999999
floor(...)        → 0 .. 5
+ min             → 1 .. 6
```

Bad: `Math.floor(Math.random() * 6)` → `0..5`. Good: `Math.floor(Math.random() * 6) + 1` → `1..6`.

It is **not** cryptographically secure. For tokens use `crypto.randomUUID()` or `crypto.getRandomValues()`.

---

# 3. Precision: Floating Point, Safe Integers, BigInt

### Floating point issues

Most decimals have no exact binary form.

```javascript
console.log(0.1 + 0.2);
console.log(0.1 + 0.2 === 0.3);
console.log((0.1 + 0.2).toFixed(2));
```

Output:

```text
0.30000000000000004
false
0.30
```

Bad:

```javascript
if (0.1 + 0.2 === 0.3) approve(); // never runs
```

Good:

```javascript
const close = Math.abs(0.1 + 0.2 - 0.3) < Number.EPSILON; // true
const shown = (0.1 + 0.2).toFixed(2);                      // "0.30"
```

`Number.EPSILON` (≈ `2.22e-16`) is the gap between `1` and the next representable double — the standard tolerance.

### Safe integers

Integers are exact only up to **2⁵³ − 1**.

```javascript
console.log(Number.MAX_SAFE_INTEGER);
console.log(2 ** 53, 9007199254740993);
console.log(Number.isSafeInteger(9007199254740993));
```

Output:

```text
9007199254740991
9007199254740992 9007199254740992
false
```

`9007199254740993` prints as `...992`: the **literal** was rounded at parse time. That is why snowflake-style IDs travel as strings.

### BigInt

`BigInt` holds arbitrarily large integers. Use the `n` suffix or `BigInt()`.

```javascript
const big = 9007199254740993n;
console.log(big, typeof big, 2n ** 64n);
console.log(10n / 3n, Number(10n));
```

Output:

```text
9007199254740993n bigint 18446744073709551616n
3n 10
```

- Division **truncates**: `10n / 3n` → `3n`.
- No mixing: `1n + 1` throws `TypeError`. Convert first: `1n + BigInt(1)`.
- No `Math` methods; `typeof` is `"bigint"`.
- `JSON.stringify({ n: 1n })` throws — send a string.
- `10n > 5` is fine; comparisons allow mixed types.

---

# 4. Creating and Reading Dates

### Date creation

A `Date` is one instant, stored as milliseconds since **1 Jan 1970 UTC**.

```javascript
console.log(new Date(0).toISOString());
console.log(new Date(2026, 0, 15).toDateString());
console.log(new Date("2026-01-15").toISOString());
console.log(new Date(2026, 11, 0).toDateString());
```

Output:

```text
1970-01-01T00:00:00.000Z
Thu Jan 15 2026
2026-01-15T00:00:00.000Z
Mon Nov 30 2026
```

**Month is 0-indexed** (`0` = January). Day `0` is the last day of the previous month. `new Date(y, m, d)` is **local** and silently rolls overflow over:

```javascript
console.log(new Date(2026, 12, 1).toDateString()); // month 12 → next year
console.log(new Date(2026, 1, 29).toDateString()); // 2026 is not a leap year
```

Output:

```text
Fri Jan 01 2027
Sun Mar 01 2026
```

### Date methods

| Getter | Returns | Range |
| --- | --- | --- |
| `getFullYear()` | year | `2026` |
| `getMonth()` | month | `0`–`11` |
| `getDate()` | day of month | `1`–`31` |
| `getDay()` | weekday | `0` Sun – `6` Sat |
| `getTime()` | epoch ms | number |

```javascript
const d = new Date(2026, 0, 15);
console.log(d.getFullYear(), d.getMonth(), d.getDate(), d.getDay());
d.setDate(d.getDate() + 20);
console.log(d.toDateString());
```

Output:

```text
2026 0 15 4
Wed Feb 04 2026
```

`setDate` handles rollover — use it to add days. Never do `setMonth(getMonth() + 1)` on the 31st: you get 3 March, not the end of February.

### Timestamps

`Date.now()` and `getTime()` both give **milliseconds**.

```javascript
const then = Date.UTC(2026, 0, 15);
const now = Date.now();
console.log(then);
console.log(typeof now, now > then);
```

Output:

```text
1768435200000
number true
```

Think:

```text
Date  ──getTime()──→  1768435200000 (ms)
                          ├── / 1000      → seconds (APIs)
                          └── new Date(ms) → Date again
```

`Date.UTC(y, m, d)` reads parts as **UTC**; `new Date(y, m, d)` reads them as **local**. Mixing them shifts results by the offset.

---

# 5. Formatting and Time Zones

### Intl.DateTimeFormat

Never concatenate `getDate() + "/" + getMonth()`. `Intl` knows month names, ordering and 12/24-hour conventions.

```javascript
const t = new Date("2026-01-15T10:30:00Z");
console.log(new Intl.DateTimeFormat("en-IN", { dateStyle: "medium", timeZone: "UTC" }).format(t));
console.log(new Intl.DateTimeFormat("en-GB", {
  weekday: "long", day: "2-digit", month: "long", year: "numeric", timeZone: "UTC",
}).format(t));
```

Output:

```text
15 Jan 2026
Thursday, 15 January 2026
```

`dateStyle` / `timeStyle` (`"short" | "medium" | "long" | "full"`) are shortcuts; you can also pass `day`, `month`, `year`, `hour`, `minute`, `weekday`. Build the formatter **once** and reuse it in loops.

### Time zones

A `Date` stores an instant, not a zone. The same instant prints differently depending on where you read it.

```javascript
const t = new Date("2026-01-15T10:30:00Z");
console.log(t.getUTCHours());          // 10
console.log(t.getTimezoneOffset());    // -330 on IST
console.log(t.toISOString().slice(0, 10));
```

Output:

```text
10
-330
2026-01-15
```

- `getHours()` = local, `getUTCHours()` = UTC.
- `toISOString()` is always UTC and ends with `Z`.
- `getTimezoneOffset()` is minutes to add to local time to reach UTC (IST = `-330`).
- `Intl`'s `timeZone` option only changes **display**, not the stored instant.

Trap — **date-only strings parse as UTC**, date-time strings without `Z` parse as **local**:

```javascript
console.log(new Date("2026-01-15").toISOString());
console.log(new Date("2026-01-15T00:00:00").toISOString());
```

Output:

```text
2026-01-15T00:00:00.000Z
2026-01-14T18:30:00.000Z
```

The second line is the previous day because this machine is IST (UTC+5:30). Always send ISO strings **with** `Z`; store UTC, display local.

---

# Real-World Uses

- **Money** — integer paise internally, `toFixed(2)` only for display.
- **IDs** — string or BigInt above `Number.MAX_SAFE_INTEGER`.
- **Pagination** — `Math.ceil(total / pageSize)`.
- **Charts** — `Math.max(...values)`, guarded when the array may be empty.
- **Bookings** — `setDate(getDate() + n)` for days, `Intl` with explicit `timeZone` for confirmations.
- **Logs** — `new Date().toISOString()`, never a hand-built string.

---

# Quick Revision

| Concept | Remember |
| --- | --- |
| **Number formats** | One `number` type; `0x`, `0b`, `0o`, `1_000`; `toString(radix)` → string |
| **parseInt / parseFloat** | Read from the start, stop at junk; pass radix `10`; `Number()` is strict |
| **toFixed / toPrecision** | Both return strings; `toFixed` rounds the binary value (`1.005` → `"1.00"`) |
| **Math methods** | `round(-2.5)` = `-2`; `trunc` ≠ `floor` for negatives; `Math.max()` = `-Infinity` |
| **Random numbers** | `0 ≤ r < 1`; `floor(r * (max - min + 1)) + min`; not secure |
| **BigInt** | `10n`, `typeof "bigint"`, `/` truncates, no mixing, `JSON.stringify` throws |
| **Safe integers** | Exact to `2^53 - 1`; beyond that the literal is already rounded |
| **Floating point** | `0.1 + 0.2 !== 0.3`; compare with `Number.EPSILON` |
| **Date creation** | Month `0`-indexed; `(y, m, d)` is local; overflow rolls over |
| **Date methods** | `getMonth` 0-based, `getDate` 1-based, `getDay` 0 = Sunday |
| **Timestamps** | Milliseconds since epoch; `Date.now()`; `/ 1000` for seconds |
| **Intl.DateTimeFormat** | `dateStyle`, `timeStyle`, `timeZone`; reuse the formatter |
| **Time zones** | `Date` is an instant; `getX` local vs `getUTCX`; date-only strings are UTC |

## The Mental Model

```text
                  JavaScript numbers
                          │
            ┌─────────────┴─────────────┐
            ↓                           ↓
      one float type                 BigInt
            │                    (huge integers)
   ┌────────┼────────┐                  │
   ↓        ↓        ↓                  ↓
parse    round   compare         no mixing,
parseInt toFixed EPSILON         JSON throws
   │        │
   └───┬────┘
       ↓
     Math ──→ round / floor / ceil / random
       │
       ↓
     Date = ms since 1970
       │
  ┌────┴─────┐
  ↓          ↓
local      UTC (toISOString, getUTC*)
  │          │
  └────┬─────┘
       ↓
 Intl.DateTimeFormat → human text
```

### The 3 things to remember

**1. Floats are not exact.**

```javascript
0.1 + 0.2;                                  // 0.30000000000000004
Math.abs(0.1 + 0.2 - 0.3) < Number.EPSILON; // true
```

**2. Months start at zero.**

```javascript
new Date(2026, 0, 15).getMonth(); // 0 → January
new Date(2026, 0, 15).getDate();  // 15 → the 15th
```

**3. Dates are instants; format with `Intl`.**

```javascript
new Intl.DateTimeFormat("en-IN", { dateStyle: "medium", timeZone: "UTC" })
  .format(new Date("2026-01-15T10:30:00Z")); // "15 Jan 2026"
```

> **One imprecise float type, `Math` for rounding and randomness, and `Date` as an epoch millisecond count that only `Intl` and the getters make readable.**
