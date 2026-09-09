# Python Context Managers

Context managers are used when you need to **set something up, use it, and reliably clean it up afterward**.

The most common example is opening a file:

```python
with open("data.txt") as file:
    content = file.read()
```

You don't need to manually close the file.

The basic mental model is:

```text
Setup
  ↓
Use resource
  ↓
Cleanup
```

Context managers make sure the **cleanup happens even if an error occurs**.

---

# 1. `with` Statement

The `with` statement is used to work with a **context manager**.

For example:

```python
with open("data.txt") as file:
    content = file.read()
```

Python automatically:

1. Opens the file
2. Gives you the file object
3. Runs your code
4. Closes the file

Conceptually:

```text
with
 │
 ├── enter resource
 │
 ├── execute code
 │
 └── exit / cleanup
```

---

## Why not just use `open()`?

You could write:

```python
file = open("data.txt")

content = file.read()

file.close()
```

But what if:

```python
file = open("data.txt")

content = file.read()  # error happens here

file.close()
```

The error could prevent `file.close()` from running.

With:

```python
with open("data.txt") as file:
    content = file.read()
```

Python handles cleanup automatically.

---

# 2. What Is a Context Manager?

A context manager is an object that defines what should happen when entering and leaving a `with` block.

It typically implements:

```python
__enter__()
__exit__()
```

Example:

```python
class MyContext:
    def __enter__(self):
        print("Entering")

    def __exit__(self, exc_type, exc_value, traceback):
        print("Exiting")
```

Use it:

```python
with MyContext():
    print("Inside")
```

Output:

```text
Entering
Inside
Exiting
```

So:

```text
__enter__
   ↓
with block
   ↓
__exit__
```

---

# 3. `__enter__`

`__enter__()` runs when execution enters the `with` block.

Example:

```python
class MyContext:
    def __enter__(self):
        print("Starting")

    def __exit__(self, exc_type, exc_value, traceback):
        print("Finished")
```

Then:

```python
with MyContext():
    print("Working...")
```

Output:

```text
Starting
Working...
Finished
```

---

## Returning a value from `__enter__`

This is very useful.

```python
class Database:
    def __enter__(self):
        print("Connecting...")
        return self

    def __exit__(self, exc_type, exc_value, traceback):
        print("Closing connection")
```

Now:

```python
with Database() as db:
    print(db)
```

The value returned by:

```python
__enter__()
```

is assigned to:

```python
db
```

So:

```text
__enter__()
    │
    └── returns object
             ↓
        as db
```

---

# 4. `__exit__`

`__exit__()` runs when the `with` block finishes.

```python
class MyContext:
    def __enter__(self):
        print("Start")

    def __exit__(self, exc_type, exc_value, traceback):
        print("Cleanup")
```

```python
with MyContext():
    print("Doing work")
```

Output:

```text
Start
Doing work
Cleanup
```

The important part is that `__exit__()` is also called when an exception occurs inside the block.

---

# 5. The Three `__exit__` Arguments

`__exit__()` receives:

```python
def __exit__(self, exc_type, exc_value, traceback):
    ...
```

These describe an exception if one occurred.

### `exc_type`

The exception class.

For example:

```python
ValueError
```

### `exc_value`

The actual exception object.

For example:

```text
Invalid value
```

### `traceback`

Information about where the exception occurred.

---

## No exception

If everything succeeds:

```python
exc_type
exc_value
traceback
```

are all:

```python
None
```

---

## Exception

Example:

```python
class MyContext:
    def __enter__(self):
        print("Start")

    def __exit__(self, exc_type, exc_value, traceback):
        print(exc_type)
        print(exc_value)
```

Then:

```python
with MyContext():
    raise ValueError("Something went wrong")
```

You'll get information about the exception inside `__exit__`.

---

# 6. Suppressing Exceptions

`__exit__()` can control whether an exception is propagated.

If it returns:

```python
True
```

the exception is suppressed.

Example:

```python
class IgnoreErrors:
    def __enter__(self):
        return self

    def __exit__(self, exc_type, exc_value, traceback):
        return True
```

Now:

```python
with IgnoreErrors():
    raise ValueError("Something went wrong")

print("Program continues")
```

Output:

```text
Program continues
```

The exception was swallowed.

---

## Returning `False` or `None`

Normally:

```python
def __exit__(self, exc_type, exc_value, traceback):
    return False
```

means:

> Don't suppress the exception.

The exception continues normally.

### Important

Don't suppress exceptions unless you have a good reason.

Usually `__exit__()` should perform cleanup and let unexpected errors propagate.

---

# 7. Creating Custom Context Managers

You can create your own context manager using a class.

Example:

```python
class Connection:
    def __enter__(self):
        print("Opening connection")
        return self

    def __exit__(self, exc_type, exc_value, traceback):
        print("Closing connection")
```

Use:

```python
with Connection() as connection:
    print("Using connection")
```

Output:

```text
Opening connection
Using connection
Closing connection
```

---

# 8. A More Realistic Example

Imagine a resource that needs to be opened and closed:

```python
class Resource:
    def __enter__(self):
        print("Resource acquired")
        return self

    def use(self):
        print("Using resource")

    def __exit__(self, exc_type, exc_value, traceback):
        print("Resource released")
```

Usage:

```python
with Resource() as resource:
    resource.use()
```

Output:

```text
Resource acquired
Using resource
Resource released
```

The benefit is that whoever uses `Resource` doesn't have to remember cleanup.

---

# 9. Context Managers and Exceptions

This is where context managers become especially useful.

```python
class Resource:
    def __enter__(self):
        print("Resource acquired")
        return self

    def __exit__(self, exc_type, exc_value, traceback):
        print("Resource released")
```

Now:

```python
with Resource() as resource:
    print("Doing work")
    raise ValueError("Oops")
```

Even though an exception occurs:

```text
Resource acquired
Doing work
Resource released
```

`__exit__()` still runs.

The exception is then propagated unless `__exit__()` returns `True`.

---

# 10. `contextlib`

Python's `contextlib` module provides utilities for creating and working with context managers.

One of the most useful tools is:

```python
@contextmanager
```

It lets you create a context manager using a generator instead of writing a class.

Import it:

```python
from contextlib import contextmanager
```

Then:

```python
@contextmanager
def my_context():
    print("Start")

    yield

    print("Cleanup")
```

Use it:

```python
with my_context():
    print("Working")
```

Output:

```text
Start
Working
Cleanup
```

---

# 11. Understanding `@contextmanager`

This:

```python
@contextmanager
def my_context():
    print("Start")

    yield

    print("Cleanup")
```

has a very useful structure:

```text
Before yield
     ↓
__enter__
     ↓
with block
     ↓
yield
     ↓
After yield
     ↓
__exit__
```

Think of:

```python
yield
```

as the point where Python says:

> "Now execute the code inside the `with` block."

---

# 12. Returning a Value with `contextlib`

You can also provide an object to the `as` variable.

```python
from contextlib import contextmanager


@contextmanager
def database_connection():
    print("Connecting")

    connection = "Database Connection"

    try:
        yield connection
    finally:
        print("Disconnecting")
```

Use:

```python
with database_connection() as db:
    print(db)
```

Output:

```text
Connecting
Database Connection
Disconnecting
```

The value passed to:

```python
yield connection
```

becomes:

```python
as db
```

---

# 13. Why `try/finally` Is Important

When using `@contextmanager`, cleanup should generally go in `finally`.

```python
from contextlib import contextmanager


@contextmanager
def resource():
    print("Opening")

    try:
        yield
    finally:
        print("Closing")
```

Now even if:

```python
with resource():
    raise ValueError("Oops")
```

the cleanup happens:

```text
Opening
Closing
```

This is the same fundamental idea behind context managers:

```text
Acquire resource
       ↓
      try
       ↓
   use resource
       ↓
    finally
       ↓
  release resource
```

---

# 14. Real-World Context Managers

You've probably already used context managers without realizing it.

## Files

```python
with open("data.txt") as file:
    data = file.read()
```

Automatically closes the file.

---

## Database connections

Database libraries commonly provide patterns like:

```python
with connection:
    ...
```

so transactions/connections can be managed safely.

---

## Locks

Threading locks can be used as context managers:

```python
with lock:
    # protected code
    ...
```

The lock is acquired when entering and released when leaving.

---

## Temporary resources

Context managers are useful for:

* Temporary files
* Database transactions
* Network connections
* Locks
* Resource allocation
* Changing configuration temporarily

---

# 15. Context Manager vs `try/finally`

You can manually manage cleanup:

```python
resource = acquire()

try:
    use(resource)
finally:
    release(resource)
```

A context manager packages this pattern:

```python
with resource_manager():
    use(resource)
```

So the context manager provides a cleaner abstraction around:

```python
try:
    ...
finally:
    ...
```

---

# 16. Custom Context Manager: Timer

A simple example:

```python
from time import perf_counter


class Timer:
    def __enter__(self):
        self.start = perf_counter()
        return self

    def __exit__(self, exc_type, exc_value, traceback):
        self.end = perf_counter()
        self.elapsed = self.end - self.start
        print(f"Elapsed: {self.elapsed:.4f} seconds")
```

Use:

```python
with Timer():
    total = sum(range(1_000_000))
```

When the block finishes, the elapsed time is printed.

This pattern is useful for understanding how context managers can automatically perform an action **after** a block finishes.

---

# 17. Context Manager with `as`

You don't have to use `as`.

This:

```python
with MyContext():
    print("Hello")
```

is valid.

But if `__enter__()` returns something useful:

```python
with MyContext() as value:
    print(value)
```

The returned value becomes `value`.

Example:

```python
class UserSession:
    def __enter__(self):
        return "Kakade session"

    def __exit__(self, exc_type, exc_value, traceback):
        print("Session closed")
```

```python
with UserSession() as session:
    print(session)
```

Output:

```text
Kakade session
Session closed
```

---

# 18. Multiple Context Managers

You can use multiple resources:

```python
with open("input.txt") as source, open("output.txt", "w") as destination:
    data = source.read()
    destination.write(data)
```

Both resources are automatically cleaned up.

You can also nest them:

```python
with open("input.txt") as source:
    with open("output.txt", "w") as destination:
        data = source.read()
        destination.write(data)
```

The first form is usually cleaner.

---

# Custom Class vs `contextlib`

There are two common ways to create context managers.

### Class-based

```python
class MyContext:
    def __enter__(self):
        ...

    def __exit__(self, exc_type, exc_value, traceback):
        ...
```

Good when:

* You need to maintain state
* The context manager is complex
* You want reusable object behavior

### `contextlib`

```python
from contextlib import contextmanager

@contextmanager
def my_context():
    ...
    yield
    ...
```

Good when:

* The logic is simple
* You want concise code
* Setup/cleanup naturally fits around one `yield`

---

# Quick Revision

| Concept                           | Remember                                        |
| --------------------------------- | ----------------------------------------------- |
| **`with`**                        | Safely manage a resource/context                |
| **Context manager**               | Object that controls entering/leaving a context |
| **`__enter__`**                   | Runs when entering `with`                       |
| **`__exit__`**                    | Runs when leaving `with`                        |
| **`as`**                          | Receives the value returned by `__enter__()`    |
| **`exc_type`**                    | Exception class, if any                         |
| **`exc_value`**                   | Exception instance, if any                      |
| **`traceback`**                   | Exception traceback information                 |
| **`return True` from `__exit__`** | Suppresses exception                            |
| **`contextlib`**                  | Utilities for context managers                  |
| **`@contextmanager`**             | Create context manager using a generator        |
| **`yield`**                       | Separates setup from the `with` block/cleanup   |

---

# The Mental Model

### Class-based context manager

```text
with MyContext() as value:
            │
            ↓
      __enter__()
            │
            ↓
      value assigned
            │
            ↓
       with block
            │
            ↓
       __exit__()
            │
            ↓
         cleanup
```

### `contextlib`

```text
@contextmanager
      │
      ↓
   function
      │
   setup
      │
      ↓
    yield
      │
      ↓
  with block
      │
      ↓
   cleanup
```

### The key idea

> **A context manager guarantees that setup and cleanup happen around a block of code, even when an exception occurs.**

And the most important pattern to remember is:

```python
with resource() as value:
    # use resource
```

which conceptually means:

```text
__enter__()
    ↓
use resource
    ↓
__exit__()
```

This is why `with` is much more than just syntactic sugar—it gives you a reliable **resource-lifecycle pattern**.
