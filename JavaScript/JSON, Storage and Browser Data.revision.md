# JSON, Storage and Browser Data — Revision Notes (Intermediate)

> One-line mental model: JSON turns values into text, storage keeps that text, and the safe store depends on size, lifetime and who may read it.

## Core recall

- **JSON.stringify**: value → JSON **string**. Keys double-quoted; optional replacer and indentation.
- **JSON.parse**: text → value; strict syntax (`SyntaxError` on single quotes, trailing commas, comments).
- **Replacer/reviver**: replacer filters or transforms on write; reviver transforms on read; `undefined` deletes a key.
- **Serialization limits**: `Date` → ISO string, `undefined`/function/symbol dropped (null in arrays), `NaN`/`Infinity` → `null`, `Map`/`Set` → `{}`, `BigInt` and cycles throw.
- **localStorage**: string-only, synchronous, per origin, persists across tabs and restarts, ~5 MB.
- **sessionStorage**: same API, scoped to one tab, survives reload, dies on close.
- **Cookies**: ~4 KB, sent with every matching request; `HttpOnly` hidden from JS; `Secure` HTTPS only; `SameSite=Lax/Strict/None`.
- **IndexedDB**: async, transactional, structured clone, large records, event-based API with version upgrades.
- **Cache API**: `Request`/`Response` pairs for service workers and offline assets, not app data.
- **Storage quota**: Web Storage ~5 MB; overflow throws `QuotaExceededError`; `navigator.storage.estimate()`.
- **Security**: XSS reads all Web Storage; cookies face CSRF; never keep secrets or passwords client-side.
- **Choosing storage**: preference → localStorage, tab → sessionStorage, session → HttpOnly cookie, bulk → IndexedDB.

## Must-remember code

```javascript
// safe JSON round trip
const text = JSON.stringify(user);
const copy = JSON.parse(text);
```

```javascript
// string-only storage
localStorage.setItem("user", JSON.stringify(user));
const saved = JSON.parse(localStorage.getItem("user") ?? "null");
```

```javascript
// revive a Date
const obj = JSON.parse(text, (key, value) =>
  key === "createdAt" ? new Date(value) : value
);
```

```javascript
// session cookie (server)
// Set-Cookie: session=abc; HttpOnly; Secure; SameSite=Lax; Path=/
```

## Interview traps

- `localStorage.setItem("user", obj)` stores `"[object Object]"`.
- `localStorage.getItem("count")` returns `"42"`, not `42`.
- `JSON.parse(undefined)` throws; `JSON.parse(null)` returns `null`.
- `JSON.parse(JSON.stringify(date))` gives a string; `.getFullYear()` then throws.
- `JSON.stringify` drops `undefined` object keys silently, but writes `null` in arrays.
- `Map`/`Set` serialize as `{}`; `BigInt` and cycles throw.
- `HttpOnly` cookies are invisible to `document.cookie` — JS cannot read or set them.
- Disabled storage can throw `SecurityError` even on `getItem`.
- A `sessionStorage` token is gone in a new tab; users think they were logged out.
- IndexedDB is not promise-based out of the box; its requests use events.

## 3 things to remember

1. **JSON is lossy.**
   ```javascript
   JSON.stringify({ d: new Date(), u: undefined }); // {"d":"..."}
   ```

2. **Web Storage is strings only.**
   ```javascript
   localStorage.setItem("user", JSON.stringify(user));
   ```

3. **Sessions belong in HttpOnly cookies.**
   ```javascript
   // Set-Cookie: session=...; HttpOnly; Secure; SameSite=Lax
   ```
