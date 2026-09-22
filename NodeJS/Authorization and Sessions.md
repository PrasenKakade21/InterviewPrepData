# Authorization and Sessions

AuthZ clicks as: `authentication says who` → `roles say what` → `middleware enforces`. AuthN (login) without AuthZ (permissions) leaves every endpoint open to any logged-in user.

The core idea:

```text
JWT → req.user (+ roles)
    ↓
requireRole("admin") checks
    ↓
403 forbidden vs 401 unauthenticated for Kakade
```

401 = "log in"; 403 = "logged in, still no" — the classic interview distinction.

---

# 1. Roles and RBAC Middleware and Protected Routes

`Roles and RBAC middleware` (`requireRole(...allowed)`) compares `req.user.roles` against the endpoint's needs. `Protected routes` layer `auth` then `requireRole` — order matters, identity before permission.

```js
function requireRole(...allowed) {
  return (req, res, next) => {
    const roles = req.user?.roles || [];
    if (!allowed.some((r) => roles.includes(r))) {
      return res.status(403).json({ error: "Not for Kakade's role" });
    }
    next();
  };
}
```

Output:

```text
Admin hits /billing → next(); Kakade (user) → 403 JSON.
```

Wiring:

```js
// app.get("/billing", auth, requireRole("admin"), (req, res) => res.json({ ok: true }));
```

Think: `auth` answers *who*, `requireRole` answers *may-they* — two middlewares, two jobs, always in that order.

---

# 2. Refresh Tokens Brief and Logout Handling

`Refresh tokens brief`: short access JWT (minutes) + long refresh token (days, stored hashed server-side); `/refresh` swaps a valid refresh for a new access token. `Logout handling`: delete/deny the refresh token server-side and clear the client copy — access tokens simply expire.

```js
const refreshStore = new Map();

function logout(userId) {
  refreshStore.delete(userId);
  console.log(`Kakade (${userId}) logged out: refresh revoked`);
}
logout("kakade-1");
```

Output:

```text
Kakade (kakade-1) logged out: refresh revoked
```

Refresh sketch:

```js
// app.post("/refresh", (req, res) => {
//   const saved = refreshStore.get(req.body.userId);
//   if (saved !== req.body.refresh) return res.status(401).json({ error: "Login again" });
//   res.json({ access: signAccess(req.body.userId) }); // fresh 15-min token.
// });
```

Rule: rotate refresh tokens on use (new refresh each swap) — stolen ones die with first legitimate use.

---

# Real-World Uses

Kakade's billing routes require `admin`, the app uses 15-min access + rotating refresh tokens, and logout revokes server-side so stolen clients go dark within minutes.

```js
function requireRole(...allowed) {
  return (req, res, next) => next();
}
console.log(typeof requireRole("admin"));
```

Output:

```text
function
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **Roles and RBAC middleware** | Allowed-roles gate per route |
| **Protected routes** | auth → requireRole → handler |
| **Refresh tokens brief** | Short access + long rotating refresh |
| **Logout handling** | Revoke server-side, clear client |

## The Mental Model

```text
Request (JWT → req.user + roles)
  │
  ├── no token? → 401 login first
  ├── wrong role? → 403 not yours, Kakade
  │
  └── access expired? → /refresh → new access
        └── logout → revoke refresh
```

### The 3 things to remember

**1. Gate roles:**

```js
app.get("/billing", auth, requireRole("admin"), handler);
```

**2. Distinguish:**

```js
res.status(401).json({ error: "Login first" });
res.status(403).json({ error: "Not for Kakade's role" });
```

**3. Revoke:**

```js
refreshStore.delete(userId);
```

> **Authenticate who, authorize what, and revoke refresh tokens on logout.**
