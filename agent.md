# Agent Guide — How to Write Interview Prep Explanations

## 0. Current Session (read first)

- Active track: **JavaScript** (DONE, verified).
- Python (37), React (30), Node.js (30), HTML (20 + 10 tag docs), CSS (22 + 10 property docs), JavaScript (34) are DONE.
- Source of truth: `JavaScript/roadmap.json` (34 topics).
- Progress: all 34 topics have `.md` + `.mock.json` + `.revision.md`; JSON, sequence, structure, and closing-section checks passed.
- Default output dir: `JavaScript/` directly. No reference subfolder.
- "Python way" = Python-track depth: granular `###` sub-sections, runnable examples, verbose explanations, 8-question mocks, ~2.5KB revisions. Do not use the shorter HTML/CSS style.

This repo holds interview-prep notes as Markdown. Match the existing style exactly.
Reference files: `Python/Decorators.md`, `Python/Python Standard Library.md`, `Python/HTTP and REST APIs.md`.

## 1. Scope / Input

- Active track is JavaScript: `JavaScript/roadmap.json` is source of truth for topics + sub-topics.
- Python, React, Node.js, HTML, and CSS tracks are done — ignore unless the user asks for them.
- `sequence.json` (if present) defines order, including subfolders. Don't reorder without asking.
- One `.md` file per roadmap topic. Cover EVERY sub-topic listed for it — none skipped.
- Only read the active track's `roadmap.json` + one existing `.md` for style unless the user asks for more.

## 2. Filename Convention

- Title Case with spaces, matching existing files.
- HTML examples: `HTML Fundamentals.md`, `Forms Fundamentals.md`, `HTML Interview Traps.md`.
- CSS examples: `CSS Fundamentals.md`, `Box Model.md`, `CSS Interview Traps.md`.
- JavaScript examples: `JavaScript Fundamentals.md`, `Closures and the Scope Chain.md`, `JavaScript Interview Traps.md`.
- Place new topic files directly under the track folder. Reference subfolders use category names, e.g. `HTML/Tags/Form Tags.md`, `CSS/Properties/Layout Properties.md`.

## 3. Document Structure (follow in order)

1. `# <Topic Name>` + 2-4 sentence beginner intro stating the chain: `A → B → C`.
2. Core-idea `text` diagram:
   ````text
   ```text
   Concept
       ↓
   does X
       ↓
   gives Y
   ```
   ````
3. Numbered `# 1.`, `# 2.`, ... sections — one per sub-topic group. Never use `##` for main sections.
4. End with (in order):
   - `# Real-World Uses` (or `# Real-World Example`)
   - `# Quick Revision` table
   - `## The Mental Model` text diagram
   - `### The 3 things to remember` + closing `> **...**` one-line summary.

## 4. Per-Section Pattern

Each numbered section must have:

- Short plain-English explanation (2-5 lines, no jargon dump).
- Small snippet (5-15 lines max). Python track: `python` (stdlib-only unless pip needed). React track: `jsx`/`js`. HTML track: `html` (plain document snippets, `Kakade` examples).
- `Output:` + `text` block showing exact output / render result.
- `---` separator after each major section.
- Where helpful: `Think:` text diagram, `Compare:` block, table, or `Notice:` callout.

Example (React track):

````md
```jsx
function Greet() {
  return <h1>Hello Kakade</h1>;
}
```

Output:

```text
Hello Kakade
```
````

Example:

````md
```python
print("Hello Kakade")
```

Output:

```text
Hello Kakade
```
````

## 5. Style Rules

- Beginner-first, interview-relevant only. No deep API dumps.
- Use `Kakade` (and `Asha`, `Rohan` for extras) in all examples. Keeps continuity.
- Show Bad vs Good explicitly when there is a common mistake:
  ```python
  # Bad
  # Good
  ```
- Call out interview traps: e.g. `PUT vs PATCH`, `match vs search`, `strftime vs strptime`, `401 vs 403`.
- Prefer tables for comparisons and format codes.
- Use `text` diagrams with `↓`, `│`, `├──`, `└──` for flows (request lifecycle, wrapper nesting, etc.).
- Headers/terms in `code` on first use where it aids scanning.
- Keep snippets copy-paste runnable. Prefer `jsonplaceholder.typicode.com` for HTTP examples.
- Install block at top if pip package needed:
  ```bash
  pip install requests
  ```

## 6. Closing Sections (required)

- `Quick Revision`: `| Concept | Remember |` table, one row per sub-topic.
- `Mental Model`: single `text` tree summarizing whole file.
- `3 things to remember`: 3 minimal code snippets + one `> **Bold summary.**`.

## 7. Verification

- Python track: run key snippets with `python` before finishing (use `C:\Users\PRASEN~1\AppData\Local\Temp\opencode\` for temp scripts).
- React track: snippets are `jsx`/`js` for Vite — no execution needed. Just re-read each file once for syntax (balanced tags/backticks, every sub-topic covered).
- HTML track: snippets are plain `html` — no execution needed. Verify valid nesting, balanced code fences, every roadmap sub-topic covered, and all `.mock.json` files parse.
- JavaScript track: snippets are plain `javascript` (Node-compatible) or `html` for DOM/browser examples. Run key Node snippets with `node` before finishing when practical. Verify balanced code fences, every roadmap sub-topic covered, and all `.mock.json` files parse.
- Required Python packages (reference only): `pydantic 2.x` (use `model_dump`/`model_validate`, NOT `.dict()`), `fastapi`, `flask`, `requests`.
- Don't commit, push, or update `sequence.json` unless explicitly asked.

## 8. Batching

- HTML: on `start` / `next 5` / `next five`, write the next 5 unwritten topics in roadmap order — `.md` plus `.mock.json` and `.revision.md`. No need to ask batch size.
- JavaScript: same batching as Python/HTML — 5 topics per batch, each with `.md`, `.mock.json`, `.revision.md`.
- Write one file per `write` call. Update todos as you go.
