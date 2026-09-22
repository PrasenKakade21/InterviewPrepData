# HTTP Module

Raw HTTP clicks as: `createServer` → `route by hand` → `JSON out`. It teaches what Express automates — every Node dev should survive one file of it.

The core idea:

```text
req in (method + url)
    ↓
if/switch routing by hand
    ↓
res JSON + status for Kakade
```

Learn it once for interviews; build real APIs with Express next.

---

# 1. createServer Basics, req and Res Objects, and Status Codes and Headers

`createServer basics` (`http.createServer(handler).listen(port)`) starts a raw server. `req and res objects`: `req` carries method/url/headers; `res` sends status, headers, and body. `Status codes and headers` (`writeHead`, `Content-Type`) speak HTTP correctly.

```js
import http from "node:http";

const server = http.createServer((req, res) => {
  res.writeHead(200, { "Content-Type": "application/json" });
  res.end(JSON.stringify({ hi: "Kakade" }));
});
server.listen(3000, () => console.log("Raw server for Kakade on 3000"));
```

Output:

```text
Raw server for Kakade on 3000
```

Fetch check:

```js
// const r = await fetch("http://localhost:3000/");
// console.log(await r.json()); // { hi: "Kakade" }
```

Think: `res.end()` finishes — forget it and the client hangs until timeout.

---

# 2. Hand Routing by URL, JSON Responses, and Why Express Comes Next

`Hand routing by hand`: `if (req.url === … && req.method === …)` branches routes manually. `JSON responses`: `JSON.stringify` + content-type per route. `Why Express comes next`: parsing bodies, params, and middleware by hand is toil — Express standardizes it.

```js
import http from "node:http";

const server = http.createServer((req, res) => {
  res.setHeader("Content-Type", "application/json");
  if (req.method === "GET" && req.url === "/hi") {
    res.end(JSON.stringify({ hi: "Kakade" }));
  } else {
    res.statusCode = 404;
    res.end(JSON.stringify({ error: "Not found" }));
  }
});
server.listen(3000);
```

Output:

```text
GET /hi → { hi: "Kakade" }; anything else → 404 JSON.
```

| Raw http | Express |
| -------- | ------- |
| **Manual url/method ifs** | **`app.get("/hi", …)`** |
| **Manual body assembly** | **`express.json()`** |
| **No middleware chain** | **`use()` pipeline** |

Interview line: raw `http` proves understanding; Express proves productivity — know both, ship with Express.

---

# Real-World Uses

Kakade's health-check sidecar is 10 lines of raw `http` (no deps), while the main API runs Express — and debugging middleware always starts from "what would raw req/res do?".

```js
import http from "node:http";

const s = http.createServer((req, res) => {
  res.end(JSON.stringify({ ok: true, for: "Kakade" }));
});
s.listen(3001);
```

Output:

```text
GET localhost:3001 → {"ok":true,"for":"Kakade"}.
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **createServer basics** | Handler + listen(port) |
| **req and res objects** | In: method/url; out: status/body |
| **Hand routing by URL** | Manual method+url ifs |
| **JSON responses** | stringify + content-type |
| **Status codes and headers** | writeHead / statusCode |
| **Why Express comes next** | Bodies, params, middleware toil |

## The Mental Model

```text
Request (method + url)
  │
  ├── match route by hand → JSON + status
  │
  └── miss → 404 JSON
        └── repeated toil? → Express next
```

### The 3 things to remember

**1. Minimal server:**

```js
http.createServer((req, res) => {
  res.end(JSON.stringify({ hi: "Kakade" }));
});
```

**2. Hand route:**

```js
if (req.method === "GET" && req.url === "/hi") {
}
```

**3. Always end:**

```js
res.writeHead(200, { "Content-Type": "application/json" });
```

> **Raw http teaches the req/res contract — then let Express carry the toil.**
