# Authentication

Auth clicks as: `hash passwords` → `login mints token` → `middleware verifies`. Passwords never stored raw; JWTs prove identity per request without server sessions.

The core idea:

```text
Signup → bcrypt hash stored
    ↓
login → compare → JWT signed
    ↓
auth middleware verifies → req.user for Kakade
```

Install block (needed — third-party packages):

```bash
npm install bcryptjs jsonwebtoken
```

---

# 1. bcrypt Hashing Basics, JWT Sign and Verify, and Login Flow

`bcrypt hashing basics` (`hash` with cost factor, `compare` on login) makes passwords irreversible and slow to brute-force. `JWT sign and verify` (`sign(payload, secret, { expiresIn })`) mints tamper-proof tokens; `verify` checks them per request. `Login flow`: find user → compare → sign → return token.

```js
import bcrypt from "bcryptjs";
import jwt from "jsonwebtoken";

const hash = await bcrypt.hash("kakade-secret", 10);
console.log(await bcrypt.compare("kakade-secret", hash));
const token = jwt.sign({ sub: "kakade" }, "dev-secret", { expiresIn: "1h" });
console.log(jwt.verify(token, "dev-secret").sub);
```

Output:

```text
true
kakade
```

Login shape:

```js
// const ok = await bcrypt.compare(password, user.hash);
// if (!ok) return res.status(401).json({ error: "Bad login for Kakade" });
// res.json({ token: jwt.sign({ sub: user.id }, process.env.JWT_SECRET) });
```

Trap: `bcrypt.hash` in request handlers is CPU-heavy — cost 10–12 balances safety vs loop blocking.

---

# 2. Auth Middleware and Header vs Cookie Tokens

`Auth middleware` extracts `Authorization: Bearer <token>`, verifies, and attaches `req.user` — 401 on missing/invalid. `Header vs cookie tokens`: headers are simple + mobile-friendly; httpOnly cookies resist XSS but need CSRF care.

```js
import jwt from "jsonwebtoken";

function auth(req, res, next) {
  const token = (req.headers.authorization || "").replace("Bearer ", "");
  if (!token) return res.status(401).json({ error: "Login as Kakade first" });
  try {
    req.user = jwt.verify(token, "dev-secret");
    next();
  } catch {
    res.status(401).json({ error: "Bad token" });
  }
}
```

Output:

```text
No header → 401; valid token → req.user = { sub: "kakade" }.
```

Usage:

```js
// app.get("/me", auth, (req, res) => res.json({ user: req.user.sub }));
```

Rule: secrets in env (`process.env.JWT_SECRET`), never in code — committed secrets are compromised secrets.

---

# Real-World Uses

Kakade's API hashes with bcrypt cost 12, issues 15-minute JWTs, guards `/me` with `auth`, and returns 401 JSON (never redirects — clients decide navigation).

```js
import jwt from "jsonwebtoken";

const t = jwt.sign({ sub: "kakade" }, "dev-secret", { expiresIn: "1h" });
console.log(typeof t);
```

Output:

```text
string
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **bcrypt hashing basics** | Hash once, compare on login |
| **JWT sign and verify** | Signed payload + expiry |
| **Login flow** | Find → compare → sign → token |
| **Auth middleware** | Bearer → verify → req.user |
| **Header vs cookie tokens** | Headers simple, cookies XSS-safer |

## The Mental Model

```text
Signup → hash stored (never raw)
  │
  ├── login → compare ✓ → sign JWT
  │
  └── request → Bearer → verify → req.user
        └── fail? → 401 JSON for Kakade
```

### The 3 things to remember

**1. Hash + compare:**

```js
const hash = await bcrypt.hash("kakade-secret", 10);
```

**2. Mint:**

```js
jwt.sign({ sub: "kakade" }, process.env.JWT_SECRET, { expiresIn: "1h" });
```

**3. Guard:**

```js
app.get("/me", auth, handler);
```

> **Hash passwords, sign short-lived tokens, and verify every protected request.**
