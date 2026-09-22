# HTTP and REST APIs — Revision Notes (Intermediate)
> One-line mental model: Request (method+URL+headers+body) → server → response (status+headers+JSON); REST maps methods to resources.
## Core recall
- HTTP: text-based, request-response, stateless (tokens/cookies add memory); request = method+URL+headers+body, response = status+headers+body.
- Resources: `/users` collection, `/users/5` item; scheme/host/path-param/query-param parts.
- Methods: GET read (no body, cacheable), POST create (body → 201), PUT replace full (idempotent), PATCH partial update, DELETE remove (200/204, then 404).
- PUT vs PATCH: PUT sends full object/replace; PATCH sends partial/update.
- Params: path = which resource (`/users/5`), query = filter (`?role=admin&page=2`, starts `?`, joined `&`, never secrets).
- Headers/auth: `Content-Type: application/json`, `Accept`, `Authorization: Bearer <token>`; cookies = browser session via Set-Cookie.
## Must-remember code
- ```python
import requests
r = requests.get("https://api.example.com/users/5", timeout=5)
print(r.status_code, r.json())
  ```
- ```python
import requests
r = requests.post("https://api.example.com/users", json={"name": "Kakade"}, timeout=5)
print(r.status_code)  # 201
  ```
- ```python
# PUT replaces whole object; PATCH updates one field
import requests
requests.put("https://api.example.com/users/5", json={"name": "Kakade", "email": "k@x.com"})
requests.patch("https://api.example.com/users/5", json={"email": "new@x.com"})
  ```
- ```python
# Flask path param vs query param
from flask import request
uid = 5  # from /users/<int:uid>
role = request.args.get("role", "all")  # from ?role=admin
  ```
## Interview traps
- PUT ≠ PATCH: full replace vs partial update — don't conflate.
- 401 (who are you? bad/missing token) vs 403 (no permission) vs 404/409/422/400 distinctions.
- GET/DELETE normally no body; POST/PUT/PATCH carry JSON with `Content-Type: application/json`.
- Secrets in query params leak via logs/history → use headers/body instead.
## 3 things to remember
1. GET read, POST create→201, PUT replace, PATCH update, DELETE remove.
2. Path = which, query = filter, body = data, headers = metadata/auth.
3. 2xx worked, 4xx your fault (401/403/404/409/422), 5xx server bug.
