# Sets — Revision Notes (Intermediate)
> One-line mental model: Lists are sequences; sets are unique-item collections for membership and relationships between collections.
## Core recall
- Properties: unique, unordered (no index/slice), mutable; hash-table backed → `in` is O(1) avg vs O(n) list.
- Create: `{1,2,3}`, `set(iterable)`, `set("hello")` dedups; empty = `set()` (`{}` is dict).
- Dedup pattern: `list(set(nums))` — order not preserved.
- Mutate: `add(x)` single, `update(iterable)` multi; `remove(x)` → KeyError if missing, `discard(x)` → silent, `pop()` → arbitrary element, `clear()` → `set()`.
- Membership: `x in s`, `x not in s`.
- Ops: `|` union (all), `&` intersection (common), `-` difference (directional: A-B ≠ B-A), `^` symmetric diff (either but not both). Methods: `.union/.intersection/.difference/.symmetric_difference`.
- Sets vs list/tuple: no duplicates, no indexing, fast membership + math ops; use for tags, permissions, dedup, filtering.
## Must-remember code
- ```python
kakade_tags = ["python", "api", "python", "fastapi"]
print(list(set(kakade_tags)))
langs = {"Python"}; langs.add("Go"); langs.update(["Rust", "Java"]); print(langs)
  ```
- ```python
langs = {"Python", "Go"}; langs.discard("Rust"); print(langs)  # no error
try: langs.remove("Rust")
except KeyError: print("missing")
  ```
- ```python
a, b = {1, 2, 3}, {2, 3, 4}
print(a | b, a & b, a - b, a ^ b)  # {1,2,3,4} {2,3} {1} {1,4}
  ```
- ```python
admin = {"read", "write", "delete"}; editor = {"read", "write"}
print(admin - editor, admin & editor, admin | editor)
  ```
- ```python
users = {"Kakade", "Asha"}; print("Kakade" in users)  # True, O(1)
  ```
## Interview traps
- `{}` is empty dict, not set → use `set()` for empty set.
- `remove` vs `discard`: remove raises KeyError on missing; discard silent — use discard for idempotent cleanup.
- `pop()` removes arbitrary element, takes no index → `s.pop(0)` fails.
- Difference is directional (`A-B ≠ B-A`); symmetric diff `^` excludes common elements.
- `list(set(x))` loses order — don't rely on ordering; sort if needed: `sorted(set(x))`.
## 3 things to remember
1. Unique + unordered + O(1) membership — pick sets for dedup and `in` checks.
2. `| & - ^` = union, intersection, difference, symmetric-difference.
3. `add/update` vs `remove/discard/pop/clear` — know which raises.
