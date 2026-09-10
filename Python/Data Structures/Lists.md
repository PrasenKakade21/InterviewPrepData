
A **list** is an ordered, mutable collection that can contain multiple values.

```python
users = ["Prasen", "John", "Alice"]
numbers = [10, 20, 30, 40]
mixed = [10, "hello", True, None]
```

Unlike strings, lists are **mutable**, meaning you can change their contents after creation.

---

#  Indexing

Lists use **zero-based indexing**.

```python
languages = ["Python", "JavaScript", "TypeScript", "Go"]
```

```text
Python      JavaScript      TypeScript      Go
   0             1              2           3
  -4            -3             -2          -1
```

```python
print(languages[0])   # Python
print(languages[2])   # TypeScript

print(languages[-1])  # Go
print(languages[-2])  # TypeScript
```

You can also modify an element:

```python
languages[0] = "Rust"

print(languages)
# ['Rust', 'JavaScript', 'TypeScript', 'Go']
```

This is possible because **lists are mutable**.

---

#  Slicing

Slicing extracts part of a list.

Syntax:

```python
list[start:stop:step]
```

The `stop` index is **excluded**.

```python
numbers = [10, 20, 30, 40, 50]
```

```python
print(numbers[1:4])
# [20, 30, 40]
```

Because:

```text
10  20  30  40  50
0   1   2   3   4
    ↑   ↑   ↑
    included
            4 excluded
```

### Beginning to index

```python
print(numbers[:3])
# [10, 20, 30]
```

### Index to end

```python
print(numbers[2:])
# [30, 40, 50]
```

### Copy the entire list

```python
copy = numbers[:]
```

### Step

```python
print(numbers[::2])
# [10, 30, 50]
```

### Reverse

```python
print(numbers[::-1])
# [50, 40, 30, 20, 10]
```

---

#  `append()`

`append()` adds **one item to the end** of a list.

```python
numbers = [1, 2, 3]

numbers.append(4)

print(numbers)
# [1, 2, 3, 4]
```

### Appending a list

This is an important distinction:

```python
numbers = [1, 2, 3]

numbers.append([4, 5])

print(numbers)
```

Result:

```python
[1, 2, 3, [4, 5]]
```

The entire list becomes **one element**.

Visualize it as:

```text
[1, 2, 3, [4, 5]]
             ↑
        one element
```

If you want to add `4` and `5` separately, use `extend()`.

---

#  `extend()`

`extend()` adds **each element from another iterable** to the list.

```python
numbers = [1, 2, 3]

numbers.extend([4, 5])

print(numbers)
# [1, 2, 3, 4, 5]
```

### `append()` vs `extend()`

```python
a = [1, 2]

a.append([3, 4])

print(a)
# [1, 2, [3, 4]]
```

Whereas:

```python
b = [1, 2]

b.extend([3, 4])

print(b)
# [1, 2, 3, 4]
```

Think:

```text
append(x)
    ↓
add x as ONE item

extend(x)
    ↓
add items from x
```

### `extend()` works with other iterables

For example:

```python
letters = ["a", "b"]

letters.extend("cd")

print(letters)
# ['a', 'b', 'c', 'd']
```

Because a string is iterable.

---

#  `insert()`

`insert()` adds an element at a specific index.

Syntax:

```python
list.insert(index, value)
```

Example:

```python
numbers = [10, 20, 40]

numbers.insert(2, 30)

print(numbers)
# [10, 20, 30, 40]
```

The existing elements are shifted to the right.

```text
Before:
10  20  40
0   1   2

insert(2, 30)

After:
10  20  30  40
0   1   2   3
```

### Insert at beginning

```python
numbers.insert(0, 5)
```

### Insert beyond the end

```python
numbers = [1, 2, 3]

numbers.insert(100, 4)

print(numbers)
# [1, 2, 3, 4]
```

If the index is beyond the end, the item is added at the end.

---

#  `remove()`

`remove()` removes the **first occurrence of a value**.

```python
numbers = [10, 20, 30, 20]

numbers.remove(20)

print(numbers)
# [10, 30, 20]
```

Only the first `20` was removed.

### Important

`remove()` expects a **value**, not an index.

```python
numbers.remove(30)
```

means:

> Find the value `30` and remove it.

If you want to remove by index, use `pop()`.

### What if the value doesn't exist?

```python
numbers = [1, 2, 3]

numbers.remove(10)
```

Raises:

```text
ValueError
```

You can check first:

```python
if 10 in numbers:
    numbers.remove(10)
```

---

#  `pop()`

`pop()` removes and **returns** an element.

```python
numbers = [10, 20, 30]

value = numbers.pop()

print(value)
# 30

print(numbers)
# [10, 20]
```

Without an argument, it removes the **last element**.

---

## Pop by index

```python
numbers = [10, 20, 30]

value = numbers.pop(1)

print(value)
# 20

print(numbers)
# [10, 30]
```

This makes `pop()` useful when you need both:

 Remove the item
 Use the removed item

Example:

```python
tasks = ["coding", "testing", "deployment"]

current_task = tasks.pop()

print(current_task)
# deployment
```

### `remove()` vs `pop()`

| Method       | Removes by | Returns removed item? |
| ------------ | ---------- | --------------------- |
| `remove(x)`  | Value      | No                    |
| `pop(index)` | Index      | Yes                   |
| `pop()`      | Last item  | Yes                   |

---

#  `sort()`

`sort()` sorts the list **in place**.

```python
numbers = [5, 2, 8, 1, 3]

numbers.sort()

print(numbers)
# [1, 2, 3, 5, 8]
```

Important:

```python
result = numbers.sort()
```

does **not** give you the sorted list.

```python
print(result)
# None
```

Because `sort()` modifies the existing list.

---

## Descending order

```python
numbers = [5, 2, 8, 1, 3]

numbers.sort(reverse=True)

print(numbers)
# [8, 5, 3, 2, 1]
```

---

## Sorting strings

```python
names = ["Charlie", "Alice", "Bob"]

names.sort()

print(names)
# ['Alice', 'Bob', 'Charlie']
```

---

## `key=`

One of the most useful intermediate features of `sort()` is `key`.

Suppose:

```python
users = [
    {"name": "John", "age": 30},
    {"name": "Alice", "age": 25},
    {"name": "Bob", "age": 35}
]
```

Sort by age:

```python
users.sort(key=lambda user: user["age"])
```

Result:

```python
[
    {"name": "Alice", "age": 25},
    {"name": "John", "age": 30},
    {"name": "Bob", "age": 35}
]
```

We'll go deeper into `lambda` and sorting later, but remember the pattern:

```python
items.sort(key=...)
```

---

#  `reverse()`

`reverse()` reverses the list **in place**.

```python
numbers = [1, 2, 3, 4, 5]

numbers.reverse()

print(numbers)
# [5, 4, 3, 2, 1]
```

Like `sort()`, it returns `None`:

```python
result = numbers.reverse()

print(result)
# None
```

### `reverse()` vs `[::-1]`

```python
numbers = [1, 2, 3]

numbers.reverse()
```

modifies the original list.

Whereas:

```python
numbers = [1, 2, 3]

reversed_numbers = numbers[::-1]
```

creates a **new list**.

So:

```text
reverse()  → modifies original
[::-1]     → creates new list
```

---

#  `copy()`

`copy()` creates a **shallow copy** of a list.

```python
original = [1, 2, 3]

new_list = original.copy()

new_list.append(4)

print(original)
# [1, 2, 3]

print(new_list)
# [1, 2, 3, 4]
```

The two lists are different objects:

```python
print(original is new_list)
# False
```

---

## Why copying matters

Consider:

```python
a = [1, 2, 3]
b = a

b.append(4)

print(a)
# [1, 2, 3, 4]
```

`b = a` doesn't copy the list.

It makes both variables point to the same object.

```text
a ──────┐
        ↓
     [1, 2, 3]
        ↑
b ──────┘
```

With `copy()`:

```python
a = [1, 2, 3]
b = a.copy()

b.append(4)
```

you get:

```text
a → [1, 2, 3]

b → [1, 2, 3, 4]
```

---

## Shallow copy gotcha

`copy()` only copies the **outer list**.

```python
original = [[1, 2], [3, 4]]

new_list = original.copy()

new_list[0].append(99)

print(original)
```

Output:

```python
[[1, 2, 99], [3, 4]]
```

Why?

The inner lists are still shared.

For a fully independent nested structure, you'd use `deepcopy()`:

```python
from copy import deepcopy

new_list = deepcopy(original)
```

You'll encounter this when working with nested data structures.

---

#  List Unpacking

List unpacking allows you to assign list elements directly to variables.

```python
numbers = [10, 20, 30]

a, b, c = numbers

print(a)  # 10
print(b)  # 20
print(c)  # 30
```

Think:

```text
numbers = [10, 20, 30]

a ← 10
b ← 20
c ← 30
```

The number of variables normally needs to match the number of elements.

This:

```python
a, b = [10, 20, 30]
```

raises:

```text
ValueError
```

---

#  Extended Unpacking with `*`

Python allows one variable to collect multiple elements.

```python
numbers = [1, 2, 3, 4, 5]

first, *middle, last = numbers

print(first)
# 1

print(middle)
# [2, 3, 4]

print(last)
# 5
```

This is extremely useful.

### Collect everything after the first

```python
first, *rest = [1, 2, 3, 4]

print(first)
# 1

print(rest)
# [2, 3, 4]
```

### Collect everything before the last

```python
*start, last = [1, 2, 3, 4]

print(start)
# [1, 2, 3]

print(last)
# 4
```

The starred variable always receives a **list**.

---

#  Unpacking in Function Arguments

The `*` operator can also unpack a list when calling a function.

```python
numbers = [10, 20, 30]

print(*numbers)
```

Equivalent to:

```python
print(10, 20, 30)
```

This becomes especially useful with functions:

```python
numbers = [3, 7]

print(max(*numbers))
```

Although in this specific case `max(numbers)` is simpler.

A more practical example:

```python
args = ["Hello", "Python"]

print(*args)
```

---

#  Nested Lists

Lists can contain other lists.

```python
matrix = [
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9]
]
```

Access elements using multiple indexes:

```python
print(matrix[0])
# [1, 2, 3]

print(matrix[0][1])
# 2

print(matrix[2][2])
# 9
```

You can modify nested elements:

```python
matrix[1][1] = 99

print(matrix)
```

Result:

```python
[
    [1, 2, 3],
    [4, 99, 6],
    [7, 8, 9]
]
```

---

#  Methods vs Built-in Functions

You'll frequently encounter both:

```python
numbers.sort()
```

and:

```python
sorted(numbers)
```

These are different.

### `list.sort()`

Modifies the original list:

```python
numbers = [3, 1, 2]

numbers.sort()

print(numbers)
# [1, 2, 3]
```

### `sorted()`

Creates a new sorted result:

```python
numbers = [3, 1, 2]

result = sorted(numbers)

print(numbers)
# [3, 1, 2]

print(result)
# [1, 2, 3]
```

This distinction is important in real code.

Similarly:

```python
numbers.reverse()
```

modifies the list, while:

```python
reversed(numbers)
```

returns an iterator.

---

#  A Realistic Example

Imagine you're processing API/user data:

```python
users = ["  Prasen  ", " Alice", "BOB ", " John "]
```

First clean the names:

```python
users = [user.strip().title() for user in users]
```

Now:

```python
print(users)
```

Result:

```python
['Prasen', 'Alice', 'Bob', 'John']
```

Add another user:

```python
users.append("David")
```

Add several users:

```python
users.extend(["Mike", "Sarah"])
```

Sort them:

```python
users.sort()
```

Remove one:

```python
users.remove("Bob")
```

Take the last user:

```python
last_user = users.pop()
```

This combination of list operations is much closer to how you'll actually use lists in applications.

---

#  Quick Revision Cheat Sheet

```text
LIST CREATION
users = ["Prasen", "Alice", "Bob"]

INDEXING
users[0]       → first item
users[-1]      → last item

SLICING
users[1:3]     → index 1 to 2
users[:3]      → beginning → index 2
users[2:]      → index 2 → end
users[::2]     → every second item
users[::-1]    → reversed copy

ADD
append(x)      → add ONE item
extend(x)      → add MANY items
insert(i, x)   → insert at index

REMOVE
remove(x)      → remove first matching VALUE
pop()          → remove + return last item
pop(i)         → remove + return item at index

ORDER
sort()         → sort in-place
sort(reverse=True)
reverse()      → reverse in-place

COPY
copy()         → shallow copy

UNPACKING
a, b, c = [1, 2, 3]

EXTENDED UNPACKING
first, *middle, last = [1, 2, 3, 4, 5]

IMPORTANT
append([3,4])
→ [1,2,[3,4]]

extend([3,4])
→ [1,2,3,4]

sort() / reverse()
→ modify original, return None

sorted(list)
→ creates new sorted list

list.copy()
→ creates shallow copy
```

## The key distinctions to remember

| If you want to...                  | Use         |
| ---------------------------------- | ----------- |
| Add one item                       | `append()`  |
| Add multiple items                 | `extend()`  |
| Add at a specific position         | `insert()`  |
| Remove by value                    | `remove()`  |
| Remove by index and get the value  | `pop()`     |
| Sort the existing list             | `sort()`    |
| Get a sorted copy                  | `sorted()`  |
| Reverse existing list              | `reverse()` |
| Get reversed copy                  | `[::-1]`    |
| Create an independent outer list   | `copy()`    |
| Assign list elements to variables  | unpacking   |
| Capture multiple unpacked elements | `*variable` |

**One particularly important intermediate concept:** when working with lists, always be conscious of whether an operation **mutates the existing list** or **creates a new object**. That distinction becomes very important when you start working with function arguments, nested data, APIs, and state in Flask/FastAPI applications.
