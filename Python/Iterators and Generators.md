# Python Iterators & Generators

Iterators and generators are mainly about **processing values one at a time instead of creating everything at once**.

This becomes especially useful when working with:

* Large files
* Large datasets
* API responses
* Database results
* Data pipelines
* Infinite sequences

The core idea is:

```text
Iterable → can give you an iterator
Iterator → gives you values one at a time
Generator → an easy way to create an iterator
```

---

# 1. Iterable vs Iterator

These two are related but **not the same**.

## Iterable

An **iterable** is an object that you can loop over.

Examples:

```python
numbers = [1, 2, 3]
name = "Kakade"
items = {"python", "flask", "fastapi"}
```

You can use:

```python
for number in numbers:
    print(number)
```

A list is iterable.

Other common iterables:

```text
list
tuple
string
set
dictionary
range
files
```

---

## Iterator

An **iterator** is an object that produces values **one at a time**.

It implements:

```python
__iter__()
__next__()
```

You can get an iterator from an iterable using:

```python
iter()
```

Example:

```python
numbers = [10, 20, 30]

iterator = iter(numbers)
```

Now:

```python
print(next(iterator))
print(next(iterator))
print(next(iterator))
```

Output:

```text
10
20
30
```

Another `next()`:

```python
print(next(iterator))
```

causes:

```text
StopIteration
```

because there are no more values.

---

## The difference

Think:

```text
Iterable
    ↓
"Give me an iterator"

Iterator
    ↓
"Give me the next value"
```

Example:

```python
numbers = [10, 20, 30]

iterator = iter(numbers)

print(next(iterator))  # 10
print(next(iterator))  # 20
print(next(iterator))  # 30
```

The **list** is the iterable.

The object returned by `iter(numbers)` is the iterator.

---

# 2. `iter()`

`iter()` converts an iterable into an iterator.

```python
numbers = [10, 20, 30]

iterator = iter(numbers)
```

You can then use:

```python
next(iterator)
```

Example:

```python
names = ["Kakade", "Alex", "Sam"]

iterator = iter(names)

print(next(iterator))
print(next(iterator))
```

Output:

```text
Kakade
Alex
```

The iterator remembers where it currently is.

```text
Iterator
   │
   ├── next() → "Kakade"
   │
   ├── next() → "Alex"
   │
   └── next() → "Sam"
```

---

# 3. `next()`

`next()` retrieves the **next value from an iterator**.

```python
numbers = iter([10, 20, 30])

print(next(numbers))
```

Output:

```text
10
```

Then:

```python
print(next(numbers))
```

Output:

```text
20
```

Then:

```python
print(next(numbers))
```

Output:

```text
30
```

After that:

```python
next(numbers)
```

raises:

```python
StopIteration
```

---

## `next()` with a default value

You can provide a default:

```python
numbers = iter([10, 20])

print(next(numbers, "Done"))
print(next(numbers, "Done"))
print(next(numbers, "Done"))
```

Output:

```text
10
20
Done
```

This avoids `StopIteration` being raised.

---

# 4. How `for` loops actually work

A `for` loop uses the iterator protocol internally.

When you write:

```python
numbers = [10, 20, 30]

for number in numbers:
    print(number)
```

Python conceptually does something similar to:

```python
iterator = iter(numbers)

while True:
    try:
        number = next(iterator)
        print(number)
    except StopIteration:
        break
```

So:

```text
for loop
   ↓
iter()
   ↓
next()
   ↓
next()
   ↓
next()
   ↓
StopIteration
   ↓
loop ends
```

This is why understanding `iter()` and `next()` helps you understand how Python's `for` loop works.

---

# 5. Creating Custom Iterators

You can create your own iterator by implementing:

```python
__iter__()
__next__()
```

Example:

```python
class CountUp:
    def __init__(self, limit):
        self.current = 1
        self.limit = limit

    def __iter__(self):
        return self

    def __next__(self):
        if self.current > self.limit:
            raise StopIteration

        value = self.current
        self.current += 1

        return value
```

Now:

```python
counter = CountUp(3)

print(next(counter))
print(next(counter))
print(next(counter))
```

Output:

```text
1
2
3
```

Then:

```python
next(counter)
```

raises:

```text
StopIteration
```

You can also use it with a `for` loop:

```python
counter = CountUp(3)

for number in counter:
    print(number)
```

Output:

```text
1
2
3
```

---

## What is happening?

The iterator stores its state:

```text
current = 1
limit = 3
```

Every `next()`:

```text
current → return value → increment current
```

Eventually:

```text
current > limit
       ↓
StopIteration
```

---

# 6. Generators

A **generator** is a simpler way to create an iterator.

Instead of manually implementing:

```python
__iter__()
__next__()
```

you can use:

```python
yield
```

Example:

```python
def count_up(limit):
    current = 1

    while current <= limit:
        yield current
        current += 1
```

Use it:

```python
numbers = count_up(3)

print(next(numbers))
print(next(numbers))
print(next(numbers))
```

Output:

```text
1
2
3
```

---

# 7. `yield`

`yield` is the key feature of generators.

Compare:

### `return`

```python
def get_number():
    return 10
```

`return` ends the function completely.

### `yield`

```python
def get_number():
    yield 10
    yield 20
    yield 30
```

Each `yield` **pauses the function** and remembers its state.

```python
numbers = get_number()

print(next(numbers))
```

Output:

```text
10
```

The function pauses.

Then:

```python
print(next(numbers))
```

Output:

```text
20
```

It resumes from where it stopped.

Then:

```python
print(next(numbers))
```

Output:

```text
30
```

---

## Visualizing `yield`

```text
get_number()
     │
     ↓
yield 10
     │
     └── pause
          │
       next()
          ↓
     yield 20
          │
          └── pause
               │
            next()
               ↓
          yield 30
```

This ability to **pause and resume** is what makes generators powerful.

---

# 8. Generator vs Normal Function

Normal function:

```python
def get_numbers():
    return [1, 2, 3, 4, 5]
```

The entire list is created immediately.

Generator:

```python
def get_numbers():
    for number in range(1, 6):
        yield number
```

Values are produced one at a time.

```text
Normal function

[1, 2, 3, 4, 5]
      ↓
all values exist immediately


Generator

1 → 2 → 3 → 4 → 5
    ↑
one value at a time
```

---

# 9. Generator Expressions

A **generator expression** looks similar to a list comprehension but uses `()` instead of `[]`.

### List comprehension

```python
numbers = [x * 2 for x in range(5)]
```

This creates a list immediately:

```python
[0, 2, 4, 6, 8]
```

### Generator expression

```python
numbers = (x * 2 for x in range(5))
```

This creates a generator.

Values are produced when needed:

```python
print(next(numbers))
print(next(numbers))
```

Output:

```text
0
2
```

---

## Comparison

```python
# List
numbers = [x * 2 for x in range(1000000)]
```

versus:

```python
# Generator
numbers = (x * 2 for x in range(1000000))
```

The list creates and stores all one million values.

The generator produces them as they're requested.

---

# 10. Lazy Evaluation

**Lazy evaluation** means:

> Don't calculate something until it's actually needed.

Generators are lazy.

Example:

```python
def numbers():
    print("Generating 1")
    yield 1

    print("Generating 2")
    yield 2

    print("Generating 3")
    yield 3
```

Creating the generator:

```python
values = numbers()
```

doesn't execute the function body yet.

Nothing is printed.

When you do:

```python
print(next(values))
```

Python starts executing until the first `yield`.

Output:

```text
Generating 1
1
```

Then:

```python
print(next(values))
```

Output:

```text
Generating 2
2
```

The generator only performs work when you request another value.

---

# 11. Memory Efficiency

Generators are particularly useful when dealing with **large amounts of data**.

Suppose you need numbers from:

```text
1 → 10,000,000
```

A list:

```python
numbers = [x for x in range(10_000_000)]
```

stores all those values in memory.

A generator:

```python
numbers = (x for x in range(10_000_000))
```

produces values as needed.

Conceptually:

```text
List:

RAM
┌─────────────────────────────┐
│ 1 2 3 4 5 6 7 ... 10M       │
└─────────────────────────────┘


Generator:

RAM
┌───────┐
│ state │
└───────┘
    │
    ↓
generate next value
```

The generator itself doesn't hold all generated values.

---

# Real-World Example: Reading a Large File

Imagine `logs.txt` contains millions of lines.

You could do:

```python
with open("logs.txt") as file:
    lines = file.readlines()

for line in lines:
    print(line)
```

`readlines()` loads all lines into memory.

Instead:

```python
with open("logs.txt") as file:
    for line in file:
        print(line)
```

The file object itself is iterable and provides lines incrementally.

This is much more memory-friendly for large files.

---

# Real-World Example: Custom Generator

Suppose you want to process IDs:

```python
def generate_ids(start, end):
    for id in range(start, end + 1):
        yield id
```

Use:

```python
for id in generate_ids(1, 5):
    print(id)
```

Output:

```text
1
2
3
4
5
```

You don't need to construct the entire sequence beforehand.

---

# Infinite Generators

Generators can even represent sequences that never end.

```python
def infinite_numbers():
    number = 1

    while True:
        yield number
        number += 1
```

You can take values from it:

```python
numbers = infinite_numbers()

print(next(numbers))
print(next(numbers))
print(next(numbers))
```

Output:

```text
1
2
3
```

It doesn't need to store an infinite list.

---

# Iterator vs Generator

A generator **is an iterator**.

```text
Iterable
   │
   ├── list
   ├── tuple
   ├── string
   └── range
        │
        ↓
      iter()
        │
        ↓
    Iterator
        │
        ├── custom iterator
        └── generator
```

So:

```python
generator = (x * 2 for x in range(5))
```

is both:

* iterable
* iterator

You can do:

```python
iter(generator) is generator
```

which evaluates to:

```text
True
```

---

# Iterator vs Iterable — Interview Question

### Is a list an iterator?

**No.**

A list is an **iterable**.

```python
numbers = [1, 2, 3]
```

This works:

```python
iter(numbers)
```

and gives you an iterator.

```python
iterator = iter(numbers)
```

Now `iterator` is an iterator.

### Does an iterator have to be iterable?

Yes.

An iterator implements both:

```python
__iter__()
__next__()
```

and `__iter__()` returns itself.

---

# `yield` vs `return`

| `return`                      | `yield`                   |
| ----------------------------- | ------------------------- |
| Ends function                 | Pauses function           |
| Returns one final result      | Produces values over time |
| Function can't resume         | Generator can resume      |
| Usually immediate computation | Supports lazy evaluation  |
| Used in normal functions      | Used to create generators |

Example:

```python
def normal():
    return 1
```

versus:

```python
def generator():
    yield 1
    yield 2
    yield 3
```

---

# Memory Efficiency: Important Nuance

Generators are **not automatically faster**.

Their biggest advantage is often **lower memory usage**.

For example:

```python
numbers = [x * 2 for x in range(100)]
```

may actually be more convenient and sometimes faster if you need to repeatedly access the results.

But:

```python
numbers = (x * 2 for x in range(100_000_000))
```

is useful when:

* You process values once
* You don't need random access
* The dataset is large
* You don't need all results simultaneously

---

# Quick Revision

| Concept                  | Remember                                 |
| ------------------------ | ---------------------------------------- |
| **Iterable**             | Something you can loop over              |
| **Iterator**             | Produces values one at a time            |
| **`iter()`**             | Gets an iterator from an iterable        |
| **`next()`**             | Gets the next value                      |
| **`StopIteration`**      | Signals that iteration is finished       |
| **Custom iterator**      | Implements `__iter__()` and `__next__()` |
| **Generator**            | Easy way to create an iterator           |
| **`yield`**              | Produces a value and pauses              |
| **Generator expression** | Lazy expression using `()`               |
| **Lazy evaluation**      | Calculate only when needed               |
| **Memory efficiency**    | Don't store all values at once           |

## The Core Mental Model

```text
ITERABLE
   │
   │ iter()
   ↓
ITERATOR
   │
   │ next()
   ↓
VALUE
   │
   │ next()
   ↓
VALUE
   │
   │ next()
   ↓
VALUE
   │
   ↓
StopIteration
```

And generators simplify creating iterators:

```text
Generator function
       │
       │ yield
       ↓
   Generator
       │
       │ next()
       ↓
   value
       │
       │ next()
       ↓
   value
       │
       ↓
   ...
```

### One-liner to remember

> **Iterable = can be looped over; Iterator = produces the next value; Generator = an easy, lazy way to create an iterator.**
