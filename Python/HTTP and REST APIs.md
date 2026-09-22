# HTTP and REST APIs

HTTP looks intimidating at first, but once you understand **request → method + URL → server → status + JSON response**, everything else becomes easier.

The core idea:

```text
Client (browser / Python)
    ↓
sends HTTP request
    ↓
Server (Flask / FastAPI)
    ↓
sends HTTP response
```

HTTP is just a **messaging format** between client and server. REST is a **set of conventions** for using HTTP to manage resources like users, tasks, and orders.

This file covers **theory only**. The practical Python code with `requests` is in the next file: `Requests and HTTP Clients`.

---

# 1. HTTP Fundamentals

HTTP means **HyperText Transfer Protocol**.

It is:

* Text-based
* Request-response based
* Stateless

Stateless means:

```text
Request 1
    ↓
server responds, forgets you

Request 2
    ↓
server treats it as brand new
```

If you need memory across requests, you use tokens, cookies, or sessions on top of HTTP.

---

## Request vs Response

Every HTTP interaction has two halves:

```text
REQUEST                    RESPONSE
-------                    --------
Method: GET                Status: 200 OK
URL: /users/5              Headers: Content-Type: application/json
Headers: ...               Body: {"id": 5, "name": "Kakade"}
Body: (often empty)
```

Think:

```text
Request
    │
    ├── WHAT to do (method)
    ├── WHERE to do it (URL)
    ├── EXTRA info (headers)
    └── DATA (body, optional)

Response
    │
    ├── DID it work (status code)
    ├── EXTRA info (headers)
    └── DATA (body, usually JSON)
```

---

# 2. Resources and URLs

In REST, everything is a **resource**.

Examples:

```text
/users
/tasks
/orders/123
```

A URL has parts:

```text
https://api.example.com/users/5?active=true
│        │                │        │
│        │                │        └── query parameter
│        │                └── path parameter (5)
│        └── host
└── scheme
```

Rule of thumb for interviews:

```text
/users      → collection
/users/5    → single item
```

---

# 3. GET

`GET` reads data. It does not change anything.

Example:

```text
GET /users
```

means **give me all users**.

Example:

```text
GET /users/5
```

means **give me user 5**.

Rules:

* No request body
* Safe to repeat
* Can be cached and bookmarked

Python preview (full details in next file):

```python
import requests

response = requests.get("https://api.example.com/users/5")
```

---

# 4. POST

`POST` creates new data.

Example:

```text
POST /users
Body: {"name": "Kakade"}
```

means **create a user named Kakade**.

Rules:

* Has a request body
* Not safe to repeat blindly (may create duplicates)
* Server usually returns `201 Created`

```text
POST /users
    ↓
creates new resource
    ↓
response contains new id
```

---

# 5. PUT

`PUT` replaces an entire resource.

Example:

```text
PUT /users/5
Body: {"name": "Kakade", "email": "kakade@example.com"}
```

You must send the **complete object**, not just changed fields.

Think:

```text
old user 5
    ↓
PUT with full new object
    ↓
old object fully replaced
```

`PUT` is **idempotent**. Calling it 1 time or 5 times gives the same final result.

---

# 6. PATCH

`PATCH` updates **part** of a resource.

Example:

```text
PATCH /users/5
Body: {"email": "new@example.com"}
```

Only email changes. Name stays the same.

Compare:

```text
PUT
    ↓
replace whole object

PATCH
    ↓
update only given fields
```

Interview trap: many beginners say PUT and PATCH are the same. They are not.

| Method | Sends | Effect |
| ------ | ----- | ------ |
| `PUT` | Full object | Replace |
| `PATCH` | Partial object | Update |

---

# 7. DELETE

`DELETE` removes a resource.

Example:

```text
DELETE /users/5
```

Usually has no body. Success is often `200` or `204 No Content`.

```text
DELETE /users/5
    ↓
first call → deletes user

second call → 404, already gone
```

---

# 8. HTTP Headers

Headers are extra key-value info sent with request or response.

Common request headers:

```text
Content-Type: application/json
Authorization: Bearer <token>
Accept: application/json
```

Common response headers:

```text
Content-Type: application/json
Set-Cookie: session=abc123
```

Example:

```text
GET /users
Authorization: Bearer abc123
Accept: application/json
```

Think:

```text
URL → WHERE
Method → WHAT
Headers → HOW / WHO
Body → DATA
```

---

# 9. Query Parameters

Query parameters filter, sort, or paginate.

Example:

```text
GET /users?role=admin&active=true
```

Another:

```text
GET /tasks?page=2&limit=10
```

Rules:

* Start with `?`
* Separated by `&`
* Visible in URL
* Good for filtering, bad for secrets

Do not put passwords in query params. They get logged in browser history and server logs.

---

# 10. Path Parameters

Path parameters identify a specific resource.

Example:

```text
/users/5
│      │
│      └── path parameter: id = 5
└── resource
```

Another:

```text
/orders/2026/09
```

Compare:

```text
Path param: /users/5
    ↓
WHICH resource

Query param: /users?role=admin
    ↓
WHICH SUBSET / HOW to filter
```

In Flask:

```python
@app.get("/users/<int:user_id>")
def get_user(user_id):
    ...
```

In FastAPI:

```python
@app.get("/users/{user_id}")
def get_user(user_id: int):
    ...
```

---

# 11. Request Body

The body carries data for POST, PUT, PATCH.

Almost always JSON for modern APIs:

```json
{
  "name": "Kakade",
  "email": "kakade@example.com"
}
```

GET and DELETE usually have **no body**.

```text
GET     → params in URL
POST    → data in body
PUT     → data in body
PATCH   → partial data in body
DELETE  → usually no body
```

The `Content-Type` header tells the server how to parse the body:

```text
Content-Type: application/json
```

---

# 12. Cookies

Cookies store small data in the browser. Server sends them, browser sends them back.

Flow:

```text
1. Client logs in
       ↓
2. Server responds:
   Set-Cookie: session=abc123
       ↓
3. Browser saves cookie
       ↓
4. Next request sends:
   Cookie: session=abc123
```

Use cases:

* Login sessions
* Preferences
* Tracking

For mobile apps and modern REST APIs, tokens in `Authorization` headers are more common than cookies.

---

# 13. Authentication Headers

Most APIs need to know **who you are**.

Common pattern:

```text
Authorization: Bearer <token>
```

Example:

```text
GET /tasks
Authorization: Bearer eyJhbGciOiJIUzI1NiJ9...
```

Other schemes:

```text
Basic <base64 user:pass>
ApiKey <key>
```

Flow:

```text
login with username/password
    ↓
server returns token
    ↓
client sends token in every request
    ↓
server verifies token
```

JWT and OAuth2 details are in FastAPI Intermediate. For now remember the header format.

---

# 14. JSON APIs

JSON is the standard body format for REST.

Python dict:

```python
{"id": 5, "name": "Kakade", "active": True}
```

JSON string:

```json
{"id": 5, "name": "Kakade", "active": true}
```

Notice:

```text
Python True  → JSON true
Python None  → JSON null
Python dict  → JSON object
```

Rules:

* Keys are always strings
* Values can be string, number, boolean, null, array, object
* `Content-Type` must be `application/json`

---

# 15. HTTP Status Codes

Status codes tell you **what happened**.

Groups:

```text
2xx → success
4xx → client mistake
5xx → server mistake
```

You must know these for interviews:

## Success: 200 / 201

```text
200 OK
    ↓
GET succeeded
PUT/PATCH succeeded
DELETE succeeded
```

```text
201 Created
    ↓
POST succeeded
new resource created
```

Example:

```text
POST /users → 201 Created
GET /users/5 → 200 OK
```

Some APIs also use `204 No Content` for successful DELETE with empty body.

---

## Client errors: 400 / 401 / 403 / 404 / 409 / 422

```text
400 Bad Request
    ↓
invalid JSON, missing fields
```

```text
401 Unauthorized
    ↓
not logged in / bad token
```

Think: **401 = WHO are you?**

```text
403 Forbidden
    ↓
logged in, but no permission
```

Think: **403 = you CANNOT do this.**

Example:

```python
# Kakade tries to delete admin user
# Logged in, but not admin
# → 403
```

```text
404 Not Found
    ↓
/users/9999 does not exist
```

```text
409 Conflict
    ↓
duplicate email, already exists
```

Example:

```text
POST /users with existing email
→ 409 Conflict
```

```text
422 Unprocessable Entity
    ↓
valid JSON, but validation failed
```

FastAPI uses `422` heavily with Pydantic. Example: sending `"age": "not-a-number"`.

Quick trick:

```text
401 → login missing/invalid
403 → permission missing
404 → resource missing
409 → duplicate/conflict
422 → validation failed
400 → general bad request
```

---

## Server error: 500

```text
500 Internal Server Error
    ↓
server crashed, bug in code
```

Client did everything right, server failed.

Never return `500` intentionally. It means unhandled exception.

---

# Real-World Example

Create user, then fetch it:

```text
POST /users
Content-Type: application/json

{"name": "Kakade"}
```

Response:

```text
201 Created
{"id": 5, "name": "Kakade"}
```

Then:

```text
GET /users/5
Authorization: Bearer abc123
```

Response:

```text
200 OK
{"id": 5, "name": "Kakade"}
```

If user does not exist:

```text
GET /users/9999
```

Response:

```text
404 Not Found
{"detail": "User not found"}
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **Request** | Method + URL + headers + body |
| **Response** | Status + headers + body |
| **GET** | Read, no body |
| **POST** | Create, full body → 201 |
| **PUT** | Replace whole object |
| **PATCH** | Update partial fields |
| **DELETE** | Remove, often 200/204 |
| **Headers** | Metadata, auth, content type |
| **Query params** | `?key=value`, filtering |
| **Path params** | `/users/5`, identifies resource |
| **Body** | JSON data for POST/PUT/PATCH |
| **Cookies** | Browser-stored session data |
| **Auth header** | `Authorization: Bearer <token>` |
| **200** | OK |
| **201** | Created |
| **400/422** | Bad request / validation |
| **401** | Not logged in |
| **403** | No permission |
| **404** | Not found |
| **409** | Conflict / duplicate |
| **500** | Server bug |

## The Mental Model

```text
           HTTP
            │
            ↓
   Request vs Response
            │
    ┌───────┴───────┐
    ↓               ↓
  Method           Status
    │               │
    ├── GET → 200   ├── 2xx success
    ├── POST → 201  ├── 4xx client error
    ├── PUT         └── 5xx server error
    ├── PATCH
    └── DELETE
            │
            ↓
   URL structure
            │
    ├── Path: /users/5
    ├── Query: ?active=true
    ├── Headers: auth, content-type
    ├── Body: JSON
    └── Cookies: session
```

### The 3 things to remember

**1. Methods have jobs:**

```text
GET = read, POST = create, PUT = replace, PATCH = update, DELETE = remove
```

**2. URL parts have jobs:**

```text
Path = which resource, Query = filter, Body = data, Headers = metadata/auth
```

**3. Status tells the story:**

```text
2xx = worked, 4xx = your fault, 5xx = server fault
```

> **HTTP is a request-response format; REST uses methods, URLs, headers, and status codes to manage resources with JSON.**
