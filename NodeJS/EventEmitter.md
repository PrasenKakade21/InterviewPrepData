# EventEmitter

Emitter clicks as: `subscribe with on` → `publish with emit` → `react anywhere`. It is Node's built-in pub/sub — the pattern behind servers, streams, and resolvers.

The core idea:

```text
emitter.on("event", handler)
    ↓
emitter.emit("event", data)
    ↓
handler runs for Kakade
```

Decouple producers from consumers; always handle `error` events.

---

# 1. on and Emit, once, and Removing Listeners

`on and emit`: `on` registers, `emit` fires with arguments. `once`: fires a single time, then auto-removes. `Removing listeners` (`off`/`removeListener`) prevents leaks in long-lived processes.

```js
import { EventEmitter } from "node:events";

const bus = new EventEmitter();
bus.on("hi", (name) => console.log(`Hi ${name}`));
bus.emit("hi", "Kakade");
```

Output:

```text
Hi Kakade
```

once + off:

```js
bus.once("welcome", () => console.log("Welcome Kakade (once)"));
bus.emit("welcome");
bus.emit("welcome");
```

Output:

```text
Welcome Kakade (once)
```

Rule: every `on` inside a request/connection handler needs a matching `off` — otherwise listeners pile up per request.

---

# 2. Error Events, Custom Emitter Classes, and Max Listeners Warning

`Error events`: emitting `"error"` with no listener throws — always attach one. `Custom emitter classes` extend `EventEmitter` to give domain objects (chat rooms, job queues) their own events. `Max listeners warning`: >10 listeners on one event warns of a leak; raise with `setMaxListeners` only deliberately.

```js
import { EventEmitter } from "node:events";

class Chat extends EventEmitter {
  join(name) {
    this.emit("join", name);
  }
}

const room = new Chat();
room.on("join", (n) => console.log(`${n} joined`));
room.on("error", (e) => console.error(`Room error: ${e.message}`));
room.join("Kakade");
```

Output:

```text
Kakade joined
```

Think:

```text
Chat extends EventEmitter
  ├── join("Kakade") → emit("join")
  └── failure → emit("error") → caught, no crash
```

Interview line: unhandled `"error"` events crash the process by design — it forces you to face failures.

---

# Real-World Uses

Kakade's job queue extends `EventEmitter` (`done`, `failed`), servers listen for `"request"`, and every emitter gets an `error` handler so one bad job never kills the process.

```js
import { EventEmitter } from "node:events";

const jobs = new EventEmitter();
jobs.on("done", (id) => console.log(`Job ${id} done for Kakade`));
jobs.emit("done", 7);
```

Output:

```text
Job 7 done for Kakade
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **on and emit** | Subscribe, then publish with args |
| **once** | Single-fire, auto-removes |
| **Error events** | No listener → throws; always handle |
| **Removing listeners** | `off` per `on` in hot paths |
| **Custom emitter classes** | Extend for domain events |
| **Max listeners warning** | >10 warns; fix leak, don't just raise |

## The Mental Model

```text
Producer (emit)
  │
  ├── on → every time (Kakade notified)
  ├── once → first time only
  │
  └── error → must have listener
        └── custom class extends EventEmitter
```

### The 3 things to remember

**1. Basic bus:**

```js
bus.on("hi", (name) => console.log(`Hi ${name}`));
```

**2. Domain emitter:**

```js
class Chat extends EventEmitter {}
```

**3. Always catch errors:**

```js
room.on("error", (e) => console.error(e.message));
```

> **Subscribe with on, publish with emit, and never leave error events unhandled.**
