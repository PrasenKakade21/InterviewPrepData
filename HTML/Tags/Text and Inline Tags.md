# Text and Inline Tags

This is the largest tag family: headings, paragraphs, quotes, emphasis, code, dates, and inline containers. The rule for all of them is the same — choose by meaning, style with CSS.

The core idea:

```text
text content
    ├── structure → h1-h6, p, pre, blockquote
    ├── meaning   → strong, em, cite, abbr, dfn, time
    ├── technical → code, kbd, samp, var
    └── inline    → span, b, i, u, s, mark, small, sub, sup
```

---

# Headings: `<h1>` to `<h6>`

Six levels of section headings. `h1` is the page topic; deeper levels are subsections. They form the document outline used by screen readers and search engines.

```html
<h1>Kakade's Notes</h1>
<h2>HTML</h2>
<h3>Text</h3>
```

Output:

```text
H1 Kakade's Notes
  H2 HTML
    H3 Text
```

Gotchas: don't skip levels for size; one `h1` per page is the safe convention; style with CSS, never choose a level for appearance.

---

# `<p>`

A paragraph of prose. Browsers add margin above and below.

```html
<p>Kakade writes HTML notes every morning.</p>
<p>CSS comes after structure.</p>
```

Output:

```text
Kakade writes HTML notes every morning.

CSS comes after structure.
```

Gotchas: `p` cannot contain block elements (`div`, `p`, `section`); the parser closes it automatically.

---

# `<br>`

A forced line break inside text. Void element.

```html
<p>Kakade.dev<br />221B Interview Street</p>
```

Output:

```text
Kakade.dev
221B Interview Street
```

Gotchas: not for spacing between paragraphs; use CSS margins.

---

# `<hr>`

A thematic break between sections, rendered as a horizontal rule. Void element.

```html
<p>Before the break.</p>
<hr />
<p>After the break.</p>
```

Output:

```text
Before the break.
──────────────
After the break.
```

Gotchas: it means "topic change", not "draw a line"; style it with CSS if needed.

---

# `<wbr>`

An optional line-break opportunity inside a long word. Void element.

```html
<p>Supercalifragilistic<wbr />expialidocious</p>
```

Output:

```text
Wraps at the marked point only when space is tight.
```

Gotchas: it does not force a break; use `<br>` for that.

---

# `<pre>`

Preformatted text: preserves spaces and line breaks. Usually paired with `<code>`.

```html
<pre><code>function greet() {
  return "Hello Kakade";
}</code></pre>
```

Output:

```text
function greet() {
  return "Hello Kakade";
}
```

Gotchas: content inside `pre` is not wrapped by default and can overflow; combine with `overflow-x: auto`.

---

# `<blockquote>`

A block-level quotation from another source, optionally with a `cite` URL.

```html
<blockquote cite="https://kakade.dev/quotes">
  <p>Ship small, ship often.</p>
</blockquote>
```

Output:

```text
    Ship small, ship often.
```

Gotchas: `cite` is machine-readable and not displayed; browsers indent by default but style it yourself.

---

# `<q>`

A short inline quotation. Browsers add quotation marks automatically.

```html
<p>Kakade said <q>ship small</q> and meant it.</p>
```

Output:

```text
Kakade said "ship small" and meant it.
```

Gotchas: don't add your own quotes around `q`; nested quotes are handled by the browser.

---

# `<cite>`

Names a creative work: book, article, film, standard.

```html
<p>Defined in <cite>HTML Living Standard</cite>.</p>
```

Output:

```text
Defined in HTML Living Standard.   (italic by default)
```

Gotchas: `cite` is for the title of a work, not for the author or a quotation source label.

---

# `<abbr>`

Marks an abbreviation or acronym; `title` holds the expansion.

```html
<p><abbr title="HyperText Markup Language">HTML</abbr> structures the web.</p>
```

Output:

```text
HTML structures the web.   (tooltip: HyperText Markup Language)
```

Gotchas: tooltips are not available on touch; spell out the term on first use when it matters.

---

# `<address>`

Contact information for the nearest `article` or page.

```html
<address>
  Contact <a href="mailto:kakade@example.com">Kakade</a><br />
  Pune, India
</address>
```

Output:

```text
Contact Kakade
Pune, India
```

Gotchas: it is for contact details, not any postal address or a generic footer.

---

# `<b>` and `<strong>`

`b` applies visual bold with no extra meaning; `strong` marks importance.

```html
<p><strong>Warning:</strong> deploy today.</p>
<p>The <b>Kakade</b> brand.</p>
```

Output:

```text
Warning: deploy today.   (importance conveyed)
The Kakade brand.        (style only)
```

Gotchas: use `strong` when the emphasis matters to meaning; `b` for stylistic offset such as brand names.

---

# `<i>` and `<em>`

`i` applies italic styling; `em` marks stress emphasis.

```html
<p>Deploy is <em>today</em>.</p>
<p>The <i>Kakade.dev</i> title.</p>
```

Output:

```text
Deploy is today.        (stress emphasis)
The Kakade.dev title.   (alternate voice/style)
```

Gotchas: don't use `em` just to italicize; screen readers may change intonation.

---

# `<u>` and `<s>`

`u` marks unarticulated text (a proper name in Chinese, a misspelling); `s` marks content that is no longer accurate.

```html
<p lang="zh">Kakade <u>先生</u></p>
<p><s>₹599</s> ₹499</p>
```

Output:

```text
Kakade 先生
₹599 (struck through) → ₹499
```

Gotchas: `u` is not for underlining links; `s` is not for deleted text in edits — use `del`/`ins` there.

---

# `<small>`

Fine print, side comments, legal text.

```html
<p><small>Offer ends Friday. Terms apply.</small></p>
```

Output:

```text
Offer ends Friday. Terms apply.   (smaller)
```

Gotchas: not for de-emphasizing important content; it means "side comment".

---

# `<mark>`

Highlights text for reference/relevance, like a marker pen.

```html
<p>Search: <mark>Kakade</mark> appears 3 times.</p>
```

Output:

```text
Search: Kakade appears 3 times.   (highlighted)
```

Gotchas: `mark` is relevance, not emphasis — use `strong`/`em` for that.

---

# `<sub>` and `<sup>`

Subscript and superscript text.

```html
<p>H<sub>2</sub>O and x<sup>2</sup></p>
```

Output:

```text
H₂O and x²
```

Gotchas: for dates like 1<sup>st</sup>, consider `<sup>` but keep the text readable; not for styling.

---

# `<code>`, `<kbd>`, `<samp>`, `<var>`

Technical inline elements: code, keyboard input, program output, variable.

```html
<p>Run <code>npm run dev</code>, press <kbd>Ctrl</kbd> + <kbd>C</kbd>.</p>
<p>Output: <samp>Server running on 3000</samp></p>
<p>Total is <var>price</var> × <var>qty</var>.</p>
```

Output:

```text
Run npm run dev, press Ctrl + C.
Output: Server running on 3000
Total is price × qty.
```

Gotchas: `code` inside `pre` is the standard block; don't use `kbd` for any styled keycap text.

---

# `<time>`

A machine-readable date/time with human-readable text.

```html
<p>Published <time datetime="2026-10-28">28 Oct 2026</time>.</p>
```

Output:

```text
Published 28 Oct 2026.   (machine value: 2026-10-28)
```

Gotchas: `datetime` must be a valid format (`YYYY-MM-DD`, `HH:MM`, ISO 8601); without it the text content is parsed.

---

# `<data>`

Pairs visible text with a machine-readable value.

```html
<p>Product: <data value="SKU-1042">Kakade Notebook</data></p>
```

Output:

```text
Product: Kakade Notebook   (machine value: SKU-1042)
```

Gotchas: use `data` for generic machine values; `time` is specifically for dates/times.

---

# `<dfn>`

Marks the defining instance of a term.

```html
<p><dfn>DOM</dfn> is the browser's object model for a page.</p>
```

Output:

```text
DOM is the browser's object model for a page.   (italic by default)
```

Gotchas: only the first/defining occurrence should be `dfn`; the definition itself can follow in the same paragraph.

---

# `<bdi>` and `<bdo>`

`bdi` isolates bidirectional text so it doesn't affect surrounding direction; `bdo` overrides direction explicitly.

```html
<p><bdi>مرحبا</bdi> Kakade</p>
<p><bdo dir="rtl">Kakade</bdo></p>
```

Output:

```text
مرحبا Kakade   (isolated direction)
Kakade         (rendered right-to-left)
```

Gotchas: use `bdi` for user-generated names mixed with RTL; `bdo` is rare and overrides layout direction.

---

# `<ruby>`, `<rt>`, `<rp>`

Ruby annotations for pronunciation (Japanese, Chinese). `rt` is the annotation, `rp` is fallback parentheses.

```html
<ruby>漢<rp>(</rp><rt>kan</rt><rp>)</rp></ruby>
```

Output:

```text
漢 with "kan" above it (or 漢(kan) where ruby is unsupported)
```

Gotchas: `rp` only shows in non-ruby browsers; keep annotations short.

---

# `<span>`

A generic inline container with no meaning, used for styling or scripting a text fragment.

```html
<p>Price: <span class="price">₹499</span></p>
```

Output:

```text
Price: ₹499   (price can be styled inline)
```

Gotchas: use a semantic inline element when one exists; `span` is the fallback.

---

# Quick Reference

| Tag | Meaning |
| --- | ------- |
| `h1`–`h6` | Heading outline |
| `p` | Paragraph |
| `br` / `wbr` | Forced / optional break |
| `hr` | Thematic break |
| `pre` | Preformatted text |
| `blockquote` / `q` | Block / inline quote |
| `cite` | Work title |
| `abbr` | Abbreviation |
| `address` | Contact info |
| `strong` / `b` | Importance / bold |
| `em` / `i` | Emphasis / italic |
| `u` / `s` | Unarticulated / outdated |
| `small` / `mark` | Fine print / highlight |
| `sub` / `sup` | Sub/superscript |
| `code` / `kbd` / `samp` / `var` | Code / input / output / variable |
| `time` / `data` | Date/time / machine value |
| `dfn` | Defining instance |
| `bdi` / `bdo` | Direction isolation / override |
| `ruby` / `rt` / `rp` | Pronunciation annotations |
| `span` | Generic inline |

> **Pick text elements by meaning — headings outline, strong/em carry importance, and code/kbd/time make content machine-readable.**
