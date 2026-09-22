# Caching and Performance

Speed clicks as: `measure first` → `cache repeated work` → `use all cores`. Caching beats optimizing — a cache hit skips the work entirely.

The core idea:

```text
Slow repeated response
    ↓
cache (memory → Redis) with TTL
    ↓
cores via cluster for Kakade scale
```

Profile before tuning; the fastest query is the one never run.

---

# 1. In-Memory Cache Pattern, Redis Brief, and Compression with gzip

`In-memory cache pattern` (`Map` + expiry timestamps) serves hot data per process. `Redis brief`: shared external cache across processes/servers with `SET key value EX 60`. `Compression with gzip` (`compression` middleware) shrinks JSON/text over the wire.

```js
const cache = new Map();

function cached(key, ttlMs, build) {
  const hit = cache.get(key);
  if (hit && hit.until > Date.now()) return hit.value;
  return build().then((v) => {
    cache.set(key, { value: v, until: Date.now() + ttlMs });
    return v;
  });
}
const v = await cached("kakade", 60_000, async () => ({ user: "Kakade" }));
console.log(v);
```

Output:

```text
{ user: 'Kakade' } (built once, cached 60s after).
```

Redis sketch:

```js
// await redis.set("kakade", JSON.stringify(user), { EX: 60 });
// const hit = await redis.get("kakade"); // shared across Kakade's servers.
```

Install for wire savings:

```bash
npm install compression
```

Rule: cache at the slowest layer (DB results, rendered pages) with TTLs — invalidation by expiry beats manual busting.

---

# 2. Clustering Concept and Worker Threads Concept

`Clustering concept` (`node:cluster`) forks one worker per core sharing the port — N× throughput for I/O servers. `Worker threads concept` (`node:worker_threads`) runs CPU sprints (hashing, parsing) off the event loop in the same process.

```js
import cluster from "node:cluster";
import os from "node:os";

if (cluster.isPrimary) {
  console.log(`Primary plans ${os.cpus().length > 0 ? "workers" : "none"} for Kakade`);
} else {
  console.log("Worker serving Kakade");
}
```

Output:

```text
Primary plans workers for Kakade (one fork per core in real setup).
```

| Scale tool | For |
| ---------- | --- |
| **Cluster** | **More I/O throughput (per core)** |
| **Worker threads** | **CPU sprints off the loop** |
| **Redis** | **Shared cache across boxes** |

Interview line: cluster scales requests, workers unblock the loop, cache avoids the work — three different bottlenecks, three tools.

---

# Real-World Uses

Kakade's API caches profiles in Redis (60s TTL), gzips JSON, clusters to core count in prod, and pushes thumbnail rendering to worker threads.

```js
const cache2 = new Map([["kakade", 1]]);
console.log(cache2.has("kakade"));
```

Output:

```text
true
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **In-memory cache pattern** | Map + TTL per process |
| **Redis brief** | Shared cache with EX |
| **Compression with gzip** | Shrink JSON/text wire |
| **Clustering concept** | Fork per core, shared port |
| **Worker threads concept** | CPU off the event loop |

## The Mental Model

```text
Slow for Kakade?
  │
  ├── repeat? → cache (memory → Redis, TTL)
  │
  ├── big JSON? → gzip wire
  │
  └── cores idle? → cluster; loop blocked? → workers
```

### The 3 things to remember

**1. Cache with TTL:**

```js
cache.set(key, { value: v, until: Date.now() + ttlMs });
```

**2. Share via Redis:**

```js
await redis.set("kakade", JSON.stringify(user), { EX: 60 });
```

**3. Fork per core:**

```js
os.cpus().length;
```

> **Cache repeated work, compress the wire, and spend every core — after measuring.**
