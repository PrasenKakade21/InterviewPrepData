# Uploads and Static

Files click as: `static serves reads` → `multer handles writes` → `limits guard abuse`. Serving is one line; uploads need validation, caps, and storage decisions.

The core idea:

```text
GET /files/x.png → express.static
    ↓
POST multipart → multer → disk/memory
    ↓
validate type + size → URL for Kakade
```

Install block (needed — third-party package):

```bash
npm install multer
```

---

# 1. Serving Static Files and multer Basics

`Serving static files` (`express.static("public")`) serves built frontends, images, and docs with caching headers. `multer basics` (`upload.single("avatar")`) parses multipart bodies into `req.file`.

```js
import express from "express";

const app = express();
app.use("/files", express.static("public"));
app.get("/logo", (req, res) => res.json({ at: "/files/kakade.png" }));
app.listen(3000);
```

Output:

```text
GET /files/kakade.png → image bytes; GET /logo → {"at":"/files/kakade.png"}.
```

Upload shape:

```js
// import multer from "multer";
// const upload = multer({ dest: "uploads/" });
// app.post("/avatar", upload.single("avatar"), (req, res) => {
//   res.status(201).json({ saved: req.file.filename, for: "Kakade" });
// });
```

Rule: static for public reads, multer for authenticated writes — never accept uploads anonymously.

---

# 2. File Limits and Filters and S3 Concept Overview

`File limits and filters` (`limits: { fileSize }`, `fileFilter` by mimetype) reject oversized or dangerous files before storage. `S3 concept overview`: local disk works for dev; production streams to object storage (S3/R2) and serves via CDN.

```js
import multer from "multer";

const upload = multer({
  dest: "uploads/",
  limits: { fileSize: 2 * 1024 * 1024 },
  fileFilter: (req, file, cb) => {
    if (!file.mimetype.startsWith("image/")) {
      return cb(new Error("Images only for Kakade"));
    }
    cb(null, true);
  },
});
console.log("Uploader ready: 2MB images only");
```

Output:

```text
Uploader ready: 2MB images only
```

Think:

```text
POST avatar (5MB exe)
  ├── size? ✗ → 413
  ├── type? ✗ → 400 Images only
  └── ✓ → stored → URL for Kakade
```

Production path: multer memory → stream to S3 → store URL in DB; local `uploads/` never survives deploys.

---

# Real-World Uses

Kakade's profile upload caps at 2MB images, renames with random ids (no user filenames on disk), and serves avatars from a CDN while dev uses `express.static`.

```js
import multer from "multer";

const up = multer({ dest: "uploads/" });
console.log(typeof up.single);
```

Output:

```text
function
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **Serving static files** | `express.static` for public reads |
| **multer basics** | `upload.single` → `req.file` |
| **File limits and filters** | Size caps + mimetype gates |
| **S3 concept overview** | Object storage + CDN in prod |

## The Mental Model

```text
Files for Kakade
  │
  ├── reads → express.static (cached)
  │
  └── writes → multer → limits + filter
        ├── dev → local disk
        └── prod → S3 → CDN URL in DB
```

### The 3 things to remember

**1. Serve:**

```js
app.use("/files", express.static("public"));
```

**2. Accept:**

```js
app.post("/avatar", upload.single("avatar"), handler);
```

**3. Guard:**

```js
limits: { fileSize: 2 * 1024 * 1024 },
```

> **Serve reads statically, gate uploads by size and type, and store production files in object storage.**
