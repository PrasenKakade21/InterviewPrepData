
A **dictionary (`dict`)** stores data as **key-value pairs**.

Instead of accessing data by position like a list:

```python
users[0]
```

you access it using a **key**:

```python
user["name"]
```

Dictionaries are one of the most important Python data structures, especially when working with **JSON, APIs, databases, Flask/FastAPI, configuration, and automation**.

---

# 1. Key-Value Pairs

A dictionary looks like this:

```python
user = {
    "name": "Prasen",
    "age": 25,
    "role": "Developer"
}
```

The structure is:

```text
key       value
 ↓          ↓
"name" → "Prasen"
"age"  → 25
"role" → "Developer"
```

Each key maps to a value.

You can have different types of values:

```python
user = {
    "name": "Prasen",
    "age": 25,
    "is_active": True,
    "skills": ["Python", "React"],
}
```

Keys are commonly strings, but they can also be other **hashable** types:

```python
data = {
    1: "one",
    2: "two",
    (10, 20): "coordinate"
}
```

Lists cannot be dictionary keys because they're mutable:

```python
data = {
    [1, 2]: "value"  # TypeError
}
```

---

## Dictionary keys must be unique

```python
user = {
    "name": "Prasen",
    "name": "Rahul"
}

print(user)
```

Result:

```python
{"name": "Rahul"}
```

The second `"name"` replaces the first one.

---

# 2. Accessing Values

Use the key inside square brackets:

```python
user = {
    "name": "Prasen",
    "age": 25
}

print(user["name"])
# Prasen

print(user["age"])
# 25
```

You can also store the result:

```python
name = user["name"]
```

---

## What happens if the key doesn't exist?

```python
print(user["email"])
```

Python raises:

```text
KeyError: 'email'
```

This is where `get()` becomes useful.

---

# 3. `get()`

`get()` allows you to retrieve a value **without raising `KeyError`** if the key doesn't exist.

```python
user = {
    "name": "Prasen",
    "age": 25
}

print(user.get("name"))
# Prasen

print(user.get("email"))
# None
```

You can provide a default value:

```python
email = user.get("email", "Not provided")

print(email)
# Not provided
```

This is especially useful when processing API/JSON data where a field may be missing.

For example:

```python
response = {
    "name": "Prasen",
    "age": 25
}

email = response.get("email", "")
```

Instead of:

```python
email = response["email"]
```

which would crash if `"email"` doesn't exist.

### `[]` vs `get()`

| Code                       | Missing key       |
| -------------------------- | ----------------- |
| `user["email"]`            | Raises `KeyError` |
| `user.get("email")`        | Returns `None`    |
| `user.get("email", "N/A")` | Returns `"N/A"`   |

---

# 4. `keys()`

`keys()` returns the dictionary's keys.

```python
user = {
    "name": "Prasen",
    "age": 25,
    "role": "Developer"
}

print(user.keys())
```

You get a dictionary view:

```text
dict_keys(['name', 'age', 'role'])
```

You can iterate over it:

```python
for key in user.keys():
    print(key)
```

Output:

```text
name
age
role
```

Actually, you often don't even need `.keys()`:

```python
for key in user:
    print(key)
```

This is equivalent and more idiomatic.

---

# 5. `values()`

`values()` gives you the values.

```python
user = {
    "name": "Prasen",
    "age": 25,
    "role": "Developer"
}

print(user.values())
```

Output:

```text
dict_values(['Prasen', 25, 'Developer'])
```

You can iterate:

```python
for value in user.values():
    print(value)
```

Output:

```text
Prasen
25
Developer
```

You can also check membership:

```python
if "Prasen" in user.values():
    print("User found")
```

---

# 6. `items()`

`items()` gives you **key-value pairs**.

```python
user = {
    "name": "Prasen",
    "age": 25,
    "role": "Developer"
}

print(user.items())
```

Output:

```text
dict_items([
    ('name', 'Prasen'),
    ('age', 25),
    ('role', 'Developer')
])
```

The pairs are represented as tuples.

This is very commonly used with dictionary unpacking in loops:

```python
for key, value in user.items():
    print(key, "=", value)
```

Output:

```text
name = Prasen
age = 25
role = Developer
```

This is one of the most important dictionary patterns to remember:

```python
for key, value in dictionary.items():
    ...
```

---

# 7. `update()`

`update()` adds or modifies multiple key-value pairs.

```python
user = {
    "name": "Prasen",
    "age": 25
}

user.update({
    "role": "Developer",
    "city": "Mumbai"
})

print(user)
```

Result:

```python
{
    "name": "Prasen",
    "age": 25,
    "role": "Developer",
    "city": "Mumbai"
}
```

If the key already exists, it gets updated:

```python
user.update({
    "age": 26
})
```

Now:

```python
{
    "name": "Prasen",
    "age": 26,
    "role": "Developer",
    "city": "Mumbai"
}
```

You can also pass keyword arguments:

```python
user.update(
    role="Senior Developer",
    city="Pune"
)
```

---

## Updating from another dictionary

```python
defaults = {
    "theme": "dark",
    "language": "en"
}

settings = {
    "theme": "light"
}

settings.update(defaults)

print(settings)
```

The resulting value for `"theme"` is `"dark"` because the later update wins.

---

# 8. `pop()`

`pop()` removes a key and **returns its value**.

```python
user = {
    "name": "Prasen",
    "age": 25,
    "role": "Developer"
}

role = user.pop("role")

print(role)
# Developer

print(user)
# {'name': 'Prasen', 'age': 25}
```

This is useful when you want to both **remove and retrieve** something.

---

## `pop()` with a default

If the key doesn't exist:

```python
user.pop("email")
```

you get:

```text
KeyError: 'email'
```

You can provide a default:

```python
email = user.pop("email", None)

print(email)
# None
```

This is similar to the difference between `[]` and `get()`.

---

# 9. Nested Dictionaries

A dictionary can contain another dictionary as a value.

```python
user = {
    "name": "Prasen",
    "profile": {
        "age": 25,
        "role": "Developer",
        "city": "Mumbai"
    }
}
```

You access nested data step by step:

```python
print(user["profile"])
```

Result:

```python
{
    "age": 25,
    "role": "Developer",
    "city": "Mumbai"
}
```

To access `"city"`:

```python
print(user["profile"]["city"])
# Mumbai
```

You can go multiple levels deep:

```python
data = {
    "user": {
        "profile": {
            "location": {
                "city": "Mumbai"
            }
        }
    }
}

print(data["user"]["profile"]["location"]["city"])
# Mumbai
```

---

## Nested dictionaries are extremely common with JSON

For example, an API might return:

```python
response = {
    "id": 101,
    "name": "Prasen",
    "company": {
        "name": "Example Corp",
        "address": {
            "city": "Mumbai",
            "country": "India"
        }
    }
}
```

You can access:

```python
company = response["company"]

print(company["name"])
# Example Corp

print(response["company"]["address"]["city"])
# Mumbai
```

This is something you'll encounter constantly when working with **REST APIs, FastAPI, Flask, and JavaScript/TypeScript JSON data**.

---

## Safely accessing nested dictionaries

Suppose `"profile"` might not exist.

This can crash:

```python
city = user["profile"]["city"]
```

You can use `get()`:

```python
city = user.get("profile", {}).get("city")
```

If either key doesn't exist, you'll get `None` instead of `KeyError`.

For example:

```python
user = {
    "name": "Prasen"
}

city = user.get("profile", {}).get("city")

print(city)
# None
```

The `{}` acts as a fallback empty dictionary.

---

# 10. Dictionary Unpacking

Dictionary unpacking uses `**`.

Suppose:

```python
user = {
    "name": "Prasen",
    "age": 25
}
```

You can unpack it into another dictionary:

```python
user_with_role = {
    **user,
    "role": "Developer"
}

print(user_with_role)
```

Result:

```python
{
    "name": "Prasen",
    "age": 25,
    "role": "Developer"
}
```

Think of:

```python
**user
```

as:

> "Take all key-value pairs from `user` and put them here."

---

## Combining dictionaries

```python
basic_info = {
    "name": "Prasen",
    "age": 25
}

job_info = {
    "role": "Developer",
    "company": "Example"
}

user = {
    **basic_info,
    **job_info
}

print(user)
```

Result:

```python
{
    "name": "Prasen",
    "age": 25,
    "role": "Developer",
    "company": "Example"
}
```

This is very useful for creating a new dictionary **without modifying the originals**.

---

## Duplicate keys when unpacking

The later value wins:

```python
defaults = {
    "theme": "dark",
    "language": "en"
}

settings = {
    **defaults,
    "theme": "light"
}

print(settings)
```

Result:

```python
{
    "theme": "light",
    "language": "en"
}
```

Because:

```text
defaults → theme = dark
              ↓
new value → theme = light
```

---

# Dictionary Unpacking in Function Calls

This is particularly important when working with Python APIs.

Suppose:

```python
def create_user(name, age, role):
    print(name, age, role)
```

And you have:

```python
user = {
    "name": "Prasen",
    "age": 25,
    "role": "Developer"
}
```

You can pass the dictionary using `**`:

```python
create_user(**user)
```

Python effectively converts it into:

```python
create_user(
    name="Prasen",
    age=25,
    role="Developer"
)
```

This pattern is extremely common in Python frameworks.

---

# Dictionary Iteration

You should be comfortable with these three patterns:

### Keys

```python
for key in user:
    print(key)
```

### Values

```python
for value in user.values():
    print(value)
```

### Keys + values

```python
for key, value in user.items():
    print(key, value)
```

The third one is probably the most useful.

---

# Practical Example

Imagine you're processing an API response:

```python
user = {
    "id": 101,
    "name": "Prasen",
    "email": "prasen@example.com",
    "skills": ["Python", "React"],
    "profile": {
        "role": "Full Stack Developer",
        "experience": 3
    }
}
```

### Get the name

```python
name = user["name"]
```

### Safely get phone number

```python
phone = user.get("phone")
```

### Get role from nested dictionary

```python
role = user["profile"]["role"]
```

### Get all fields

```python
for key, value in user.items():
    print(key, value)
```

### Add a field

```python
user["location"] = "India"
```

### Update fields

```python
user.update({
    "experience": 4,
    "is_active": True
})
```

### Remove a field

```python
email = user.pop("email")
```

### Create a modified copy

```python
updated_user = {
    **user,
    "role": "Senior Full Stack Developer"
}
```

This last pattern is especially useful when you want to treat your original dictionary as unchanged.

---

# Important Dictionary Concepts to Remember

### Basic structure

```python
user = {
    "name": "Prasen",
    "age": 25
}
```

### Access

```python
user["name"]
```

### Safe access

```python
user.get("name")
user.get("email", "Not provided")
```

### Keys

```python
user.keys()
```

### Values

```python
user.values()
```

### Key + value

```python
user.items()
```

### Update

```python
user.update({"age": 26})
```

### Remove + return value

```python
user.pop("age")
```

### Nested access

```python
user["profile"]["city"]
```

### Dictionary unpacking

```python
new_user = {
    **user,
    "role": "Developer"
}
```

### Function argument unpacking

```python
create_user(**user)
```

---

## The Mental Model

Think of the four major Python collections this way:

```text
LIST
[10, 20, 30]
  ↓
Ordered collection where position matters


TUPLE
(10, 20, 30)
  ↓
Ordered collection that shouldn't change


SET
{10, 20, 30}
  ↓
Unique collection where membership/relationships matter


DICT
{
    "name": "Prasen",
    "age": 25
}
  ↓
Collection where you identify data by a key
```

For your backend/API work, **dictionaries are particularly important** because JSON objects map very naturally to Python dictionaries:

```json
{
  "name": "Prasen",
  "age": 25
}
```

becomes:

```python
{
    "name": "Prasen",
    "age": 25
}
```

So I'd consider `dict`, `dict.get()`, `.items()`, nested dictionaries, and `**` unpacking **must-know Python fundamentals** before moving deeper.
