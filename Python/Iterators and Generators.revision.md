# Iterators and Generators — Revision Notes (Intermediate)
> One-line mental model: Iterable gives iterator (`iter()`), iterator gives values (`next()`), generator is the lazy easy iterator (`yield`).
## Core recall
- Iterable (list/tuple/str/set/dict/range/file) loopable; Iterator implements `__iter__+__next__`, stateful, raises `StopIteration` at end.
- `it = iter([10,20,30]); next(it)` → 10,20,30 then StopIteration; `next(it, "Done")` default avoids raise.
- `for` ≈ `it=iter(x); while True: try: v=next(it) except StopIteration: break`.
- Custom iterator: `class CountUp: __iter__ returns self; __next__ raises StopIteration when done`.
- Generator func with `yield` pauses/resumes, lazy (body runs only on `next()`); `yield` vs `return` (pause many vs end once).
- Generator expr `(x*2 for x in range(n))` vs list comp `[...]` — lazy vs eager; `iter(gen) is gen` True.
- Memory: list 10M ints in RAM vs generator holds only state; prefer generator for single-pass large/infinite streams.
- Infinite generator `while True: yield n` — consume with `islice`/break; files are iterable — `for line in file` not `readlines()`.
## Must-remember code
- ```python
  nums = iter([10, 20])
  print(next(nums, "Done")); print(next(nums, "Done")); print(next(nums, "Done"))
  ```
- ```python
  def count_up(limit):
      cur = 1
      while cur <= limit:
          yield cur; cur += 1
  print(list(count_up(3)))
  ```
- ```python
  gen = (x * 2 for x in range(5))
  print(next(gen), next(gen))
  ```
- ```python
  def kakade_ids(s, e):
      for i in range(s, e + 1): yield i
  for i in kakade_ids(1, 3): print(i)
  ```
## Interview traps
- List is NOT iterator → `next([1,2])` TypeError; correct `next(iter([1,2]))`.
- Generator exhausted after one pass → second `list(g)` is `[]`; recreate or tee.
- Assuming generators are faster → win is memory, not speed; need re-access → use list.
- Forgetting `StopIteration` handling with manual `next()` → use default or try/except.
- `return` in generator ends iteration; `yield` inside function makes calling it lazy (no code runs until `next()`).
## 3 things to remember
1. `iter()` → iterator, `next()` → value, `StopIteration` → done.
2. `yield` pauses + remembers; generator = lazy iterator.
3. Big/infinite/one-pass data → generator; repeated/random access → list.
