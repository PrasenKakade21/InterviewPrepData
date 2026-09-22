# Control Flow — Revision Notes (Intermediate)
> One-line mental model: `if` chooses, `for` repeats over items, `while` repeats until condition flips.

## Core recall
- `if/elif/else`: top-down, first True wins; indentation defines block.
- Nesting: flatten with `and` or guard-clauses/early `return`; ternary `x if c else y` for simple cases only.
- Truthiness: falsy `False None 0 0.0 "" [] () {} set()`; prefer `if users:` / `if not users:`; use `is None` when None-specific.
- `for` iterates list/tuple/set/str/dict (keys; `.values()/.items()` for rest); `while` for condition-driven loops; avoid infinite loop by updating state.
- `range(n)` → 0..n-1 (stop exclusive); `range(start,stop,step)`; `range` is lazy — `list(range(5))` to materialize.
- `break` exits loop; `continue` skips iteration; `pass` no-op placeholder (does not skip).
- `enumerate(it, start=0)` → index+value; `zip(a,b)` pairs until shortest; combine: `enumerate(zip(a,b), start=1)`.

## Must-remember code
```python
age = 25
status = "Adult" if age >= 18 else "Minor"
print(status)
users = []
print("No users" if not users else "Users exist")
```
```python
names = ["Kakade", "Rahul", "Amit"]; ages = [25, 28, 30]
for i, (n, a) in enumerate(zip(names, ages), start=1):
    print(i, n, a)
```
```python
for n in [10, -5, 20, -3, 30]:
    if n < 0:
        continue
    print(n)  # 10 20 30
```
```python
attempts = 0
while attempts < 3:
    attempts += 1
    if attempts == 2:
        print("Kakade connected"); break
```

## Interview traps
- `range(5)` gives 0–4, not 1–5; `zip` silently drops extras from longer iterable.
- `pass` vs `continue`: `pass` falls through and runs rest; `continue` jumps to next iteration.
- `if not value:` conflates `None/0/""/[]` → use `is None` for API missing-field checks.
- Missing loop update → infinite `while`; chained `if/if` vs `if/elif` changes semantics (both vs first-only).
- `for` over dict yields keys only — need `.items()` for key+value unpack.

## 3 things to remember
1. `for` for collections, `while` for conditions; `range` stop excluded.
2. `break` stops, `continue` skips, `pass` does nothing.
3. Prefer `enumerate`/`zip` over `range(len(...))`; check truthiness idiomatically.
