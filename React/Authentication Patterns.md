# Authentication Patterns

Auth clicks as: `login stores session` → `context shares user` → `routes + API enforce`. Frontend auth is UX + token plumbing — real security lives on the server.

The core idea:

```text
Login form → token → persist (memory/httpOnly)
    ↓
AuthContext { user, login, logout }
    ↓
protected route + 401 refresh → redirect
```

Never store tokens in `localStorage` for XSS-prone apps if httpOnly cookies are an option.

---

# 1. Login Forms, JWT in Frontend, and Storing Tokens

`Login forms` collect credentials with controlled inputs + pending/error states. `JWT in frontend`: decode claims for UI (`name`, `exp`), never trust them for access. `Storing tokens`: memory (safest, lost on reload) > httpOnly cookie (server-set) > `localStorage` (XSS-readable, convenient).

```jsx
import { useState } from "react";

function Login({ onLogin }) {
  const [email, setEmail] = useState("kakade@dev.io");
  const [pw, setPw] = useState("");
  const [busy, setBusy] = useState(false);
  async function submit(e) {
    e.preventDefault();
    setBusy(true);
    await onLogin({ email, pw });
    setBusy(false);
  }
  return (
    <form onSubmit={submit}>
      <input value={email} onChange={(e) => setEmail(e.target.value)} />
      <input type="password" value={pw} onChange={(e) => setPw(e.target.value)} />
      <button disabled={busy}>{busy ? "…" : "Login Kakade"}</button>
    </form>
  );
}
```

Output:

```text
Login Kakade → … → dashboard (or error stays on form).
```

Token sketch:

```js
// jwt = header.payload.signature; payload { sub: "kakade", exp: 123 }.
// Attach: fetch(url, { headers: { Authorization: `Bearer ${token}` } }).
```

Interview line: JWT proves *who*; the API still authorizes *what* on every request.

---

# 2. Auth Context, Protected Routes, and Redirect After Login

`Auth context` shares `{ user, login, logout, loading }` app-wide. `Protected routes` render `<Navigate to="/login">` when logged out. `Redirect after login` returns to `location.state.from` (the page Kakade wanted).

```jsx
import { createContext, useContext, useState } from "react";
import { Navigate, useLocation, useNavigate } from "react-router-dom";

const AuthCtx = createContext(null);

export function AuthProvider({ children }) {
  const [user, setUser] = useState({ name: "Kakade" });
  return <AuthCtx.Provider value={{ user, setUser }}>{children}</AuthCtx.Provider>;
}

export function Guard({ children }) {
  const { user } = useContext(AuthCtx);
  const loc = useLocation();
  if (!user) return <Navigate to="/login" state={{ from: loc.pathname }} replace />;
  return children;
}
```

Output:

```text
/logout → /login; login returns to saved page.
```

Login redirect:

```js
// const nav = useNavigate();
// nav(state.from || "/dashboard", { replace: true }) → no back-to-login loop.
```

Keep auth loading state (`checking session…`) so guards do not flash login on refresh.

---

# 3. Persisting Sessions, Logout Handling, and 401 Handling and Refresh

`Persisting sessions` rehydrates on mount (`/me` with cookie, or stored refresh token). `Logout handling` clears tokens + user + cached queries, then redirects. `401 handling and refresh`: an interceptor catches 401, refreshes once, retries the original request, else logs out.

```jsx
function Logout({ onOut }) {
  return <button onClick={onOut}>Logout Kakade</button>;
}
```

Output:

```text
Logout Kakade → tokens cleared → /login.
```

Refresh sketch:

```js
// on 401: if (!retried) { await refresh(); retry(original); } else logout().
// Queue concurrent 401s behind one refresh — no token stampede.
```

Also: short access + rotating refresh tokens, `Secure/HttpOnly/SameSite` cookies, and log all auth failures without leaking which field was wrong.

---

# Real-World Uses

A Kakade dashboard persists the session via `/me`, guards `/billing` with `Guard`, redirects post-login to the deep link, and silently refreshes tokens so long forms never die to a 401.

```jsx
function Me() {
  return <p>Hi Kakade (from /me)</p>;
}
```

Output:

```text
Hi Kakade (session rehydrated on reload).
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **Login forms** | Controlled + busy/error |
| **JWT in frontend** | UI claims only, verify server-side |
| **Storing tokens** | Memory/cookie over localStorage |
| **Auth context** | `{ user, login, logout }` shared |
| **Protected routes** | `Guard` → `<Navigate>` |
| **Redirect after login** | `state.from` deep link |
| **Persisting sessions** | `/me` rehydrate on mount |
| **Logout handling** | Clear all + redirect |
| **401 handling and refresh** | Refresh once, retry, else logout |

## The Mental Model

```text
Login → token (memory/cookie)
  │
  ├── AuthContext → user app-wide
  │
  ├── Guard → !user? /login (save from)
  │
  └── API 401 → refresh once → retry → logout
        └── reload → /me rehydrate
```

### The 3 things to remember

**1. Guard routes:**

```jsx
if (!user) return <Navigate to="/login" state={{ from: loc.pathname }} replace />;
```

**2. Bearer calls:**

```jsx
fetch(url, { headers: { Authorization: `Bearer ${token}` } });
```

**3. Refresh once:**

```jsx
await refresh();
retry(original);
```

> **Persist sessions safely, guard routes with return links, and refresh-then-retry on 401.**
