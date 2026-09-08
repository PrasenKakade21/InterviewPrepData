## 1. Defining Functions


Functions in Python are reusable blocks of code designed to perform a specific task. They prevent code repetition (the DRY principle: Don't Repeat Yourself), make code modular, and break down complex logic into manageable, testable components.

In Python, functions are **first-class objects**, meaning they can be passed around, assigned to variables, and returned from other functions just like integers, strings, or lists.

### Basic Syntax

```python
def function_name(parameters):
    """Optional docstring explaining the function."""
    # Function body
    return statement  # Optional

```

### Examples

```python
# Simple function with no return value
def greet_user():
    """Prints a static greeting message."""
    print("Welcome to the API service!")

greet_user()
# Welcome to the API service!

```

```python
# Function with logic and explicit return
def calculate_tax(amount, tax_rate=0.05):
    """Calculates total tax based on amount and rate."""
    return amount * tax_rate

tax = calculate_tax(100.0)
print(tax)
# 5.0

```

### Explain the Code

* `def`: The keyword used to declare a function block.
* `greet_user()` / `calculate_tax()`: The identifier used to reference and invoke the function.
* `"""Docstring"""`: A string literal placed as the first statement in a function body. Python assigns this to `function_name.__doc__`, which IDEs and documentation tools read.
* `return`: Exits the function immediately and sends a value back to the caller.

### Important Edge Cases / Gotchas

* **Missing Return Value Defaults to `None`:** If a function does not explicitly encounter a `return` statement, or uses a bare `return` with no argument, Python automatically returns `None`.

```python
def process_data(data):
    if not data:
        return
    # Imagine doing work here

result = process_data([])
print(result)
# None

```

### Practical Usage (Backend / API Development)

Defining modular utility functions to clean or extract incoming payload data in a Flask/FastAPI route.

```python
def extract_bearer_token(auth_header):
    """Extracts raw JWT token from HTTP Authorization header."""
    if not auth_header or not auth_header.startswith("Bearer "):
        return None
    return auth_header.split(" ")[1]

# Simulating request header from web framework
header = "Bearer eyJhbGciOiJIUzI1NiIn..."
token = extract_bearer_token(header)
print(token)
# eyJhbGciOiJIUzI1NiIn...

```

### Mental Model

A function is like a **factory machine**: You feed raw materials in (parameters), inside it processes those materials, and it outputs a finished product (`return` value). If you don't build an output ramp (`return`), it drops `None` onto the floor.

---

## 2. Parameters & Positional / Keyword Arguments

### Clear Explanation

* **Parameters** are the variable names defined in a function signature.
* **Arguments** are the actual values passed into the function when calling it.

Python allows arguments to be passed in two core ways:

1. **Positional Arguments:** Matched based on their order/position in the function call.
2. **Keyword Arguments:** Matched explicitly using `name=value` in the function call, ignoring order.

### Basic Syntax

```python
def func(param1, param2):
    pass

# Calling with positional arguments
func(val1, val2)

# Calling with keyword arguments
func(param2=val2, param1=val1)

```

### Examples

```python
def format_user_profile(username, user_id, role):
    return f"ID: {user_id} | User: {username} | Role: {role}"

# Positional arguments (Order matters strictly)
profile1 = format_user_profile("alex99", 1042, "Admin")
print(profile1)
# ID: 1042 | User: alex99 | Role: Admin

# Keyword arguments (Order does not matter)
profile2 = format_user_profile(role="Editor", username="sara_m", user_id=2055)
print(profile2)
# ID: 2055 | User: sara_m | Role: Editor

```

### Explain the Code

In `profile1`, `"alex99"` binds to `username`, `1042` to `user_id`, and `"Admin"` to `role` strictly by their placement.
In `profile2`, explicitly naming `role="Editor"` bypasses positional order.

### Important Edge Cases / Gotchas

* **Positional arguments MUST come before Keyword arguments in a function call:**

```python
# SyntaxError: positional argument follows keyword argument
# format_user_profile(username="alex99", 1042, "Admin")

```

### Practical Usage (Automation Script)

Keyword arguments improve code clarity when passing multiple flags or configuration parameters where order is hard to remember.

```python
def connect_database(host, port, database, timeout, ssl):
    return f"Connecting to {host}:{port}/{database} (SSL: {ssl})"

# Expressive, self-documenting call
connection = connect_database(
    host="127.0.0.1",
    port=5432,
    database="production_db",
    timeout=30,
    ssl=True
)
print(connection)
# Connecting to 127.0.0.1:5432/production_db (SSL: True)

```

### Mental Model

* **Positional Arguments:** Named seating in a cinema row based on tickets: seat 1, seat 2, seat 3.
* **Keyword Arguments:** Name-tagged seats: your ticket specifically says "Alice's Seat", regardless of where it is in the row.

---

## 3. Return Values

### Clear Explanation

The `return` statement serves two purposes:

1. It halts execution of the function immediately.
2. It sends back data to the execution context that called the function.

Python functions can return any valid object, including collections, functions, modules, or multiple comma-separated values (which Python packs into a single **tuple**).

### Basic Syntax

```python
# Returning multiple values
def func():
    return val1, val2

a, b = func()  # Unpacking tuple return

```

### Examples

```python
def parse_status_code(code):
    if code == 200:
        return "OK", True
    elif code == 404:
        return "Not Found", False
    return "Unknown Error", False

# Receiving multiple return values (Tuple Unpacking)
status_msg, is_success = parse_status_code(200)
print(f"Message: {status_msg}, Success: {is_success}")
# Message: OK, Success: True

```

### Explain the Code

`return "OK", True` is implicitly interpreted by Python as returning a single tuple `("OK", True)`. The caller then unpacks this tuple directly into two variables `status_msg` and `is_success`.

### Important Edge Cases / Gotchas

* **Early Return Pattern vs. Nested Else:** Using `return` early reduces deep nesting and makes functions easier to follow.

```python
# Less Pythonic (Deep nesting)
def validate_payload(data):
    if "id" in data:
        if isinstance(data["id"], int):
            return True
        else:
            return False
    else:
        return False

# Idiomatic Pythonic Early Return (Guard Clauses)
def validate_payload_clean(data):
    if "id" not in data:
        return False
    return isinstance(data["id"], int)

```

### Practical Usage (API Endpoint Guard)

```python
def authenticate_request(headers):
    token = headers.get("Authorization")
    if not token:
        return False, {"error": "Missing token", "code": 401}
    if token != "secret-token":
        return False, {"error": "Invalid token", "code": 403}
    
    return True, {"user_id": 99}

is_valid, response = authenticate_request({"Authorization": "secret-token"})
print(f"Allowed: {is_valid} | Payload: {response}")
# Allowed: True | Payload: {'user_id': 99}

```

---

## 4. Default Arguments

### Clear Explanation

Default arguments allow parameters to fall back to a predefined value if no argument is passed during the function call. This makes arguments optional and allows for cleaner function calls with sensible defaults.

### Basic Syntax

```python
def function_name(param1, param2=default_value):
    pass

```

### Examples

```python
def paginate_query(query, page=1, per_page=20):
    offset = (page - 1) * per_page
    return f"SELECT * FROM {query} LIMIT {per_page} OFFSET {offset}"

# Using defaults
print(paginate_query("users"))
# SELECT * FROM users LIMIT 20 OFFSET 0

# Overriding defaults
print(paginate_query("users", page=3, per_page=50))
# SELECT * FROM users LIMIT 50 OFFSET 100

```

### Explain the Code

In `paginate_query`, `page` and `per_page` are given standard default values (`1` and `20`). If the user calls `paginate_query("users")`, Python fills in those values automatically.

### Important Edge Cases / Gotchas

* **The Mutable Default Argument Bug:** Default arguments are evaluated **ONCE** when the function is defined (at module load time), NOT each time the function is called. Never use a mutable object (`list`, `dict`, `set`) as a default value!

```python
# ❌ DANGEROUS: Persistent state across calls
def add_item_bad(item, item_list=[]):
    item_list.append(item)
    return item_list

print(add_item_bad("Apple"))
# ['Apple']
print(add_item_bad("Banana"))
# ['Apple', 'Banana']  <-- UNEXPECTED PERSISTENCE!

# ✅ CORRECT: Use None as sentinel value
def add_item_good(item, item_list=None):
    if item_list is None:
        item_list = []
    item_list.append(item)
    return item_list

print(add_item_good("Apple"))
# ['Apple']
print(add_item_good("Banana"))
# ['Banana']

```

---

## 5. `*args` and `**kwargs`

### Clear Explanation

* `*args`: Captures any number of additional **positional** arguments into a **tuple**.
* `**kwargs`: Captures any number of additional **keyword** arguments into a **dictionary**.

The names `args` and `kwargs` are strong Python conventions, but the actual magic lies in the asterisks (`*` and `**`).

### Basic Syntax

```python
def func(*args, **kwargs):
    # args is a tuple of positional arguments
    # kwargs is a dict of keyword arguments
    pass

```

### Examples

```python
def build_log_entry(level, *messages, **meta):
    full_message = " ".join(messages)
    log_data = {
        "level": level,
        "message": full_message,
        "metadata": meta
    }
    return log_data

log = build_log_entry(
    "INFO", 
    "User logged in.", 
    "Session initiated.", 
    user_id=42, 
    ip="192.168.1.1"
)

print(log)
# {'level': 'INFO', 'message': 'User logged in. Session initiated.', 'metadata': {'user_id': 42, 'ip': '192.168.1.1'}}

```

### Explain the Code

* `level` captures `"INFO"` positionally.
* `*messages` collects `"User logged in."` and `"Session initiated."` into the tuple `("User logged in.", "Session initiated.")`.
* `**meta` collects `user_id=42` and `ip="192.168.1.1"` into the dictionary `{'user_id': 42, 'ip': '192.168.1.1'}`.

### Important Edge Cases / Gotchas

* **Strict Parameter Ordering Rule:** When combining parameter types in a function signature, they MUST follow this order:
1. Standard positional arguments
2. Default positional arguments / `*args`
3. Keyword-only parameters (or `*args`)
4. `**kwargs`



```python
# Correct signature ordering pattern
def complex_function(a, b=1, *args, key=None, **kwargs):
    pass

```

### Practical Usage (Decorator Base & Wrapper Functions)

`*args` and `**kwargs` allow wrapper functions to accept and forward whatever arguments the original function needed without knowing its precise signature in advance.

```python
def audit_wrapper(func):
    def wrapper(*args, **kwargs):
        print(f"[AUDIT] Calling function: {func.__name__}")
        return func(*args, **kwargs)
    return wrapper

```

### Mental Model

* `*args` is a **box that packs extra single items** sitting on the counter into an ordered list (tuple).
* `**kwargs` is a **box that packs extra labeled packages** into a labeled filing system (dictionary).

---

## 6. Argument Unpacking

### Clear Explanation

While `*` and `**` in a function signature **collect** parameters, using `*` and `**` in a function *call* does the exact opposite: it **unpacks** iterable items (lists, tuples) into positional arguments, and dictionary items into keyword arguments.

### Basic Syntax

```python
args_list = [1, 2, 3]
kwargs_dict = {"a": 10, "b": 20}

func(*args_list)        # equivalent to func(1, 2, 3)
func(**kwargs_dict)    # equivalent to func(a=10, b=20)

```

### Examples

```python
def create_point(x, y, z):
    return f"Point at X={x}, Y={y}, Z={z}"

coords = [10, 20, 30]
# Unpacking a list positionally
print(create_point(*coords))
# Point at X=10, Y=20, Z=30

point_data = {"x": 5, "y": 15, "z": 25}
# Unpacking a dictionary into keyword arguments
print(create_point(**point_data))
# Point at X=5, Y=15, Z=25

```

### Explain the Code

* `*coords` expands the 3-element list into three standalone positional arguments (`10, 20, 30`).
* `**point_data` expands the dictionary keys and values into `x=5, y=15, z=25`.

### Important Edge Cases / Gotchas

* **Key Name Mismatch:** Unpacking a dictionary with `**` requires dictionary key names to match the function's parameter names exactly. Otherwise, Python raises a `TypeError`.

```python
invalid_data = {"x": 5, "y": 15, "altitude": 25}
# TypeError: create_point() got an unexpected keyword argument 'altitude'
# create_point(**invalid_data)

```

### Practical Usage (API Request Data Passing)

Passing dynamically generated settings or payload dictionaries directly into client library functions.

```python
import json

def payload_builder(endpoint, method, headers, timeout):
    return {
        "url": f"https://api.service.com{endpoint}",
        "method": method,
        "headers": headers,
        "timeout": timeout
    }

config = {
    "endpoint": "/v1/users",
    "method": "POST",
    "headers": {"Content-Type": "application/json"},
    "timeout": 10
}

request_obj = payload_builder(**config)
print(request_obj["url"])
# https://api.service.com/v1/users

```

---

## 7. Variable Scope & Resolution (LEGB Rule)

### Clear Explanation

**Scope** refers to the visibility and lifecycle of variables in different parts of a program. Python resolves variable names using the **LEGB** lookup rule:

1. **L (Local):** Names defined inside the currently executing function body.
2. **E (Enclosing / Nonlocal):** Names defined inside outer functions (nested structures).
3. **G (Global):** Names declared at the module level (top-level script file).
4. **B (Built-in):** Reserved names built into Python (e.g., `len`, `print`, `range`).

Python searches this chain strictly in order: **L ➔ E ➔ G ➔ B**.

---

### A. Local vs. Global Scope & `global` Keyword

#### Clear Explanation

Variables created inside a function live in its **Local** scope and are destroyed when the function exits. Variables defined outside any function belong to the **Global** scope.

To *modify* (reassign) a global variable from inside a local function scope, you must explicitly declare it using the `global` keyword.

#### Examples

```python
app_version = "1.0.0"  # Global scope

def check_version():
    local_status = "Active"  # Local scope
    print(f"App Version: {app_version} | Status: {local_status}")

check_version()
# App Version: 1.0.0 | Status: Active

# Trying to access local_status outside raises NameError
# print(local_status) # NameError: name 'local_status' is not defined

```

```python
counter = 0  # Global

def increment_bad():
    # UnboundLocalError: local variable 'counter' referenced before assignment
    # counter += 1
    pass

def increment_good():
    global counter  # Explicitly targeting global counter
    counter += 1

increment_good()
print(counter)
# 1

```

---

### B. Enclosing Scope & `nonlocal` Keyword

#### Clear Explanation

When a function is nested inside another function, the outer function's local scope becomes the **Enclosing** (or non-local) scope for the inner function.

To *modify* a variable in the outer function's scope from inside the inner function, use the `nonlocal` keyword.

#### Examples

```python
def outer_tracker():
    request_count = 0  # Enclosing scope variable

    def inner_logger():
        nonlocal request_count  # Bind to outer scope's request_count
        request_count += 1
        return f"Total requests logged: {request_count}"

    return inner_logger

tracker = outer_tracker()  # Returns the inner_logger function (Closure)
print(tracker())
# Total requests logged: 1
print(tracker())
# Total requests logged: 2

```

### Explain the Code

`nonlocal request_count` instructs Python not to create a new local variable `request_count` inside `inner_logger`, nor look for a global variable, but to mutate the existing `request_count` defined within `outer_tracker`.

### Comparisons: Scope Modifiers

| Keyword | Target Scope | Primary Use Case |
| --- | --- | --- |
| **`global`** | Top-level module scope | Mutating module-level configuration or counters from inside a function. |
| **`nonlocal`** | Immediate outer enclosing function scope | Mutating state inside closures or nested functions. |

### Important Edge Cases / Gotchas

* **Shadowing Built-ins:** Avoid assigning variables to names in the Built-in scope (e.g., `list`, `dict`, `str`, `min`, `max`, `sum`). Doing so shadows the built-in functionality and breaks it in local context.

```python
# ❌ NEVER DO THIS
# list = [1, 2, 3]
# new_list = list("hello")  # TypeError: 'list' object is not callable

```

### Mental Model: LEGB

Think of standard **nested translucent boxes**:

* From inside the inner-most box (**Local**), you can look through the clear walls to see what's outside in **Enclosing**, **Global**, and **Built-in**.
* But if you want to **replace** something in an outer box from inside your current box, you must tag it with `global` or `nonlocal` first—otherwise Python builds a brand new object right inside your local box instead.

---

## 8. Lambda Functions

### Clear Explanation

Lambda functions are small, anonymous functions defined in a single inline line. They can accept any number of positional or keyword arguments, but can contain only a **single expression** whose result is implicitly returned.

They are typically used for quick inline operations, short-lived callbacks, or key functions passed into higher-order functions like `sorted()`, `map()`, or `filter()`.

### Basic Syntax

```python
lambda argument1, argument2: expression

```

### Examples

```python
# Standard function equivalent
def multiply_def(x, y):
    return x * y

# Equivalent lambda function
multiply_lambda = lambda x, y: x * y

print(multiply_lambda(4, 5))
# 20

```

```python
# Practical use: Custom sorting key
users = [
    {"username": "dev_bob", "age": 34},
    {"username": "admin_alice", "age": 28},
    {"username": "user_charlie", "age": 22}
]

# Sort user dictionaries by 'age' key
users_sorted = sorted(users, key=lambda u: u["age"])
print(users_sorted)
# [{'username': 'user_charlie', 'age': 22}, {'username': 'admin_alice', 'age': 28}, {'username': 'dev_bob', 'age': 34}]

```

### Explain the Code

`key=lambda u: u["age"]`: The `sorted()` function calls this inline lambda for each element `u` in the `users` list and sorts items based on the value returned (`u["age"]`).

### Important Edge Cases / Gotchas

* **No Statements Allowed:** Lambdas cannot contain complex Python control structures like `for` loops, `while` loops, `pass`, or `raise` statements, nor can they perform assignments (`=`).
* **Avoid Binding Lambdas to Variable Names:** Assigning a lambda to a variable (`foo = lambda x: ...`) defeats its purpose as an anonymous function. PEP 8 guidelines explicitly advise using standard `def` for named functions because `def` provides clearer stack traces during debugging.

```python
# ❌ Discouraged by PEP 8
add = lambda x, y: x + y

# ✅ Preferred
def add(x, y):
    return x + y

```

### Comparisons: `def` vs `lambda`

| Feature | Standard Function (`def`) | Lambda Function (`lambda`) |
| --- | --- | --- |
| **Name** | Explicit name required | Anonymous |
| **Statements** | Unlimited statements and loops | Single inline expression |
| **Readability** | High for complex logic | High for brief single-use callbacks |
| **Debugging** | Clear function name in stack trace | Shown generically as `<lambda>` |

---

## 9. Higher-Order Functions: `map()`, `filter()`, `reduce()`

### Clear Explanation

A **Higher-Order Function** is a function that does at least one of the following:

* Accepts one or more functions as arguments.
* Returns a function as its result.

`map()`, `filter()`, and `reduce()` are classic higher-order functional programming built-ins used to transform, extract, or aggregate iterables without explicit `for` loops.

---

### A. `map()`

#### Clear Explanation

Applies a given transformation function to every item in an iterable and returns an **iterator** containing the updated elements.

#### Basic Syntax

```python
map(transformation_function, iterable)

```

#### Example

```python
prices_usd = [10.0, 25.5, 100.0]
usd_to_eur_rate = 0.92

# Transform all prices to EUR
prices_eur = list(map(lambda price: round(price * usd_to_eur_rate, 2), prices_usd))
print(prices_eur)
# [9.2, 23.46, 92.0]

```

---

### B. `filter()`

#### Clear Explanation

Evaluates every item in an iterable against a predicate function that returns `True` or `False`. It returns an **iterator** containing only the items that passed the condition (`True`).

#### Basic Syntax

```python
filter(predicate_function, iterable)

```

#### Example

```python
api_logs = [
    {"status": 200, "path": "/home"},
    {"status": 500, "path": "/checkout"},
    {"status": 404, "path": "/unknown"},
    {"status": 502, "path": "/payment"}
]

# Extract only server error logs (status >= 500)
server_errors = list(filter(lambda log: log["status"] >= 500, api_logs))
print(server_errors)
# [{'status': 500, 'path': '/checkout'}, {'status': 502, 'path': '/payment'}]

```

---

### C. `reduce()`

#### Clear Explanation

Unlike `map` and `filter`, `reduce()` is imported from the standard library's `functools` module. It repeatedly applies a two-argument function cumulative to the items of a sequence, rolling the sequence down into a **single aggregate final value**.

#### Basic Syntax

```python
from functools import reduce

reduce(binary_function, iterable[, initializer])

```

#### Example

```python
from functools import reduce

cart_items = [
    {"name": "Laptop", "price": 1000},
    {"name": "Mouse", "price": 25},
    {"name": "Monitor", "price": 300}
]

# Aggregate total bill cost starting at initial value 0
total_bill = reduce(lambda acc, item: acc + item["price"], cart_items, 0)
print(total_bill)
# 1325

```

### Explain the Code

In `reduce`:

1. `acc` (accumulator) holds the running aggregate result (initialized to `0`).
2. `item` is the current element pulled from `cart_items`.
3. In each iteration, `acc + item["price"]` calculates a new combined value that becomes the `acc` for the subsequent item.

### Modern Pythonic Alternative: List / Dict Comprehensions

While `map()` and `filter()` are common in functional languages, modern idiomatic Python generally favors **List Comprehensions** and **Generator Expressions**. They are more readable and eliminate the need for wrapping functions in explicit `lambda`s or `list()` calls.

```python
prices_usd = [10.0, 25.5, 100.0]

# map() approach
eur_map = list(map(lambda p: round(p * 0.92, 2), prices_usd))

# Pythonic List Comprehension equivalent (Preferred)
eur_comp = [round(p * 0.92, 2) for p in prices_usd]
print(eur_comp)
# [9.2, 23.46, 92.0]

```

```python
# filter() approach
errors_filter = list(filter(lambda log: log["status"] >= 500, api_logs))

# Pythonic List Comprehension equivalent (Preferred)
errors_comp = [log for log in api_logs if log["status"] >= 500]
print(errors_comp)
# [{'status': 500, 'path': '/checkout'}, {'status': 502, 'path': '/payment'}]

```

### Mental Model Summary

* **`map()`:** Assembly Line Transformer—Takes 5 items in, applies an action to each, releases 5 transformed items out.
* **`filter()`:** Security Bouncer—Takes 5 items in, inspects against a checklist, passes through only approved items (<= 5).
* **`reduce()`:** Trash Compactor—Takes multiple items in, smushes them continuously step-by-step, produces 1 single block out.

---

## Revision Cheat Sheet

### 1. Function Arguments Summary

```python
def example(a, b=2, *args, **kwargs):
    """
    a       -> Mandatory Positional
    b       -> Optional Default Positional
    args    -> Positional collector tuple
    kwargs  -> Keyword collector dict
    """
    pass

```

### 2. Scope Lookup Order (LEGB)

1. **L**ocal: Inside current function definition.
2. **E**nclosing: In enclosing nested outer functions (`nonlocal`).
3. **G**lobal: At script module top-level (`global`).
4. **B**uilt-in: Reserved keywords/functions built into Python engine (`len`, `sum`).

### 3. Core Syntax Quick Reference

* **Positional/Keyword Unpacking:** `func(*[1, 2], **{"c": 3})`
* **Safe Mutable Argument Pattern:** `def func(data=None): data = data or []`
* **Lambda Syntax:** `lambda x, y: x + y`
* **Map:** `map(func, seq)`
* **Filter:** `filter(pred, seq)`
* **Reduce:** `functools.reduce(func, seq, init)`

---

## Self-Assessment Practice Problems

1. **The Default Argument Trap:**
Analyze the code snippet below. What will calling `append_to_cache("key1", "val1")` followed immediately by `append_to_cache("key2", "val2")` return? How would you modify it to prevent shared state?
```python
def append_to_cache(key, value, cache={}):
    cache[key] = value
    return cache

```


2. **Refactoring Functional Code to Comprehensions:**
Rewrite the following line using a single Pythonic list comprehension instead of `map()` and `filter()`:
```python
raw_nums = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
processed = list(map(lambda x: x ** 2, filter(lambda x: x % 2 == 0, raw_nums)))

```


3. **Scope Variable Resolution:**
What will be printed to the terminal when executing `main()`? Trace variable bindings manually using LEGB rules:
```python
x = 10

def main():
    x = 20
    def sub_func():
        nonlocal x
        x = 30
    sub_func()
    print(x)

main()

```