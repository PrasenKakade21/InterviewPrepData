# React Router

Router clicks as: `URL` → `matched Route` → `component renders`. React Router syncs the address bar with UI so SPAs navigate without reloads.

The core idea:

```text
User visits /users/kakade
    ↓
Routes matches path
    ↓
Route renders element
    ↓
hooks read params + query
```

Install once: `npm i react-router-dom`. All snippets assume v6+.

---

# 1. BrowserRouter, Routes and Route, and 404 Routes

`BrowserRouter` provides history + location to the tree. `Routes and Route` map paths to elements — first best match wins. `404 routes` use `path="*"` as a catch-all fallback.

```jsx
import { BrowserRouter, Routes, Route } from "react-router-dom";

function Home() {
  return <h1>Home of Kakade</h1>;
}

function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="*" element={<p>Not found</p>} />
      </Routes>
    </BrowserRouter>
  );
}
```

Output:

```text
/ → Home of Kakade. /nope → Not found.
/users → Not found (no matching Route).
```

Think:

```text
BrowserRouter (context: location)
    └── Routes (picks one)
          ├── / → <Home />
          └── * → fallback
```

Interview trap: v6 uses `element={<Home />}`, not v5's `component={Home}`. One `BrowserRouter` at the root only.

---

# 2. Link vs NavLink and useNavigate

`Link vs NavLink`: `Link` navigates without reload; `NavLink` additionally knows when it is active for styling menus. `useNavigate` navigates from code — after login, submit, or delete.

```jsx
import { Link, NavLink } from "react-router-dom";

function Nav() {
  return (
    <nav>
      <Link to="/">Home</Link>
      <NavLink to="/users" className={({ isActive }) => (isActive ? "on" : "")}>
        Users of Kakade
      </NavLink>
    </nav>
  );
}
```

Output:

```text
Two client-side links; Users link highlights when on /users.
```

Programmatic navigation:

```jsx
import { useNavigate } from "react-router-dom";

function Login() {
  const navigate = useNavigate();
  function handleLogin() {
    navigate("/dashboard");
  }
  return <button onClick={handleLogin}>Log in Kakade</button>;
}
```

Output:

```text
Click → URL becomes /dashboard, no reload.
```

Use `<Link>` for clicks, `navigate()` for post-action redirects. Variants: `navigate(-1)` goes back, `navigate("/x", { replace: true })` replaces history.

---

# 3. useParams, useSearchParams, and Nested Routes with Outlet

`useParams` reads path segments (`/users/:name`). `useSearchParams` reads `?tab=posts` query state. `Nested routes` + `Outlet` render child routes inside a shared layout.

```jsx
import { useParams } from "react-router-dom";

function UserPage() {
  const { name } = useParams();
  return <h2>User: {name}</h2>;
}

// Route: <Route path="/users/:name" element={<UserPage />} />
```

Output:

```text
/users/kakade → User: kakade
```

Search params + nesting:

```jsx
import { Outlet, useSearchParams } from "react-router-dom";

function Users() {
  const [params, setParams] = useSearchParams();
  const tab = params.get("tab") || "all";
  return (
    <div>
      <button onClick={() => setParams({ tab: "pro" })}>Pro for Kakade</button>
      <p>Tab: {tab}</p>
      <Outlet />
    </div>
  );
}

// <Route path="/users" element={<Users />}>
//   <Route path=":name" element={<UserPage />} />
// </Route>
```

Output:

```text
/users?tab=pro → Tab: pro. /users/kakade renders UserPage inside Outlet.
```

Think: params identify *which* resource, search params hold *how to view* it (sort, tab, page).

---

# 4. Protected Routes Pattern

`Protected routes pattern` wraps private pages: if no user, redirect to `/login`; after login, return to the saved destination. A layout component with `Outlet` + `Navigate` is the standard shape.

```jsx
import { Navigate, Outlet } from "react-router-dom";

function Guard({ user }) {
  if (!user) {
    return <Navigate to="/login" replace />;
  }
  return <Outlet />;
}

// <Route element={<Guard user={user} />}>
//   <Route path="/dashboard" element={<p>Hi Kakade</p>} />
// </Route>
```

Output:

```text
Guest → /login. Kakade → dashboard content.
```

Login restoring the target:

```jsx
import { useLocation, useNavigate } from "react-router-dom";

function LoginBox({ onLogin }) {
  const navigate = useNavigate();
  const location = useLocation();
  const to = location.state?.from || "/dashboard";
  function handleGo() {
    onLogin("Kakade");
    navigate(to, { replace: true });
  }
  return <button onClick={handleGo}>Enter</button>;
}
```

Output:

```text
After login, Kakade lands back where they came from.
```

Keep auth state in Context so `Guard` reads it anywhere without prop drilling.

---

# Real-World Uses

A blog uses every piece: layout with `NavLink` menu, nested `/posts/:slug` + `Outlet`, `?tag=` search filtering, `*` fallback, and a guard so only Kakade edits drafts.

```jsx
import { Link } from "react-router-dom";

function PostLink({ slug, title }) {
  return <Link to={`/posts/${slug}`}>{title} by Kakade</Link>;
}
```

Output:

```text
Click → /posts/hello-world without reload.
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **BrowserRouter** | Root provider of routing context |
| **Routes and Route** | `path` → `element`, best match wins |
| **Link vs NavLink** | Link navigates; NavLink + active styling |
| **useNavigate** | Code-driven navigation + redirects |
| **useParams** | `/users/:name` path values |
| **useSearchParams** | `?tab=` query read + set |
| **Nested routes** | Child `<Route>` inside parent path |
| **Outlet** | Slot where children render |
| **404 routes** | `path="*"` fallback last |
| **Protected routes pattern** | Guard + `Navigate` + `Outlet` |

## The Mental Model

```text
BrowserRouter
  │
  └── Routes
        ├── / → layout (NavLink menu + Outlet)
        │       ├── users/:name → useParams
        │       └── users?tab= → useSearchParams
        ├── guard → logged in? Outlet : Navigate /login
        └── * → 404 fallback
```

### The 3 things to remember

**1. Map URLs:**

```jsx
<Routes>
  <Route path="/" element={<Home />} />
  <Route path="*" element={<p>Not found</p>} />
</Routes>
```

**2. Link + navigate:**

```jsx
<Link to="/users">Users of Kakade</Link>
navigate("/dashboard");
```

**3. Guard private pages:**

```jsx
if (!user) {
  return <Navigate to="/login" replace />;
}
return <Outlet />;
```

> **Map each URL to an element, link without reloads, and guard private branches with a redirect.**
