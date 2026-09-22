# Buffers

Buffers click as: `binary arrives` → `Buffer holds raw bytes` → `decode to string when ready`. They exist because network and file data is bytes first, text second.

The core idea:

```text
Raw bytes (file, socket)
    ↓
Buffer (fixed raw memory)
    ↓
.toString("utf8") → Kakade reads it
```

Reach for Buffers at I/O boundaries; work in strings everywhere else.

---

# 1. What Buffers Are For, Buffer.alloc and Buffer.from, and Encodings

`What buffers are for`: holding binary (images, TCP chunks, file bytes) that is not yet valid text. `Buffer.alloc and Buffer.from`: `alloc` makes zeroed memory of a size; `from` wraps existing string/array data. `Encodings` (`utf8`, `hex`, `base64`) translate between bytes and text.

```js
const buf = Buffer.from("Kakade");
console.log(buf.length);
console.log(buf.toString("base64"));
```

Output:

```text
6
S2FrYWRl
```

Alloc shape:

```js
const safe = Buffer.alloc(4);
console.log(safe);
```

Output:

```text
<Buffer 00 00 00 00>
```

Trap: `new Buffer()` is deprecated and unsafe (uninitialized memory) — always `alloc` or `from`.

---

# 2. Concat and Slice and Buffer to JSON Pattern

`Concat and slice`: `Buffer.concat(list)` joins chunks from streams/sockets; `subarray` views a range without copying. `Buffer to JSON pattern`: `.toJSON()` serializes bytes for transport, `Buffer.from(arr)` rebuilds them.

```js
const a = Buffer.from("Kak");
const b = Buffer.from("ade");
const full = Buffer.concat([a, b]);
console.log(full.toString());
console.log(full.subarray(0, 3).toString());
```

Output:

```text
Kakade
Kak
```

JSON round-trip:

```js
const packed = Buffer.from("Kakade").toJSON();
console.log(packed.type);
const back = Buffer.from(packed.data).toString();
console.log(back);
```

Output:

```text
Buffer
Kakade
```

Think: TCP delivers `Kak` + `ade` separately — concat reassembles before decoding, or multi-byte characters corrupt.

---

# Real-World Uses

Kakade's upload handler concats request chunks into one Buffer, decodes base64 avatars, and serializes binary payloads to JSON for the queue.

```js
const chunks = [Buffer.from("Hi "), Buffer.from("Kakade")];
console.log(Buffer.concat(chunks).toString());
```

Output:

```text
Hi Kakade
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **What buffers are for** | Raw bytes before text |
| **Buffer.alloc and Buffer.from** | Zeroed size vs wrap data |
| **Encodings** | utf8, hex, base64 bridges |
| **Concat and slice** | Reassemble chunks; view ranges |
| **Buffer to JSON pattern** | `.toJSON()` + `from(data)` round-trip |

## The Mental Model

```text
Bytes in (socket, file)
  │
  ├── alloc (empty) / from (data)
  │
  ├── concat chunks → complete Buffer
  │
  └── decode (utf8/base64) → text for Kakade
```

### The 3 things to remember

**1. Wrap text:**

```js
const buf = Buffer.from("Kakade");
```

**2. Reassemble:**

```js
const full = Buffer.concat([a, b]);
```

**3. Decode late:**

```js
full.toString("utf8");
```

> **Bytes first with alloc/from, concat chunks at boundaries, decode to text last.**
