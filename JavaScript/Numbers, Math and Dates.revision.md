# Numbers, Math and Dates — Revision Notes (Intermediate)

> One-line mental model: One imprecise float type, `Math` for rounding, `Date` as epoch milliseconds, `Intl` for humans.

## Core recall

- **One type**: every number is a 64-bit float. Literals: `0xff`, `0b1010`, `0o17`, `1_000_000`, `1.5e3`. `(255).toString(16)` → `"ff"` (string).
- **Parsing**: `parseInt`/`parseFloat` read from the left and **stop at junk** (`"12px"` → 12); `Number()` requires the whole string (`"12px"` → `NaN`). Always `parseInt(x, 10)`; without a radix `"0x10"` is hex (16). `parseInt` stringifies first, so `parseInt(0.0000005)` → `5`.
- **Formatting**: `toFixed(n)` = n decimals, `toPrecision(n)` = n significant digits, **both return strings**. They round the *binary* value: `(1.005).toFixed(2)` → `"1.00"`, `(1.45).toFixed(1)` → `"1.4"`.
- **Math**: `round(-2.5)` → `-2` (toward `+∞`), `floor(-2.1)` → `-3`, `trunc(-2.7)` → `-2`. `Math.max()` → `-Infinity`, `Math.min()` → `Infinity`.
- **Random**: `0 ≤ Math.random() < 1`; inclusive integer = `Math.floor(r * (max - min + 1)) + min`. Not crypto-secure.
- **Float trap**: `0.1 + 0.2` → `0.30000000000000004`; compare with `Math.abs(a - b) < Number.EPSILON`. Money → integer paise, `Math.round(x * 100)`.
- **Safe ints**: `MAX_SAFE_INTEGER` = `2^53 - 1` = `9007199254740991`; above it literals are rounded at parse time, so IDs travel as strings.
- **BigInt**: `10n`, `typeof "bigint"`, `10n / 3n` → `3n`, mixing with number throws, no `Math`, `JSON.stringify` throws.
- **Date creation**: `new Date(0)` = epoch; `new Date(y, m, d)` is **local** and `m` is **0-indexed**; overflow rolls (`new Date(2026, 1, 29)` → 1 Mar); day `0` = last day of previous month.
- **Methods**: `getMonth` 0-based, `getDate` 1-based, `getDay` 0 = Sunday, `getTime` = ms. `setDate(getDate() + n)` adds days safely; `setMonth` on the 31st rolls into the next month.
- **Timestamps**: `Date.now()` / `getTime()` = ms; `/1000` for seconds; `Date.UTC(y,m,d)` reads parts as UTC, `new Date(y,m,d)` as local.
- **Intl**: `new Intl.DateTimeFormat(locale, { dateStyle, timeStyle, timeZone })`; build the formatter once. `dateStyle` shortcuts: short/medium/long/full.
- **Zones**: a `Date` is an instant. `getX` local vs `getUTCX`; `toISOString()` always UTC with `Z`; `getTimezoneOffset()` = minutes to add to local to reach UTC (IST `-330`). **Date-only strings parse as UTC**, date-time strings without `Z` parse as local.

## Must-remember code

```javascript
const randomInt = (min, max) =>
  Math.floor(Math.random() * (max - min + 1)) + min;
```

```javascript
const toPaise = (rupees) => Math.round(rupees * 100);
const formatINR = (paise) =>
  new Intl.NumberFormat("en-IN", { style: "currency", currency: "INR" })
    .format(paise / 100);
```

```javascript
const daysBetween = (a, b) => {
  const utc = (d) => Date.UTC(d.getFullYear(), d.getMonth(), d.getDate());
  return Math.round((utc(b) - utc(a)) / 86_400_000);
};
```

## Interview traps

- `0.1 + 0.2 !== 0.3` → never `===` on arithmetic floats; use `Number.EPSILON` or integers.
- `parseInt("08")` is fine nowadays, but `parseInt("0x10")` is 16 → always pass radix `10`.
- `Number("")` is `0`, `Number(" 42 ")` is `42`, `Number(null)` is `0`, `Number(undefined)` is `NaN`.
- `toFixed` does not round like a human (`1.005` → `"1.00"`); don't use it to compute money.
- Month is `0`-indexed, `getDay()` is **not** the day of month.
- `setMonth(getMonth() + 1)` on the 31st gives March, not February.
- `new Date("2026-01-15")` is UTC midnight; `new Date("2026-01-15T00:00:00")` is local midnight.
- `JSON.stringify` on BigInt throws; on a Date it silently produces a string.
- `Math.max(...[])` is `-Infinity`, not `0`.

## 3 things to remember

1. **Floats are binary approximations.**
   ```javascript
   Math.abs(0.1 + 0.2 - 0.3) < Number.EPSILON; // true
   ```

2. **Months are zero-based.**
   ```javascript
   new Date(2026, 0, 15).getMonth(); // 0 → January
   ```

3. **Dates are instants; format at the edge.**
   ```javascript
   new Intl.DateTimeFormat("en-IN", { dateStyle: "medium", timeZone: "UTC" })
     .format(new Date("2026-01-15T10:30:00Z")); // "15 Jan 2026"
   ```
