# Security — Revision Notes (Beginner)

> One-line mental model: the server authorizes, the browser enforces origins and CSP, and every untrusted value is encoded for its sink.

## Core recall

- **XSS**: stored (database → everyone), reflected (request → response), DOM (client source → sink). Runs with your origin, so it bypasses same-origin.
- **innerHTML risks**: parses HTML; inserted `<script>` does not run, but `<img onerror>`/`<svg onload>` do. Use `textContent` for text.
- **CSP**: response header allowlist; `default-src 'self'`; scripts via nonce or hash, not `'unsafe-inline'`; `object-src 'none'`, `base-uri 'self'`.
- **CSRF**: browser auto-sends cookies; defend with `SameSite=Lax/Strict`, a per-session token, and an `Origin`/`Referer` check; never mutate on GET.
- **CORS**: server opt-in headers; preflight for JSON/custom headers/PUT; credentials require an exact origin and `Allow-Credentials: true`.
- **Same-origin policy**: scheme + host + port; blocks cross-origin **reads**, not sends. A simple POST can arrive — hence CSRF.
- **Sanitization**: allowlist with DOMPurify only when HTML is required; sanitize stored HTML server-side too; validate URL schemes (`javascript:`).
- **Secrets in frontend**: everything shipped is public, including `import.meta.env.VITE_*`; proxy through your backend. Publishable keys are the exception.
- **Prototype pollution**: `__proto__` in a recursive merge reaches `Object.prototype`; block `__proto__`/`constructor`/`prototype`, use `Object.hasOwn`, `Object.create(null)` or `Map`.
- **Dependency security**: lockfile, `npm ci`, `npm audit`, few deps, review install scripts; zero advisories is not proof of safety.
- **Secure storage**: localStorage/sessionStorage/IndexedDB are XSS-readable; session cookies need `HttpOnly; Secure; SameSite=Lax`.
- **Clickjacking**: transparent iframe over decoy UI; `frame-ancestors 'none'` / `X-Frame-Options: DENY`, not JS frame-busting.

## Must-remember code

```javascript
// safe text rendering
el.textContent = userInput;

// sanitize only when HTML is required
el.innerHTML = DOMPurify.sanitize(dirty, { ALLOWED_TAGS: ["p", "strong"] });
```

```javascript
// CSRF token on a mutating request
await fetch("/api/transfer", {
  method: "POST",
  headers: { "Content-Type": "application/json", "X-CSRF-Token": token },
  body: JSON.stringify({ to: "Rohan", amount: 5000 }),
});
```

```javascript
// block prototype pollution keys
const BLOCKED = new Set(["__proto__", "constructor", "prototype"]);
for (const key of Object.keys(source)) {
  if (BLOCKED.has(key)) continue;
  target[key] = source[key];
}
```

```text
Set-Cookie: session=abc123; HttpOnly; Secure; SameSite=Lax; Path=/
Content-Security-Policy: default-src 'self'; frame-ancestors 'none'
```

## Interview traps

- Inserting `<script>` via `innerHTML` does not execute, but `<img onerror>` does — test with the latter.
- `Access-Control-Allow-Origin: *` plus `credentials: "include"` is rejected; echo the exact origin.
- CORS does not stop a cross-site form POST, and it is not server authorization.
- `HttpOnly` protects against cookie theft by XSS, not against XSS itself.
- `key in obj` walks the prototype chain; use `Object.hasOwn(obj, key)` for permission maps.
- `SameSite=Lax` still sends cookies on top-level GET — do not mutate on GET.
- DOMPurify on the client does not protect other users if the server stores and serves raw HTML.
- `<meta>` CSP ignores `frame-ancestors`, `report-uri` and `sandbox`; use an HTTP header.

## 3 things to remember

1. **Text goes in via `textContent`; HTML only via an allowlist sanitizer.**
   ```javascript
   el.textContent = input;                          // safe
   el.innerHTML = DOMPurify.sanitize(html);         // justified case
   ```

2. **Sessions: `HttpOnly` + `Secure` + `SameSite`, plus a CSRF token.**
   ```text
   Set-Cookie: session=abc123; HttpOnly; Secure; SameSite=Lax; Path=/
   ```

3. **CORS is a browser read policy, not authorization.**
   ```javascript
   // Server must still check the session and the action on every request.
   ```
