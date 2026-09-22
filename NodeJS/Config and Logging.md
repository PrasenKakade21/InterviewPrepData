# Config and Logging

Config + logs click as: `env validated at boot` → `structured logs per request` → `levels filter noise`. Fail fast on missing config; log JSON machines can grep.

The core idea:

```text
.env → validated env.js (crash if bad)
    ↓
pino(info/warn/error) + request ids
    ↓
Kakade debugs prod from logs, not guesses
```

Install block (needed — third-party packages):

```bash
npm install dotenv pino
```

---

# 1. dotenv Basics, Validating Env at Startup, and Console vs Logger

`dotenv basics` (`config()` loads `.env` into `process.env`) keeps secrets out of code. `Validating env at startup` (required-keys check that throws) fails in seconds, not at 3am on first request. `Console vs logger`: `console.log` is unstructured stdout; pino gives levels, JSON, and speed.

```js
import "dotenv/config";

const required = ["PORT", "JWT_SECRET"];
for (const k of required) {
  if (!process.env[k]) throw new Error(`Missing ${k} for Kakade`);
}
console.log("Config ok");
```

Output:

```text
Error: Missing PORT for Kakade (fast, at boot — before listen).
```

With env set:

```bash
PORT=3000 JWT_SECRET=dev node server.js
```

```text
Config ok
```

Rule: every `process.env.X` read goes through one validated module — no scattered unchecked reads.

---

# 2. pino Basics, Log Levels, and Request Logging

`pino basics` (`pino()` + `.info/.error`) emit line-delimited JSON. `Log levels` (`debug < info < warn < error`) filter by environment — debug locally, warn+ in prod. `Request logging` (tiny middleware stamping method/url/status/ms) traces every call.

```js
import pino from "pino";

const log = pino({ level: "info" });
log.info({ user: "Kakade" }, "logged in");
log.debug("hidden at info level");
```

Output:

```text
{"level":30,"msg":"logged in","user":"Kakade",...} (debug line absent).
```

Request middleware:

```js
// app.use((req, res, next) => {
//   const t = Date.now();
//   res.on("finish", () => log.info({ method: req.method, url: req.url, status: res.statusCode, ms: Date.now() - t }, "req"));
//   next();
// });
```

Never log passwords, tokens, or full headers — redact at the logger (`redact: ["req.headers.authorization"]`).

---

# Real-World Uses

Kakade's API boots only with valid env, logs JSON per request with ids, and ships logs to a collector — prod issues start from a log query, not a reproduction.

```js
import pino from "pino";

const log = pino({ level: "warn" });
log.info("quiet for Kakade");
log.warn("visible warning");
```

Output:

```text
{"level":40,"msg":"visible warning",...} (info filtered).
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **dotenv basics** | `.env` → `process.env`, never committed |
| **Validating env at startup** | Throw fast on missing keys |
| **Console vs logger** | Unstructured vs levels + JSON |
| **pino basics** | `pino()` + leveled methods |
| **Log levels** | debug/info/warn/error per env |
| **Request logging** | Method/url/status/ms per call |

## The Mental Model

```text
Boot
  │
  ├── dotenv → validate required → crash fast
  │
  └── pino (level per env) → request middleware
        └── JSON lines → collector for Kakade
```

### The 3 things to remember

**1. Load + guard:**

```js
import "dotenv/config";
if (!process.env.JWT_SECRET) throw new Error("Missing JWT_SECRET for Kakade");
```

**2. Levelled log:**

```js
const log = pino({ level: "info" });
```

**3. Per-request:**

```js
log.info({ method: req.method, url: req.url }, "req");
```

> **Validate config at boot, log structured lines with levels, and never log secrets.**
