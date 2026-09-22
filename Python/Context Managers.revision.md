# Context Managers — Revision Notes (Intermediate)
> One-line mental model: Setup → use → guaranteed cleanup (`__enter__` / `__exit__`), even on error.
## Core recall
- `with open("data.txt") as f:` auto-closes; manual `open/close` leaks if exception before `close()`.
- Class protocol: `__enter__(self)` (runs on entry, return value → `as x`) + `__exit__(self, exc_type, exc_val, tb)` (runs on exit, even on error).
- `__exit__` args are `(None,None,None)` on success; else exception info; `return True` suppresses, `False/None` re-raises (default: don't suppress).
- `@contextmanager` generator: code before `yield` = enter, `yield val` = `as val`, after `yield` = exit; wrap in `try/finally` so cleanup runs on error.
- Real uses: files, DB transactions, `with lock:`, temp files, Timer (`perf_counter` in enter/exit).
- Multiple: `with open(a) as s, open(b,"w") as d:` (cleaner than nesting); `as` optional if no useful return.
- Class-based = stateful/complex/reusable; `@contextmanager` = simple concise setup→yield→cleanup.
- `with` ≈ `try: use finally: release` packaged cleanly.
## Must-remember code
- ```python
  with open("data.txt") as f: data = f.read()
  with open("a.txt") as s, open("b.txt", "w") as d: d.write(s.read())
  ```
- ```python
  class Timer:
      from time import perf_counter
      def __enter__(self):
          from time import perf_counter; self.s = perf_counter(); return self
      def __exit__(self, *e):
          from time import perf_counter; print(f"{perf_counter()-self.s:.4f}s")
  ```
- ```python
  from contextlib import contextmanager
  @contextmanager
  def kakade_session():
      print("Opening")
      try: yield "Kakade session"
      finally: print("Closing")
  with kakade_session() as s: print(s)
  ```
- ```python
  class IgnoreErrors:
      def __enter__(self): return self
      def __exit__(self, *e): return True  # suppress
  ```
## Interview traps
- Forgetting `try/finally` in `@contextmanager` → cleanup skipped on exception → always use finally.
- Returning `True` from `__exit__` unintentionally swallows bugs → default return None/False unless suppression intended.
- Assuming `as x` is the manager → it's `__enter__`'s return value; must `return self` to get manager.
- Manual `f.close()` after error never runs → always prefer `with`.
- Yielding without value then using `as x` gives None → `yield conn` to expose resource.
## 3 things to remember
1. `__enter__` in, `__exit__` out — always runs, even on raise.
2. `True` suppresses, falsy propagates; `try/finally` for generator version.
3. Prefer `with` over manual try/finally; multi-resource on one line.
