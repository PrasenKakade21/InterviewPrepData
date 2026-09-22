# Build Tools and Project Setup

Setup clicks as: `Vite scaffolds` → `structure + aliases scale` → `env + lint guard quality`. Tooling is invisible when right — builds fast, imports clean, prod safe.

The core idea:

```text
npm create vite → src/ routes, features, lib
    ↓
@ alias + .env files → clean imports, per-env config
    ↓
ESLint + Prettier + build check → ship
```

Vite is the default; CRA is legacy knowledge for interviews.

---

# 1. Vite Setup, CRA Legacy Overview, and Project Structure

`Vite setup` (`npm create vite@latest … -- --template react`) gives instant HMR and Rollup builds. `CRA legacy overview` (`react-scripts`, webpack, slow) still appears in old codebases. `Project structure` groups by feature (`features/auth/`, `components/`, `lib/`) once past toy size.

```bash
npm create vite@latest kakade-app -- --template react
cd kakade-app
npm install
npm run dev
```

Output:

```text
Vite ready in ms → HMR at localhost:5173 for Kakade.
```

Structure:

```text
src/
├── components/ (Button, Modal)
├── features/ (auth/, users/)
├── lib/ (api, query client)
├── routes/ (page entries)
└── main.jsx (providers)
```

Colocate tests, styles, and types with features — flat `components/` only for truly shared UI.

---

# 2. Absolute Imports, Environment Variables, and Vite Env Handling

`Absolute imports` (`@/...`) avoid `../../../` via `vite.config.js` alias + `jsconfig/tsconfig` paths. `Environment variables` inject per-env config; `Vite env handling` exposes only `VITE_*` via `import.meta.env`.

```js
// vite.config.js
// import { defineConfig } from "vite";
// export default defineConfig({ resolve: { alias: { "@": "/src" } } });
```

```jsx
import Button from "@/components/Button";

function App() {
  return <Button>Hi Kakade</Button>;
}
```

Output:

```text
@/components/Button resolves to src/components/Button.
```

Env shape:

```bash
VITE_API_URL=https://api.kakade.dev
```

```js
// fetch(`${import.meta.env.VITE_API_URL}/users`) → per-env API.
// Never VITE_SECRET — client env is public.
```

Restart dev after env changes; validate required vars at startup with a tiny `env.js` guard.

---

# 3. ESLint Basics, Prettier Basics, and Dev vs Production Builds

`ESLint basics` (`eslint react/recommended`, hooks rules) catches bugs; `Prettier basics` formats on save. `Dev vs production builds`: dev favors speed + sourcemaps; `npm run build` minifies, hashes, and tree-shakes for deploy.

```bash
npm run dev
npm run build
npm run preview
```

Output:

```text
dev → HMR for Kakade; build → dist/ hashed assets; preview → local prod check.
```

Checklist: `react-hooks` rules on error, no-unused-vars fails CI, format on save, and always `preview` (or serve `dist/`) before blaming production.

---

# Real-World Uses

A Kakade app scaffolded with Vite uses `@/` imports, `VITE_API_URL` per env, feature folders, and a lint + build CI gate so broken hooks or missing envs never reach Vercel.

```bash
npm run build
```

Output:

```text
dist/ ready for Kakade (hashed, minified).
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **Vite setup** | `create vite` + HMR default |
| **CRA legacy overview** | `react-scripts`, old/webpack |
| **Project structure** | Features + shared components |
| **Absolute imports** | `@` alias, no `../../..` |
| **Environment variables** | Per-env config, validated |
| **Vite env handling** | `VITE_*` + `import.meta.env` |
| **ESLint basics** | Hooks rules on error |
| **Prettier basics** | Format on save |
| **Dev vs production builds** | HMR vs minified `dist/` |

## The Mental Model

```text
Scaffold (Vite)
  │
  ├── structure: features + @ alias
  │
  ├── config: VITE_* env per deploy
  │
  └── quality: ESLint + Prettier → build → preview
```

### The 3 things to remember

**1. Scaffold:**

```bash
npm create vite@latest kakade-app -- --template react
```

**2. Alias import:**

```jsx
import Button from "@/components/Button";
```

**3. Env + build:**

```bash
VITE_API_URL=https://api.kakade.dev
npm run build
```

> **Scaffold with Vite, alias + feature-folders for scale, and gate deploys on lint + prod build.**
