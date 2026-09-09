# Python Decorators

Decorators are one of those Python topics that initially look confusing, but once you understand **functions as objects → nested functions → closures → decorators**, the whole thing becomes much easier.

The core idea:

```text
Function
   ↓
pass it to another function
   ↓
modify/wrap its behavior
   ↓
get back a new function
```

A decorator lets you **add behavior to an existing function without changing the function's original code**.

---

# 1. Functions as Objects

In Python, functions are **first-class objects**.

That means you can:

* Store a function in a variable
* Pass a function as an argument
* Return a function from another function
* Store functions in lists/dictionaries

### Assign a function to a variable

```python
def greet():
    return "Hello Kakade"


message = greet

print(message())
```

Output:

```text
Hello Kakade
```

Both:

```python
greet
message
```

refer to the same function.

---

## Functions as arguments

You can pass a function to another function:

```python
def greet():
    return "Hello Kakade"


def execute(function):
    return function()


print(execute(greet))
```

Output:

```text
Hello Kakade
```

Notice:

```python
execute(greet)
```

not:

```python
execute(greet())
```

Because we want to pass the **function itself**, not the result of calling it.

---

## Functions can be returned

```python
def create_greeting():
    def greet():
        return "Hello Kakade"

    return greet
```

Now:

```python
message = create_greeting()

print(message())
```

Output:

```text
Hello Kakade
```

This idea is extremely important for understanding decorators.

---

# 2. Nested Functions

A **nested function** is a function defined inside another function.

```python
def outer():

    def inner():
        print("Hello Kakade")

    inner()
```

Calling:

```python
outer()
```

Output:

```text
Hello Kakade
```

The `inner()` function exists inside the scope of `outer()`.

---

## Returning a nested function

This is more interesting:

```python
def outer():

    def inner():
        print("Hello Kakade")

    return inner
```

Now:

```python
function = outer()

function()
```

Output:

```text
Hello Kakade
```

Think:

```text
outer()
   │
   └── creates inner()
            │
            ↓
       returns inner
            │
            ↓
        function()
```

This is the foundation of decorators.

---

# 3. Closures

A **closure** happens when an inner function remembers variables from the outer function even after the outer function has finished executing.

Example:

```python
def create_greeting(name):

    def greet():
        return f"Hello {name}"

    return greet
```

Now:

```python
greet_kakade = create_greeting("Kakade")

print(greet_kakade())
```

Output:

```text
Hello Kakade
```

Where did `name` come from?

`create_greeting()` has already finished.

But `greet()` **remembers** the value of `name`.

That's a closure.

---

## Another example

```python
def multiplier(x):

    def multiply(number):
        return number * x

    return multiply
```

Create different functions:

```python
double = multiplier(2)
triple = multiplier(3)
```

Now:

```python
print(double(10))
print(triple(10))
```

Output:

```text
20
30
```

The `double` function remembers:

```text
x = 2
```

while `triple` remembers:

```text
x = 3
```

---

## Mental model

```text
multiplier(2)
     │
     └── x = 2
           │
           ↓
       multiply()
           │
           └── remembers x = 2


multiplier(3)
     │
     └── x = 3
           │
           ↓
       multiply()
           │
           └── remembers x = 3
```

This "remembering" behavior is what makes closures useful for decorators.

---

# 4. Basic Decorators

Now we can understand decorators.

Suppose we have:

```python
def greet():
    print("Hello Kakade")
```

We want to add:

```text
Before function
After function
```

without modifying `greet()` itself.

Create a decorator:

```python
def my_decorator(function):

    def wrapper():
        print("Before function")

        function()

        print("After function")

    return wrapper
```

Apply it:

```python
@my_decorator
def greet():
    print("Hello Kakade")
```

Then:

```python
greet()
```

Output:

```text
Before function
Hello Kakade
After function
```

---

# What does `@my_decorator` actually mean?

This:

```python
@my_decorator
def greet():
    print("Hello Kakade")
```

is essentially:

```python
def greet():
    print("Hello Kakade")

greet = my_decorator(greet)
```

This is **extremely important**.

The decorator receives the original function and replaces it with the wrapper.

```text
Original function
       │
       ↓
my_decorator()
       │
       ↓
   wrapper()
       │
       ↓
New greet
```

---

# 5. Decorators with Arguments

The previous decorator works for a function with no arguments:

```python
def greet():
    ...
```

But what if:

```python
def greet(name):
    ...
```

Our previous wrapper:

```python
def wrapper():
```

wouldn't accept `name`.

We need to handle arguments.

---

# 6. `*args` and `**kwargs` in Decorators

This is where `*args` and `**kwargs` become very useful.

```python
def my_decorator(function):

    def wrapper(*args, **kwargs):
        print("Before function")

        result = function(*args, **kwargs)

        print("After function")

        return result

    return wrapper
```

Now it can wrap functions with different arguments.

Example:

```python
@my_decorator
def greet(name):
    print(f"Hello {name}")
```

Call:

```python
greet("Kakade")
```

Output:

```text
Before function
Hello Kakade
After function
```

---

## Why `*args`?

`*args` collects positional arguments.

For:

```python
greet("Kakade", "Python")
```

they become:

```python
args = ("Kakade", "Python")
```

---

## Why `**kwargs`?

`**kwargs` collects keyword arguments.

For:

```python
greet(name="Kakade", language="Python")
```

you get:

```python
kwargs = {
    "name": "Kakade",
    "language": "Python"
}
```

So:

```python
def wrapper(*args, **kwargs):
```

makes your decorator flexible enough to work with many different functions.

---

# 7. Returning the Function's Result

A common mistake when writing decorators is forgetting to return the wrapped function's result.

Bad:

```python
def decorator(function):

    def wrapper(*args, **kwargs):
        result = function(*args, **kwargs)

    return wrapper
```

If the original function returns something, `wrapper()` doesn't return it.

Correct:

```python
def decorator(function):

    def wrapper(*args, **kwargs):
        result = function(*args, **kwargs)
        return result

    return wrapper
```

Or simply:

```python
def decorator(function):

    def wrapper(*args, **kwargs):
        return function(*args, **kwargs)

    return wrapper
```

---

# 8. A Practical Decorator: Logging

Decorators are commonly used for things like logging.

```python
def log_call(function):

    def wrapper(*args, **kwargs):
        print(f"Calling {function.__name__}")

        result = function(*args, **kwargs)

        print(f"Finished {function.__name__}")

        return result

    return wrapper
```

Use it:

```python
@log_call
def add(a, b):
    return a + b
```

Then:

```python
result = add(10, 20)

print(result)
```

Output:

```text
Calling add
Finished add
30
```

The original `add()` didn't need to know anything about logging.

---

# 9. `functools.wraps`

There's an important problem with decorators.

Consider:

```python
def decorator(function):

    def wrapper(*args, **kwargs):
        return function(*args, **kwargs)

    return wrapper
```

Then:

```python
@decorator
def greet():
    """Greets the user."""
    return "Hello Kakade"
```

Now:

```python
print(greet.__name__)
```

might give:

```text
wrapper
```

instead of:

```text
greet
```

The wrapper has replaced the original function's metadata.

---

## `functools.wraps`

Python provides:

```python
from functools import wraps
```

Use it like this:

```python
from functools import wraps


def decorator(function):

    @wraps(function)
    def wrapper(*args, **kwargs):
        return function(*args, **kwargs)

    return wrapper
```

Now:

```python
@decorator
def greet():
    """Greets the user."""
    return "Hello Kakade"
```

You get:

```python
print(greet.__name__)
```

```text
greet
```

And:

```python
print(greet.__doc__)
```

```text
Greets the user.
```

### Best practice

When writing decorators, generally use:

```python
@wraps(function)
```

This preserves important metadata from the original function.

---

# 10. Decorators with Arguments

This phrase can mean something slightly different.

Compare:

```python
@decorator
def greet():
    ...
```

with:

```python
@decorator("INFO")
def greet():
    ...
```

In the second case, **the decorator itself receives an argument**.

You need an additional function layer.

Example:

```python
def repeat(times):

    def decorator(function):

        def wrapper(*args, **kwargs):
            for _ in range(times):
                function(*args, **kwargs)

        return wrapper

    return decorator
```

Now:

```python
@repeat(3)
def greet():
    print("Hello Kakade")
```

Calling:

```python
greet()
```

Output:

```text
Hello Kakade
Hello Kakade
Hello Kakade
```

---

## Why three functions?

This is one of the most important decorator patterns.

```text
repeat(3)
    │
    ↓
decorator(function)
    │
    ↓
wrapper(*args, **kwargs)
    │
    ↓
actual function
```

There are three levels because there are three jobs:

### Level 1

```python
def repeat(times):
```

Receives the decorator's configuration.

### Level 2

```python
def decorator(function):
```

Receives the function being decorated.

### Level 3

```python
def wrapper(*args, **kwargs):
```

Receives the function's arguments when it is actually called.

---

# 11. Another Decorator with Arguments

Let's create a permission-style decorator:

```python
from functools import wraps


def require_role(role):

    def decorator(function):

        @wraps(function)
        def wrapper(user_role, *args, **kwargs):

            if user_role != role:
                raise PermissionError("Access denied")

            return function(user_role, *args, **kwargs)

        return wrapper

    return decorator
```

Use:

```python
@require_role("admin")
def delete_user(user_role, username):
    return f"Deleted {username}"
```

Now:

```python
print(delete_user("admin", "Kakade"))
```

works.

But:

```python
delete_user("guest", "Kakade")
```

raises:

```text
PermissionError: Access denied
```

This pattern is conceptually similar to decorators you may encounter in web frameworks.

---

# 12. Multiple Decorators

You can apply multiple decorators to the same function.

Example:

```python
def first(function):

    @wraps(function)
    def wrapper(*args, **kwargs):
        print("First")
        return function(*args, **kwargs)

    return wrapper
```

```python
def second(function):

    @wraps(function)
    def wrapper(*args, **kwargs):
        print("Second")
        return function(*args, **kwargs)

    return wrapper
```

Apply both:

```python
@first
@second
def greet():
    print("Hello Kakade")
```

Calling:

```python
greet()
```

Output:

```text
First
Second
Hello Kakade
```

---

# Decorator Order

This:

```python
@first
@second
def greet():
    ...
```

is equivalent to:

```python
greet = first(second(greet))
```

So the decorators are applied **from bottom to top**.

Think:

```text
greet
  ↓
second
  ↓
first
  ↓
final function
```

But when the final function is called, the outermost wrapper (`first`) runs first.

---

# 13. Decorator That Modifies a Result

Decorators don't have to only perform actions before/after a function.

They can modify the result.

```python
from functools import wraps


def uppercase(function):

    @wraps(function)
    def wrapper(*args, **kwargs):
        result = function(*args, **kwargs)
        return result.upper()

    return wrapper
```

Use:

```python
@uppercase
def greet():
    return "Hello Kakade"
```

Now:

```python
print(greet())
```

Output:

```text
HELLO KAKADE
```

The decorator changed the function's result.

---

# 14. Decorator Execution vs Function Execution

This is another important concept.

Consider:

```python
def decorator(function):
    print("Decorator executed")

    def wrapper():
        print("Function executed")
        return function()

    return wrapper


@decorator
def greet():
    print("Hello Kakade")
```

The line:

```python
@decorator
```

causes the decorator to run when the function is **defined**.

So:

```text
Decorator executed
```

happens before you call:

```python
greet()
```

Then calling:

```python
greet()
```

runs the wrapper.

Output sequence:

```text
Decorator executed
Function executed
Hello Kakade
```

### Important distinction

```text
Decoration time
    ↓
decorator(function)

Call time
    ↓
wrapper(*args, **kwargs)
    ↓
original function
```

---

# Real-World Uses of Decorators

You'll see decorators frequently in real Python applications.

### Logging

```python
@log_call
def process_data():
    ...
```

### Authentication

```python
@require_login
def dashboard():
    ...
```

### Authorization

```python
@require_role("admin")
def delete_user():
    ...
```

### Caching

```python
@cache
def expensive_calculation():
    ...
```

### Timing

```python
@measure_time
def process_data():
    ...
```

### Web frameworks

For example, Flask uses decorators for routes:

```python
@app.route("/users")
def users():
    return "Users"
```

The decorator registers the function as a route.

---

# The Complete Decorator Pattern

This is the pattern worth memorizing:

```python
from functools import wraps


def decorator(function):

    @wraps(function)
    def wrapper(*args, **kwargs):

        # Before
        print("Before")

        result = function(*args, **kwargs)

        # After
        print("After")

        return result

    return wrapper
```

Then:

```python
@decorator
def greet(name):
    return f"Hello {name}"
```

Use:

```python
print(greet("Kakade"))
```

Output:

```text
Before
After
Hello Kakade
```

---

# Decorator with Configuration

For:

```python
@decorator("something")
```

memorize this structure:

```python
from functools import wraps


def decorator(argument):

    def actual_decorator(function):

        @wraps(function)
        def wrapper(*args, **kwargs):

            # decorator logic

            return function(*args, **kwargs)

        return wrapper

    return actual_decorator
```

The nesting looks complicated, but each layer has a specific responsibility:

```text
decorator(argument)
       ↓
 receives configuration

actual_decorator(function)
       ↓
 receives original function

wrapper(*args, **kwargs)
       ↓
 receives function call arguments
```

---

# Quick Revision

| Concept                  | Remember                                      |
| ------------------------ | --------------------------------------------- |
| **Functions as objects** | Functions can be stored, passed, and returned |
| **Nested function**      | Function defined inside another function      |
| **Closure**              | Inner function remembers outer variables      |
| **Decorator**            | Wraps/modifies another function               |
| **`@decorator`**         | Shortcut for `function = decorator(function)` |
| **`*args`**              | Captures positional arguments                 |
| **`**kwargs`**           | Captures keyword arguments                    |
| **`functools.wraps`**    | Preserves original function metadata          |
| **Decorator arguments**  | Requires another function layer               |
| **Multiple decorators**  | Applied bottom-to-top                         |
| **Wrapper**              | Function that surrounds the original function |

## The Mental Model

```text
              FUNCTIONS
                  │
                  ↓
          Functions are objects
                  │
                  ↓
          Pass functions around
                  │
                  ↓
           Nested functions
                  │
                  ↓
             Closures
                  │
                  ↓
             Decorators
                  │
        ┌─────────┴─────────┐
        ↓                   ↓
   @decorator          @decorator(...)
        │                   │
        ↓                   ↓
   wrapper()        decorator factory
                            │
                            ↓
                         wrapper()
```

### The 3 things to remember

**1. Basic decorator:**

```python
@decorator
def function():
    ...
```

means:

```python
function = decorator(function)
```

**2. General-purpose wrapper:**

```python
def wrapper(*args, **kwargs):
    return function(*args, **kwargs)
```

**3. Preserve metadata:**

```python
from functools import wraps

@wraps(function)
```

> **A decorator is essentially a function that takes another function, wraps or modifies its behavior, and returns a new function.**
