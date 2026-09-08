
**Comprehensions** are a concise way to create collections from existing iterables.

Instead of writing:

```python
numbers = [1, 2, 3, 4, 5]
squares = []

for number in numbers:
    squares.append(number ** 2)
```

you can write:

```python
squares = [number ** 2 for number in numbers]
```

The second version is called a **list comprehension**.

The main types you'll encounter are:

```text
List comprehension       → [...]
Dictionary comprehension → {...}
Set comprehension        → {...}
Generator expression    → (...)
```

---

# 1. List Comprehensions

A list comprehension creates a new list from an iterable.

### Normal loop

```python
numbers = [1, 2, 3, 4, 5]

squares = []

for number in numbers:
    squares.append(number ** 2)

print(squares)
# [1, 4, 9, 16, 25]
```

### List comprehension

```python
numbers = [1, 2, 3, 4, 5]

squares = [number ** 2 for number in numbers]

print(squares)
# [1, 4, 9, 16, 25]
```

The general syntax is:

```python
[expression for item in iterable]
```

Think about it as:

```text
[WHAT you want      FOR each item     IN this collection]
```

So:

```python
[number ** 2 for number in numbers]
```

means:

> Give me `number ** 2` for every `number` in `numbers`.

---

## Transforming values

Comprehensions are often used to transform data.

```python
names = ["prasen", "rahul", "amit"]

uppercase_names = [name.upper() for name in names]

print(uppercase_names)
# ['PRASEN', 'RAHUL', 'AMIT']
```

Another example:

```python
prices = [100, 200, 300]

with_tax = [price * 1.18 for price in prices]

print(with_tax)
# [118.0, 236.0, 354.0]
```

---

## Working with strings

```python
word = "Python"

characters = [char.upper() for char in word]

print(characters)
# ['P', 'Y', 'T', 'H', 'O', 'N']
```

---

# 2. Conditional List Comprehensions

You can add an `if` condition to filter elements.

### Normal loop

```python
numbers = [1, 2, 3, 4, 5, 6]

even_numbers = []

for number in numbers:
    if number % 2 == 0:
        even_numbers.append(number)
```

### Comprehension

```python
even_numbers = [
    number
    for number in numbers
    if number % 2 == 0
]
```

Result:

```python
[2, 4, 6]
```

Syntax:

```python
[expression for item in iterable if condition]
```

Notice the order:

```text
WHAT
 ↓
for
 ↓
FROM WHERE
 ↓
if
 ↓
FILTER
```

---

## Filtering strings

```python
users = ["Prasen", "Amit", "", "Rahul", ""]

valid_users = [user for user in users if user]

print(valid_users)
# ['Prasen', 'Amit', 'Rahul']
```

This works because an empty string is **falsy**.

---

## Transform + filter

You can both filter and transform:

```python
numbers = [1, 2, 3, 4, 5, 6]

squares_of_even = [
    number ** 2
    for number in numbers
    if number % 2 == 0
]

print(squares_of_even)
# [4, 16, 36]
```

The process is:

```text
1 → odd → skip
2 → even → 2² = 4
3 → odd → skip
4 → even → 4² = 16
5 → odd → skip
6 → even → 6² = 36
```

---

## Conditional expression inside a comprehension

There's another form that's easy to confuse with filtering.

You can put an `if/else` **inside the expression**:

```python
numbers = [1, 2, 3, 4, 5]

result = [
    "even" if number % 2 == 0 else "odd"
    for number in numbers
]

print(result)
# ['odd', 'even', 'odd', 'even', 'odd']
```

Compare:

### Filtering

```python
[number for number in numbers if number % 2 == 0]
```

Result:

```python
[2, 4]
```

### Conditional expression

```python
["even" if number % 2 == 0 else "odd" for number in numbers]
```

Result:

```python
["odd", "even", "odd", "even", "odd"]
```

**Important distinction:**

```text
... for x in items if condition
```

→ filters items

```text
value_if_true if condition else value_if_false
```

→ transforms every item conditionally

---

# 3. Nested List Comprehensions

A comprehension can contain another loop.

Suppose you have:

```python
matrix = [
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9]
]
```

You want to flatten it into:

```python
[1, 2, 3, 4, 5, 6, 7, 8, 9]
```

Normal loops:

```python
flattened = []

for row in matrix:
    for number in row:
        flattened.append(number)
```

Nested comprehension:

```python
flattened = [
    number
    for row in matrix
    for number in row
]
```

Result:

```python
[1, 2, 3, 4, 5, 6, 7, 8, 9]
```

The order is important.

This:

```python
for row in matrix
for number in row
```

corresponds directly to:

```python
for row in matrix:
    for number in row:
```

---

## Another example

```python
matrix = [
    [1, 2],
    [3, 4],
    [5, 6]
]

doubled = [
    number * 2
    for row in matrix
    for number in row
]

print(doubled)
# [2, 4, 6, 8, 10, 12]
```

---

## Nested loops with conditions

You can add filtering:

```python
matrix = [
    [1, 2, 3],
    [4, 5, 6]
]

even_numbers = [
    number
    for row in matrix
    for number in row
    if number % 2 == 0
]

print(even_numbers)
# [2, 4, 6]
```

---

## Be careful with readability

You **can** write extremely complicated comprehensions:

```python
result = [
    x * y
    for x in numbers
    for y in numbers
    if x != y
    if x * y > 10
]
```

But just because Python allows it doesn't mean you should use it.

If a comprehension becomes difficult to understand, use a normal loop.

**Good intermediate Python means knowing when *not* to use a comprehension.**

---

# 4. Dictionary Comprehensions

Dictionary comprehensions create dictionaries.

Syntax:

```python
{key_expression: value_expression for item in iterable}
```

Suppose:

```python
numbers = [1, 2, 3, 4, 5]
```

Create a dictionary where each number maps to its square:

```python
squares = {
    number: number ** 2
    for number in numbers
}

print(squares)
```

Result:

```python
{
    1: 1,
    2: 4,
    3: 9,
    4: 16,
    5: 25
}
```

---

## Transforming an existing dictionary

Suppose:

```python
prices = {
    "laptop": 50000,
    "mouse": 2000,
    "keyboard": 3000
}
```

Apply a discount:

```python
discounted = {
    product: price * 0.9
    for product, price in prices.items()
}
```

Result:

```python
{
    "laptop": 45000.0,
    "mouse": 1800.0,
    "keyboard": 2700.0
}
```

This pattern is very useful when processing API responses or configuration data.

---

## Conditional dictionary comprehension

You can filter entries:

```python
prices = {
    "laptop": 50000,
    "mouse": 2000,
    "keyboard": 3000
}

expensive = {
    product: price
    for product, price in prices.items()
    if price > 5000
}
```

Result:

```python
{
    "laptop": 50000
}
```

---

## Conditional values

Just like list comprehensions, you can use `if/else` inside the expression:

```python
numbers = [1, 2, 3, 4]

result = {
    number: "even" if number % 2 == 0 else "odd"
    for number in numbers
}
```

Result:

```python
{
    1: "odd",
    2: "even",
    3: "odd",
    4: "even"
}
```

---

# 5. Set Comprehensions

Set comprehensions work similarly to list comprehensions, but produce a **set**.

Use `{}` with an expression and `for`:

```python
numbers = [1, 2, 2, 3, 3, 4]

squares = {
    number ** 2
    for number in numbers
}

print(squares)
```

Result:

```python
{1, 4, 9, 16}
```

Duplicates automatically disappear.

Compare:

```python
[number ** 2 for number in numbers]
```

Result:

```python
[1, 4, 4, 9, 9, 16]
```

while:

```python
{number ** 2 for number in numbers}
```

gives:

```python
{1, 4, 9, 16}
```

---

## Filtering with set comprehensions

```python
numbers = range(20)

even_numbers = {
    number
    for number in numbers
    if number % 2 == 0
}

print(even_numbers)
```

You get a set of unique even numbers.

---

# 6. Generator Expressions

Generator expressions look similar to list comprehensions but use `()`.

List comprehension:

```python
squares = [number ** 2 for number in range(10)]
```

Generator expression:

```python
squares = (number ** 2 for number in range(10))
```

The important difference is that a list comprehension creates the entire list **immediately**, while a generator produces values **lazily**, as you iterate over it.

---

## List comprehension

```python
numbers = [number ** 2 for number in range(5)]

print(numbers)
# [0, 1, 4, 9, 16]
```

The values are already stored in the list.

---

## Generator expression

```python
numbers = (number ** 2 for number in range(5))

print(numbers)
```

You'll see something similar to:

```text
<generator object <genexpr> at 0x...>
```

The values haven't all been produced yet.

You can iterate:

```python
for number in numbers:
    print(number)
```

Output:

```text
0
1
4
9
16
```

---

## Why use generators?

Generators are useful when working with **large amounts of data**.

Imagine:

```python
numbers = [number ** 2 for number in range(10_000_000)]
```

This creates a huge list in memory.

Instead:

```python
numbers = (number ** 2 for number in range(10_000_000))
```

The generator produces values as they're requested rather than storing all 10 million results at once.

Conceptually:

```text
List comprehension:

[0, 1, 4, 9, 16, ...]
 ↓
All values exist in memory


Generator:

0 → produce
1 → produce
4 → produce
9 → produce
...
 ↓
Produce values when needed
```

---

# Generator Expressions with Functions

This is a very common pattern.

Instead of:

```python
numbers = [1, 2, 3, 4, 5]

total = sum([number ** 2 for number in numbers])
```

you can write:

```python
total = sum(number ** 2 for number in numbers)
```

There is no need to create an intermediate list.

Similarly:

```python
any(number > 100 for number in numbers)
```

or:

```python
all(number > 0 for number in numbers)
```

This can be both memory-efficient and clean.

---

# Comprehension Cheat Sheet

## List

```python
result = [expression for item in iterable]
```

Example:

```python
squares = [x ** 2 for x in numbers]
```

---

## List + filter

```python
result = [
    expression
    for item in iterable
    if condition
]
```

Example:

```python
evens = [x for x in numbers if x % 2 == 0]
```

---

## List + conditional expression

```python
result = [
    value_if_true if condition else value_if_false
    for item in iterable
]
```

Example:

```python
labels = [
    "even" if x % 2 == 0 else "odd"
    for x in numbers
]
```

---

## Nested list comprehension

```python
result = [
    expression
    for outer in outer_iterable
    for inner in inner_iterable
]
```

Example:

```python
flattened = [
    x
    for row in matrix
    for x in row
]
```

---

## Dictionary

```python
result = {
    key: value
    for item in iterable
}
```

Example:

```python
squares = {
    x: x ** 2
    for x in numbers
}
```

---

## Set

```python
result = {
    expression
    for item in iterable
}
```

Example:

```python
unique_squares = {
    x ** 2
    for x in numbers
}
```

---

## Generator

```python
result = (
    expression
    for item in iterable
)
```

Example:

```python
squares = (
    x ** 2
    for x in numbers
)
```

---

# The Most Important Difference

Given:

```python
numbers = [1, 2, 2, 3, 4]
```

### List comprehension

```python
[x * 2 for x in numbers]
```

→ Creates a **list**, keeps duplicates.

```python
[2, 4, 4, 6, 8]
```

### Set comprehension

```python
{x * 2 for x in numbers}
```

→ Creates a **set**, removes duplicates.

```python
{2, 4, 6, 8}
```

### Generator expression

```python
(x * 2 for x in numbers)
```

→ Creates a **generator**, calculates values lazily.

---

# Real-World Example

Suppose you're processing users:

```python
users = [
    {"name": "Prasen", "age": 25, "active": True},
    {"name": "Rahul", "age": 17, "active": False},
    {"name": "Amit", "age": 30, "active": True},
]
```

### Get names

```python
names = [user["name"] for user in users]
```

Result:

```python
["Prasen", "Rahul", "Amit"]
```

### Get active users

```python
active_users = [
    user
    for user in users
    if user["active"]
]
```

### Get active adult users

```python
active_adults = [
    user
    for user in users
    if user["active"] and user["age"] >= 18
]
```

### Create name → age dictionary

```python
user_ages = {
    user["name"]: user["age"]
    for user in users
}
```

Result:

```python
{
    "Prasen": 25,
    "Rahul": 17,
    "Amit": 30
}
```

### Get unique roles

```python
roles = {
    user["role"]
    for user in users
}
```

if the dictionaries contained `"role"`.

This is exactly the kind of transformation you'll frequently do when working with **API responses and backend data**.

---

# One Important Rule

Don't use comprehensions simply because they're shorter.

This:

```python
result = [
    complicated_transformation(x)
    for x in data
    if complex_condition(x)
]
```

might technically be valid, but if it's difficult to understand, use a normal loop:

```python
result = []

for x in data:
    if complex_condition(x):
        result.append(complicated_transformation(x))
```

**Comprehensions are best when they make the transformation clearer, not merely shorter.**

---

# Mental Model

Think of comprehensions as:

```text
LIST
[ WHAT       FOR       EACH ITEM       IN       COLLECTION ]
```

```python
[x * 2 for x in numbers]
```

↓

> "Give me `x * 2` for every `x` in `numbers`."

Add:

```python
if condition
```

↓

> "Give me `x * 2` for every `x` in `numbers`, but only when the condition is true."

And the collection type determines the result:

```text
[...]       → List
{...}       → Set
{key: val}  → Dictionary
(...)       → Generator
```

If you understand that pattern, **most Python comprehensions become straightforward to read and write.**
