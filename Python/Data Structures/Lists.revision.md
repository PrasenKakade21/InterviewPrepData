# Lists — Revision Notes (Intermediate)
> One-line mental model: Ordered + mutable; know whether each op mutates in place or returns new.
## Core recall
- Create `users=["Kakade","Alice"]`, mixed types ok; mutable (`l[0]="Rust"` works, unlike strings).
- Indexing 0-based + negative (`l[0]`, `l[-1]`); slicing `l[start:stop:step]`, stop excluded; `[:3]`, `[2:]`, `[:]`, `[::2]`, `[::-1]` (new list).
- `append(x)` one item (nested if list → `[1,2,[3,4]]`); `extend(iter)` many (`[1,2,3,4]`); `insert(i,x)` shifts right, `insert(100,x)` appends.
- `remove(value)` first match by value, ValueError if missing (guard `if x in l`); `pop()` last / `pop(i)` by index, returns item.
- `sort()` in-place + returns None, `reverse=True`, `key=lambda u: u["age"]`; `reverse()` in-place vs `[::-1]` new copy.
- `copy()` shallow (outer only; nested shared → use `deepcopy`); `b = a` aliases same object (`is` True).
- Unpacking `a,b,c=[10,20,30]` (count must match else ValueError); `first,*mid,last` starred → list; `print(*nums)` call-unpack.
- `list.sort()` mutates vs `sorted()` new list; `list.reverse()` mutates vs `reversed()` iterator; nested `matrix[1][1]`.
## Must-remember code
- ```python
  a = [1, 2]; a.append([3, 4]); print(a)  # [1, 2, [3, 4]]
  b = [1, 2]; b.extend([3, 4]); print(b)  # [1, 2, 3, 4]
  ```
- ```python
  users = [{"name": "Kakade", "age": 25}, {"name": "Asha", "age": 22}]
  users.sort(key=lambda u: u["age"]); print(users[0]["name"])
  ```
- ```python
  first, *middle, last = [1, 2, 3, 4, 5]
  print(first, middle, last)
  ```
- ```python
  orig = [1, 2, 3]; c = orig.copy(); c.append(4)
  print(orig, c)  # alias vs copy
  ```
## Interview traps
- `result = nums.sort()` → None, list mutated; want copy → `sorted(nums)`.
- `append([3,4])` vs `extend([3,4])` → nested vs flat; `extend("cd")` adds chars.
- `b = a; b.append(4)` mutates `a` too → use `.copy()` (shallow; nested needs `deepcopy`).
- `remove(10)` missing → ValueError; `pop(99)` → IndexError; check membership/bounds.
- `[::-1]` new list vs `.reverse()` in-place None → pick by whether caller may mutate.
## 3 things to remember
1. append=one, extend=many, insert=position; remove=value, pop=index+returns.
2. sort/reverse mutate (None); sorted/slicing create new.
3. `=` aliases, `.copy()` shallow — watch nested mutation.
