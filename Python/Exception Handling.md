## 1. The `try` / `except` Block

### Clear Explanation

In Python, errors during runtime are called **exceptions**. When an unhandled exception occurs, Python halts program execution and prints a traceback.

The `try` / `except` structure provides a way to **catch** and **handle** runtime errors gracefully. Code that might fail is placed inside the `try` block. If an error occurs, Python stops executing the rest of the `try` block and jumps immediately into the matching `except` block.

### Basic Syntax

```python
try:
    # Code that might raise an exception
    dangerous_operation()
except ExceptionType as error_variable:
    # Code that runs if the exception occurs
    handle_error(error_variable)

```

### Examples

```python
# Standard handling of a potential runtime crash
raw_input = "not_a_number"

try:
    user_age = int(raw_input)
    print(f"Age set to {user_age}")
except ValueError as e:
    print(f"Failed to convert input: {e}")

# Output:
# Failed to convert input: invalid literal for int() with base 10: 'not_a_number'

```

### Explain the Code

* `try:` marks the code block Python will monitor for exceptions.
* `int(raw_input)` raises a `ValueError` because `"not_a_number"` cannot be converted to an integer.
* `except ValueError as e:` intercepts only `ValueError` instances and assigns the exception instance to variable `e`.
* The `print()` inside the `except` executes, and the script continues running safely instead of crashing.

### Important Edge Cases / Gotchas

* **Bare `except:` is dangerous:** Writing a bare `except:` catches **all** exceptions, including keyboard interrupts (`Ctrl+C`) and system exit requests (`sys.exit()`), making program termination or debugging extremely difficult.

```python
# ❌ DANGEROUS: Traps SystemExit, KeyboardInterrupt, and masks bugs
try:
    do_something()
except:
    print("An error occurred!")

# ✅ CORRECT: Catch explicit, expected exception types or Exception base class
try:
    do_something()
except Exception as e:
    print(f"Application error: {e}")

```

### Practical Usage (Backend API Input Validation)

Parsing incoming JSON payloads in web applications without crashing the server thread on malformed data.

```python
import json

def parse_user_payload(raw_json_str):
    try:
        payload = json.loads(raw_json_str)
        return payload, True
    except json.JSONDecodeError as err:
        # Return structured error details for API consumer
        return {"error": "Invalid JSON string provided", "line": err.lineno}, False

data, success = parse_user_payload('{"username": "dev_alex",}') # Trailing comma is invalid JSON
print(f"Success: {success} | Response: {data}")
# Output:
# Success: False | Response: {'error': 'Invalid JSON string provided', 'line': 1}

```

### Mental Model

Think of `try` / `except` as an **airbag system**: You drive normal code through the `try` tunnel. If code crashes (`impact`), the `except` airbag inflates immediately, absorbing the crash so your program doesn't hit a fatal brick wall.

---

## 2. Complementary Blocks: `else` and `finally`

### Clear Explanation

Python provides two optional control blocks to pair with `try`/`except`:

* **`else`:** Runs **ONLY IF NO exceptions** were raised in the `try` block. It separates code that might fail from code that should only run after a successful attempt.
* **`finally`:** Runs **ALWAYS**, regardless of whether an exception occurred, was handled, or went unhandled—and even if a `return`, `break`, or `continue` statement was hit! It is designed for clean-up tasks (closing DB connections, unlocking files, releasing network sockets).

### Basic Syntax

```python
try:
    # Risky operation
    pass
except SpecificError:
    # Runs on failure
    pass
else:
    # Runs ONLY on success
    pass
finally:
    # Runs ALWAYS (Cleanup)
    pass

```

### Examples

```python
def process_file_data(file_path):
    f = None
    try:
        f = open(file_path, "r")
    except FileNotFoundError:
        print(f"Error: File '{file_path}' does not exist.")
    else:
        # Executed only if open() succeeded
        data = f.read()
        print(f"Successfully read {len(data)} characters.")
    finally:
        # Executed no matter what happened
        if f and not f.closed:
            f.close()
            print("File stream safely closed.")

process_file_data("non_existent_file.txt")
# Output:
# Error: File 'non_existent_file.txt' does not exist.
# File stream safely closed.

```

### Explain the Code

1. `open()` fails because the file doesn't exist, raising `FileNotFoundError`.
2. Execution jumps straight to `except FileNotFoundError:`.
3. The `else:` block is **skipped** because an error occurred.
4. The `finally:` block runs, safely validating and closing resource handles.

### Important Edge Cases / Gotchas

* **`finally` overwrites return values!** If a `finally` block explicitly returns a value, it overrides any preceding `return` statement executed in the `try` or `except` blocks.

```python
def tricky_return():
    try:
        return "FROM TRY"
    finally:
        return "FROM FINALLY"  # ❌ Overrides the return statement from try!

print(tricky_return())
# Output:
# FROM FINALLY

```

### Practical Usage (Database Connection Cleanup)

Ensuring database transactions are cleanly committed or rolled back, and the pool connection is ALWAYS surrendered back.

```python
def execute_db_query(query, connection_pool):
    conn = connection_pool.get_connection()
    try:
        cursor = conn.cursor()
        cursor.execute(query)
        result = cursor.fetchall()
    except Exception as query_err:
        conn.rollback() # Cancel changes on error
        print(f"Query execution failed: {query_err}")
        return None
    else:
        conn.commit()   # Apply changes on success
        return result
    finally:
        # Ensure connection goes back to pool regardless of outcome
        connection_pool.release(conn)

```

---

## 3. Raising Exceptions (`raise`)

### Clear Explanation

The `raise` keyword allows developers to trigger exceptions explicitly in their code. It is used to enforce preconditions, signal domain-specific validation errors, or re-raise an intercepted exception after logging it.

### Basic Syntax

```python
# Instantiating and raising an exception
raise ValueError("Invalid configuration parameter provided.")

# Re-raising an exception inside an except block
try:
    ...
except Exception:
    # Log error...
    raise  # Re-raises the active exception caught by this block

```

### Examples

```python
# Validation check raising an exception
def calculate_discount(price, discount_percent):
    if not (0 <= discount_percent <= 100):
        raise ValueError(f"Discount must be between 0 and 100. Got: {discount_percent}")
    return price * (1 - discount_percent / 100)

try:
    calculate_discount(50, 150)
except ValueError as err:
    print(f"Validation Blocked Action: {err}")

# Output:
# Validation Blocked Action: Discount must be between 0 and 100. Got: 150

```

```python
# Exception Re-raising pattern (Audit/Logging)
def fetch_user_record(user_id):
    try:
        # Imagine an external DB call failing here
        raise ConnectionError("DB host unreachable")
    except ConnectionError as err:
        print(f"[LOG] Failure fetching user {user_id}: {err}")
        raise  # Re-raises 'ConnectionError' so upstream callers know it failed

```

### Explain the Code

In the second example, `except ConnectionError` traps the error to write an audit log message (`[LOG]...`). Calling a bare `raise` immediately re-triggers the exact active `ConnectionError` up the call stack for higher-level framework handlers to intercept.

### Important Edge Cases / Gotchas

* **Implicit vs. Explicit Exception Chaining (`from` keyword):** When raising a new exception from inside an `except` block, use `raise NewException(...) from original_err` to maintain clean exception cause chains in tracebacks.

```python
try:
    int("abc")
except ValueError as cause:
    # Explicitly links the new exception to the original cause
    raise KeyTypeError("Parsing failed due to bad input") from cause

```

---

## 4. Built-in Exceptions & The Exception Hierarchy

### Clear Explanation

Python structures all built-in exceptions in a strict class inheritance hierarchy. Knowing this hierarchy is crucial because an `except` block catching a parent class will catch instances of **any child class** inheriting from it.

### Python Exception Hierarchy (Abbreviated Core)

```text
BaseException
 ├── SystemExit
 ├── KeyboardInterrupt
 ├── GeneratorExit
 └── Exception  <-- (Catch ALL application/logic errors from here)
      ├── ArithmeticError
      │    └── ZeroDivisionError
      ├── LookupError
      │    ├── IndexError
      │    └── KeyError
      ├── TypeError
      ├── ValueError
      │    └── UnicodeDecodeError
      └── OSError
           └── FileNotFoundError

```

### Common Built-in Exceptions Overview

* `ValueError`: Raised when an operation receives an argument of right type but inappropriate value (`int("abc")`).
* `TypeError`: Raised when an operation is applied to an object of inappropriate type (`"2" + 2`).
* `KeyError`: Raised when a mapping (dict) key is not found.
* `IndexError`: Raised when a sequence index is out of range.
* `AttributeError`: Raised when an attribute reference or assignment fails on an object.

### Examples

```python
# Demonstration of catching by Parent Class
try:
    my_list = [1, 2, 3]
    print(my_list[10]) # Raises IndexError
except LookupError as e:  # LookupError is parent of IndexError!
    print(f"Caught by parent LookupError class: {type(e).__name__} - {e}")

# Output:
# Caught by parent LookupError class: IndexError - list index out of range

```

### Important Edge Cases / Gotchas

* **`Exception` vs. `BaseException`:** NEVER catch `BaseException` unless you are building custom root-level runner frameworks. Catching `BaseException` traps `KeyboardInterrupt` (`Ctrl+C`) and `SystemExit`, preventing users from stopping scripts from the CLI! Always use `except Exception:` as your broadest safe handler.

---

## 5. Multiple Exceptions

### Clear Explanation

A single block of code might fail in multiple distinct ways. Python allows handling multiple distinct exception types using multiple `except` blocks or combining them in a tuple inside a single `except` block.

### Basic Syntax

```python
# Multiple separate blocks
try:
    ...
except ExceptionTypeA:
    ...
except ExceptionTypeB:
    ...

# Grouped tuple block
try:
    ...
except (ExceptionTypeA, ExceptionTypeB) as err:
    ...

```

### Examples

```python
def process_dict_element(data_dict, key, index):
    try:
        target_list = data_dict[key]
        element = target_list[index]
        return element / 2
    except KeyError:
        print(f"Error: Key '{key}' does not exist in dictionary.")
    except IndexError:
        print(f"Error: Index {index} is out of bounds for the list.")
    except ZeroDivisionError:
        print("Error: Divisor value was zero.")
    except (TypeError, ValueError) as err:
        print(f"Data type conversion error: {err}")

process_dict_element({"items": [10, 20]}, "items", 5)
# Output:
# Error: Index 5 is out of bounds for the list.

```

### Important Edge Cases / Gotchas

* **Order Matters! Specific Exceptions First, Generic Later:** Python matches `except` blocks sequentially from top to bottom. If you place a parent class (like `Exception` or `LookupError`) before a subclass (like `KeyError`), the subclass block will **never execute**.

```python
# ❌ INCORRECT: Generic Exception hides specific handlers below it
try:
    data = {}["id"]
except Exception:
    print("Caught general exception!")
except KeyError:
    print("This will NEVER be executed!") # Unreachable code

# ✅ CORRECT: Specific subclasses first, general Exception last
try:
    data = {}["id"]
except KeyError:
    print("Caught specific KeyError!")
except Exception:
    print("Caught unexpected fallback exception!")

```

---

## 6. Custom Exceptions

### Clear Explanation

Custom exceptions are application-specific error classes created by inheriting from Python's built-in `Exception` class (or one of its sub-classes).

They improve readability, allow callers to catch business-logic failures specifically, and make APIs self-documenting.

### Basic Syntax

```python
class CustomErrorName(Exception):
    """Docstring explaining domain error condition."""
    pass

```

### Examples

```python
# Custom Exception Hierarchy for an API Service
class APIError(Exception):
    """Base exception class for all API runtime errors."""
    pass

class UnauthorizedError(APIError):
    """Raised when request lacks valid authentication credentials."""
    def __init__(self, message="Authentication required", status_code=401):
        super().__init__(message)
        self.status_code = status_code

class InsufficientFundsError(APIError):
    """Raised when account balance cannot cover transaction."""
    def __init__(self, balance, required):
        msg = f"Available balance ${balance} insufficient for required ${required}"
        super().__init__(msg)
        self.balance = balance
        self.required = required

# Usage
def process_payment(account_balance, charge_amount):
    if account_balance < charge_amount:
        raise InsufficientFundsError(account_balance, charge_amount)
    return account_balance - charge_amount

try:
    process_payment(20, 100)
except InsufficientFundsError as err:
    print(f"Payment Blocked: {err}")
    print(f"Shortfall Amount: ${err.required - err.balance}")

# Output:
# Payment Blocked: Available balance $20 insufficient for required $100
# Shortfall Amount: $80

```

### Explain the Code

* `CustomErrorName` inherits from `Exception`.
* Custom attributes (`status_code`, `balance`, `required`) are set via `__init__()` and passed to `super().__init__(message)` so traceback error text works seamlessly.

---

## 7. When to Catch vs. Propagate Exceptions

### Clear Explanation

Not every function should catch errors! Software architecture follows two fundamental paths when handling errors:

1. **Catching:** Intercepting an exception to handle it locally, recover, log, or translate it into an expected return state.
2. **Propagating:** Allowing an exception to pass through the current function context unhandled, delegating resolution to higher-level caller frames.

```text
  [Low-level Data Reader]  ──(Raises FileNotFoundError)──>  Propagates Up
            │
  [Service Controller]     ──(Translates to HTTP 404)───>   Catches & Handles

```

### Decision Matrix

| Scenario | Action | Rationale |
| --- | --- | --- |
| **Low-level Utility / Library code** | **Propagate** | You don't know how the caller wants to handle failures (e.g., CLI vs Web API vs GUI). |
| **Known, expected operational failures** | **Catch** | Recover gracefully (e.g., loading default configs if file missing). |
| **Top-level Framework Route / Entry point** | **Catch** | Prevent raw app crashes; convert errors into user messages or HTTP 500 error payloads. |
| **Resource allocation / Clean-up code** | **Propagate with `finally` / Context Managers** | Ensure resources are released, then let the exception continue up. |

### Idiomatic Strategy: EAFP vs. LBYL

Python heavily favors **EAFP** (*Easier to Ask for Forgiveness than Permission*) over **LBYL** (*Look Before You Leap*).

```python
payload = {"user": {"profile": {"name": "Alex"}}}

# ❌ LBYL (Look Before You Leap) - Cluttered with checks
if "user" in payload and "profile" in payload["user"] and "name" in payload["user"]["profile"]:
    name = payload["user"]["profile"]["name"]
else:
    name = "Guest"

# ✅ EAFP (Easier to Ask for Forgiveness than Permission) - Idiomatic Python
try:
    name = payload["user"]["profile"]["name"]
except (KeyError, TypeError):
    name = "Guest"

```

---

## Revision Cheat Sheet

### 1. The Complete Exception Block Architecture

```python
try:
    # 1. Action that might fail
    res = 10 / divisor
except ZeroDivisionError as e:
    # 2. Handles specific exception
    res = 0
else:
    # 3. Runs ONLY if try succeeded (No exceptions)
    log_success(res)
finally:
    # 4. ALWAYS executes (Cleanup)
    cleanup_resources()

```

### 2. Core Best Practices

* **Don't use bare `except:**` -> Catch `except Exception:` at minimum.
* **Order matters in `except` chains** -> Specific child exceptions first, parent/generic exceptions last.
* **Don't suppress exceptions silently** -> Avoid `except Exception: pass` without logging.
* **Keep `try` blocks small** -> Enclose only the specific line(s) that might raise an expected error.

---

## Self-Assessment Practice Problems

1. **Trace Execution with `else` and `finally`:**
Trace the printed console output step-by-step when executing `run_test(0)` vs `run_test(2)`:
```python
def run_test(val):
    try:
        result = 10 / val
    except ZeroDivisionError:
        print("DIV_ZERO")
        return "ERROR"
    else:
        print("SUCCESS")
        return "OK"
    finally:
        print("CLEANUP")

print(f"Result: {run_test(0)}")
print(f"Result: {run_test(2)}")

```


2. **Fixing the Exception Inheritance Trap:**
Why will the `ValueError` custom handler never execute in this script? How would you reorganize the `except` blocks to fix it?
```python
class DataProcessingError(Exception): pass
class InvalidValueError(DataProcessingError, ValueError): pass

try:
    raise InvalidValueError("Bad input data")
except DataProcessingError:
    print("Caught Data Processing Error")
except InvalidValueError:
    print("Caught Invalid Value Error")

```


3. **EAFP Conversion Challenge:**
Refactor this defensive LBYL code into clean, idiomatic Pythonic EAFP style using `try`/`except`:
```python
def get_file_size_kb(file_path):
    import os
    if os.path.exists(file_path):
        if os.path.isfile(file_path):
            size = os.path.getsize(file_path)
            return size / 1024
    return 0

```