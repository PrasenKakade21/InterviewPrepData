# Deployment and Production

Deploy clicks as: `build passes` → `static host + rewrites` → `cache + env per deploy`. React ships static files — routing and caching decide whether prod feels instant or broken.

The core idea:

```text
npm run build → dist/
    ↓
host (Vercel/Netlify) + SPA rewrites
    ↓
CDN cache + env configs → checklist
```

Preview `dist/` locally before blaming the host.

---

# 1. Production Builds, Build Size Checks, and Environment Configs

`Production builds` (`npm run build`) minify, hash, and tree-shake into `dist/`. `Build size checks` (`vite-bundle-visualizer`, `source-map-explorer`) find heavy deps before users do. `Environment configs` (`.env.production`, host vars) point each deploy at its API.

```bash
npm run build
```

Output:

```text
dist/ for Kakade: index.html + hashed js/css.
```

Size sketch:

```bash
npx vite-bundle-visualizer
```

```text
chart.js 220kb → lazy-load for Kakade admin only.
```

Rule: budgets over hope — fail CI past limits, lazy-load charts/maps, and never ship sourcemaps publicly unless intended.

---

# 2. Vercel Deployment, Netlify Deployment, and Client-Side Routing Rewrites

`Vercel deployment` (`vercel` / git import, framework preset Vite) and `Netlify deployment` (`netlify deploy`, build `npm run build` → `dist`) both serve static + set env in dashboards. `Client-side routing rewrites` send all paths to `index.html` so `/users/kakade` refresh works.

```bash
vercel
```

Output:

```text
Kakade app live on https://…vercel.app.
```

Netlify config:

```toml
# netlify.toml
[build]
  command = "npm run build"
  publish = "dist"
[[redirects]]
  from = "/*"
  to = "/index.html"
  status = 200
```

Vercel equivalent (`vercel.json`): `{ "rewrites": [{ "source": "/(.*)", "destination": "/index.html" }] }`. Without rewrites, deep links 404 on refresh.

---

# 3. Caching and CDN Basics and Production Checklist

`Caching and CDN basics`: hashed assets cache forever (`immutable`), `index.html` never caches (always fresh). `Production checklist` verifies env, routes, errors, and perf before announcing.

```text
/dist/assets/app-abc123.js → Cache-Control: immutable, 1y
/dist/index.html → Cache-Control: no-cache
```

Checklist:

```text
[ ] VITE_API_URL correct for Kakade prod
[ ] /users/kakade refresh works (rewrites)
[ ] 404 + error boundary render
[ ] Lighthouse + bundle budget pass
[ ] Sentry DSN set, sourcemaps uploaded
```

Purge CDN on rollback; keep preview deploys per PR so QA tests the real bundle.

---

# Real-World Uses

A Kakade portfolio deploys previews per PR on Vercel, ships hashed assets via CDN, rewrites SPA routes, and blocks merge unless build + budget + Lighthouse pass.

```bash
npm run preview
```

Output:

```text
Local prod check for Kakade before deploy.
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **Production builds** | `npm run build` → hashed `dist/` |
| **Build size checks** | Visualize, budget, lazy-load |
| **Environment configs** | Per-deploy `VITE_*` vars |
| **Vercel deployment** | Git import / `vercel` |
| **Netlify deployment** | Build `dist` + dashboard env |
| **Client-side routing rewrites** | `/* → /index.html` (200) |
| **Caching and CDN basics** | Assets immutable, html fresh |
| **Production checklist** | Env, routes, errors, perf |

## The Mental Model

```text
build (hashed dist/)
  │
  ├── host: Vercel / Netlify + env vars
  │
  ├── rewrites: /* → index.html
  │
  └── CDN: assets immutable, html no-cache → checklist
```

### The 3 things to remember

**1. Build:**

```bash
npm run build
```

**2. SPA rewrite:**

```toml
[[redirects]]
from = "/*"
to = "/index.html"
```

**3. Cache split:**

```text
assets → immutable; index.html → no-cache
```

> **Build hashed bundles, rewrite SPA routes, and cache assets forever but HTML never.**
