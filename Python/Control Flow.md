
**Control flow** determines **which code runs, when it runs, and how many times it runs**.

The main things you'll use are:

```text
if / elif / else     → make decisions
for                  → repeat over a collection
while                → repeat while a condition is true
break                → stop a loop
continue             → skip current iteration
pass                 → do nothing
enumerate()          → get index + value
zip()                → iterate over multiple collections together
```

---

# 1. `if`

`if` executes code when a condition evaluates to `True`.

```python
age = 25

if age >= 18:
    print("Adult")
```

Output:

```text
Adult
```

The condition can contain comparisons:

```python
score = 85

if score >= 80:
    print("Excellent")
```

You can also combine conditions:

```python
age = 25
has_license = True

if age >= 18 and has_license:
    print("Can drive")
```

### Important: indentation

Python uses indentation to define the block.

```python
if age >= 18:
    print("Adult")
    print("Can vote")
```

Both `print()` statements belong to the `if`.

This would be different:

```python
if age >= 18:
    print("Adult")

print("This always runs")
```

---

# 2. `elif`

`elif` means **"else if"**.

Use it when you have multiple possible conditions.

```python
score = 75

if score >= 90:
    grade = "A"
elif score >= 80:
    grade = "B"
elif score >= 70:
    grade = "C"
else:
    grade = "F"

print(grade)
# C
```

Python checks from top to bottom.

Once a condition is `True`, the remaining conditions are skipped.

For example:

```python
score = 95

if score >= 90:
    print("A")
elif score >= 80:
    print("B")
```

Only:

```text
A
```

is printed.

---

# 3. `else`

`else` runs when **none of the previous conditions are true**.

```python
age = 15

if age >= 18:
    print("Adult")
else:
    print("Minor")
```

Output:

```text
Minor
```

You don't have to use `elif`.

```python
is_logged_in = False

if is_logged_in:
    print("Dashboard")
else:
    print("Login required")
```

---

# 4. Nested Conditions

A condition inside another condition is called a **nested condition**.

```python
age = 25
is_verified = True

if age >= 18:
    if is_verified:
        print("Access granted")
    else:
        print("Verification required")
else:
    print("Must be 18 or older")
```

Here:

```text
if age >= 18
    └── if is_verified
```

Nested conditions are useful, but excessive nesting can make code difficult to read.

Instead of:

```python
if user:
    if user.is_active:
        if user.has_permission:
            print("Allowed")
```

you can often combine conditions:

```python
if user and user.is_active and user.has_permission:
    print("Allowed")
```

Or use early returns in functions.

---

# 5. Truthy and Falsy Values

Python doesn't require conditions to literally be `True` or `False`.

Many values are automatically treated as either **truthy** or **falsy**.

### Common falsy values

```python
False
None
0
0.0
""
[]
()
{}
set()
```

Almost everything else is truthy.

For example:

```python
name = ""

if name:
    print("Name exists")
else:
    print("Name is empty")
```

Output:

```text
Name is empty
```

With a value:

```python
name = "Prasen"

if name:
    print("Name exists")
```

Output:

```text
Name exists
```

---

## Very common Python pattern

Instead of:

```python
if len(users) > 0:
    print("Users exist")
```

you can write:

```python
if users:
    print("Users exist")
```

And:

```python
if not users:
    print("No users")
```

This works because an empty list is falsy.

---

## `None` vs falsy values

Be careful when you specifically need to check for `None`.

Don't always do:

```python
if not value:
    ...
```

because that treats all of these the same:

```python
None
0
False
""
[]
```

If you specifically mean `None`, use:

```python
if value is None:
    print("No value")
```

This distinction becomes important when handling API data.

---

# 6. Ternary Expressions

A **ternary expression** is a compact way to write a simple `if/else`.

Normal:

```python
age = 25

if age >= 18:
    status = "Adult"
else:
    status = "Minor"
```

Ternary:

```python
age = 25

status = "Adult" if age >= 18 else "Minor"
```

Syntax:

```python
value_if_true if condition else value_if_false
```

Another example:

```python
score = 75

result = "Pass" if score >= 40 else "Fail"

print(result)
# Pass
```

### Don't overuse it

This is readable:

```python
message = "Active" if is_active else "Inactive"
```

But something like this quickly becomes difficult to read:

```python
result = "A" if score >= 90 else "B" if score >= 80 else "C" if score >= 70 else "F"
```

For multiple conditions, normal `if/elif/else` is usually better.

---

# 7. `for` Loops

A `for` loop iterates over an iterable.

```python
languages = ["Python", "JavaScript", "Go"]

for language in languages:
    print(language)
```

Output:

```text
Python
JavaScript
Go
```

The loop essentially says:

> For every element in `languages`, assign it to `language` and run this block.

---

## Looping through a string

Strings are iterable:

```python
for char in "Python":
    print(char)
```

Output:

```text
P
y
t
h
o
n
```

---

## Looping through a tuple

```python
numbers = (10, 20, 30)

for number in numbers:
    print(number)
```

---

## Looping through a set

```python
skills = {"Python", "React", "Docker"}

for skill in skills:
    print(skill)
```

Remember that sets don't provide sequence-style indexing.

---

## Looping through a dictionary

By default, you iterate over keys:

```python
user = {
    "name": "Prasen",
    "age": 25
}

for key in user:
    print(key)
```

For values:

```python
for value in user.values():
    print(value)
```

For both:

```python
for key, value in user.items():
    print(key, value)
```

---

# 8. `while` Loops

A `while` loop keeps running **while a condition is true**.

```python
count = 1

while count <= 5:
    print(count)
    count += 1
```

Output:

```text
1
2
3
4
5
```

The flow is:

```text
check condition
      ↓
   True?
   /   \
 yes    no
  ↓      ↓
execute  stop
  ↓
update
  ↓
check again
```

---

## Infinite loops

Be careful:

```python
count = 1

while count <= 5:
    print(count)
```

`count` never changes, so the condition remains true forever.

You need something that eventually makes the condition false:

```python
count += 1
```

---

## When should you use `while`?

Use `for` when you know you're iterating over something:

```python
for user in users:
    ...
```

Use `while` when the number of iterations depends on a condition:

```python
while not connected:
    try_to_connect()
```

---

# 9. `range()`

`range()` generates a sequence of numbers, commonly used with `for`.

```python
for i in range(5):
    print(i)
```

Output:

```text
0
1
2
3
4
```

Important:

> The stop value is **exclusive**.

So:

```python
range(5)
```

means:

```text
0 1 2 3 4
```

---

## `range(start, stop)`

```python
for i in range(2, 6):
    print(i)
```

Output:

```text
2
3
4
5
```

---

## `range(start, stop, step)`

```python
for i in range(0, 10, 2):
    print(i)
```

Output:

```text
0
2
4
6
8
```

You can use a negative step:

```python
for i in range(10, 0, -2):
    print(i)
```

Output:

```text
10
8
6
4
2
```

---

## `range()` is not a list

```python
numbers = range(5)

print(type(numbers))
# <class 'range'>
```

If you need an actual list:

```python
numbers = list(range(5))

print(numbers)
# [0, 1, 2, 3, 4]
```

---

# 10. `break`

`break` immediately **terminates the loop**.

```python
for number in range(1, 10):
    if number == 5:
        break

    print(number)
```

Output:

```text
1
2
3
4
```

When `number == 5`, `break` exits the loop completely.

---

## Practical example

Search for a user:

```python
users = ["Amit", "Rahul", "Prasen", "Sneha"]

for user in users:
    if user == "Prasen":
        print("User found")
        break
```

There's no reason to continue searching after finding the user.

---

# 11. `continue`

`continue` skips the **current iteration** and moves to the next one.

```python
for number in range(1, 6):
    if number == 3:
        continue

    print(number)
```

Output:

```text
1
2
4
5
```

Compare:

```text
break       → stop the entire loop

continue    → skip this iteration
```

---

## Practical example

Process only valid values:

```python
numbers = [10, -5, 20, -3, 30]

for number in numbers:
    if number < 0:
        continue

    print(number)
```

Output:

```text
10
20
30
```

---

# 12. `pass`

`pass` means:

> **Do nothing.**

It's essentially a placeholder.

```python
for number in range(5):
    if number == 3:
        pass

    print(number)
```

`pass` does **not** skip the iteration.

The output is still:

```text
0
1
2
3
4
```

Compare:

```python
if number == 3:
    pass
```

with:

```python
if number == 3:
    continue
```

`pass`:

```text
do nothing
↓
continue normally
```

`continue`:

```text
skip rest of current iteration
↓
go to next iteration
```

---

## Common use of `pass`

When you're designing something but haven't implemented it yet:

```python
def process_payment():
    pass
```

Or:

```python
class UserService:
    pass
```

It prevents Python from complaining about an empty block.

---

# 13. `enumerate()`

`enumerate()` lets you iterate over a collection while getting both the **index and value**.

Without `enumerate()`:

```python
languages = ["Python", "JavaScript", "Go"]

for i in range(len(languages)):
    print(i, languages[i])
```

With `enumerate()`:

```python
for index, language in enumerate(languages):
    print(index, language)
```

Output:

```text
0 Python
1 JavaScript
2 Go
```

This is much more Pythonic.

---

## Start from a different index

By default:

```python
enumerate(languages)
```

starts at `0`.

You can specify a starting value:

```python
for index, language in enumerate(languages, start=1):
    print(index, language)
```

Output:

```text
1 Python
2 JavaScript
3 Go
```

This is useful when displaying numbered lists.

---

# 14. `zip()`

`zip()` allows you to iterate over **multiple iterables simultaneously**.

Suppose:

```python
names = ["Prasen", "Rahul", "Amit"]
ages = [25, 28, 30]
```

You can do:

```python
for name, age in zip(names, ages):
    print(name, age)
```

Output:

```text
Prasen 25
Rahul 28
Amit 30
```

Conceptually:

```text
names       ages

Prasen  ←→  25
Rahul   ←→  28
Amit    ←→  30
```

---

## `zip()` with three lists

```python
names = ["Prasen", "Rahul", "Amit"]
ages = [25, 28, 30]
roles = ["Developer", "Designer", "Manager"]

for name, age, role in zip(names, ages, roles):
    print(name, age, role)
```

Output:

```text
Prasen 25 Developer
Rahul 28 Designer
Amit 30 Manager
```

---

## `zip()` stops at the shortest iterable

```python
names = ["Prasen", "Rahul", "Amit"]
ages = [25, 28]

for name, age in zip(names, ages):
    print(name, age)
```

Output:

```text
Prasen 25
Rahul 28
```

`"Amit"` has no corresponding age, so it's not included.

---

# `enumerate()` + `zip()`

You can combine them.

```python
names = ["Prasen", "Rahul", "Amit"]
roles = ["Developer", "Designer", "Manager"]

for index, (name, role) in enumerate(zip(names, roles), start=1):
    print(index, name, role)
```

Output:

```text
1 Prasen Developer
2 Rahul Designer
3 Amit Manager
```

This is a good example of Python's unpacking features working together.

---

# Practical Example: Processing API Data

Suppose you receive:

```python
users = [
    {"name": "Prasen", "age": 25, "active": True},
    {"name": "Rahul", "age": 17, "active": True},
    {"name": "Amit", "age": 30, "active": False},
]
```

You could process active adult users:

```python
for user in users:
    if not user["active"]:
        continue

    if user["age"] < 18:
        continue

    print(f"{user['name']} can access the system")
```

Output:

```text
Prasen can access the system
```

Here you're combining:

* `for`
* `if`
* `continue`
* dictionary access
* truthy/falsy logic

---

# Practical Example: Retry Logic

`while` loops are useful for retry behavior:

```python
attempts = 0
max_attempts = 3

while attempts < max_attempts:
    attempts += 1

    success = make_request()

    if success:
        print("Request successful")
        break

    print("Request failed")
```

The loop stops early if the request succeeds.

This type of pattern is particularly relevant to **API calls, automation, network requests, and backend development**.

---

# `for` vs `while`

A useful rule:

### Use `for`

When you're iterating over a known collection:

```python
for user in users:
    process(user)
```

or a known range:

```python
for i in range(10):
    print(i)
```

### Use `while`

When you want to continue until some condition changes:

```python
while not job_finished:
    check_status()
```

---

# Quick Mental Model

```text
if
↓
Should I execute this?


elif
↓
What about this other condition?


else
↓
None of the conditions worked.


for
↓
Repeat for every item.


while
↓
Repeat while condition is true.


break
↓
STOP the loop completely.


continue
↓
Skip this iteration.


pass
↓
Do nothing.


range()
↓
Generate a sequence of numbers.


enumerate()
↓
Give me index + value.


zip()
↓
Give me corresponding values from multiple collections.
```

---

# Must-Know Examples

### Conditional

```python
if age >= 18:
    status = "adult"
elif age >= 13:
    status = "teen"
else:
    status = "child"
```

### Ternary

```python
status = "adult" if age >= 18 else "minor"
```

### Truthy/falsy

```python
if users:
    print("Users exist")
```

### For loop

```python
for user in users:
    print(user)
```

### Range

```python
for i in range(5):
    print(i)
```

### While

```python
while count < 5:
    count += 1
```

### Break

```python
for item in items:
    if item == target:
        break
```

### Continue

```python
for item in items:
    if not item:
        continue

    process(item)
```

### Enumerate

```python
for index, item in enumerate(items):
    print(index, item)
```

### Zip

```python
for name, age in zip(names, ages):
    print(name, age)
```

