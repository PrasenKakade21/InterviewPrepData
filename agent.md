# Agent Guide — How to Write Interview Prep Explanations

## 0. Current Session (read first)

- Active track: **Node.js**.
- Python track is DONE. React track is DONE (30 topics + sequence.json).
- Source of truth: `NodeJS/roadmap.json`.
- Progress: roadmap created (30 topics). Content writing NOT started. Start from `01. Node Fundamentals`.
- When user says `start` (or `next 5`, `next five`): write the next 5 unwritten Node.js topics, one `.md` per topic, then verify and summarize. Do not ask for batch size.
- Default output dir: `NodeJS/` directly (no subfolders for new files).

This repo holds interview-prep notes as Markdown. Match the existing style exactly.
Reference files: `Python/Decorators.md`, `Python/Python Standard Library.md`, `Python/HTTP and REST APIs.md`.

## 1. Scope / Input

- Active track is React: `React/roadmap.json` is source of truth for topics + sub-topics.
- Python track (`Python/roadmap.json`) is done — ignore unless user asks for Python.
- `sequence.json` (if present) defines order. Don't reorder without asking.
- One `.md` file per roadmap topic. Cover EVERY sub-topic listed for it — none skipped.
- Only read `React/roadmap.json` + one `.md` for style unless user asks for more.

## 2. Filename Convention

- Title Case with spaces, matching Python files.
- React examples: `React Fundamentals.md`, `JSX.md`, `State and useState.md`, `React Router.md`.
- Place new React files directly under `React/` (ignore old `React/React-Fundamentals/` kebab-case layout for new work).

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
- Small snippet (5-15 lines max). Python track: `python` (stdlib-only unless pip needed). React track: `jsx`/`js` (copy-paste into Vite/CRA, `Kakade` examples).
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
- Required Python packages (reference only): `pydantic 2.x` (use `model_dump`/`model_validate`, NOT `.dict()`), `fastapi`, `flask`, `requests`.
- Don't commit, push, or update `sequence.json` unless explicitly asked.

## 8. Batching

- React: on `start` / `next 5` / `next five`, write the next 5 unwritten topics in roadmap order. No need to ask batch size.
- Write one file per `write` call. Update todos as you go.
