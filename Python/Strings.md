
## 1. String Creation

A string is a sequence of Unicode characters.

```python
name = "Prasen"
language = 'Python'
```

Both single and double quotes work.

```python
single = 'Hello'
double = "Hello"

print(single == double)  # True
```

### Quotes inside strings

You can use different quote types:

```python
message = "I'm learning Python"
message = 'He said "Hello"'
```

Or escape the quote:

```python
message = 'I\'m learning Python'
```

### Empty string

```python
name = ""

print(len(name))  # 0
```

### Strings are Unicode

Python strings can contain Unicode characters:

```python
name = "Prasen"
emoji = "🚀"
hindi = "नमस्ते"
```

---

# 2. Indexing

A string is an ordered sequence, so every character has an index.

```python
text = "Python"
```

Indexes:

```text
 P   y   t   h   o   n
 0   1   2   3   4   5
```

So:

```python
print(text[0])  # P
print(text[1])  # y
print(text[3])  # h
```

### Negative indexing

Python also supports indexes from the end:

```text
 P   y   t   h   o   n
-6  -5  -4  -3  -2  -1
```

```python
print(text[-1])  # n
print(text[-2])  # o
print(text[-6])  # P
```

This is particularly useful when you want the last character:

```python
filename = "report.pdf"

print(filename[-1])
# f
```

---

# 3. Slicing

Slicing extracts part of a string.

Syntax:

```python
string[start:stop:step]
```

Important: **`stop` is excluded**.

```python
text = "Python"

print(text[0:3])
# Pyt
```

Indexes:

```text
 P   y   t   h   o   n
 0   1   2   3   4   5
```

`text[0:3]` means:

```text
0 → included
1 → included
2 → included
3 → excluded
```

---

## Omitting `start`

```python
text = "Python"

print(text[:3])
# Pyt
```

Means:

```python
text[0:3]
```

---

## Omitting `stop`

```python
print(text[3:])
# hon
```

Means from index `3` until the end.

---

## Copying the entire string

```python
print(text[:])
# Python
```

---

## Using a step

```python
text = "Python"

print(text[::2])
```

Output:

```text
Pto
```

It takes every second character:

```text
P y t h o n
↑   ↑   ↑
P   t   o
```

---

## Reversing a string

One of the most useful slicing tricks:

```python
text = "Python"

print(text[::-1])
```

Output:

```text
nohtyP
```

`-1` means move backwards one character at a time.

---

# 4. String Immutability

This is **very important**.

Python strings are **immutable**.

Once a string is created, its individual characters cannot be changed.

This is invalid:

```python
text = "Python"

text[0] = "J"
```

You'll get:

```text
TypeError: 'str' object does not support item assignment
```

Instead, create a new string:

```python
text = "Python"

text = "J" + text[1:]

print(text)
# Jython
```

### Methods also return new strings

For example:

```python
name = "prasen"

upper_name = name.upper()

print(name)        # prasen
print(upper_name)  # PRASEN
```

`.upper()` didn't modify `name`.

It created a **new string**.

This matters when writing code like:

```python
name.upper()
```

If you need to keep the result:

```python
name = name.upper()
```

---

# 5. String Methods

Python provides many methods for working with strings.

Some of the most useful ones are:

```python
upper()
lower()
strip()
replace()
split()
join()
startswith()
endswith()
```

You can inspect available methods with:

```python
print(dir(str))
```

Or:

```python
help(str.upper)
```

---

# 6. `upper()` / `lower()`

Convert a string to uppercase/lowercase.

```python
text = "Hello Python"

print(text.upper())
# HELLO PYTHON

print(text.lower())
# hello python
```

### Useful for comparisons

Suppose you receive user input:

```python
role = input("Enter role: ")

if role.lower() == "admin":
    print("Admin access")
```

Now all of these work:

```text
ADMIN
Admin
admin
AdMiN
```

because they're converted to lowercase before comparison.

### `casefold()`

For more robust case-insensitive comparisons, Python also provides:

```python
text.casefold()
```

For normal English application logic, `.lower()` is often sufficient.

---

# 7. `strip()`

`strip()` removes whitespace from the **beginning and end** of a string.

```python
username = "   prasen   "

print(username.strip())
# prasen
```

It doesn't remove spaces in the middle:

```python
text = "  hello   world  "

print(text.strip())
# hello   world
```

### `lstrip()`

Removes whitespace from the left:

```python
text = "   hello"

print(text.lstrip())
# hello
```

### `rstrip()`

Removes whitespace from the right:

```python
text = "hello   "

print(text.rstrip())
# hello
```

---

## `strip()` can remove specific characters

```python
text = "---hello---"

print(text.strip("-"))
# hello
```

Important: it removes matching characters from the **edges**, not an exact substring.

```python
text = "abcHelloabc"

print(text.strip("abc"))
```

This doesn't mean "remove the string `abc` once from both sides"; it removes any combination of the characters `a`, `b`, and `c` from the edges.

---

# 8. `replace()`

Replaces occurrences of one substring with another.

```python
text = "I like Java"

result = text.replace("Java", "Python")

print(result)
# I like Python
```

Again, because strings are immutable, the original isn't changed.

---

## Replace multiple occurrences

```python
text = "cat dog cat"

print(text.replace("cat", "bird"))
```

Output:

```text
bird dog bird
```

---

## Limit replacements

`replace()` accepts a third argument:

```python
text = "one one one"

print(text.replace("one", "two", 1))
```

Output:

```text
two one one
```

Only the first occurrence was replaced.

---

# 9. `split()`

`split()` converts a string into a **list of strings**.

```python
text = "apple,banana,orange"

fruits = text.split(",")

print(fruits)
```

Output:

```python
['apple', 'banana', 'orange']
```

This is extremely common when processing:

* CSV-like data
* command-line input
* user input
* logs
* configuration strings

### Default separator

If you don't provide a separator:

```python
text = "hello world python"

print(text.split())
```

Output:

```python
['hello', 'world', 'python']
```

It splits on whitespace.

It also handles multiple spaces:

```python
text = "hello    world   python"

print(text.split())
```

Result:

```python
['hello', 'world', 'python']
```

---

## Limiting splits

```python
text = "a-b-c-d"

print(text.split("-", 2))
```

Output:

```python
['a', 'b', 'c-d']
```

The second argument specifies the maximum number of splits.

---

# 10. `join()`

`join()` does essentially the opposite of `split()`.

It combines strings from an iterable into one string.

```python
words = ["Python", "is", "awesome"]

result = " ".join(words)

print(result)
```

Output:

```text
Python is awesome
```

### Comma-separated values

```python
fruits = ["apple", "banana", "orange"]

result = ", ".join(fruits)

print(result)
```

Output:

```text
apple, banana, orange
```

### No separator

```python
letters = ["P", "y", "t", "h", "o", "n"]

print("".join(letters))
```

Output:

```text
Python
```

### Important

`join()` expects strings.

This won't work:

```python
numbers = [1, 2, 3]

",".join(numbers)
```

You need:

```python
numbers = [1, 2, 3]

result = ",".join(map(str, numbers))

print(result)
# 1,2,3
```

---

# 11. `startswith()` / `endswith()`

These check whether a string starts or ends with a particular substring.

They return a Boolean.

```python
filename = "report.pdf"

print(filename.startswith("report"))
# True

print(filename.endswith(".pdf"))
# True
```

Very useful for file processing:

```python
filename = "data.json"

if filename.endswith(".json"):
    print("JSON file")
```

### Multiple possibilities

You can pass a tuple:

```python
filename = "photo.png"

if filename.endswith((".jpg", ".jpeg", ".png")):
    print("Image file")
```

Similarly:

```python
url = "https://example.com"

if url.startswith(("http://", "https://")):
    print("Valid URL scheme")
```

---

# 12. String Formatting

String formatting means inserting dynamic values into strings.

There are several approaches.

---

## Old `%` formatting

You'll see this in older Python code:

```python
name = "Prasen"
age = 25

message = "My name is %s and I am %d years old" % (name, age)

print(message)
```

Works, but generally avoid it in new code.

---

## `.format()`

```python
name = "Prasen"
age = 25

message = "My name is {} and I am {} years old".format(name, age)

print(message)
```

You can use named placeholders:

```python
message = "My name is {name} and I am {age}".format(
    name="Prasen",
    age=25
)
```

This is better than `%` formatting, but modern Python usually favors **f-strings**.

---

# 13. f-Strings

f-strings are the preferred way to format strings in modern Python.

```python
name = "Prasen"
age = 25

message = f"My name is {name} and I am {age} years old"

print(message)
```

The `f` before the string enables expressions inside `{}`.

---

## Expressions inside f-strings

```python
a = 10
b = 20

print(f"Sum = {a + b}")
```

Output:

```text
Sum = 30
```

You can even call methods:

```python
name = "prasen"

print(f"Hello, {name.upper()}!")
```

Output:

```text
Hello, PRASEN!
```

---

## Number formatting

```python
price = 1234.5678

print(f"{price:.2f}")
```

Output:

```text
1234.57
```

`.2f` means:

```text
2 decimal places
floating-point format
```

### Thousands separator

```python
number = 1000000

print(f"{number:,}")
```

Output:

```text
1,000,000
```

You can combine them:

```python
price = 1234567.891

print(f"₹{price:,.2f}")
```

Output:

```text
₹1,234,567.89
```

---

## Width and alignment

```python
name = "Python"

print(f"{name:>10}")  # right aligned
print(f"{name:<10}")  # left aligned
print(f"{name:^10}")  # centered
```

Useful when generating CLI tables/reports.

---

## Debugging with f-strings

A very useful feature:

```python
name = "Prasen"
age = 25

print(f"{name=}")
print(f"{age=}")
```

Output:

```text
name='Prasen'
age=25
```

You can also do:

```python
x = 10
y = 20

print(f"{x + y=}")
```

Output:

```text
x + y=30
```

---

# 14. Escape Characters

Escape characters allow you to represent special characters inside strings.

They use a backslash:

```text
\
```

### Newline — `\n`

```python
print("Hello\nWorld")
```

Output:

```text
Hello
World
```

### Tab — `\t`

```python
print("Name:\tPrasen")
```

Output:

```text
Name:   Prasen
```

### Quote characters

```python
print("He said \"Hello\"")
```

Output:

```text
He said "Hello"
```

Or:

```python
print('He said "Hello"')
```

Usually the second version is cleaner.

### Backslash

```python
print("C:\\Users\\Prasen")
```

Output:

```text
C:\Users\Prasen
```

Because `\\` represents a literal backslash.

---

# 15. Common Escape Sequences

| Escape | Meaning         |
| ------ | --------------- |
| `\n`   | Newline         |
| `\t`   | Tab             |
| `\\`   | Backslash       |
| `\'`   | Single quote    |
| `\"`   | Double quote    |
| `\r`   | Carriage return |
| `\b`   | Backspace       |
| `\0`   | Null character  |

You'll most commonly use:

```text
\n
\t
\\
\'
\"
```

---

# 16. Raw Strings

Raw strings treat backslashes mostly as literal characters.

Add `r` before the string:

```python
path = r"C:\Users\Prasen\Documents"
```

Without a raw string:

```python
path = "C:\\Users\\Prasen\\Documents"
```

Raw strings are especially useful for:

* Windows paths
* Regular expressions

Example:

```python
pattern = r"\d+\.\d+"
```

The backslashes don't need to be double-escaped.

---

# 17. Multiline Strings

Triple quotes allow strings to span multiple lines.

```python
message = """
Hello Prasen,

Welcome to Python.

Good luck!
"""

print(message)
```

You can use either:

```python
"""
...
"""
```

or:

```python
'''
...
'''
```

---

## Multiline f-strings

You can combine triple quotes with f-strings:

```python
name = "Prasen"
age = 25

message = f"""
Name: {name}
Age: {age}
Language: Python
"""

print(message)
```

This is useful for generating:

* emails
* reports
* HTML
* SQL
* prompts
* configuration text

---

# 18. Useful Combination: Cleaning User Input

A very common real-world pattern:

```python
username = input("Username: ")

username = username.strip().lower()

if username == "prasen":
    print("Welcome!")
```

Here we're combining:

```text
input()
  ↓
strip()
  ↓
lower()
  ↓
comparison
```

So these all become equivalent:

```text
" Prasen "
"PRASEN"
" prasen"
"PrAsEn "
```

---

# 19. Useful Combination: Processing CSV-like Data

Suppose you receive:

```python
data = "Prasen,25,Developer"
```

You can parse it:

```python
name, age, role = data.split(",")

age = int(age)

print(name)
print(age)
print(role)
```

Now:

```text
name → "Prasen"
age  → 25
role → "Developer"
```

---

# 20. Useful Combination: Building a String

Suppose you have:

```python
languages = ["Python", "JavaScript", "TypeScript"]
```

You can create:

```python
result = ", ".join(languages)

print(result)
```

Output:

```text
Python, JavaScript, TypeScript
```

Then format it:

```python
print(f"I know: {result}")
```

Output:

```text
I know: Python, JavaScript, TypeScript
```

---

# 21. Important Intermediate Gotchas

### `strip()` doesn't remove all internal spaces

```python
text = "  hello   world  "

print(text.strip())
```

Result:

```text
"hello   world"
```

If you want to normalize all whitespace:

```python
text = "  hello   world  "

result = " ".join(text.split())

print(result)
# hello world
```

---

### `replace()` doesn't modify the original

```python
text = "hello"

text.replace("hello", "hi")

print(text)
# hello
```

You need:

```python
text = text.replace("hello", "hi")
```

---

### `split()` returns a list

```python
result = "a,b,c".split(",")

print(type(result))
# <class 'list'>
```

---

### `join()` is called on the separator

This:

```python
", ".join(["a", "b", "c"])
```

is correct.

Not:

```python
["a", "b", "c"].join(", ")
```

Think:

```text
SEPARATOR.join(STRINGS)
```

---

# 22. Quick Revision Cheat Sheet

```text
STRING CREATION
name = "Prasen"
name = 'Prasen'
name = """Multiline"""

INDEXING
text[0]       → first character
text[-1]      → last character

SLICING
text[1:4]     → index 1 to 3
text[:4]      → beginning to 3
text[4:]      → 4 to end
text[::2]     → every 2nd character
text[::-1]    → reverse

IMMUTABILITY
text[0] = "X" → TypeError
text = "X" + text[1:] → creates new string

METHODS
upper()       → uppercase
lower()       → lowercase
strip()       → remove edge whitespace
lstrip()      → left whitespace
rstrip()      → right whitespace
replace()     → replace substring
split()       → string → list
join()        → iterable of strings → string
startswith()  → check beginning
endswith()    → check ending

FORMATTING
f"Hello {name}"

ESCAPES
\n            → newline
\t            → tab
\\            → backslash
\"            → quote

RAW STRING
r"C:\Users\Prasen"

MULTILINE
"""
line 1
line 2
"""
```

