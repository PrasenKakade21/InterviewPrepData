# Exception Handling — Revision Notes (Intermediate)
> One-line mental model: `try` the risky line, `except` the expected failure, `else` on success, `finally` always clean up.

## Core recall
- `try/except E as e:` catches and continues; never bare `except:` — use `except Exception:` so `KeyboardInterrupt/SystemExit` still work.
- `else` runs only if no exception (separate risky vs success logic); `finally` always runs (close/rollback/release) even after `return/break`.
- `raise ValueError("...")` enforces contracts; bare `raise` re-raises active error after logging; `raise New(...) from cause` chains.
- Hierarchy: `BaseException > Exception > (ValueError, TypeError, LookupError→Key/IndexError, OSError→FileNotFoundError)`; parent `except` catches children.
- Multiple: separate `except KeyError:` / `except (TypeError, ValueError) as e:`; order specific→generic or later blocks unreachable.
- Custom: `class APIError(Exception)` + fields via `super().__init__(msg)`; e.g. `InsufficientFundsError(balance, required)`.
- Strategy: low-level propagate, top-level catch→HTTP/message; EAFP (`try` nested access) > LBYL chains; keep `try` small, never silent `pass`.

## Must-remember code
```python
try:
    age = int("Kakade")
except ValueError as e:
    print(f"Bad age: {e}")
else:
    print(f"Age {age}")
finally:
    print("done")
```
```python
def discount(price, pct):
    if not 0 <= pct <= 100:
        raise ValueError(f"Bad pct {pct}")
    return price * (1 - pct / 100)
try:
    discount(50, 150)
except ValueError as e:
    print(e)
```
```python
try:
    print([1, 2][10])
except LookupError as e:  # parent of IndexError
    print(f"lookup: {e}")
try:
    {}["id"]
except KeyError:
    print("specific first")
except Exception:
    print("fallback last")
```
```python
payload = {"user": {"profile": {"name": "Kakade"}}}
try:
    name = payload["user"]["profile"]["name"]
except (KeyError, TypeError):
    name = "Guest"
print(name)
```

## Interview traps
- Bare `except:` / `except BaseException:` swallows Ctrl+C, `sys.exit`, hides bugs → `except Exception:`.
- `return` in `finally` overrides `try/except` return — never return from `finally`.
- Generic `except Exception:` before `except KeyError:` makes specific unreachable — order specific first.
- `except: pass` silences errors — at least log; keep `try` to 1-2 risky lines.
- Raising new error in `except` without `from` loses cause — use `raise X(...) from e`.

## 3 things to remember
1. `else` = success-only, `finally` = always-cleanup.
2. Specific exceptions first, `Exception` last; never bare.
3. EAFP + small `try` + custom domain errors for APIs.
