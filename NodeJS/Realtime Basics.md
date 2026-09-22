# Realtime Basics

Realtime clicks as: `HTTP request/response ends` → `WebSocket stays open` → `both sides push anytime`. Use sockets for chat, live dashboards, and notifications — not for CRUD.

The core idea:

```text
Client ↔ persistent socket
    ↓
message → server → broadcast
    ↓
Kakade sees Asha's chat instantly
```

Install block (needed — third-party package):

```bash
npm install ws
```

---

# 1. WebSocket Concept, ws Server Basics, and Broadcast Pattern

`WebSocket concept`: one handshake upgrades HTTP to a full-duplex channel — no polling. `ws server basics` (`new WebSocketServer({ server })`) shares the Express port. `Broadcast pattern`: loop `wss.clients` and send to every open socket.

```js
import http from "node:http";
import { WebSocketServer } from "ws";

const server = http.createServer();
const wss = new WebSocketServer({ server });
wss.on("connection", (ws) => {
  ws.send(JSON.stringify({ hi: "Kakade" }));
});
server.listen(3000, () => console.log("Sockets for Kakade on 3000"));
```

Output:

```text
Sockets for Kakade on 3000 (client receives {"hi":"Kakade"} on connect).
```

Broadcast sketch:

```js
// wss.on("connection", (ws) => {
//   ws.on("message", (raw) => {
//     for (const c of wss.clients) c.send(raw.toString()); // everyone hears Kakade.
//   });
// });
```

Think: HTTP is letters; WebSocket is a phone call — keep the call only when conversation is continuous.

---

# 2. Rooms Concept and When Sockets Beat Polling

`Rooms concept`: sets of sockets per topic (`room:js`, `dm:kakade`) so broadcasts target groups, not the world. `When sockets beat polling`: sub-second updates with many listeners (chat, scores, cursors) — polling wins for rare or cacheable data.

```js
const rooms = new Map();

function join(room, ws) {
  if (!rooms.has(room)) rooms.set(room, new Set());
  rooms.get(room).add(ws);
  console.log(`Kakade joined ${room}: ${rooms.get(room).size} inside`);
}
```

Output:

```text
Kakade joined js: 1 inside
```

| Use sockets | Use polling/fetch |
| ----------- | ----------------- |
| **Chat for Kakade** | **Daily report** |
| **Live scores** | **Profile fetch** |
| **Collaborative cursors** | **Paginated lists** |

Cleanup rule: remove sockets on `close` or rooms leak — every `join` needs a `close` handler with `delete`.

---

# Real-World Uses

Kakade's notifications fan out per-user rooms, the chat broadcasts per channel, and everything else stays REST — sockets only where pushes are truly live.

```js
import { WebSocketServer } from "ws";

console.log(typeof WebSocketServer);
```

Output:

```text
function
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **WebSocket concept** | Persistent full-duplex channel |
| **ws server basics** | Share port, greet on connect |
| **Broadcast pattern** | Loop open clients, send all |
| **Rooms concept** | Sets per topic, targeted fan-out |

## The Mental Model

```text
HTTP (request → response, done)
  │
  └── live? → WebSocket (open call)
        ├── connect → greet Kakade
        ├── message → broadcast / room fan-out
        └── close → remove from rooms
```

### The 3 things to remember

**1. Attach:**

```js
const wss = new WebSocketServer({ server });
```

**2. Broadcast:**

```js
for (const c of wss.clients) c.send(raw.toString());
```

**3. Room:**

```js
rooms.get(room).add(ws);
```

> **Sockets for live pushes with room fan-out — CRUD stays on REST.**
