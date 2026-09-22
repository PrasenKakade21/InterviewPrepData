# Requests and HTTP Clients — Revision Notes (Intermediate)
> One-line mental model: Method + URL + params/headers/json + timeout → Response (status/json/text) with explicit error handling.
## Core recall
- Verbs: `get(url, params=)`, `post(url, json=)→201`, `put` full replace, `patch` partial, `delete` no body.
- Send: `params={...}` builds `?k=v` (encoded); `headers={...}` metadata/auth; `json={...}` → JSON + content-type; `data={...}` → form-encoded; `auth=(u,p)` → Basic.
- Auth: `{"Authorization": "Bearer <token>"}`; load token from `os.environ`, never hardcode.
- Timeout: always `timeout=5` (or `(3,10)` connect/read); prevents infinite hang.
- Read: `.status_code`, `.json()` (parsed dict, raises if non-JSON), `.text` (raw/HTML/debug), `.headers`, `.url`, `.ok`.
- Errors: requests never raises for 4xx/5xx — check manually or `raise_for_status()`; catch `Timeout`, `HTTPError`, parent `RequestException`.
## Must-remember code
- ```python
import requests
r = requests.get("https://api.example.com/tasks", params={"page": 2, "limit": 10}, timeout=5)
print(r.url, r.status_code)
  ```
- ```python
import requests
r = requests.post("https://api.example.com/users", json={"name": "Kakade"}, timeout=5)
print(r.status_code, r.json())  # 201
  ```
- ```python
import os, requests
tok = os.environ.get("API_TOKEN", "demo-token")
r = requests.get("https://api.example.com/tasks", headers={"Authorization": f"Bearer {tok}"}, timeout=5)
  ```
- ```python
import requests
try:
    r = requests.get("https://api.example.com/users/5", timeout=5)
    r.raise_for_status(); user = r.json()
except requests.exceptions.Timeout: print("slow server")
except requests.exceptions.HTTPError as e: print(f"HTTP: {e}")
except requests.exceptions.RequestException as e: print(f"fail: {e}")
  ```
## Interview traps
- `json=` vs `data=`: json sends `application/json`; data sends form-encoded — wrong one breaks APIs.
- No timeout → hangs forever; always set `timeout`.
- `.json()` on non-JSON body raises; check content-type or try/except.
- `raise_for_status()` needed — 404/500 don't raise by themselves.
## 3 things to remember
1. Send: `params/headers/json` + `timeout=5`.
2. Read: `status_code/json()/text/headers`.
3. Fail safe: `raise_for_status()` in try/except (Timeout/HTTPError).
