# Streams

Streams click as: `data flows in chunks` → `pipe connects stages` → `pipeline cleans up`. They process gigabytes with megabytes of memory — never `readFile` a huge file.

The core idea:

```text
Readable (source)
    ↓ pipe / pipeline
Writable (destination)
    ↓
Kakade moves GBs in KBs of RAM
```

Default to `pipeline` over `pipe` — it forwards errors and closes everything.

---

# 1. Readable and Writable, pipe Basics, and pipeline for Safety

`Readable and writable`: sources emit `data` chunks; destinations consume via `write`/`end`. `pipe basics` (`src.pipe(dst)`) wires flow with backpressure handled. `pipeline for safety` (from `node:stream/promises`) propagates errors and destroys all stages on failure.

```js
import { Readable, Writable } from "node:stream";
import { pipeline } from "node:stream/promises";

const src = Readable.from(["Hi ", "Kakade"]);
let out = "";
const dst = new Writable({
  write(chunk, _enc, next) {
    out += chunk.toString();
    next();
  },
});
await pipeline(src, dst);
console.log(out);
```

Output:

```text
Hi Kakade
```

Think:

```text
pipe:    src ──→ dst (errors silent!)
pipeline: src ──→ dst (errors thrown, all closed)
```

Trap: bare `.pipe()` swallows mid-chain errors — `pipeline` (or `finished`) is the interview-correct answer.

---

# 2. Object Mode Concept, Backpressure Concept, and fs Streams Example

`Object mode concept` (`objectMode: true`) streams JS values instead of bytes — rows, events, log lines. `Backpressure concept`: when the destination is slow, the source pauses; ignoring `write()`'s `false` return balloons memory. `fs streams example` copies large files chunk-by-chunk.

```js
import { createReadStream, createWriteStream } from "node:fs";
import { pipeline } from "node:stream/promises";

await pipeline(
  createReadStream("big-in.txt"),
  createWriteStream("big-out.txt")
);
console.log("Copied for Kakade without loading it all");
```

Output:

```text
Copied for Kakade without loading it all
```

Object-mode sketch:

```js
// import { Readable } from "node:stream";
// const rows = Readable.from([{ user: "Kakade" }], { objectMode: true });
```

Rule: files, sockets, compression, and CSV parsing stream; small JSON configs can `readFile`.

---

# Real-World Uses

Kakade's log processor pipelines a 2GB file through parse → filter → gzip → upload, using object mode for rows and backpressure to keep memory flat.

```js
import { Readable } from "node:stream";

const names = [];
for await (const n of Readable.from(["Kakade", "Asha"])) {
  names.push(n);
}
console.log(names.join(" and "));
```

Output:

```text
Kakade and Asha
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **Readable and writable** | Sources emit, destinations consume |
| **pipe basics** | Wire flow, handles backpressure |
| **pipeline for safety** | Errors propagate, stages close |
| **Object mode concept** | Stream values, not bytes |
| **Backpressure concept** | Slow dst pauses src |
| **fs streams example** | Copy big files in chunks |

## The Mental Model

```text
Big data for Kakade
  │
  ├── readable → pipeline → writable
  │     └── errors thrown, all destroyed
  │
  ├── objects? → objectMode rows
  │
  └── slow dst? → backpressure pauses src
```

### The 3 things to remember

**1. Safe copy:**

```js
await pipeline(createReadStream("in"), createWriteStream("out"));
```

**2. Object rows:**

```js
Readable.from([{ user: "Kakade" }], { objectMode: true });
```

**3. Consume async:**

```js
for await (const chunk of stream) {
}
```

> **Stream big data, connect with pipeline, and let backpressure guard memory.**
