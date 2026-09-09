## Module 1: JSON Data Processing

### Clear Explanation

**JSON** (JavaScript Object Notation) is a lightweight, human-readable data-interchange format heavily used in modern web APIs, microservices, and configuration files.

In Python, the built-in `json` module translates between JSON formatted strings or files and Python’s native data structures (`dict`, `list`, `str`, `int`, `float`, `bool`, `None`).

The module exposes four primary functions. Understanding the final letter **`s`** is the key:

* **`loads()` / `dumps()`:** Operating on **S**trings in memory.
* **`load()` / `dump()`:** Operating directly on **File streams / File objects** on disk.

---

### Python Data Types vs. JSON Equivalents

| Python Data Structure | JSON Format Equivalent |
| --- | --- |
| `dict` | Object (`{"key": "value"}`) |
| `list`, `tuple` | Array (`[1, 2, 3]`) |
| `str` | String (`"hello"`) |
| `int`, `float` | Number (`42`, `3.14`) |
| `True` / `False` | Boolean (`true` / `false`) |
| `None` | Null (`null`) |

---

### Basic Syntax

```python
import json

# String operations (loads / dumps)
python_dict = json.loads(json_string)
json_string = json.dumps(python_dict)

# File stream operations (load / dump)
with open("data.json", "r", encoding="utf-8") as f:
    python_dict = json.load(f)

with open("data.json", "w", encoding="utf-8") as f:
    json.dump(python_dict, f)

```

---

### A. Working with JSON Strings: `json.loads()` & `json.dumps()`

#### Examples

```python
import json

#  json.loads() - String to Python Dictionary
json_payload = '{"user_id": 101, "is_active": true, "roles": ["admin", "editor"], "quota": null}'

user_data = json.loads(json_payload)
print(type(user_data))
print(f"User ID: {user_data['user_id']} | Active: {user_data['is_active']}")
# Output:
# <class 'dict'>
# User ID: 101 | Active: True

#  json.dumps() - Python Dictionary to JSON String
response_dict = {
    "status": 200,
    "message": "Resource updated",
    "debug_mode": False
}

# Pretty-printing with indent and sorting keys
json_out = json.dumps(response_dict, indent=2, sort_keys=True)
print(json_out)
# Output:
# {
#   "debug_mode": false,
#   "message": "Resource updated",
#   "status": 200
# }

```

#### Explain the Code

* `json.loads()` converts `"true"` ➔ `True`, `"null"` ➔ `None`, and JSON objects ➔ native Python `dict`.
* `json.dumps(..., indent=2)` adds newlines and 2-space indentation to format raw strings for human logging. `sort_keys=True` sorts dictionary keys alphabetically.

---

### B. Working with JSON Files: `json.load()` & `json.dump()`

#### Examples

```python
import json
from pathlib import Path

config_path = Path("config.json")
data_to_save = {"theme": "dark", "timeout": 30, "retry_attempts": 3}

# Writing JSON directly to file stream using dump()
with open(config_path, "w", encoding="utf-8") as file:
    json.dump(data_to_save, file, indent=4)

# Reading JSON directly from file stream using load()
with open(config_path, "r", encoding="utf-8") as file:
    loaded_config = json.load(file)

print(f"Theme: {loaded_config['theme']} | Timeout: {loaded_config['timeout']}s")
# Output:
# Theme: dark | Timeout: 30s

```

---

### C. Parsing Complex & Nested JSON

API payloads frequently contain deeply nested combinations of dictionaries and lists. Accessing nested JSON in Python requires chaining dictionary key lookups `[]` and list indices `[]`.

#### Examples

```python
import json

api_response = '''
{
  "status": "success",
  "data": {
    "page": 1,
    "users": [
      {"id": 1, "profile": {"name": "Alice", "email": "alice@corp.com"}},
      {"id": 2, "profile": {"name": "Bob", "email": "bob@corp.com"}}
    ]
  }
}
'''

payload = json.loads(api_response)

# Safely extract the second user's email address
second_user_email = payload["data"]["users"][1]["profile"]["email"]
print(f"Second User Email: {second_user_email}")
# Output:
# Second User Email: bob@corp.com

```

#### Practical Usage (FastAPI / Flask Payload Processing)

Sanitizing and extracting user records from an API response array using list comprehensions.

```python
import json

def extract_active_user_emails(json_str):
    data = json.loads(json_str)
    # Filter users where active is True and extract email
    active_emails = [
        user["email"] 
        for user in data.get("users", []) 
        if user.get("is_active")
    ]
    return active_emails

raw_json = '''
{
  "users": [
    {"email": "a@dev.com", "is_active": true},
    {"email": "b@dev.com", "is_active": false},
    {"email": "c@dev.com", "is_active": true}
  ]
}
'''

print(extract_active_user_emails(raw_json))
# Output:
# ['a@dev.com', 'c@dev.com']

```

---

### Important Edge Cases / Gotchas

* **Unserializable Data Types (`TypeError`):** Python standard `datetime`, `set`, and custom class instances are **NOT** serializable by default in JSON!

```python
import json
import datetime

# ❌ Raises TypeError: Object of type datetime is not JSON serializable
# json.dumps({"timestamp": datetime.datetime.now()})

# ✅ FIX: Convert datetime to string ISO format or pass custom default handler
json_str = json.dumps({
    "timestamp": datetime.datetime.now().isoformat()
})
print(json_str)
# Output:
# {"timestamp": "2026-09-09T03:25:03.000000"}

```

* **Single vs. Double Quotes:** JSON standards strictly enforce **double quotes (`"`)** for keys and strings. Single-quoted strings (`'`) inside JSON raw payloads cause `json.JSONDecodeError`.

```python
import json

# ❌ Raises json.JSONDecodeError
# json.loads("{'key': 'value'}")

# ✅ Valid JSON
json.loads('{"key": "value"}')

```

---

### Mental Model

* **`loads` / `dumps`:** Think **"String in Memory"**. You are turning in-memory string text into Python objects, or turning Python objects into string text.
* **`load` / `dump`:** Think **"Disk File Storage"**. You are shoveling data back and forth through a pipe connected to a physical file on disk.

---

## Revision Cheat Sheet

###  JSON Quick Function Matrix

* **String Operations:**
* `json.loads(str)` ➔ String ➔ Dict/List
* `json.dumps(obj)` ➔ Dict/List ➔ String


* **File Operations:**
* `json.load(f)` ➔ File Handle ➔ Dict/List
* `json.dump(obj, f)` ➔ Dict/List ➔ File Handle

