# npm and Project Setup

Setup clicks as: `npm init` → `install saves to package.json` → `scripts run the workflow`. `package.json` is the project contract; the lockfile makes installs reproducible.

The core idea:

```text
npm init -y → package.json
    ↓
npm install <pkg> → node_modules + lock
    ↓
npm run <script> → dev, test, start for Kakade
```

Commit `package.json` + lockfile; never commit `node_modules`.

---

# 1. npm Init and package.json, Dependencies vs devDependencies, and Lockfiles and node_modules

`npm init and package.json` declares name, version, type (module), and scripts. `Dependencies vs devDependencies`: runtime needs (`express`) vs tooling (`vitest`, `nodemon`). `Lockfiles and node_modules`: `package-lock.json` pins exact trees; `node_modules` is the disposable install output.

```json
{
  "name": "kakade-api",
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "dev": "node --watch server.js",
    "start": "node server.js"
  }
}
```

Output:

```text
npm run dev → watched server for Kakade; npm start → production run.
```

Commands:

```bash
npm init -y
npm install express
npm install -D nodemon
```

Output:

```text
express → dependencies; nodemon → devDependencies for Kakade.
```

Rule: `npm ci` (clean install from lock) in CI/deploy; `npm install` for local adds.

---

# 2. npm Scripts, npx Basics, and gitignore for Node

`npm scripts` alias commands (`dev`, `test`, `start`) so contributors run one word. `npx basics` executes packages without installing (`npx prettier .`). `gitignore for Node` excludes `node_modules`, `.env`, and `dist/`.

```bash
npx prettier --check .
npm run dev
```

Output:

```text
Format checked, dev server for Kakade starts.
```

`.gitignore`:

```text
node_modules/
.env
dist/
```

Convention: `dev` (watch), `start` (prod entry), `test` (suite) — every Node project needs all three.

---

# Real-World Uses

Kakade's API repo has `type: module`, `dev/start/test` scripts, a committed lockfile, and CI running `npm ci && npm test` so installs are identical everywhere.

```bash
npm ci
npm test
```

Output:

```text
Clean install for Kakade → tests pass on identical tree.
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **npm init and package.json** | Project contract + scripts |
| **Dependencies vs devDependencies** | Runtime vs tooling |
| **npm scripts** | One-word workflows |
| **npx basics** | Run without installing |
| **Lockfiles and node_modules** | Pin + disposable output |
| **gitignore for Node** | Exclude modules, env, dist |

## The Mental Model

```text
npm init -y
  │
  ├── deps vs devDeps → lockfile pins tree
  │
  └── scripts: dev / start / test
        └── node_modules ignored, never committed
```

### The 3 things to remember

**1. Init ESM:**

```bash
npm init -y
```

**2. Install split:**

```bash
npm install express
npm install -D nodemon
```

**3. CI install:**

```bash
npm ci
```

> **Contract in package.json, reproducible via lockfile, workflows in scripts — ignore the rest.**
