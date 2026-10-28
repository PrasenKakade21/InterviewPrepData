# Security

Frontend security starts with one rule: **the browser is untrusted**. The chain is **validate input → encode for the sink → restrict origins → protect the session**: the server authorizes, the browser enforces same-origin and CSP, and your code must never turn text into executable HTML.

```text
Untrusted input
   ↓
validate        type, length, shape
   ↓
encode          HTML / attribute / URL / JS
   ↓
browser guards  same-origin · CORS · CSP
   ↓
server checks   auth · CSRF · headers
   ↓
safe output     textContent, not innerHTML
```

---

# 1. Cross-Site Scripting

### XSS

XSS injects attacker JavaScript into a page other users trust: **stored** (saved and served to everyone), **reflected** (echoed from the request), and **DOM-based** (client source → dangerous sink).

| Type | Source | Who is hit |
| --- | --- | --- |
| Stored | database, comments, profile | every visitor |
| Reflected | query string, form echo | the victim's response |
| DOM-based | `location`, `referrer` | client-side render |

```javascript
// Browser only — DOM XSS: the hash is attacker-controlled
const name = decodeURIComponent(location.hash.slice(1));
document.querySelector("#greeting").innerHTML = `Hello ${name}`;
```

Output:

```text
URL: #<img src=x onerror="alert(document.cookie)">
A broken image appears, then alert() runs with this site's origin.
```

XSS runs with your origin, so it bypasses the same-origin policy entirely. Treat URLs, storage, API data and `postMessage` as hostile.

### innerHTML risks

`innerHTML` parses a string as HTML. A `<script>` inserted this way does **not** run, but event attributes do: `<img onerror>`, `<svg onload>`, `<iframe srcdoc>`.

```javascript
const name = '<img src=x onerror="alert(1)">';

// Bad — parses and executes the handler
document.querySelector("#bad").innerHTML = name;

// Good — renders characters, never HTML
document.querySelector("#good").textContent = name;
```

Output:

```text
#bad:  broken image icon, then alert(1)
#good: <img src=x onerror="alert(1)"> shown as literal text
```

`outerHTML`, `insertAdjacentHTML`, `document.write` and jQuery `.html()` carry the same risk. Need rich text? Sanitize it first.

---

# 2. Content Security Policy

CSP is a response header listing allowed sources for scripts, styles, images, frames and connections. It is a second line of defence: even if a payload reaches the page, `script-src` can stop it running.

```text
Content-Security-Policy:
  default-src 'self'; script-src 'self' 'nonce-r4nd0m';
  img-src 'self' https://images.example.com;
  object-src 'none'; base-uri 'self'; frame-ancestors 'none'
```

```html
<script nonce="r4nd0m">console.log("allowed");</script>
<script>alert("blocked");</script>
```

Output:

```text
allowed
Refused to execute inline script: violates "script-src 'self' 'nonce-r4nd0m'".
```

Nonce must be random per response and match header. `'unsafe-inline'` re-enables inline scripts and removes most protection; prefer nonces or hashes. `default-src 'self'` is a good baseline, and `object-src 'none'` + `base-uri 'self'` close classic bypasses.

Notice: `frame-ancestors`, `report-uri` and `sandbox` are **ignored in `<meta>`** — send CSP as an HTTP header. Start with `Content-Security-Policy-Report-Only` to find breakage before enforcing.

---

# 3. CSRF

CSRF tricks a logged-in browser into sending a state-changing request to your site. Cookies attach automatically; the attacker does not need to read the response.

```html
<!-- Rohan's page -->
<form action="https://bank.example.com/transfer" method="POST">
  <input type="hidden" name="to" value="Asha">
  <input type="hidden" name="amount" value="5000">
</form>
<script>document.forms[0].submit();</script>
```

Output:

```text
SameSite=None cookie: 200 OK — ₹5,000 transferred.
SameSite=Lax (default): cookie omitted on cross-site POST → 401.
```

Defences: `SameSite=Lax` or `Strict`; a per-session CSRF token checked on the server; an `Origin`/`Referer` check; and never mutate state on `GET`. A custom header such as `X-CSRF-Token` also forces a CORS preflight, which a cross-site form cannot satisfy, so read the token from a meta tag and send it on every mutating `fetch`; the server returns 403 when it is missing or stale.

CSRF is a **server** responsibility. CORS does not stop a form POST, and JavaScript cannot read the response — but the transfer still happened.

---

# 4. CORS and the Same-Origin Policy

### Same-origin policy

An origin is **scheme + host + port**. A page may read responses only from its own origin; cross-origin requests may be sent, but the response is hidden unless the server opts in.

| Page | Request URL | Same origin? |
| --- | --- | --- |
| `https://app.example.com` | `https://app.example.com/api` | yes |
| `https://app.example.com` | `http://app.example.com/api` | no (scheme) |
| `https://app.example.com` | `https://api.example.com` | no (host) |
| `https://app.example.com` | `https://app.example.com:8443` | no (port) |

```javascript
fetch("https://api.example.com/orders")
  .then((response) => response.json())
  .catch((error) => console.log(error.name, error.message));
```

Output:

```text
TypeError Failed to fetch
(a CORS policy error also appears; the request may still have reached the server)
```

A "simple" POST can arrive and mutate data; the browser only blocks your code from reading the response. That gap is why CSRF exists.

### CORS

CORS is the server's opt-in list, sent as response headers. Non-simple requests (JSON, custom headers, PUT/DELETE) trigger an `OPTIONS` preflight first.

```text
Access-Control-Allow-Origin: https://app.example.com
Access-Control-Allow-Credentials: true
Vary: Origin
```

```javascript
await fetch("https://api.example.com/orders", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  credentials: "include",
  body: JSON.stringify({ item: "book" }),
});
```

Output:

```text
OPTIONS /orders → 204 (preflight allowed)
POST /orders    → 201 Created
```

With credentials the server must name the exact origin; `*` plus credentials is rejected. CORS is enforced by the browser for the user, **not** server authentication — any script or curl can ignore it.

---

# 5. Sanitization

Sanitization removes dangerous markup while keeping safe formatting. It is needed only when you must render HTML; for plain text, `textContent` is better. Never write a regex sanitizer.

```javascript
import DOMPurify from "dompurify";

const dirty = `<p>Hello Kakade</p><img src=x onerror="alert(1)">`;
const clean = DOMPurify.sanitize(dirty, {
  ALLOWED_TAGS: ["p", "strong", "em"],
  ALLOWED_ATTR: [],
});

document.querySelector("#out").innerHTML = clean;
```

Output:

```text
<p>Hello Kakade</p>
The img element and its onerror attribute are removed.
```

DOMPurify parses in the browser and applies an allowlist; configure it to your content model. Sanitize **on the server** too when HTML is stored for other users. Also validate URL schemes — `javascript:` in an `href` is script execution.

---

# 6. Secrets and Storage

### Secrets in frontend

Everything shipped to the browser is public: source, bundle, environment variables, keys. `import.meta.env.VITE_*` values are **inlined at build time**.

```javascript
// Bad — secret key in client code
const key = "sk_live_51KakadeSecret";
await fetch("https://api.stripe.com/v1/charges", {
  headers: { Authorization: `Bearer ${key}` },
});
```

Output:

```text
The key appears in DevTools → Sources and in the built .js file.
Anyone can copy it and make charges on Kakade's account.
```

Good: the browser calls **your** backend, which holds the secret and enforces authorization — `POST /api/checkout` with no key in the bundle. Publishable keys (Stripe `pk_...`, map tiles) are designed to be public; secret keys, database URLs and signing secrets never are.

### Secure storage

`localStorage` and `sessionStorage` are readable by any script on the origin, so one XSS steals every token there. `HttpOnly` cookies are invisible to JavaScript, and `SameSite` limits cross-site sending.

```javascript
localStorage.setItem("session", "abc123");
console.log(localStorage.getItem("session"));
```

Output:

```text
abc123
```

```text
Set-Cookie: session=abc123; HttpOnly; Secure; SameSite=Lax; Path=/; Max-Age=3600
```

Prefer a short-lived session cookie with `HttpOnly; Secure; SameSite=Lax`, and keep access tokens in memory where possible. `Secure` means HTTPS-only, `HttpOnly` hides the value from JS, and `SameSite=Lax` omits it on cross-site POSTs. `sessionStorage` and IndexedDB are XSS-readable; preventing XSS is the real control.

---

# 7. Prototype Pollution

Prototype pollution happens when untrusted keys are merged and `__proto__` reaches `Object.prototype`; every object then inherits the attacker's property, which can flip authorization checks.

```javascript
function merge(target, source) {
  for (const key in source) {
    if (typeof source[key] === "object" && source[key] !== null) {
      target[key] ??= {};
      merge(target[key], source[key]);
    } else {
      target[key] = source[key];
    }
  }
  return target;
}

merge({}, JSON.parse('{"__proto__":{"isAdmin":true}}'));
console.log({}.isAdmin);
```

Output:

```text
true — Object.prototype.isAdmin now affects every object in the process.
```

Bad: recursive merge of untrusted JSON with no key allowlist. Good: reject the dangerous keys, check own properties, and prefer a null-prototype object for user-keyed data.

```javascript
const safe = Object.create(null);
Object.assign(safe, JSON.parse('{"__proto__":{"isAdmin":true}}'));

console.log(safe.isAdmin, {}.isAdmin);
```

Output:

```text
undefined undefined
```

`Object.create(null)` has no prototype chain, so `__proto__` becomes an ordinary own key instead of a setter. Production merges should also skip `__proto__`, `constructor` and `prototype`, use `Object.hasOwn(obj, key)` rather than `key in obj`, and keep merge libraries patched — several had CVEs.

---

# 8. Dependencies and Clickjacking

### Dependency security

Most shipped code is transitive. Risks include typosquatting (`lodahs`), compromised maintainers, malicious `postinstall` scripts and abandoned packages.

```bash
npm audit --omit=dev
npm ls lodash
```

Output:

```text
found 0 vulnerabilities

kakade-app@1.0.0
└── lodash@4.17.21
```

`npm audit` only knows published advisories, so zero findings is not proof of safety. Commit the lockfile, use `npm ci` in CI, keep dependencies few and current, and prefer `npm ci --ignore-scripts` where install scripts are unnecessary. Dependabot or Renovate keeps upgrade PRs flowing; `overrides` forces a patched transitive version.

### Clickjacking

Clickjacking loads your site in a transparent iframe over a decoy button, so the user's click lands on your UI.

```html
<!-- Attacker's page -->
<iframe src="https://bank.example.com/transfer"
        style="opacity:0.01;position:fixed;inset:0"></iframe>
<button style="position:relative;z-index:1">Claim your prize</button>
```

Output:

```text
A click on "Claim your prize" lands on Transfer.
```

Defend at the header, not with JavaScript frame-busting:

```text
X-Frame-Options: DENY
Content-Security-Policy: frame-ancestors 'none'
```

Output:

```text
The iframe refuses to render: "Refused to display ... because it set
'X-Frame-Options' to 'deny'."
```

Use `SAMEORIGIN` only when your own pages must frame you. `frame-ancestors` is the modern control; `X-Frame-Options` is the legacy fallback.

---

# Real-World Uses

- **Comments and profiles** — render text with `textContent`; sanitize only rich HTML.
- **Login flows** — `HttpOnly; Secure; SameSite` cookies plus CSRF tokens.
- **Build pipelines** — `npm audit`, lockfiles and dependency review.

---

# Quick Revision

| Concept | Remember |
| --- | --- |
| **XSS** | Stored / reflected / DOM; runs with your origin |
| **innerHTML risks** | Fires `onerror`; use `textContent` |
| **CSP** | Allowlist header; nonce/hash; `frame-ancestors` |
| **CSRF** | Cookies auto-send; SameSite + token + Origin |
| **CORS** | Opt-in headers; preflight; `*` invalid with credentials |
| **Same-origin policy** | Scheme + host + port; blocks reads |
| **Sanitization** | DOMPurify allowlist; validate URLs |
| **Secrets in frontend** | Everything shipped is public |
| **Prototype pollution** | Block `__proto__`; null-prototype objects |
| **Dependency security** | Lockfile, audit, few deps |
| **Secure storage** | `HttpOnly` cookies; localStorage is readable |
| **Clickjacking** | `frame-ancestors 'none'` |

## The Mental Model

```text
                UNTRUSTED INPUT
                       │
         ┌─────────────┼─────────────┐
         ↓             ↓             ↓
      validate       encode        limit
      type/length   HTML/URL     CSP/origins
         └─────────────┼─────────────┘
                       ↓
                 SAFE OUTPUT
             textContent / sanitized
                       ↓
        HttpOnly · SameSite · CSRF token
                       ↓
          auth · authz · headers · audit
```

### The 3 things to remember

**1. Never put untrusted text into HTML.**

```javascript
el.textContent = userInput;              // safe
el.innerHTML = DOMPurify.sanitize(html); // only when HTML is required
```

**2. CORS is not authorization, and the browser is not the server.**

```javascript
// The server must check the session and CSRF token, not just Origin.
```

**3. Session cookies need `HttpOnly`, `Secure` and `SameSite`.**

```text
Set-Cookie: session=abc123; HttpOnly; Secure; SameSite=Lax; Path=/
```

> **Security is about trusting the right layer: the server authorizes, the browser enforces origins and CSP, and your code must encode every untrusted value for the sink it lands in.**
