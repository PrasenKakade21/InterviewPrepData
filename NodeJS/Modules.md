# Modules

Modules click as: `one file, one unit` → `export what others need` → `import by path or package`. CommonJS (`require`) is the legacy default; ES modules (`import`) are the modern standard.

The core idea:

```text
greet.js exports function
    ↓
app.js imports it
    ↓
Kakade reuses code without globals
```

New projects: use ESM (`"type": "module"`) unless a dependency forces CJS.

---

# 1. CommonJS Require and Exports, ES Modules Import and Export, and CJS vs ESM Differences

`CommonJS require and exports` loads synchronously via `require()` and shares via `module.exports`. `ES modules import and export` use `import/export` with static analysis and top-level await. `CJS vs ESM differences`: CJS is dynamic + sync; ESM is static + async-capable, with mandatory file extensions.

```js
// greet.cjs — CommonJS
function greet(name) {
  return `Hi ${name}`;
}
module.exports = { greet };

// app.cjs
const { greet } = require("./greet.cjs");
console.log(greet("Kakade"));
```

Output:

```text
Hi Kakade
```

ESM version:

```js
// greet.js — ESM
export function greet(name) {
  return `Hi ${name}`;
}

// app.js
import { greet } from "./greet.js";
console.log(greet("Kakade"));
```

Output:

```text
Hi Kakade
```

| Feature | CJS | ESM |
| ------- | --- | --- |
| **Syntax** | **`require` / `module.exports`** | **`import` / `export`** |
| **Loading** | **Sync, dynamic** | **Async-capable, static** |
| **Extension** | **Optional** | **Mandatory (`.js`)** |
| **Top-level await** | **No** | **Yes** |

Trap: mixing them (`require` of ESM) throws `ERR_REQUIRE_ESM` — pick one system per project.

---

# 2. Core Modules Overview, dirname in ESM, and Circular Dependency Basics

`Core modules overview` (`fs`, `path`, `os`, `http`, `events`) ship with Node — no install. `dirname in ESM`: `__dirname` does not exist in ESM; rebuild it from `import.meta.url`. `Circular dependency basics`: A requires B requires A — CJS returns a partial export, ESM handles bindings better; both signal a design smell.

```js
import path from "node:path";
console.log(path.join("users", "kakade", "notes.txt"));
```

Output:

```text
users\kakade\notes.txt
```

ESM dirname pattern:

```js
// import { fileURLToPath } from "node:url";
// import { dirname } from "node:path";
// const __dirname = dirname(fileURLToPath(import.meta.url));
```

Prefer the `node:` prefix (`node:fs`, `node:path`) — it marks core modules explicitly and avoids package shadowing.

---

# Real-World Uses

Kakade's API uses ESM imports with `node:` core modules, a `lib/` folder of small exported helpers, and a startup check that fails fast if a required module is missing.

```js
import os from "node:os";
console.log(`Kakade runs on ${os.platform()}`);
```

Output:

```text
Kakade runs on win32
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **CommonJS require and exports** | `require` + `module.exports`, sync |
| **ES modules import and export** | `import` + `export`, static |
| **CJS vs ESM differences** | Dynamic/sync vs static/async, extensions |
| **Core modules overview** | `node:fs/path/os/http/events` built in |
| **dirname in ESM** | Rebuild from `import.meta.url` |
| **Circular dependency basics** | Partial exports; refactor shared code out |

## The Mental Model

```text
Code sharing
  │
  ├── CJS → require (sync, legacy)
  │
  ├── ESM → import (static, preferred)
  │     └── node: prefix for core
  │
  └── circular? → extract shared module for Kakade
```

### The 3 things to remember

**1. ESM export/import:**

```js
export function greet(name) {
  return `Hi ${name}`;
}
```

**2. Core with prefix:**

```js
import path from "node:path";
```

**3. ESM dirname:**

```js
const __dirname = dirname(fileURLToPath(import.meta.url));
```

> **One file one module, ESM by default, node: prefix for core — and never mix systems.**
