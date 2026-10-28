# Headings, Paragraphs and Text — Revision Notes (Beginner)
> One-line mental model: pick the text element by meaning — headings outline, strong/em carry importance, code/kbd/time make text machine-readable.

## Core recall
- `h1`–`h6` build the document outline; do not skip levels and never choose a level for its size.
- `p` is a paragraph; `br` is a meaningful line break, `wbr` an optional break point, `hr` a thematic break.
- `pre` preserves whitespace (code, ASCII art); `blockquote` = long quote, `q` = short inline quote with auto quotation marks.
- `cite` names a work, `abbr title` expands an abbreviation, `dfn` marks the defining instance of a term.
- `strong` = importance, `em` = stress emphasis; `b`/`i` are styling-only fallbacks.
- `mark` highlights relevance, `small` is fine print, `sub`/`sup` handle chemistry and math.
- `code` inline code, `kbd` keyboard input, `samp` program output, `var` a variable.
- `time datetime="YYYY-MM-DD"` is machine-readable; `data value` pairs text with a machine value.
- `address` is contact info for the nearest article/page, not any postal address.
- Spacing comes from CSS, never from `<br><br>` or empty `<p>`.

## Must-remember code
```html
<h1>Kakade's HTML Notes</h1>
<h2>Text Elements</h2>
<h3>Headings</h3>
<p>Headings create the document outline.</p>
```
```html
<p>
  Published <time datetime="2026-10-28">28 Oct 2026</time> by
  <strong>Kakade</strong>.
</p>
<p><dfn>Semantic HTML</dfn> chooses elements by <em>meaning</em>.</p>
```
```html
<p>Run <code>npm run dev</code>, then press <kbd>Ctrl</kbd> + <kbd>C</kbd>.</p>
<pre><code>npm run dev</code></pre>
```
```html
<blockquote cite="https://kakade.dev/quotes">
  <p>Ship small, ship often.</p>
</blockquote>
```

## Interview traps
- Skipping heading levels (`h1` → `h4`) breaks screen-reader and SEO outlines.
- Using `b`/`i` when meaning matters instead of `strong`/`em`.
- Spacing with `<br><br>` or empty `<p>` instead of CSS.
- Treating `blockquote` as an indentation tool — it means a quotation.
- Using `address` for any postal address rather than contact info.
- Writing dates as plain text instead of `<time datetime>`.

## 3 things to remember
1. Headings are structure, not font sizes.
2. strong/em for meaning, b/i for style.
3. Technical text gets code/kbd/samp/var/time/data.
