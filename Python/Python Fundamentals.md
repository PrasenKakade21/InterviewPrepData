# 1. Variables and Naming

A variable is a **name that refers to an object** in Python.

```python
name = "Prasen"
age = 25
salary = 75000.50
```

Python is **dynamically typed**, meaning you don't declare the type explicitly.

```python
x = 10
x = "hello"
x = [1, 2, 3]
```

The variable `x` can point to objects of different types during execution.

### Multiple assignment

```python
x, y, z = 10, 20, 30
```

You can also assign the same value to multiple variables:

```python
a = b = c = 100
```

### Swapping variables

Python makes swapping easy:

```python
a = 10
b = 20

a, b = b, a

print(a)  # 20
print(b)  # 10
```

### Naming rules

Valid:

```python
user_name = "Prasen"
age2 = 25
_private = True
```

Invalid:

```python
2users = 10       # cannot start with a number
user-name = "x"   # - is interpreted as subtraction
class = "Python"  # reserved keyword
```

Python convention is **snake_case**:

```python
first_name = "John"
total_price = 500
is_authenticated = True
```

Constants are conventionally written in uppercase:

```python
MAX_RETRIES = 3
API_TIMEOUT = 30
```

Python doesn't actually enforce constants.

### Important: variables are references

```python
a = [1, 2, 3]
b = a

b.append(4)

print(a)  # [1, 2, 3, 4]
```

`a` and `b` refer to the **same list object**.

You can verify this with:

```python
print(a is b)  # True
```

This becomes especially important when working with mutable objects.

---

# 2. Data Types

Python has several built-in data types. The ones you listed are:

| Type    | Example   | Description        |
| ------- | --------- | ------------------ |
| `int`   | `42`      | Integer            |
| `float` | `3.14`    | Decimal number     |
| `str`   | `"hello"` | Text               |
| `bool`  | `True`    | Boolean            |
| `None`  | `None`    | Absence of a value |

---

## `int`

Integers represent whole numbers.

```python
age = 25
temperature = -10
count = 0
```

Python integers can be arbitrarily large:

```python
big_number = 999999999999999999999999999999
```

Unlike languages such as C/Java, Python's normal `int` doesn't have a fixed 32-bit/64-bit limit.

---

## `float`

Floating-point numbers represent decimal values.

```python
price = 99.99
temperature = 36.5
```

Scientific notation is also supported:

```python
x = 1.5e3

print(x)  # 1500.0
```

### Floating-point precision

A common gotcha:

```python
print(0.1 + 0.2)
```

Output:

```text
0.30000000000000004
```

This happens because floating-point numbers use binary representation.

For financial calculations, you may want `Decimal`:

```python
from decimal import Decimal

price = Decimal("0.1")
tax = Decimal("0.2")

print(price + tax)
```

---

# 3. `str`

Strings represent text.

```python
name = "Prasen"
message = 'Hello World'
```

Both single and double quotes work:

```python
"hello"
'hello'
```

### Multiline strings

```python
message = """
Hello
World
Python
"""
```

### String indexing

```python
name = "Python"

print(name[0])   # P
print(name[2])   # t
print(name[-1])  # n
```

### String slicing

```python
name = "Python"

print(name[0:3])  # Pyt
print(name[:3])   # Pyt
print(name[3:])   # hon
print(name[::-1]) # nohtyP
```

Strings are **immutable**.

This doesn't work:

```python
name = "Python"
name[0] = "J"  # TypeError
```

Instead:

```python
name = "J" + name[1:]

print(name)  # Jython
```

---

# 4. `bool`

Boolean values are:

```python
True
False
```

Notice the capital letters.

```python
is_logged_in = True
is_admin = False
```

Booleans are heavily used in conditions:

```python
age = 25

if age >= 18:
    print("Adult")
```

### Booleans are actually related to integers

Python has:

```python
True == 1   # True
False == 0  # True
```

And:

```python
isinstance(True, int)  # True
```

This is because `bool` is a subclass of `int`.

Don't normally rely on this behavior in application code, but it's useful to know.

---

# 5. `None`

`None` represents **the absence of a value**.

```python
result = None
```

Commonly used when:

* a value doesn't exist yet
* a function doesn't return anything meaningful
* an optional value wasn't provided

Example:

```python
user = None

if user is None:
    print("No user found")
```

### Important

Use:

```python
value is None
```

rather than:

```python
value == None
```

So:

```python
if result is None:
    ...
```

is the idiomatic Python approach.

---

# 6. Type Conversion

Type conversion means converting one type into another.

### String → Integer

```python
age = "25"

age = int(age)

print(age)       # 25
print(type(age)) # <class 'int'>
```

### Integer → String

```python
age = 25

message = "I am " + str(age)

print(message)
```

### Integer → Float

```python
x = 10

y = float(x)

print(y)  # 10.0
```

### Float → Integer

```python
x = 10.9

print(int(x))  # 10
```

Important: `int()` **truncates** the decimal portion; it doesn't round.

```python
int(10.99)   # 10
int(-10.99)  # -10
```

If you want rounding:

```python
round(10.99)  # 11
```

### String → Float

```python
price = "99.99"

price = float(price)

print(price)  # 99.99
```

### Invalid conversion

```python
int("hello")
```

Raises:

```text
ValueError
```

Likewise:

```python
int("10.5")
```

also raises `ValueError`.

Instead:

```python
int(float("10.5"))
```

---

# 7. `type()`

`type()` tells you the type/class of an object.

```python
x = 10

print(type(x))
```

Output:

```text
<class 'int'>
```

Examples:

```python
type(10)        # int
type(10.5)      # float
type("hello")   # str
type(True)      # bool
type(None)      # NoneType
```

You can also technically use `type()` for exact type comparison:

```python
if type(value) == int:
    print("Integer")
```

But for most type checking, `isinstance()` is preferable.

---

# 8. `isinstance()`

`isinstance()` checks whether an object is an instance of a particular class/type.

```python
age = 25

print(isinstance(age, int))
```

Output:

```text
True
```

Example:

```python
value = "hello"

if isinstance(value, str):
    print("It's a string")
```

### Multiple types

You can pass a tuple of types:

```python
value = 10

if isinstance(value, (int, float)):
    print("It's a number")
```

This is useful when accepting multiple valid types.

### `type()` vs `isinstance()`

This distinction is important.

```python
class Animal:
    pass

class Dog(Animal):
    pass

dog = Dog()
```

With `type()`:

```python
type(dog) == Dog
# True

type(dog) == Animal
# False
```

With `isinstance()`:

```python
isinstance(dog, Dog)
# True

isinstance(dog, Animal)
# True
```

Why?

Because `Dog` inherits from `Animal`.

### Rule of thumb

Use:

```python
isinstance(value, SomeType)
```

when you care about inheritance/subclasses.

Use:

```python
type(value) is SomeType
```

when you specifically need the **exact type**.

---

# 9. Operators

Operators are symbols/keywords used to perform operations.

Python has several categories:

```text
Arithmetic
Comparison
Logical
Assignment
Membership
Identity
```

---

# 10. Arithmetic Operators

Used for mathematical operations.

| Operator | Meaning        | Example   |
| -------- | -------------- | --------- |
| `+`      | Addition       | `10 + 3`  |
| `-`      | Subtraction    | `10 - 3`  |
| `*`      | Multiplication | `10 * 3`  |
| `/`      | Division       | `10 / 3`  |
| `//`     | Floor division | `10 // 3` |
| `%`      | Modulo         | `10 % 3`  |
| `**`     | Exponent       | `10 ** 3` |

### Addition

```python
a = 10
b = 5

print(a + b)  # 15
```

### Division

```python
print(10 / 3)
```

Output:

```text
3.3333333333333335
```

`/` always produces a floating-point result.

```python
10 / 2
# 5.0
```

### Floor division

```python
print(10 // 3)
```

Output:

```text
3
```

But be careful with negatives:

```python
print(-10 // 3)
```

Output:

```text
-4
```

Floor division rounds **toward negative infinity**, not toward zero.

### Modulo

```python
print(10 % 3)
```

Output:

```text
1
```

Very useful for checking divisibility:

```python
if number % 2 == 0:
    print("Even")
```

### Exponentiation

```python
print(2 ** 3)
```

Output:

```text
8
```

---

# 11. Comparison Operators

Comparison operators produce a Boolean.

| Operator | Meaning               |
| -------- | --------------------- |
| `==`     | Equal                 |
| `!=`     | Not equal             |
| `>`      | Greater than          |
| `<`      | Less than             |
| `>=`     | Greater than or equal |
| `<=`     | Less than or equal    |

Example:

```python
age = 25

print(age == 25)  # True
print(age != 20)  # True
print(age > 18)   # True
print(age < 18)   # False
```

### `==` vs `=`

Very important:

```python
x = 10
```

means **assignment**.

```python
x == 10
```

means **comparison**.

---

## Chained comparisons

Python allows:

```python
age = 25

if 18 <= age < 60:
    print("Working age")
```

This is equivalent to:

```python
if age >= 18 and age < 60:
    print("Working age")
```

This is a nice Python-specific feature worth remembering.

---

# 12. Logical Operators

Python has:

```python
and
or
not
```

---

## `and`

Returns true when **both conditions are true**.

```python
age = 25
has_license = True

if age >= 18 and has_license:
    print("Can drive")
```

Both must be true.

### Truth table

```text
True  and True  → True
True  and False → False
False and True  → False
False and False → False
```

---

## `or`

Returns true when **at least one condition is true**.

```python
is_admin = False
is_owner = True

if is_admin or is_owner:
    print("Access granted")
```

Truth table:

```text
True  or True  → True
True  or False → True
False or True  → True
False or False → False
```

---

## `not`

Reverses a Boolean value.

```python
is_logged_in = True

print(not is_logged_in)
```

Output:

```text
False
```

Example:

```python
if not is_logged_in:
    print("Please log in")
```

---

# 13. Logical Operators Don't Always Return `True`/`False`

This is a particularly important intermediate-level Python concept.

Consider:

```python
x = 10
y = 20

result = x and y

print(result)
```

Output:

```text
20
```

Why?

Python's `and` and `or` return **one of their operands**, not necessarily a Boolean.

### `and`

```python
10 and 20
# 20
```

If the first value is falsy:

```python
0 and 20
# 0
```

### `or`

```python
10 or 20
# 10
```

If the first value is falsy:

```python
0 or 20
# 20
```

This is frequently used for default values:

```python
username = provided_username or "Guest"
```

If `provided_username` is empty/falsy, `"Guest"` is used.

---

# 14. Short-Circuit Evaluation

Python doesn't necessarily evaluate every condition.

```python
x = 0

if x != 0 and 10 / x > 2:
    print("Yes")
```

The second condition isn't evaluated because:

```python
x != 0
```

is already `False`.

Therefore Python doesn't attempt:

```python
10 / 0
```

Similarly:

```python
x = 10

if x == 10 or expensive_function():
    print("Yes")
```

`expensive_function()` isn't called because the first condition is already `True`.

This is called **short-circuit evaluation**.

---

# 15. Assignment Operators

Basic assignment:

```python
x = 10
```

Python also provides compound assignment operators.

| Operator | Example   | Equivalent   |
| -------- | --------- | ------------ |
| `=`      | `x = 5`   | Assignment   |
| `+=`     | `x += 5`  | `x = x + 5`  |
| `-=`     | `x -= 5`  | `x = x - 5`  |
| `*=`     | `x *= 5`  | `x = x * 5`  |
| `/=`     | `x /= 5`  | `x = x / 5`  |
| `//=`    | `x //= 5` | `x = x // 5` |
| `%=`     | `x %= 5`  | `x = x % 5`  |
| `**=`    | `x **= 5` | `x = x ** 5` |

Example:

```python
score = 100

score += 10
print(score)  # 110

score -= 20
print(score)  # 90
```

### Assignment expressions: `:=`

Python also has the **walrus operator**.

```python
if (length := len("Python")) > 5:
    print(length)
```

Here:

```python
length := len("Python")
```

assigns the result while also allowing it to be used in the expression.

You'll encounter this occasionally in modern Python code, but don't overuse it.

---

# 16. Membership Operators

Membership operators check whether something exists inside a collection.

Python provides:

```python
in
not in
```

### Lists

```python
numbers = [10, 20, 30]

print(20 in numbers)
# True

print(50 in numbers)
# False
```

### Strings

```python
name = "Prasen"

print("P" in name)
# True

print("x" in name)
# False
```

### `not in`

```python
users = ["john", "alice", "bob"]

if "admin" not in users:
    print("Admin doesn't exist")
```

### Dictionaries — important

With dictionaries, `in` checks **keys**, not values.

```python
user = {
    "name": "Prasen",
    "age": 25
}

print("name" in user)
# True

print("Prasen" in user)
# False
```

To check values:

```python
print("Prasen" in user.values())
# True
```

To check keys explicitly:

```python
print("name" in user.keys())
# True
```

---

# 17. Identity Operators

Identity operators are:

```python
is
is not
```

They check whether two variables refer to the **same object in memory**, rather than whether they have equal values.

### `==` vs `is`

```python
a = [1, 2, 3]
b = [1, 2, 3]

print(a == b)
# True

print(a is b)
# False
```

Why?

The lists contain the same values, so:

```python
a == b
```

is `True`.

But they're two different list objects:

```text
a → [1, 2, 3]

b → [1, 2, 3]
```

Therefore:

```python
a is b
```

is `False`.

---

## Same object

```python
a = [1, 2, 3]
b = a

print(a == b)
# True

print(a is b)
# True
```

Now both variables point to the same object.

---

# 18. The Most Important `is` Use Case: `None`

You'll frequently see:

```python
result = some_function()

if result is None:
    print("No result")
```

And:

```python
if result is not None:
    process(result)
```

Prefer this:

```python
value is None
```

over:

```python
value == None
```

---

# 19. Don't Use `is` for Normal Value Comparison

Avoid:

```python
if x is 10:
    ...
```

Use:

```python
if x == 10:
    ...
```

`is` is about **identity**.

`==` is about **equality**.

A useful mental model:

```text
==  → "Do these objects have equal values?"

is  → "Are these literally the same object?"
```

---

# 20. Truthy and Falsy Values

This ties several of these concepts together.

Python considers some values **falsy**:

```python
False
None
0
0.0
""
[]
{}
()
set()
```

Most other objects are truthy.

Example:

```python
username = ""

if username:
    print("Username provided")
else:
    print("Username missing")
```

Because an empty string is falsy, this prints:

```text
Username missing
```

You can explicitly convert something to a Boolean:

```python
bool(0)       # False
bool(1)       # True
bool("")      # False
bool("hello") # True
bool([])      # False
bool([1, 2])  # True
bool(None)    # False
```

This is extremely common in real Python code.

---

# 21. Putting Everything Together

Here's a realistic example using most of these concepts:

```python
username = "Prasen"
age = "25"
is_verified = True

# Type conversion
age = int(age)

# Type checking
if isinstance(age, int):
    print("Age is an integer")

# Comparison + logical operators
if age >= 18 and is_verified:
    print(f"{username} can access the platform")

# Membership
allowed_roles = ["admin", "developer", "user"]
role = "developer"

if role in allowed_roles:
    print("Valid role")

# None check
profile_picture = None

if profile_picture is None:
    print("Using default profile picture")

# Arithmetic
years_until_30 = 30 - age

print(f"{years_until_30} years until age 30")
```

---

# 22. Quick Revision Cheat Sheet

```text
VARIABLES
x = 10
name = "Prasen"

DATA TYPES
int    → 10
float  → 10.5
str    → "hello"
bool   → True / False
None   → absence of a value

TYPE CHECKING
type(x)
isinstance(x, int)

TYPE CONVERSION
int("10")
float("10.5")
str(10)
bool(value)

ARITHMETIC
+    addition
-    subtraction
*    multiplication
/    division
//   floor division
%    remainder
**   exponentiation

COMPARISON
==   equal
!=   not equal
>    greater
<    smaller
>=   greater/equal
<=   smaller/equal

LOGICAL
and
or
not

ASSIGNMENT
=
+=
-=
*=
/=
%=
//=
**=

MEMBERSHIP
x in collection
x not in collection

IDENTITY
x is y
x is not y

IMPORTANT DISTINCTION
== → equality
is → identity

COMMON NONE CHECK
value is None

TRUTHY/FALSY
False
None
0
0.0
""
[]
{}
()
set()
```