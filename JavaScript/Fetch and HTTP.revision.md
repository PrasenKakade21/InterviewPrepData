# Fetch and HTTP — Revision Notes (Intermediate)

> One-line mental model: `fetch` gives you a `Response`, not an error — check `ok`, read the body once, abort with a signal.

## Core recall

- **fetch basics**: returns a promise for a `Response`; resolves when headers arrive, not when the body finishes. Network/DNS failure → `TypeError: fetch failed`.
- **Response object**: `status`, `statusText`, `ok` (true only 200–299), `headers`, `url`, `redirected`, `bodyUsed`.
- **JSON responses**: `res.json()` parses and **consumes** the stream; second read throws `TypeError: Body is unusable`. Invalid JSON → `SyntaxError`.
- **HTTP methods**: GET read, POST create, PUT replace, PATCH partial, DELETE remove. GET/HEAD cannot have a body.
- **Headers**: `Headers` is case-insensitive; `set` replaces, `append` adds. `Host`, `Cookie`, `Origin`, `Content-Length` are forbidden from JS.
- **Request body**: string, `Blob`, `ArrayBuffer`, `FormData`, `URLSearchParams`, stream. JSON needs `JSON.stringify` + `Content-Type: application/json`.
- **Status handling**: 4xx/5xx **resolve**; `res.ok` is the check. `204` has no body — return `null` before `.json()`.
- **AbortController**: pass `controller.signal`, call `abort()` → `DOMException` `AbortError`. One signal is one-shot.
- **Timeouts**: timer → abort → `clearTimeout` in `finally`; or `AbortSignal.timeout(ms)` (`TimeoutError`); combine with `AbortSignal.any`.
- **CORS**: browser-enforced; origin = scheme + host + port. Server needs `Access-Control-Allow-Origin`; Node/curl ignore CORS.
- **Credentials**: `omit` / `same-origin` (default) / `include`; `include` cannot use `*` origin and needs `Allow-Credentials: true`.
- **Error handling**: only network, CORS and abort reject; status and parse errors are yours to detect.
- **REST patterns**: noun URLs + verbs; 201 create, 204 no content, 401 vs 403, 404, 409; PUT/DELETE idempotent, POST not.

## Must-remember code

```javascript
// safe JSON request
const res = await fetch(url, { headers: { Accept: "application/json" } });
if (!res.ok) throw new Error(`HTTP ${res.status}`);
const data = res.status === 204 ? null : await res.json();
```

```javascript
// timeout without leaking a timer
const controller = new AbortController();
const timer = setTimeout(() => controller.abort(), 5000);
try {
  return await fetch(url, { signal: controller.signal });
} finally {
  clearTimeout(timer);
}
```

```javascript
// POST JSON
await fetch("/api/todos", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ title: "Revise fetch" }),
});
```

```javascript
// credentialed cross-origin call
await fetch("https://api.example.com/me", { credentials: "include" });
// server: ACAO: https://app.example.com + ACAC: true (never *)
```

## Interview traps

- “Does fetch reject on 404/500?” **No** — it resolves; only `ok` tells you.
- Reading the body twice throws; `clone()` before the first read if you need both.
- `fetch` has **no timeout** — a hanging request needs `AbortController`.
- A CORS failure and an offline failure both surface as `TypeError: Failed to fetch`; the console has the real reason.
- `credentials: "include"` + `Access-Control-Allow-Origin: *` is always blocked.
- `SameSite=Lax` cookies may not be sent on cross-site requests even with `include`.
- Sending a plain object as `body` produces `"[object Object]"`.
- A `200` with HTML makes `.json()` throw `SyntaxError`; check content type.
- `PUT`/`DELETE` retries are safe; retrying `POST` can duplicate records.

## 3 things to remember

1. **Status is data, not rejection.**
   ```javascript
   if (!res.ok) throw new Error(`HTTP ${res.status}`);
   ```

2. **The body is a one-shot stream.**
   ```javascript
   const data = await res.json(); // second call throws
   ```

3. **Cancellation is explicit.**
   ```javascript
   const c = new AbortController();
   setTimeout(() => c.abort(), 5000);
   await fetch(url, { signal: c.signal });
   ```
