# Python Modules & Packages

Modules and packages are how Python lets you **organize code into reusable files and folders**. Once your programs get bigger, you don't want everything inside one `.py` file.

---

##  `import`

`import` is used to bring a **module** into your current Python file.

For example, Python has a built-in `math` module:

```python
import math

print(math.sqrt(25))
print(math.pi)
```

Output:

```text
5.0
3.141592653589793
```

The important thing is that you access things through the module name:

```python
math.sqrt()
math.pi
```

### Import with an alias

You can give a module a shorter name:

```python
import math as m

print(m.sqrt(25))
```

Another common example:

```python
import datetime as dt

print(dt.datetime.now())
```

### Multiple imports

```python
import math
import random
import os
```

---

#  `from ... import`

Instead of importing the entire module namespace, you can import specific things.

```python
from math import sqrt

print(sqrt(25))
```

Now you don't need:

```python
math.sqrt(25)
```

You can import multiple things:

```python
from math import sqrt, pi

print(sqrt(25))
print(pi)
```

### Alias individual imports

```python
from math import sqrt as square_root

print(square_root(25))
```

### `from module import *`

You may see:

```python
from math import *
```

This imports everything from the module.

**Generally avoid this.**

Why?

```python
from math import *
from another_module import *

sqrt(25)
```

Now it becomes difficult to know where names came from, and names can accidentally overwrite each other.

Prefer:

```python
import math

math.sqrt(25)
```

or:

```python
from math import sqrt

sqrt(25)
```

---

#  Modules

A **module is simply a Python file (`.py`)** containing code that can be reused.

Suppose you have:

```text
project/
├── main.py
└── calculator.py
```

### `calculator.py`

```python
def add(a, b):
    return a + b


def subtract(a, b):
    return a - b
```

### `main.py`

```python
import calculator

print(calculator.add(10, 5))
print(calculator.subtract(10, 5))
```

Output:

```text
15
5
```

You created your own module.

---

#  Packages

A **package is a directory containing related Python modules**.

For example:

```text
project/
├── main.py
└── utils/
    ├── calculator.py
    └── string_utils.py
```

You can import:

```python
from utils import calculator

print(calculator.add(10, 5))
```

Or:

```python
from utils.calculator import add

print(add(10, 5))
```

Think of it like:

```text
Module  → file
Package → folder of modules
```

---

#  `__init__.py`

Traditionally, `__init__.py` is used to mark a directory as a Python package.

Example:

```text
project/
├── main.py
└── utils/
    ├── __init__.py
    ├── calculator.py
    └── string_utils.py
```

The file can be empty:

```python
# utils/__init__.py
```

Or it can contain package initialization code.

For example:

```python
# utils/__init__.py

from .calculator import add
```

Then:

```python
from utils import add

print(add(10, 20))
```

### Important modern Python note

Python 3 also supports **namespace packages**, where `__init__.py` isn't necessarily required.

But for learning and many normal projects, you'll still frequently see:

```text
package/
└── __init__.py
```

It's also useful when you want to explicitly control package exports or initialization behavior.

---

#  `__name__`

Every Python module has a special variable called:

```python
__name__
```

Its value depends on **how the file is being used**.

Suppose:

```python
# calculator.py

print(__name__)
```

If you directly run:

```bash
python calculator.py
```

you get:

```text
__main__
```

But if:

```python
# main.py

import calculator
```

then `calculator.py` prints:

```text
calculator
```

So:

```text
Directly executed:
__name__ == "__main__"

Imported:
__name__ == module_name
```

This distinction is extremely important.

---

#  `if __name__ == '__main__'`

This is one of the most important Python patterns.

Suppose:

```python
# calculator.py

def add(a, b):
    return a + b


print("Calculator loaded")
```

If another file does:

```python
import calculator
```

the `print()` runs during import.

That's sometimes undesirable.

Instead:

```python
# calculator.py

def add(a, b):
    return a + b


if __name__ == "__main__":
    print(add(10, 20))
```

Now:

```bash
python calculator.py
```

runs:

```text
30
```

But:

```python
import calculator
```

doesn't execute the code inside the `if`.

### Why?

When directly executed:

```python
__name__ == "__main__"
```

When imported:

```python
__name__ == "calculator"
```

So:

```python
if __name__ == "__main__":
```

basically means:

> "Only run this code when this file is executed directly, not when it is imported."

### Common use

```python
def main():
    print("Application started")


if __name__ == "__main__":
    main()
```

You'll see this pattern everywhere in Python.

---

#  Creating Your Own Modules

Suppose you're building an application:

```text
my_app/
├── main.py
├── database.py
├── authentication.py
└── utils.py
```

### `database.py`

```python
def connect():
    print("Connected to database")


def disconnect():
    print("Disconnected")
```

### `authentication.py`

```python
def login(username, password):
    return username == "admin" and password == "1234"
```

### `main.py`

```python
from database import connect
from authentication import login

connect()

if login("admin", "1234"):
    print("Login successful")
```

This is much cleaner than putting everything into `main.py`.

---

#  Package Structure

As your application grows, you can organize modules into packages.

For example:

```text
my_app/
│
├── main.py
│
├── database/
│   ├── __init__.py
│   ├── connection.py
│   └── queries.py
│
├── auth/
│   ├── __init__.py
│   ├── login.py
│   └── permissions.py
│
└── utils/
    ├── __init__.py
    ├── validators.py
    └── helpers.py
```

Now you have logical separation:

```text
database → database-related code
auth     → authentication-related code
utils    → reusable utilities
```

You can import:

```python
from auth.login import login
from database.connection import connect
from utils.validators import validate_email
```

This is the same basic idea you'll encounter in **Flask, FastAPI, Django, CLI applications, automation scripts, and larger Python projects**.

---

#  Absolute Imports

An **absolute import** starts from the project's/package's top-level location.

Example:

```text
my_app/
├── main.py
└── utils/
    ├── __init__.py
    └── validators.py
```

Inside `main.py`:

```python
from utils.validators import validate_email
```

That's an absolute import.

Another example:

```text
my_app/
├── main.py
└── services/
    ├── __init__.py
    └── user_service.py
```

```python
from services.user_service import create_user
```

Absolute imports are generally easier to understand because you can immediately see where the module comes from.

---

#  Relative Imports

Relative imports refer to modules **relative to the current package**.

Suppose:

```text
my_app/
└── app/
    ├── __init__.py
    ├── routes.py
    └── services/
        ├── __init__.py
        └── user.py
```

Inside:

```text
app/services/user.py
```

you can import something from `app/routes.py` using:

```python
from ..routes import something
```

Here:

```text
.   → current package
..  → parent package
... → parent's parent
```

For example:

```python
from .helpers import format_user
```

means:

> Import `helpers` from the current package.

And:

```python
from ..database import connect
```

means:

> Go up one package and import `database`.

---

# Absolute vs Relative

Consider:

```text
project/
└── app/
    ├── __init__.py
    ├── routes.py
    └── services/
        ├── __init__.py
        └── user.py
```

Inside `user.py`:

### Absolute

```python
from app.routes import something
```

### Relative

```python
from ..routes import something
```

Both can refer to the same module.

A useful mental model:

```text
Absolute:
"Start from the top."

Relative:
"Start from where I am."
```

---

#  A Realistic Example

Here's a small application:

```text
shop/
│
├── main.py
│
└── app/
    ├── __init__.py
    │
    ├── models/
    │   ├── __init__.py
    │   └── product.py
    │
    ├── services/
    │   ├── __init__.py
    │   └── product_service.py
    │
    └── utils/
        ├── __init__.py
        └── validators.py
```

### `product.py`

```python
class Product:
    def __init__(self, name, price):
        self.name = name
        self.price = price
```

### `product_service.py`

```python
from ..models.product import Product


def create_product(name, price):
    return Product(name, price)
```

Notice:

```python
from ..models.product import Product
```

We're going up from:

```text
services/
```

to:

```text
app/
```

and then into:

```text
models/product.py
```

### `main.py`

```python
from app.services.product_service import create_product


product = create_product("Keyboard", 2500)

print(product.name)
print(product.price)
```

Output:

```text
Keyboard
2500
```

---

#  Module Search Path

When you write:

```python
import something
```

Python needs to figure out **where `something` exists**.

Python searches locations listed in:

```python
import sys

print(sys.path)
```

You'll typically see paths including:

```text
your project directory
Python standard library
site-packages
```

That's why installed packages such as:

```python
import requests
```

can be imported.

`requests` is installed somewhere accessible through Python's module search path.

---

#  `__pycache__`

You may notice:

```text
__pycache__/
```

after running Python code.

For example:

```text
project/
├── main.py
├── utils.py
└── __pycache__/
    └── utils.cpython-314.pyc
```

Python can compile modules into **bytecode** (`.pyc`) and store them in `__pycache__`.

You generally don't need to manually manage this directory.

It's usually excluded from Git:

```gitignore
__pycache__/
*.pyc
```

---

#  Common Mistakes

### Mistake 1 — Naming your file after a standard library module

Don't do:

```text
random.py
```

and then:

```python
import random
```

You can accidentally shadow Python's real `random` module.

Similarly avoid names such as:

```text
json.py
os.py
typing.py
email.py
```

for your own modules.

---

### Mistake 2 — Running a package module directly

Suppose:

```text
app/
├── __init__.py
├── main.py
└── utils.py
```

`main.py` contains:

```python
from .utils import something
```

If you run:

```bash
python app/main.py
```

you may get:

```text
ImportError: attempted relative import with no known parent package
```

Instead, from the project directory, you would typically run:

```bash
python -m app.main
```

This tells Python to execute `app.main` **as a module inside its package context**.

This becomes especially important when working with larger applications.

---

# Quick Mental Model

```text
.py file
   ↓
MODULE
   ↓
Reusable Python code


folder containing modules
   ↓
PACKAGE
   ↓
Organized collection of modules
```

And imports:

```python
import math
```

→ Import the module.

```python
from math import sqrt
```

→ Import something specific from a module.

```python
from app.services.users import create_user
```

→ Import something from a module inside a package.

```python
from .utils import helper
```

→ Relative import from the current package.

```python
from ..utils import helper
```

→ Relative import from the parent package.

And finally:

```python
if __name__ == "__main__":
    main()
```

→ **Run this code only when the file is executed directly.**

