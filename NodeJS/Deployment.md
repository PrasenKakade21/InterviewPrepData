# Deployment

Deploy clicks as: `start script` → `process manager or container` → `health + env per deploy`. Node crashes are normal — production means auto-restart, not zero crashes.

The core idea:

```text
npm start → dist/server.js
    ↓
PM2 / Docker keeps it alive
    ↓
/health + env vars for Kakade prod
```

Preview `npm start` locally before blaming the host.

---

# 1. Production Start Scripts, PM2 Basics, and Health Check Endpoint

`Production start scripts` (`"start": "node dist/server.js"`) run compiled output with `NODE_ENV=production`. `PM2 basics` (`pm2 start`, `save`, `startup`) restart crashed apps and resurrect on reboot. `Health check endpoint` (`GET /health` → 200 + uptime) tells load balancers the app is alive.

```js
import express from "express";

const app = express();
app.get("/health", (req, res) => {
  res.json({ ok: true, uptime: Math.round(process.uptime()), for: "Kakade" });
});
const server = app.listen(3000);
const res = await fetch("http://localhost:3000/health");
console.log((await res.json()).ok);
server.close();
```

Output:

```text
true
```

PM2 shape:

```bash
npm run build
pm2 start dist/server.js --name kakade-api
pm2 save
pm2 startup
```

Interview line: PM2 is babysitting (restarts, logs, clusters) — the app still needs `/health` so outsiders can verify.

---

# 2. Docker Basics and Env per Deploy

`Docker basics`: a `Dockerfile` (slim base → deps → build → start) makes Kakade's API identical everywhere. `Env per deploy`: staging and prod differ only in host-injected vars, never code branches.

```dockerfile
FROM node:24-slim
WORKDIR /app
COPY package*.json ./
RUN npm ci --omit=dev
COPY . .
RUN npm run build
EXPOSE 3000
CMD ["node", "dist/server.js"]
```

Staging vs prod:

```bash
docker run -e PORT=3000 -e JWT_SECRET=staging-x kakade-api
docker run -e PORT=3000 -e JWT_SECRET=prod-y kakade-api
```

```text
Same image, different env — Kakade's config rule.
```

Checklist:

```text
[ ] .dockerignore skips node_modules + .env for Kakade
[ ] /health wired to orchestrator probes
[ ] logs to stdout (collector grabs them)
[ ] secrets in host vault, never baked in
```

One image, many envs — rebuilds happen for code, not config.

---

# Real-World Uses

Kakade's API ships one Docker image per commit, runs it under PM2 on a VPS (or containers on a platform), and gates traffic on `/health` with per-deploy env.

```js
console.log("start: node dist/server.js for Kakade");
```

Output:

```text
start: node dist/server.js for Kakade
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **Production start scripts** | `node dist/`, NODE_ENV set |
| **PM2 basics** | Start, save, startup |
| **Docker basics** | One image, slim, ci+build |
| **Health check endpoint** | `/health` → 200 + uptime |
| **Env per deploy** | Host vars, not code branches |

## The Mental Model

```text
build → start (dist/)
  │
  ├── PM2 / Docker → restarts for Kakade
  │
  └── /health → probe passes → traffic
        └── env injected per deploy
```

### The 3 things to remember

**1. Start:**

```bash
npm run build
node dist/server.js
```

**2. Keep alive:**

```bash
pm2 start dist/server.js --name kakade-api
```

**3. Prove alive:**

```js
app.get("/health", (req, res) => res.json({ ok: true }));
```

> **Ship one image, keep it alive with PM2 or Docker, and prove it with /health.**
