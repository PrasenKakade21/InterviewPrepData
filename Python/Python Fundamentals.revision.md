# Python Fundamentals — Revision Notes (Intermediate)
> One-line mental model: Names refer to objects; operators and checks act on objects, not boxes.

## Core recall
- Variables are references: `b = a` shares mutable objects; check with `is`. Use `snake_case`, `UPPER_CASE` for constants.
- Types: `int` (arbitrary precision), `float` (binary approx), `str` (immutable), `bool` (subclass of `int`), `None` (absence).
- `type(x)` exact type; `isinstance(x, (int, float))` respects inheritance — prefer `isinstance`.
- Conversion: `int("25")`, `str(25)`, `float("99.99")`; `int(10.9)` truncates; `int("10.5")` → `ValueError`, use `int(float("10.5"))`.
- Operators: `/` always float, `//` floors toward -inf (`-10//3 == -4`), `%`, `**`; chained `18 <= age < 60`.
- `and`/`or` return operands (`0 or "Guest"`), short-circuit; `in` checks dict keys; `==` equality vs `is` identity.
- Falsy: `False, None, 0, 0.0, "", [], {}, (), set()`; use `if users:` and `if x is None:`.
- Walrus `:=` assigns inside expression: `if (n := len(s)) > 5:`.

## Must-remember code
```python
a = [1, 2, 3]; b = a; b.append(4)
print(a, a is b)  # [1,2,3,4] True
```
```python
from decimal import Decimal
print(0.1 + 0.2)  # 0.30000000000000004
print(Decimal("0.1") + Decimal("0.2"))  # 0.3
```
```python
name = "Kakade"
print(name[::-1], "Kakade" in "Mr Kakade")  # edakaK True
print("name" in {"name": "Kakade"})  # True (keys)
```
```python
username = "" or "Guest"
print(username, bool([]), True == 1)  # Guest False True
status = "Adult" if 25 >= 18 else "Minor"
print(status)
```

## Interview traps
- `==` vs `is`: `[1,2] == [1,2]` True but `is` False; use `is None`, never `is 10`.
- `int("10.5")` raises `ValueError` → correct: `int(float("10.5"))`; `int(-1.9)` is `-1` (truncate, not floor).
- `-10 // 3` is `-4` (not `-3`); `10/2` is `5.0` (float).
- `0 or default` picks default for any falsy (`0`, `""`) — wrong if `0` is valid; check `is None` explicitly.
- Mutable aliasing: `b = a; b.append()` mutates `a` → correct: `b = a.copy()` / `list(a)`.

## 3 things to remember
1. `is` = identity, `==` = equality; always `x is None`.
2. `and`/`or` return operands and short-circuit — use for guards/defaults carefully.
3. `isinstance` > `type` for checks; `int()` truncates, `//` floors.
