# Requests and HTTP Clients

If HTTP theory is the grammar, `requests` is how you **speak it in Python**.

The core idea:

```text
requests.get() / .post() / ...
    ↓
sends HTTP request
    ↓
returns Response object
    ↓
you read status, headers, JSON
```

This file is **practical only**. For methods, status codes, and headers theory, read `HTTP and REST APIs` first.

Install first:

```bash
pip install requests
```

---

# 1. `requests.get()`

`GET` reads data.

```python
import requests

response = requests.get("https://jsonplaceholder.typicode.com/users/1")

print(response.status_code)
print(response.json())
```

Output:

```text
200
{'id': 1, 'name': 'Leanne Graham', ...}
```

Think:

```text
requests.get(url)
    │
    ↓
Response object
    │
    ├── .status_code
    ├── .json()
    ├── .text
    └── .headers
```

`GET` has no body. Data goes in URL or params.

---

# 2. `requests.post()`

`POST` creates data. Send JSON with `json=`.

```python
import requests

payload = {"name": "Kakade", "email": "kakade@example.com"}

response = requests.post(
    "https://jsonplaceholder.typicode.com/users",
    json=payload
)

print(response.status_code)
print(response.json())
```

Output:

```text
201
{'name': 'Kakade', 'email': 'kakade@example.com', 'id': 11}
```

Notice:

```python
json=payload
```

not:

```python
data=payload
```

`json=` automatically converts dict to JSON and sets `Content-Type: application/json`.

---

# 3. `requests.put()`

`PUT` replaces the whole resource.

```python
import requests

payload = {"name": "Kakade", "email": "new@example.com"}

response = requests.put(
    "https://jsonplaceholder.typicode.com/users/1",
    json=payload
)

print(response.status_code)
print(response.json())
```

Output:

```text
200
{'name': 'Kakade', 'email': 'new@example.com', ...}
```

---

# 4. `requests.patch()`

`PATCH` updates partial fields.

```python
import requests

response = requests.patch(
    "https://jsonplaceholder.typicode.com/users/1",
    json={"email": "new@example.com"}
)

print(response.status_code)
print(response.json())
```

Output:

```text
200
{..., 'email': 'new@example.com', ...}
```

Compare:

```text
PUT  → send full object
PATCH → send only changed fields
```

---

# 5. `requests.delete()`

```python
import requests

response = requests.delete(
    "https://jsonplaceholder.typicode.com/users/1"
)

print(response.status_code)
```

Output:

```text
200
```

No body needed.

---

# 6. Query Parameters

Bad: manually building URL strings.

```python
url = "https://api.example.com/tasks?page=2&limit=10"
```

Good: use `params=`.

```python
import requests

response = requests.get(
    "https://api.example.com/tasks",
    params={"page": 2, "limit": 10}
)

print(response.url)
```

Output:

```text
https://api.example.com/tasks?page=2&limit=10
```

`requests` encodes everything correctly. It handles spaces and special characters.

Another example:

```python
import requests

response = requests.get(
    "https://api.example.com/users",
    params={"role": "admin", "active": True}
)
```

Think:

```text
params={...}
    ↓
?key=value&key=value
    ↓
appended to URL
```

---

# 7. Headers

Send extra metadata with `headers=`.

```python
import requests

response = requests.get(
    "https://api.example.com/tasks",
    headers={"Accept": "application/json"}
)

print(response.request.headers)
```

Auth example:

```python
import requests

response = requests.get(
    "https://api.example.com/tasks",
    headers={"Authorization": "Bearer abc123"}
)
```

Custom headers are case-insensitive but conventionally Title-Case.

---

# 8. JSON Request Body

Modern APIs expect JSON.

```python
import requests

payload = {"name": "Kakade", "active": True}

response = requests.post(
    "https://api.example.com/users",
    json=payload
)
```

What `json=` does:

```text
Python dict
    ↓
json.dumps()
    ↓
JSON string + Content-Type: application/json
```

If you need to send pre-encoded JSON:

```python
import json
import requests

response = requests.post(
    "https://api.example.com/users",
    data=json.dumps(payload),
    headers={"Content-Type": "application/json"}
)
```

Prefer `json=` unless you have a special reason.

---

# 9. Form Data

Not all APIs use JSON. Login forms often use form encoding.

```python
import requests

response = requests.post(
    "https://api.example.com/login",
    data={"username": "kakade", "password": "secret"}
)

print(response.status_code)
```

`data=` sends:

```text
Content-Type: application/x-www-form-urlencoded
```

Compare:

```text
json={...} → application/json
data={...} → form-encoded
```

File uploads use `files=` (covered in FastAPI file uploads, same concept on client).

---

# 10. Authentication

Two common patterns.

## Bearer token

```python
import requests

headers = {"Authorization": "Bearer abc123"}

response = requests.get(
    "https://api.example.com/tasks",
    headers=headers
)
```

## Basic auth shortcut

```python
import requests

response = requests.get(
    "https://api.example.com/tasks",
    auth=("kakade", "secret")
)
```

`requests` builds the `Authorization: Basic ...` header for you.

Best practice: read tokens from environment, never hardcode.

```python
import os
import requests

token = os.environ.get("API_TOKEN")

response = requests.get(
    "https://api.example.com/tasks",
    headers={"Authorization": f"Bearer {token}"}
)
```

---

# 11. Timeouts

This is an interview favorite. Always set a timeout.

Bad:

```python
response = requests.get("https://api.example.com/users")
```

If the server hangs, your code hangs forever.

Good:

```python
import requests

response = requests.get(
    "https://api.example.com/users",
    timeout=5
)
```

`timeout=5` means **wait max 5 seconds**.

```text
no timeout
    ↓
hangs forever on slow server

timeout=5
    ↓
raises Timeout after 5s
```

You can also use a tuple:

```python
response = requests.get(url, timeout=(3, 10))
```

Means 3s to connect, 10s to read.

---

# 12. Response Handling

Every call returns a `Response`.

## `response.status_code`

```python
import requests

response = requests.get("https://jsonplaceholder.typicode.com/users/1")

print(response.status_code)
```

Output:

```text
200
```

Branch on it:

```python
if response.status_code == 200:
    print("OK")
elif response.status_code == 404:
    print("Not found")
```

---

## `response.json()`

Parses JSON body into Python dict/list.

```python
import requests

response = requests.get("https://jsonplaceholder.typicode.com/users/1")

user = response.json()

print(user["name"])
```

Output:

```text
Leanne Graham
```

If body is not JSON, `.json()` raises an error. Check content type or use try/except.

---

## `response.text`

Raw body as string.

```python
import requests

response = requests.get("https://example.com")

print(response.text[:100])
```

Use `.text` for HTML, CSV, or debugging. Use `.json()` for APIs.

---

## `response.headers`

```python
import requests

response = requests.get("https://jsonplaceholder.typicode.com/users/1")

print(response.headers.get("Content-Type"))
```

Output:

```text
application/json; charset=utf-8
```

Headers are case-insensitive dict-like.

Summary:

| Attribute | Gives you |
| --------- | --------- |
| `.status_code` | 200, 404, 500 |
| `.json()` | Parsed dict/list |
| `.text` | Raw string |
| `.headers` | Response metadata |
| `.url` | Final URL |
| `.ok` | True if status < 400 |

---

# 13. HTTP Error Handling

`requests` does **not** raise for 404/500 by default. You must check.

Option 1: manual check.

```python
import requests

response = requests.get(
    "https://jsonplaceholder.typicode.com/users/9999",
    timeout=5
)

if response.status_code == 404:
    print("User not found")
```

Option 2: `raise_for_status()`.

```python
import requests

response = requests.get("https://api.example.com/users/9999", timeout=5)
response.raise_for_status()
```

If status is 4xx/5xx, this raises:

```text
requests.exceptions.HTTPError
```

Complete pattern:

```python
import requests

try:
    response = requests.get(
        "https://api.example.com/users/5",
        timeout=5
    )
    response.raise_for_status()
    user = response.json()

except requests.exceptions.Timeout:
    print("Server took too long")

except requests.exceptions.HTTPError as e:
    print(f"HTTP error: {e}")

except requests.exceptions.RequestException as e:
    print(f"Request failed: {e}")
```

Think:

```text
Timeout → server slow
HTTPError → 4xx/5xx
RequestException → parent of all requests errors
```

Always wrap API calls in try/except in real code and interviews.

---

# Real-World Example: Fetch and Report

```python
import os
import requests

API_URL = "https://jsonplaceholder.typicode.com"
TOKEN = os.environ.get("API_TOKEN", "demo-token")

try:
    response = requests.get(
        f"{API_URL}/users",
        headers={"Authorization": f"Bearer {TOKEN}"},
        timeout=5
    )
    response.raise_for_status()
    users = response.json()

    print(f"Fetched {len(users)} users")
    for user in users[:3]:
        print(user["name"])

except requests.exceptions.RequestException as e:
    print(f"Failed: {e}")
```

This combines params, headers, timeout, status check, and JSON parsing — exactly what interviewers look for.

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **`requests.get()`** | Read, use `params=` |
| **`requests.post()`** | Create, use `json=` → 201 |
| **`requests.put()`** | Replace full object |
| **`requests.patch()`** | Partial update |
| **`requests.delete()`** | Remove |
| **Query params** | `params={...}` |
| **Headers** | `headers={...}` |
| **JSON body** | `json={...}` |
| **Form data** | `data={...}` |
| **Auth** | `Authorization: Bearer` header |
| **Timeouts** | Always set `timeout=5` |
| **`.status_code`** | Numeric result |
| **`.json()`** | Parsed body |
| **`.text`** | Raw string |
| **`.headers`** | Metadata dict |
| **Errors** | `raise_for_status()` + try/except |

## The Mental Model

```text
        REQUESTS
            │
            ↓
  method + url + options
            │
  ┌─────────┼─────────┐
  ↓         ↓         ↓
params   headers    json/data
  │         │         │
  ↓         ↓         ↓
 ?a=1    Auth/token  Body
            │
            ↓
       Response
            │
  ┌─────────┼─────────┐
  ↓         ↓         ↓
status    json/text  headers
  │
  ↓
raise_for_status()
  │
  ↓
try/except Timeout + HTTPError
```

### The 3 things to remember

**1. Send correctly:**

```python
requests.get(url, params={...}, headers={...}, timeout=5)
requests.post(url, json={...}, headers={...}, timeout=5)
```

**2. Read correctly:**

```python
response.status_code
response.json()
response.text
```

**3. Fail safely:**

```python
response.raise_for_status()
```

inside try/except with timeout.

> **The `requests` library turns HTTP theory into Python: send methods with params/headers/JSON, always set timeouts, and handle responses with status checks and error handling.**
