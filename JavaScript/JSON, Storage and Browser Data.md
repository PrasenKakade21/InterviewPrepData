# JSON, Storage and Browser Data

JSON is the text format that lets a JavaScript value travel between a browser, a server and storage. The chain is **value → `JSON.stringify` → text → `JSON.parse` → value**, then **which store holds that text**. The interview themes are what JSON silently loses, why Web Storage only holds strings, and which storage is safe for sessions and large data.

```text
JS value
   ↓  JSON.stringify
JSON text  "{ \"name\": \"Kakade\" }"
   ↓  transport / storage
   ↓  JSON.parse
JS value
   ↓
localStorage · sessionStorage · cookie · IndexedDB · Cache API
```

---

# 1. Writing JSON

### JSON.stringify

`JSON.stringify(value)` turns a value into JSON text. The result is a **string**, even when the input was an object. An optional second argument is a replacer; the third controls indentation.

```javascript
const user = { name: "Kakade", age: 30, active: true };

const json = JSON.stringify(user);
console.log(json);
console.log(typeof json);
console.log(JSON.stringify(user, null, 2));
```

Output:

```text
{"name":"Kakade","age":30,"active":true}
string
{
  "name": "Kakade",
  "age": 30,
  "active": true
}
```

JSON keys are always double-quoted. `undefined`, functions and symbols are not JSON values, so they are **dropped from objects** and become `null` inside arrays.

### Serialization limits

`JSON.stringify` only understands objects, arrays, strings, finite numbers, booleans and `null`. Everything else is transformed or throws.

```javascript
const data = { a: undefined, b: () => 1, c: NaN, d: [undefined, () => 1] };
console.log(JSON.stringify(data));

try {
  JSON.stringify({ id: 10n });
} catch (err) {
  console.log(err.name);
}
```

Output:

```text
{"c":null,"d":[null,null]}
TypeError
```

Notice: a `Date` becomes an ISO string through its `toJSON()`, `Map`/`Set` become `{}`, and a circular reference throws `TypeError: Converting circular structure to JSON`. JSON is a **lossy** format — `structuredClone` is the tool that preserves types.

---

# 2. Parsing JSON

### JSON.parse

`JSON.parse(text)` turns JSON text back into a value. It is strict: no single quotes, no trailing commas, no comments, no `undefined`, no `NaN`.

```javascript
const text = '{"name":"Asha","roles":["admin","editor"]}';
const user = JSON.parse(text);

console.log(user.name, user.roles[1]);

try {
  JSON.parse("{name:'Asha'}");
} catch (err) {
  console.log(err.name, err.message);
}
```

Output:

```text
Asha editor
SyntaxError Expected property name or '}' in JSON at position 1 (line 1 column 2)
```

Bad: `JSON.parse("{'a': 1}")`. Good: valid JSON from `JSON.stringify`, or `JSON.parse` only after checking the text. Parsing is data-only — it never executes code — but merging the result into an object can still enable prototype pollution if you copy an own `__proto__` key.

### Replacer and reviver

A **replacer** filters or transforms values on the way out. A **reviver** transforms values on the way in. Both are called for every key, including nested ones.

```javascript
const user = { name: "Kakade", password: "secret", token: "abc" };
const safe = JSON.stringify(user, (key, value) =>
  key === "password" || key === "token" ? undefined : value
);
console.log(safe);

const text = '{"name":"Asha","joined":"2026-01-15T00:00:00.000Z"}';
const parsed = JSON.parse(text, (key, value) =>
  key === "joined" ? new Date(value) : value
);
console.log(parsed.joined instanceof Date, parsed.joined.getUTCFullYear());
```

Output:

```text
{"name":"Kakade"}
true 2026
```

An **array** replacer is a whitelist of keys: `JSON.stringify({ a: 1, b: 2 }, ["a"])` → `{"a":1}`. It applies at every level, so nested keys not listed also disappear. Returning `undefined` from a reviver **deletes** that key. Revivers run bottom-up: children are revived before their parent.

---

# 3. Web Storage

### localStorage

`localStorage` is a synchronous key–value store per origin. It survives reloads, tabs and browser restarts, and every value is a **string**.

```javascript
// Browser only
localStorage.setItem("theme", "dark");
console.log(localStorage.getItem("theme"));
console.log(localStorage.getItem("missing"));
```

Output:

```text
dark
null
```

Missing keys return `null`, not `undefined`. `removeItem(key)` deletes one and `clear()` wipes the origin.

### sessionStorage

`sessionStorage` has the same API but is scoped to one **tab** and one page session. A reload keeps it; closing the tab clears it.

```javascript
// Browser only — open the same page in two tabs
sessionStorage.setItem("step", "2");
console.log(sessionStorage.getItem("step"));
console.log(localStorage.getItem("step"));
```

Output:

```text
2
null
```

Use it for a multi-step form or a one-time wizard state. Do **not** use it for a login token if you expect a new tab to stay signed in — `localStorage` persists across tabs, `sessionStorage` does not.

Both Web Storage stores are **string-only**. Writing an object gives `"[object Object]"`.

```javascript
// Browser only
localStorage.setItem("count", 42);
console.log(typeof localStorage.getItem("count"), localStorage.getItem("count"));

const user = { name: "Kakade", age: 30 };
localStorage.setItem("user", JSON.stringify(user));
const saved = JSON.parse(localStorage.getItem("user") ?? "null");
console.log(saved.name);
```

Output:

```text
string 42
Kakade
```

Notice `typeof` is `"string"` even though `42` looks numeric. Always `JSON.stringify` on write and `JSON.parse` on read; guard with `?? "null"` because `JSON.parse(null)` returns `null` but `JSON.parse(undefined)` throws.

### Storage quota

Web Storage is small — roughly **5 MB per origin** in most browsers — and synchronous. Writes that exceed the quota throw `QuotaExceededError`.

```javascript
// Browser only
try {
  localStorage.setItem("big", "x".repeat(12 * 1024 * 1024));
} catch (err) {
  console.log(err.name);
}

navigator.storage.estimate().then(({ usage, quota }) => {
  console.log(usage < quota);
});
```

Output:

```text
QuotaExceededError
true
```

Because every Web Storage call blocks the main thread, never read or write large JSON in a render loop.

---

# 4. Cookies

### Cookies

Cookies are small strings (about 4 KB each) sent with **every matching HTTP request**. JavaScript writes one cookie at a time through `document.cookie`, but reading returns all non-`HttpOnly` cookies as one `"; "`-separated string.

```javascript
// Browser only
document.cookie = "theme=dark; Path=/; Max-Age=3600; SameSite=Lax";
document.cookie = "lang=en; Path=/; SameSite=Lax";

const getCookie = (name) =>
  document.cookie
    .split("; ")
    .find((row) => row.startsWith(`${name}=`))
    ?.split("=")[1];

console.log(document.cookie);
console.log(getCookie("theme"));
```

Output:

```text
theme=dark; lang=en
dark
```

| Flag | Meaning |
| --- | --- |
| `Max-Age` / `Expires` | Lifetime; `Max-Age=0` deletes |
| `Path` | URL path scope |
| `Domain` | Subdomain scope |
| `Secure` | HTTPS only |
| `HttpOnly` | **Server-only**, hidden from JS |
| `SameSite=Strict/Lax/None` | Cross-site sending; `None` needs `Secure` |

`HttpOnly` cannot be set or read by JavaScript — that is the point: an XSS bug cannot steal the cookie. `SameSite` is the main CSRF defence. Writing a cookie with the same name, path and domain replaces it; a different path creates a second cookie.

---

# 5. IndexedDB and Cache API

### IndexedDB overview

IndexedDB is an asynchronous, transactional object database for **large structured data** — files, blobs, thousands of records. It stores structured-cloned values, not just strings.

```javascript
// Browser only
const request = indexedDB.open("interviewPrep", 1);

request.onupgradeneeded = () => {
  request.result.createObjectStore("notes", { keyPath: "id" });
};

request.onsuccess = () => {
  const db = request.result;
  const tx = db.transaction("notes", "readwrite");
  tx.objectStore("notes").put({ id: 1, title: "Closures" });
  tx.oncomplete = () => console.log("saved");
};
```

Output:

```text
saved
```

The API is **event-based**, not promise-based. A version bump runs `onupgradeneeded`, where you create stores and indexes; transactions auto-commit when the event loop yields, so keep them short.

### Cache API overview

The Cache API stores `Request`/`Response` pairs and is designed for **service workers** and offline assets. It is not a general key–value store for application data.

```javascript
// Browser only, secure context / service worker
const cache = await caches.open("static-v1");
await cache.addAll(["/", "/styles.css"]);

const hit = await cache.match("/styles.css");
console.log(hit.status);
```

Output:

```text
200
```

`caches` is available on `window` only in a secure context, and service workers can intercept `fetch` events and answer from the cache. For app records, use IndexedDB; for assets, use Cache.

---

# 6. Security and Choosing Storage

### Security

Web Storage is readable by **any** JavaScript on the origin, so one XSS bug exposes everything in it. Cookies marked `HttpOnly` are not readable, but they are automatically sent, which creates CSRF risk.

```javascript
// Browser only — Bad: a token readable by any injected script
localStorage.setItem("token", jwt);

// Good: session cookie set by the server
// Set-Cookie: session=...; HttpOnly; Secure; SameSite=Lax
```

Output:

```text
Bad: any XSS payload can read the token.
Good: JS cannot read the cookie; SameSite blocks most cross-site sends.
```

Never store passwords or long-lived admin tokens in the browser. Clear storage on logout and treat every `JSON.parse` result as untrusted. Parsing does not execute code, but merging a `__proto__` key into an object can pollute prototypes — validate and whitelist fields.

### Choosing storage

A quick decision helper:

```javascript
// Browser only
const store = (bytes, perTab = false) =>
  perTab ? "sessionStorage" : bytes > 1e6 ? "IndexedDB" : "localStorage";

console.log(store(2e3, true), store(5e6));
```

Output:

```text
sessionStorage IndexedDB
```

| Need | Use | Why |
| --- | --- | --- |
| Small preference, offline | `localStorage` | Simple, synchronous, persists |
| Per-tab wizard state | `sessionStorage` | Dies with the tab |
| Session auth | `HttpOnly` cookie | Hidden from JS, sent automatically |
| Many records / files | IndexedDB | Async, large, structured clone |
| Offline assets | Cache API | Request/Response pairs |

---

# Real-World Uses

- **Draft forms** — save `JSON.stringify(form)` to `localStorage`, restore on reload, clear after submit.
- **Offline notes app** — IndexedDB records plus a Cache API shell for the assets.

---

# Quick Revision

| Concept | Remember |
| --- | --- |
| **JSON.stringify** | Value → JSON text; double quotes; optional replacer and indent |
| **JSON.parse** | Text → value; strict syntax; `SyntaxError` on bad input |
| **Replacer and reviver** | Replacer filters on write; reviver transforms on read; `undefined` deletes |
| **localStorage** | String-only, synchronous, per origin, persists |
| **sessionStorage** | Same API, per tab, dies when the tab closes |
| **Cookies** | ~4 KB, sent every request; `HttpOnly` hidden from JS; `SameSite` stops CSRF |
| **IndexedDB overview** | Async transactional DB, structured clone, event-based |
| **Cache API overview** | Request/Response pairs for offline assets, service workers |
| **Storage quota** | ~5 MB Web Storage; `QuotaExceededError`; `navigator.storage.estimate()` |
| **Security** | XSS reads storage; cookies face CSRF; never store secrets client-side |
| **Serialization limits** | Dates → strings, `undefined`/functions dropped, `NaN` → `null`, cycles throw |
| **Choosing storage** | Preference → localStorage; tab → sessionStorage; session → cookie; bulk → IndexedDB |

## The Mental Model

Think:

```text
                   JS VALUE
                      │
          ┌───────────┴───────────┐
          ↓                       ↓
   JSON.stringify             store directly
          │                       │
     JSON text               ┌────┴─────┬─────────┬──────────┐
          │                  ↓          ↓         ↓          ↓
     JSON.parse          localStorage  cookie  IndexedDB  Cache API
          │                  │          │         │          │
          ↓                  ↓          ↓         ↓          ↓
     JS value            string    4 KB/sent   async DB   assets
          │                  │          │         │          │
          └──────────→ security: XSS / CSRF / quota ←────────┘
```

### The 3 things to remember

**1. JSON is lossy.**

```javascript
JSON.stringify({ d: new Date(), u: undefined, n: NaN });
// {"d":"2026-...","n":null}  → u gone, d is a string
```

**2. Web Storage is strings only.**

```javascript
localStorage.setItem("user", JSON.stringify(user));
const user2 = JSON.parse(localStorage.getItem("user") ?? "null");
```

**3. Session cookies belong in `HttpOnly`.**

```javascript
// Server: Set-Cookie: session=...; HttpOnly; Secure; SameSite=Lax
```

> **JSON moves values as text, storage keeps that text, and the safe choice depends on size, lifetime and whether JavaScript should be able to read it.**
