
A **set** is an unordered collection of **unique elements**.

```python
numbers = {1, 2, 3, 4}

print(numbers)
# {1, 2, 3, 4}
```

The three main properties to remember are:

* **Unique** → duplicates are automatically removed
* **Unordered** → you don't access elements using indexes
* **Mutable** → you can add and remove elements

Sets are especially useful for **membership checks** and **comparing collections**.

---

#  Creating Sets

### Basic set

```python
numbers = {10, 20, 30, 40}

print(numbers)
# {10, 20, 30, 40}
```

### Duplicates are automatically removed

```python
numbers = {1, 2, 2, 3, 3, 3}

print(numbers)
# {1, 2, 3}
```

This makes sets useful for removing duplicates:

```python
numbers = [1, 2, 2, 3, 3, 4]

unique_numbers = set(numbers)

print(unique_numbers)
# {1, 2, 3, 4}
```

You can convert it back to a list:

```python
unique_numbers = list(set(numbers))
```

Just remember that the ordering should not be relied upon.

---

## Creating a set from another iterable

```python
letters = set("hello")

print(letters)
# {'h', 'e', 'l', 'o'}
```

Notice that there is only one `"l"` even though `"hello"` contains two.

You can also create a set from a list:

```python
numbers = set([1, 2, 3, 4])
```

or a tuple:

```python
numbers = set((1, 2, 3, 4))
```

---

## Empty set

This is an important Python gotcha.

```python
x = {}
```

This creates an **empty dictionary**, not a set.

To create an empty set:

```python
x = set()

print(type(x))
# <class 'set'>
```

---

#  Adding and Removing Elements

Sets are mutable, so you can modify them.

## `add()`

Add one element:

```python
languages = {"Python", "JavaScript"}

languages.add("Go")

print(languages)
# {'Python', 'JavaScript', 'Go'}
```

If the element already exists, nothing happens:

```python
languages.add("Python")

print(languages)
# {'Python', 'JavaScript', 'Go'}
```

No duplicate is created.

---

## `update()`

Use `update()` to add multiple elements.

```python
languages = {"Python", "JavaScript"}

languages.update(["Go", "Rust", "Java"])

print(languages)
```

You can pass any iterable:

```python
languages.update(("C", "C++"))
```

or:

```python
languages.update({"Ruby", "PHP"})
```

---

## `remove()`

Remove an element:

```python
languages = {"Python", "JavaScript", "Go"}

languages.remove("Go")

print(languages)
# {'Python', 'JavaScript'}
```

But if the element doesn't exist:

```python
languages.remove("Rust")
```

Python raises:

```text
KeyError
```

---

## `discard()`

`discard()` also removes an element, but doesn't raise an error if it doesn't exist.

```python
languages = {"Python", "JavaScript", "Go"}

languages.discard("Rust")

print(languages)
# {'Python', 'JavaScript', 'Go'}
```

This distinction is useful:

| Method       | Element exists | Element doesn't exist |
| ------------ | -------------- | --------------------- |
| `remove(x)`  | Removes it     | `KeyError`            |
| `discard(x)` | Removes it     | Nothing happens       |

---

## `pop()`

`pop()` removes and returns **an arbitrary element**.

```python
numbers = {10, 20, 30, 40}

value = numbers.pop()

print(value)
print(numbers)
```

Don't assume which element will be removed.

Unlike lists:

```python
numbers.pop(0)
```

doesn't work because sets don't have indexes.

---

## `clear()`

Remove everything:

```python
numbers = {1, 2, 3}

numbers.clear()

print(numbers)
# set()
```

---

#  Membership

One of the biggest reasons to use a set is **fast membership checking**.

Use `in`:

```python
languages = {"Python", "JavaScript", "Go"}

print("Python" in languages)
# True

print("Java" in languages)
# False
```

You can also use `not in`:

```python
if "Rust" not in languages:
    print("Rust is not supported")
```

---

## Why are sets good for membership?

Imagine you have thousands of usernames:

```python
usernames = {
    "prasen",
    "rahul",
    "amit",
    # thousands more...
}

if "prasen" in usernames:
    print("Username exists")
```

Sets are implemented using a **hash table**, so membership checks are typically **O(1)** on average.

Compare that with a list:

```python
usernames = ["prasen", "rahul", "amit"]

if "prasen" in usernames:
    ...
```

A list may need to check elements one by one, making membership **O(n)** in the worst case.

This is an important practical reason to choose a set.

---

#  Union

**Union** combines the elements of two sets.

```python
python_devs = {"Prasen", "Rahul", "Amit"}
js_devs = {"Rahul", "Amit", "Sneha"}

all_devs = python_devs | js_devs

print(all_devs)
```

Result:

```text
{'Prasen', 'Rahul', 'Amit', 'Sneha'}
```

Duplicates are automatically removed.

You can also use `.union()`:

```python
all_devs = python_devs.union(js_devs)
```

Both mean the same thing:

```python
A | B
```

```python
A.union(B)
```

### Visual idea

```text
A = {1, 2, 3}
B = {3, 4, 5}

A ∪ B = {1, 2, 3, 4, 5}
```

---

#  Intersection

**Intersection** gives you elements that exist in **both sets**.

```python
python_devs = {"Prasen", "Rahul", "Amit"}
js_devs = {"Rahul", "Amit", "Sneha"}

common = python_devs & js_devs

print(common)
# {'Rahul', 'Amit'}
```

Or:

```python
common = python_devs.intersection(js_devs)
```

### Visual idea

```text
A = {1, 2, 3}
B = {2, 3, 4}

A ∩ B = {2, 3}
```

A very practical example:

```python
users_who_bought = {"A", "B", "C", "D"}
users_who_logged_in = {"B", "C", "E"}

active_customers = users_who_bought & users_who_logged_in

print(active_customers)
# {'B', 'C'}
```

These are users who **both purchased and logged in**.

---

#  Difference

Difference gives you elements that are in the **first set but not in the second**.

```python
python_devs = {"Prasen", "Rahul", "Amit"}
js_devs = {"Rahul", "Amit", "Sneha"}

python_only = python_devs - js_devs

print(python_only)
# {'Prasen'}
```

Think:

```text
A - B
```

means:

> "What's in A that isn't in B?"

### Example

```python
all_users = {"A", "B", "C", "D", "E"}
premium_users = {"A", "C"}

free_users = all_users - premium_users

print(free_users)
# {'B', 'D', 'E'}
```

---

## Difference is directional

This is important.

```python
A = {1, 2, 3}
B = {2, 3, 4}

print(A - B)
# {1}

print(B - A)
# {4}
```

They're not the same.

---

#  Symmetric Difference

Symmetric difference gives you elements that exist in **either set, but NOT in both**.

```python
A = {1, 2, 3}
B = {2, 3, 4}

result = A ^ B

print(result)
# {1, 4}
```

The common elements `2` and `3` are removed.

### Visual idea

```text
A = {1, 2, 3}
B = {2, 3, 4}

A ^ B = {1, 4}
```

Or using the method:

```python
result = A.symmetric_difference(B)
```

---

# Understanding All Four Operations

This is worth memorizing:

```text
A = {1, 2, 3}
B = {2, 3, 4}
```

### Union

Everything from both:

```python
A | B
# {1, 2, 3, 4}
```

### Intersection

Only common elements:

```python
A & B
# {2, 3}
```

### Difference

A but not B:

```python
A - B
# {1}
```

### Symmetric difference

Everything except common elements:

```python
A ^ B
# {1, 4}
```

A simple way to remember:

```text
       A              B

    1  2  3        2  3  4

Union:              1 2 3 4
Intersection:         2 3
A - B:                1
A ^ B:              1   4
```

---

# Practical Example

Imagine you're building a permission system.

```python
admin_permissions = {
    "read",
    "write",
    "delete",
    "manage_users"
}

editor_permissions = {
    "read",
    "write"
}
```

### What permissions does an admin have that an editor doesn't?

```python
extra_permissions = admin_permissions - editor_permissions

print(extra_permissions)
# {'delete', 'manage_users'}
```

### Which permissions do they share?

```python
common_permissions = admin_permissions & editor_permissions

print(common_permissions)
# {'read', 'write'}
```

### What permissions are available to either role?

```python
all_permissions = admin_permissions | editor_permissions
```

---

# Sets vs Lists vs Tuples

| Feature                 | List                         | Tuple            | Set                       |
| ----------------------- | ---------------------------- | ---------------- | ------------------------- |
| Ordered                 | Yes                          | Yes              | No*                       |
| Mutable                 | Yes                          | No               | Yes                       |
| Duplicates              | Yes                          | Yes              | No                        |
| Indexing                | Yes                          | Yes              | No                        |
| Slicing                 | Yes                          | Yes              | No                        |
| Fast membership         | No                           | No               | **Yes**                   |
| Mathematical operations | No                           | No               | **Yes**                   |
| Main use                | Collection of changing items | Fixed collection | Unique items / membership |

* Modern Python sets preserve certain iteration behavior as an implementation detail, but you should **not treat a set as an indexed or sequence-like ordered collection**.

---

# A Common Real-World Pattern

Suppose you receive duplicate tags from an API:

```python
tags = [
    "python",
    "backend",
    "python",
    "api",
    "backend",
    "fastapi"
]

unique_tags = set(tags)

print(unique_tags)
# {'python', 'backend', 'api', 'fastapi'}
```

Then you can quickly check:

```python
if "python" in unique_tags:
    print("Python is included")
```

You can also compare tags between two projects:

```python
project_a = {"python", "fastapi", "postgresql", "docker"}
project_b = {"python", "django", "postgresql", "redis"}

print(project_a & project_b)
# {'python', 'postgresql'}

print(project_a - project_b)
# {'fastapi', 'docker'}

print(project_a ^ project_b)
# {'fastapi', 'docker', 'django', 'redis'}
```

This kind of set operation becomes very useful in **data processing, APIs, permissions, filtering, deduplication, and automation scripts**.

---

## Key Things to Remember

```python
# Create
numbers = {1, 2, 3}

# Add
numbers.add(4)

# Remove
numbers.remove(4)

# Safe remove
numbers.discard(4)

# Membership
3 in numbers

# Union
A | B

# Intersection
A & B

# Difference
A - B

# Symmetric difference
A ^ B
```

The most important mental model is:

> **Lists/Tuples → think sequence. Sets → think collection of unique things and relationships between collections.**
