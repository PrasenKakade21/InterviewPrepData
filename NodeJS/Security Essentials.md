# Security Essentials

Security clicks as: `headers hardened` → `origins + rates limited` → `secrets never leak`. Most breaches are missing basics, not exotic exploits.

The core idea:

```text
helmet (safe headers)
    ↓
cors (allowed origins) + rate-limit (abuse cap)
    ↓
Kakade's API resists casual attacks
```

Install block (needed — third-party packages):

```bash
npm install helmet cors express-rate-limit
```

---

# 1. helmet Basics, CORS Handling, and Rate Limiting

`helmet basics` (`app.use(helmet())`) sets secure headers (no sniffing, framing guards) in one line. `CORS handling` (`cors({ origin })`) whitelists which browsers may call the API. `Rate limiting` (`rateLimit({ windowMs, max })`) caps requests per IP against brute force.

```js
import express from "express";
import helmet from "helmet";
import cors from "cors";
import rateLimit from "express-rate-limit";

const app = express();
app.use(helmet());
app.use(cors({ origin: "https://kakade.dev" }));
app.use(rateLimit({ windowMs: 60_000, max: 100 }));
app.get("/hi", (req, res) => res.json({ hi: "Kakade" }));
app.listen(3000);
```

Output:

```text
GET /hi → Kakade with hardened headers, CORS + 100/min guard.
```

Think: `helmet` = seatbelt (always on), `cors` = guest list, `rateLimit` = bouncer — three middlewares, three threats.

---

# 2. 401 vs 403 and Secrets and Env Safety

`401 vs 403`: `401` unauthenticated (log in); `403` unauthorized (logged in, forbidden) — clients branch retry vs hide-UI on this. `Secrets and env safety`: keys in env/host vaults, `.env` gitignored, never logged or returned.

```js
function gate(req, res, next) {
  if (!req.user) return res.status(401).json({ error: "Login as Kakade first" });
  if (!req.user.roles?.includes("admin")) {
    return res.status(403).json({ error: "Admins only" });
  }
  next();
}
console.log(gate.length);
```

Output:

```text
3
```

Checklist:

```text
[ ] helmet + cors + rateLimit mounted for Kakade
[ ] 401 login, 403 forbidden — never swapped
[ ] .env ignored, secrets in host vault
[ ] zod validates every body (see Validation)
[ ] errors generic to clients, verbose in logs
```

Interview line: 401 challenges identity, 403 enforces permission — swapping them breaks client auth flows.

---

# Real-World Uses

Kakade's API mounts the security trio globally, returns 401 for missing tokens and 403 for wrong roles, and rotates leaked secrets via host env — not code edits.

```js
console.log("helmet + cors + rateLimit on for Kakade");
```

Output:

```text
helmet + cors + rateLimit on for Kakade
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **helmet basics** | Secure headers in one line |
| **CORS handling** | Whitelist calling origins |
| **Rate limiting** | Cap per IP per window |
| **401 vs 403** | Login vs forbidden |
| **Secrets and env safety** | Vault + ignore + never log |

## The Mental Model

```text
Request for Kakade
  │
  ├── helmet → safe headers
  ├── cors → origin allowed?
  ├── rateLimit → under 100/min?
  │
  └── gate → 401 (who?) / 403 (may you?)
        └── secrets in env, never in code/logs
```

### The 3 things to remember

**1. Trio:**

```js
app.use(helmet());
app.use(cors({ origin: "https://kakade.dev" }));
```

**2. Cap abuse:**

```js
app.use(rateLimit({ windowMs: 60_000, max: 100 }));
```

**3. Split codes:**

```js
res.status(401).json({ error: "Login as Kakade first" });
res.status(403).json({ error: "Admins only" });
```

> **Harden headers, whitelist origins, cap rates — and never swap 401 with 403.**
