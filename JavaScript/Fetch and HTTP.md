# Fetch and HTTP

`fetch` is the promise-based HTTP client in browsers and Node.js 18+. You build a request, receive a `Response`, check whether the status is a success, then read the body as JSON or text. The chain is **request → Response → status check → body read → error handling**, and the interview themes are the parts people get wrong: `fetch` does not reject on 404, the body can only be read once, and CORS failures look like network failures.

```text
fetch(url, options)
      ↓
  Request   method · headers · body · signal
      ↓
  Response  status · ok · headers · body stream
      ↓
  read once .json() / .text() / .blob()
      ↓
  data  or  thrown error
```

---

# 1. Request and Response Basics

### fetch basics

`fetch` resolves as soon as response **headers** arrive, not when the body finishes downloading. That is why you await `.json()` separately.

```javascript
async function main() {
  const res = await fetch("https://jsonplaceholder.typicode.com/todos/1");
  console.log(res.status, res.ok);
  console.log(res.headers.get("content-type"));
}

main();
```

Output:

```text
200 true
application/json; charset=utf-8
```

If the host cannot be reached, the promise **rejects** with `TypeError: fetch failed`. A `404` does not reject.

### Response object

Useful members: `status`, `statusText`, `ok` (`true` only for 200–299), `headers`, `url`, `redirected`, `bodyUsed`. You can construct one offline for tests.

```javascript
const res = new Response('{"name":"Kakade"}', {
  status: 201,
  headers: { "Content-Type": "application/json" },
});

console.log(res.status, res.ok);
console.log(res.headers.get("content-type"), res.bodyUsed);
```

Output:

```text
201 true
application/json false
```

`ok` is only a status-range check: a `304` is not `ok`, and a `500` is not a rejection.

### JSON responses

`res.json()` parses the body and returns a promise; bad JSON rejects with `SyntaxError`. A body is a **stream**, so it can be consumed only once.

```javascript
const res = Response.json({ id: 7, name: "Asha" });

await res.json();

try {
  await res.json();
} catch (err) {
  console.log(err.name, err.message);
}
```

Output:

```text
TypeError Body is unusable: Body has already been read
```

Need both forms? Read once and parse yourself, or call `res.clone()` **before** reading. Cloning buffers the whole body, so only do it for small responses. A `200` that returns HTML makes `.json()` throw `SyntaxError` — a status check alone is not enough.

---

# 2. Building Requests

### HTTP methods

The method is the verb. `GET` and `HEAD` cannot have a body; the rest can.

| Method | Meaning | Idempotent |
| --- | --- | --- |
| `GET` | Read a resource | Yes |
| `POST` | Create / submit | No |
| `PUT` | Replace whole resource | Yes |
| `PATCH` | Partial update | Not guaranteed |
| `DELETE` | Remove | Yes |

```javascript
const req = new Request("https://api.example.com/todos/1", {
  method: "PATCH",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ completed: true }),
});

console.log(req.method, req.headers.get("content-type"));
```

Output:

```text
PATCH application/json
```

Bad: a body on `GET` → `TypeError`. Good: query parameters for reads, a body for writes.

### Headers

`Headers` is a case-insensitive map. `set` replaces, `append` adds another value, and names are lowercased.

```javascript
const headers = new Headers({ "Content-Type": "application/json" });

headers.append("X-Trace-Id", "abc123");
headers.set("x-trace-id", "def456");

console.log(headers.get("X-TRACE-ID"), headers.has("Authorization"));
console.log([...headers.keys()]);
```

Output:

```text
def456 false
[ 'content-type', 'x-trace-id' ]
```

Some headers are **forbidden** from JavaScript — `Host`, `Content-Length`, `Cookie`, `Origin` — because the browser owns them.

### Request body

JSON APIs need `JSON.stringify` plus `Content-Type: application/json`. Without the header the server may treat the body as plain text.

```javascript
const req = new Request("https://api.example.com/todos", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ title: "Buy milk", owner: "Rohan" }),
});

console.log(await req.text());
```

Output:

```text
{"title":"Buy milk","owner":"Rohan"}
```

The body must be a string, `Blob`, `ArrayBuffer`, `FormData`, `URLSearchParams` or stream. Passing a plain object silently sends `"[object Object]"`. For forms, `new URLSearchParams({ name: "Kakade" }).toString()` gives `name=Kakade`, and `FormData` handles file uploads.

---

# 3. Status and Errors

### Status handling

`fetch` resolves for `404` and `500`. Only network failure or abort rejects, so `try/catch` alone misses HTTP errors.

```javascript
const res = await fetch("https://jsonplaceholder.typicode.com/todos/999999");

console.log(res.status, res.ok, res.statusText);
```

Output:

```text
404 false Not Found
```

Bad: `const data = await res.json();` may parse an error page. Good:

```javascript
if (!res.ok) throw new Error(`HTTP ${res.status} ${res.statusText}`);
const data = await res.json();
```

### Error handling

A correct wrapper checks `ok`, parses safely, and keeps the URL in the message. CORS failures reject with a generic `TypeError: Failed to fetch` — the real cause is only in the browser console.

```javascript
async function getJSON(url, options = {}) {
  const res = await fetch(url, options);
  if (!res.ok) throw new Error(`HTTP ${res.status} for ${url}`);
  return res.json();
}

getJSON("https://jsonplaceholder.typicode.com/todos/1")
  .then((todo) => console.log(todo.title))
  .catch((err) => console.error(err.message));
```

Output:

```text
delectus aut autem
```

| Situation | Result |
| --- | --- |
| Offline / DNS / CORS | rejects `TypeError` |
| Aborted | rejects `DOMException` `AbortError` |
| `4xx` / `5xx` | resolves, `ok === false` |
| Invalid JSON | `.json()` rejects `SyntaxError` |
| Timeout | only if you abort it |

For `204 No Content`, return `null` before calling `.json()`, because the empty body throws.

---

# 4. Cancellation and Timeouts

### AbortController

An `AbortController` owns a `signal`. Pass it to `fetch`; `abort()` rejects the pending promise with `AbortError`. The same signal can cancel several requests.

```javascript
const controller = new AbortController();

const promise = fetch("https://api.example.com/slow", {
  signal: controller.signal,
});

controller.abort();

try {
  await promise;
} catch (err) {
  console.log(err.name);
}
```

Output:

```text
AbortError
```

Aborting after headers arrive also cancels the body stream. A signal is **one-shot**: once aborted, every future request using it fails immediately — create one per request.

### Timeouts

`fetch` has no `timeout` option. Abort from a timer and clear it in `finally`, otherwise Node keeps the event loop alive.

```javascript
async function fetchWithTimeout(url, ms = 5000) {
  const controller = new AbortController();
  const timer = setTimeout(() => controller.abort(), ms);

  try {
    return await fetch(url, { signal: controller.signal });
  } finally {
    clearTimeout(timer);
  }
}

fetchWithTimeout("https://api.example.com/slow", 100)
  .catch((err) => console.log(err.name));
```

Output:

```text
AbortError
```

Notice: `AbortSignal.timeout(5000)` rejects with `TimeoutError`. Combine it with a user cancel via `AbortSignal.any([user.signal, AbortSignal.timeout(5000)])` — the first abort wins.

---

# 5. CORS and Credentials

### CORS

An **origin** is protocol + host + port. The browser blocks cross-origin reads unless the server replies with matching CORS headers. CORS is browser-only: `curl`, Postman and Node `fetch` ignore it.

```javascript
fetch("https://api.example.com/me")
  .then((res) => res.json())
  .then(console.log)
  .catch((err) => console.log(err.message));
```

Output:

```text
Failed to fetch
```

A non-simple request (`PUT`, `PATCH`, `DELETE`, JSON content type, custom header) first sends an `OPTIONS` **preflight**. The server must allow the origin, method and headers, or the real request is never sent. `Access-Control-Max-Age` caches that answer.

### Credentials

`credentials` decides whether cookies travel with the request.

| Value | Cookies | Use |
| --- | --- | --- |
| `"omit"` | Never | Public assets |
| `"same-origin"` (default) | Same origin | Normal app calls |
| `"include"` | Cross-origin too | Session cookie on API subdomain |

```javascript
fetch("https://api.example.com/me", {
  credentials: "include",
  headers: { Accept: "application/json" },
});
```

Output:

```text
Cookie is sent.
Response is readable only if the server sends
Access-Control-Allow-Origin: https://app.example.com
and Access-Control-Allow-Credentials: true
```

Trap: with credentials, `Access-Control-Allow-Origin: *` is invalid. Also `SameSite=Lax` cookies may not be sent cross-site even with `include`; that needs `SameSite=None; Secure`.

---

# 6. REST Patterns

### REST patterns

REST treats data as **resources** addressed by noun URLs, with the method as the verb. Avoid `/getTodos`.

Think:

```text
/todos        → the collection
/todos/1      → one item
GET /todos    → read the collection
```

| Goal | Request |
| --- | --- |
| List | `GET /todos` |
| Read one | `GET /todos/1` |
| Create | `POST /todos` |
| Replace | `PUT /todos/1` |
| Update part | `PATCH /todos/1` |
| Delete | `DELETE /todos/1` |

```javascript
async function request(path, options = {}) {
  const res = await fetch(`https://jsonplaceholder.typicode.com${path}`, {
    ...options,
    headers: { Accept: "application/json", ...options.headers },
  });

  if (!res.ok) throw new Error(`HTTP ${res.status}`);
  return res.status === 204 ? null : res.json();
}

request("/todos/1").then((todo) => console.log(todo.id));
```

Output:

```text
1
```

Notice the spread order: `...options` first, then `headers` rebuilt. Spreading `options` last would drop `Accept`. Build query strings with `URL` and `searchParams.set("q", "revise fetch")`; manual concatenation breaks on `&` and spaces. `PUT` is idempotent, so retries are safe; retrying `POST` can duplicate.

---

# Real-World Uses

- **UI data fetching** — spinner, `res.ok` check, error state.
- **Form submission** — `POST` JSON or `FormData`, disable the button while pending.
- **Search-as-you-type** — abort the previous request so stale results cannot win.
- **Infinite scroll** — `GET /posts?_page=3&_limit=20`, append the page.

---

# Quick Revision

| Concept | Remember |
| --- | --- |
| **fetch basics** | Promise for a `Response`; resolves on headers, not full body |
| **Response object** | `status`, `ok` (200–299), `headers`, `bodyUsed` |
| **JSON responses** | `res.json()` parses and consumes; second read throws |
| **HTTP methods** | GET read, POST create, PUT replace, PATCH update, DELETE remove |
| **Headers** | Case-insensitive; `set` replaces, `append` adds; some forbidden |
| **Request body** | String/Blob/FormData/URLSearchParams; JSON needs the content type |
| **Status handling** | 4xx/5xx resolve; check `res.ok` yourself |
| **AbortController** | `signal` + `abort()` → `AbortError`; one-shot |
| **Timeouts** | Timer + abort + `clearTimeout`, or `AbortSignal.timeout` |
| **CORS** | Browser-enforced; server must allow the origin; Node ignores it |
| **Credentials** | `omit` / `same-origin` / `include`; `include` forbids `*` |
| **Error handling** | Only network/abort reject; wrap status and parse errors |
| **REST patterns** | Noun URLs + verbs; `PUT` idempotent, `POST` not |

## The Mental Model

```text
                    fetch(url, options)
                            │
              ┌─────────────┴─────────────┐
              ↓                           ↓
          Request                      network
   method / headers / body          DNS / TCP / TLS
   signal / credentials                  │
              │                           ↓
              └────────────────────→  Response
                                          │
                         ┌────────────────┼────────────────┐
                         ↓                ↓                ↓
                     status/ok        headers           body
                         │                │                │
                    !ok → throw      read values      read ONCE
                         │                                 │
                         └──────────────→ data ←───────────┘
                                            │
                                   CORS gate (browser)
```

### The 3 things to remember

**1. `fetch` does not reject on HTTP errors.**

```javascript
const res = await fetch(url);
if (!res.ok) throw new Error(`HTTP ${res.status}`);
```

**2. The body is consumed once.**

```javascript
const data = await res.json(); // a second call throws TypeError
```

**3. Cancel with a signal.**

```javascript
const controller = new AbortController();
setTimeout(() => controller.abort(), 5000);
await fetch(url, { signal: controller.signal });
```

> **`fetch` gives you a `Response`, not an error: check `ok`, read the body once, and abort explicitly when time or the user says stop.**
