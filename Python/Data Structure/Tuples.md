
A **tuple** is an ordered collection of values, similar to a list, but **immutable** — once created, you cannot change, add, or remove its elements.

```python
user = ("Prasen", 25, "Developer")

print(user)
# ('Prasen', 25, 'Developer')
```

Tuples are commonly used when you have a collection of values that should remain unchanged.

---

## 1. Creating Tuples

### Basic tuple

```python
numbers = (10, 20, 30, 40)

print(numbers)
# (10, 20, 30, 40)
```

Tuples can contain different data types:

```python
user = ("Prasen", 25, True, 10.5)

print(user)
# ('Prasen', 25, True, 10.5)
```

### Parentheses are optional

Python identifies a tuple primarily by the **comma**, not the parentheses.

```python
numbers = 10, 20, 30

print(type(numbers))
# <class 'tuple'>
```

This is also a tuple:

```python
numbers = (10, 20, 30)
```

### Single-element tuple

This is an important gotcha:

```python
x = (10)

print(type(x))
# <class 'int'>
```

You need a comma:

```python
x = (10,)

print(type(x))
# <class 'tuple'>
```

The same applies without parentheses:

```python
x = 10,

print(type(x))
# <class 'tuple'>
```

### Empty tuple

```python
empty = ()

print(type(empty))
# <class 'tuple'>
```

---

## 2. Indexing and Slicing

Tuples support indexing and slicing just like lists and strings.

```python
languages = ("Python", "JavaScript", "TypeScript", "Go")

print(languages[0])
# Python

print(languages[2])
# TypeScript
```

### Negative indexing

```python
print(languages[-1])
# Go

print(languages[-2])
# TypeScript
```

The indexes are:

```text
             0           1             2        3
          Python    JavaScript    TypeScript    Go
            -4          -3           -2        -1
```

### Slicing

```python
languages = ("Python", "JavaScript", "TypeScript", "Go")

print(languages[1:3])
# ('JavaScript', 'TypeScript')
```

The same slicing rules as lists apply:

```python
tuple[start:stop:step]
```

For example:

```python
numbers = (0, 1, 2, 3, 4, 5, 6)

print(numbers[:4])
# (0, 1, 2, 3)

print(numbers[2:])
# (2, 3, 4, 5, 6)

print(numbers[::2])
# (0, 2, 4, 6)

print(numbers[::-1])
# (6, 5, 4, 3, 2, 1, 0)
```

**Important:** slicing creates a new tuple.

```python
numbers = (1, 2, 3, 4)

result = numbers[1:3]

print(result)
# (2, 3)
```

---

# 3. Tuple Unpacking

**Tuple unpacking** means assigning the elements of a tuple to multiple variables.

```python
user = ("Prasen", 25, "Developer")

name, age, role = user

print(name)
# Prasen

print(age)
# 25

print(role)
# Developer
```

Python matches variables with tuple elements by position.

```text
user = ("Prasen", 25, "Developer")
        ↓         ↓          ↓
      name       age        role
```

### The number of variables must normally match

This works:

```python
data = (10, 20, 30)

a, b, c = data
```

This doesn't:

```python
data = (10, 20, 30)

a, b = data
```

You get:

```text
ValueError: too many values to unpack
```

---

## Extended unpacking

You can use `*` to collect multiple elements.

```python
numbers = (1, 2, 3, 4, 5)

first, *middle, last = numbers

print(first)
# 1

print(middle)
# [2, 3, 4]

print(last)
# 5
```

Notice that `middle` becomes a **list**, not a tuple.

You can also do:

```python
first, *rest = numbers

print(first)
# 1

print(rest)
# [2, 3, 4, 5]
```

This is useful when you don't know how many values are in the middle.

---

## Tuple unpacking in loops

This is extremely common in real Python code.

```python
users = [
    ("Prasen", 25),
    ("Rahul", 28),
    ("Amit", 30)
]

for name, age in users:
    print(name, age)
```

Output:

```text
Prasen 25
Rahul 28
Amit 30
```

Instead of:

```python
for user in users:
    print(user[0], user[1])
```

Unpacking makes the code cleaner.

---

## Swapping variables

Tuple unpacking is also used for swapping:

```python
a = 10
b = 20

a, b = b, a

print(a)
# 20

print(b)
# 10
```

Conceptually, Python creates a tuple of the right-hand side:

```python
a, b = (b, a)
```

This is one of Python's most useful features.

---

# 4. Immutability

The biggest difference between tuples and lists is that **tuples cannot be modified after creation**.

```python
numbers = (10, 20, 30)

numbers[0] = 100
```

This produces:

```text
TypeError: 'tuple' object does not support item assignment
```

You also cannot:

```python
numbers.append(40)
```

or:

```python
numbers.remove(20)
```

or:

```python
del numbers[0]
```

Because the tuple itself cannot be changed.

### But there is an important nuance

Tuple immutability applies to the **tuple's references**, not necessarily to objects stored inside it.

For example:

```python
data = ([1, 2], [3, 4])

data[0].append(5)

print(data)
# ([1, 2, 5], [3, 4])
```

Why did this work?

The tuple itself didn't change.

The tuple still contains references to the same two lists:

```text
tuple
 ├──→ [1, 2, 5]
 └──→ [3, 4]
```

You cannot replace the first element:

```python
data[0] = [100, 200]
```

That gives:

```text
TypeError
```

So remember:

> **A tuple is immutable, but objects contained inside it may themselves be mutable.**

This distinction becomes important when working with nested data structures.

---

# 5. Tuple vs List

Both are ordered collections, but they serve different purposes.

| Feature                 | Tuple       | List            |
| ----------------------- | ----------- | --------------- |
| Syntax                  | `(1, 2, 3)` | `[1, 2, 3]`     |
| Ordered                 | Yes         | Yes             |
| Mutable                 | No          | Yes             |
| `append()`              | No          | Yes             |
| `remove()`              | No          | Yes             |
| `pop()`                 | No          | Yes             |
| Indexing                | Yes         | Yes             |
| Slicing                 | Yes         | Yes             |
| Can contain mixed types | Yes         | Yes             |
| Hashable*               | Sometimes   | No              |
| Usually used for        | Fixed data  | Changeable data |

* A tuple is hashable only when all of its elements are hashable.

---

## When should you use a tuple?

Use a tuple when the collection represents a **fixed group of values**.

For example, coordinates:

```python
location = (19.0760, 72.8777)
```

You generally don't want code accidentally doing:

```python
location[0] = 20
```

Another example:

```python
rgb = (255, 128, 0)
```

Or configuration-like data:

```python
database_config = ("localhost", 5432, "my_database")
```

---

## When should you use a list?

Use a list when the collection is expected to change.

```python
tasks = ["Learn Python", "Build API"]

tasks.append("Deploy app")

print(tasks)
```

Output:

```python
["Learn Python", "Build API", "Deploy app"]
```

The list represents a collection whose contents can change.

---

# Tuples and Functions

Tuples are particularly useful with functions.

A function can return multiple values:

```python
def get_user():
    return "Prasen", 25, "Developer"
```

Technically, Python returns a tuple:

```python
user = get_user()

print(user)
# ('Prasen', 25, 'Developer')

print(type(user))
# <class 'tuple'>
```

You can immediately unpack it:

```python
name, age, role = get_user()

print(name)
# Prasen

print(age)
# 25

print(role)
# Developer
```

This pattern is extremely common in Python.

---

# Tuples as Dictionary Keys

A tuple can be used as a dictionary key if all its elements are hashable.

For example, representing a coordinate:

```python
locations = {
    (19.0760, 72.8777): "Mumbai",
    (18.5204, 73.8567): "Pune"
}

print(locations[(19.0760, 72.8777)])
# Mumbai
```

You **cannot** use a list as a dictionary key:

```python
locations = {
    [19.0760, 72.8777]: "Mumbai"
}
```

because lists are mutable and therefore unhashable.

---

# Useful Tuple Operations

Tuples have fewer methods than lists because they're immutable.

### `count()`

Count how many times a value appears:

```python
numbers = (1, 2, 2, 3, 2, 4)

print(numbers.count(2))
# 3
```

### `index()`

Find the first position of a value:

```python
numbers = (10, 20, 30, 20)

print(numbers.index(20))
# 1
```

### `in`

Check membership:

```python
languages = ("Python", "JavaScript", "Go")

print("Python" in languages)
# True

print("Java" in languages)
# False
```

### `len()`

```python
numbers = (10, 20, 30, 40)

print(len(numbers))
# 4
```

---

# Intermediate Example

Imagine you're processing users from an API:

```python
users = [
    (101, "Prasen", "Developer"),
    (102, "Rahul", "Designer"),
    (103, "Amit", "Manager")
]

for user_id, name, role in users:
    print(f"{user_id}: {name} - {role}")
```

Output:

```text
101: Prasen - Developer
102: Rahul - Designer
103: Amit - Manager
```

Here tuples make sense because each user's record has a **fixed structure**:

```text
(user_id, name, role)
```

If you needed to constantly add/remove fields, a dictionary or another structure would probably be more appropriate.

---

# Key Things to Remember

```text
Tuple = ordered + immutable
```

### Creating

```python
x = (1, 2, 3)
```

### Single element

```python
x = (1,)
```

### Indexing

```python
x[0]
x[-1]
```

### Slicing

```python
x[1:3]
x[::-1]
```

### Unpacking

```python
a, b, c = (1, 2, 3)
```

### Extended unpacking

```python
a, *middle, c = (1, 2, 3, 4, 5)
```

### Immutable

```python
x[0] = 100  # TypeError
```

### Dictionary key

```python
data = {(10, 20): "point"}
```

### Main rule

**Use a tuple when the data represents a fixed collection; use a list when the collection needs to change.**
