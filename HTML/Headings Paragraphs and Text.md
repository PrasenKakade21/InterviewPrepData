# Headings, Paragraphs and Text

Text is 90% of the web, and HTML gives every kind of text a home: `h1–h6` for structure, `p` for prose, `strong`/`em` for meaning, `code` for snippets. Picking the element that matches the *meaning* is what makes a page readable for browsers, screen readers, and search engines.

The core idea:

```text
text content
    ↓
pick the semantic element
    ↓
structure + meaning (not just looks)
```

A heading is not "big bold text" — it is an outline entry. An `em` is not "italic" — it is emphasis.

---

# 1. Headings h1 to h6 and Heading Order

`h1`–`h6` define six levels of section headings, with `h1` as the page's main topic. Heading order should be logical: don't jump from `h1` to `h4` just because `h4` looks smaller.

```html
<h1>Kakade's HTML Notes</h1>
<h2>Text Elements</h2>
<h3>Headings</h3>
<p>Headings create the document outline.</p>
```

Output:

```text
Kakade's HTML Notes        ← largest
  Text Elements            ← section
    Headings               ← subsection
Headings create the document outline.
```

Think:

```text
h1 page title
 ├── h2 major section
 │    ├── h3 subsection
 │    └── h3 subsection
 └── h2 major section
```

Interview trap: multiple `h1`s are technically valid in HTML5, but one clear `h1` per page is the safer, more accessible convention. Never choose heading level for size — use CSS.

---

# 2. Paragraphs, br, wbr and hr

`p` wraps a paragraph of text. `br` forces a line break inside text, and `wbr` marks an optional break point in a long word. `hr` represents a thematic break between sections.

```html
<p>Kakade writes HTML notes every morning.</p>
<p>
  Kakade.dev<br />
  221B Interview Street
</p>
<p>Supercalifragilistic<wbr />expialidocious</p>
<hr />
<p>Next section starts here.</p>
```

Output:

```text
Kakade writes HTML notes every morning.
Kakade.dev
221B Interview Street
Supercalifragilistic|expialidocious   ← may wrap at |
────────────────────────────────────  ← hr line
Next section starts here.
```

Bad:

```html
<!-- Don't use empty paragraphs for spacing -->
<p></p><p></p>
```

Good:

```html
<section>
  <p>Kakade's address</p>
</section>
```

Use CSS margins for spacing — `br` is for meaningful line breaks (addresses, poems), not layout.

---

# 3. pre, blockquote and q

`pre` preserves whitespace and line breaks, which makes it right for code or ASCII art. `blockquote` marks a long quotation from another source; `q` marks a short inline quote and browsers add quotation marks automatically.

```html
<pre>
function greet() {
  return "Hello Kakade";
}
</pre>

<blockquote cite="https://kakade.dev/quotes">
  <p>Ship small, ship often.</p>
</blockquote>

<p>Kakade said <q>ship small</q> and meant it.</p>
```

Output:

```text
function greet() {
  return "Hello Kakade";
}

    Ship small, ship often.

Kakade said "ship small" and meant it.
```

Notice: `cite` on `blockquote` is a URL for machines, not visible text. To show the source name, add a `<cite>` element or a `<footer>`.

---

# 4. cite, abbr, strong vs b, em vs i

`cite` names a creative work, `abbr` expands an abbreviation via `title`, `strong` marks importance, and `em` marks stress emphasis. `b` and `i` are the visual fallbacks: bold/italic without extra meaning.

```html
<p>
  <abbr title="HyperText Markup Language">HTML</abbr> was defined in
  <cite>HTML Living Standard</cite>.
</p>

<p><strong>Warning:</strong> the deploy is <em>today</em>.</p>
<p>The <b>Kakade</b> brand uses bold styling only.</p>
<p>The <i>deploy</i> word is italic for tone, not emphasis.</p>
```

Output:

```text
HTML was defined in HTML Living Standard.   ← HTML shows tooltip on hover
Warning: the deploy is today.               ← both bold, "today" stressed
The Kakade brand uses bold styling only.
The deploy word is italic for tone, not emphasis.
```

| Element | Meaning | Screen reader effect |
| ------- | ------- | -------------------- |
| `strong` | Importance | Often stressed |
| `b` | Stylistically offset | Neutral |
| `em` | Stress emphasis | Often stressed |
| `i` | Alternate voice/term | Neutral |

Interview trap: `strong` and `em` are not interchangeable with `b` and `i`. Use semantic tags when meaning matters, visual tags for pure styling.

---

# 5. mark, small, sub, sup, code, kbd, samp and var

These inline elements label very specific kinds of text: highlighted relevance (`mark`), fine print (`small`), chemistry and math (`sub`, `sup`), code (`code`), keyboard input (`kbd`), program output (`samp`), and variables (`var`).

```html
<p>Search result: <mark>Kakade</mark> appears 3 times.</p>
<p><small>Terms apply. Offer ends Friday.</small></p>
<p>H<sub>2</sub>O and x<sup>2</sup></p>
<p>Run <code>npm run dev</code>, then press <kbd>Ctrl</kbd> + <kbd>C</kbd>.</p>
<p>Output: <samp>Server running on port 3000</samp></p>
<p>The value <var>total</var> is the sum of prices.</p>
```

Output:

```text
Search result: Kakade appears 3 times.   ← Kakade highlighted
Terms apply. Offer ends Friday.          ← smaller text
H₂O and x²
Run npm run dev, then press Ctrl + C.
Output: Server running on port 3000
The value total is the sum of prices.
```

These tags carry meaning, not just styling — `code` inside `pre` is the standard code block pattern.

---

# 6. time, data, address and dfn

`time` marks dates and times in a machine-readable `datetime` value. `data` links visible text to a machine value. `address` holds contact details for the nearest article or page. `dfn` marks the defining instance of a term.

```html
<p>Published <time datetime="2026-10-28">28 October 2026</time>.</p>
<p>Product: <data value="SKU-1042">Kakade Notebook</data></p>
<address>
  Contact <a href="mailto:kakade@example.com">Kakade</a><br />
  Pune, India
</address>
<p><dfn>DOM</dfn> is the browser's object model for a page.</p>
```

Output:

```text
Published 28 October 2026.       ← datetime="2026-10-28"
Product: Kakade Notebook          ← value="SKU-1042"
Contact Kakade
Pune, India
DOM is the browser's object model for a page.
```

Notice: `address` is for contact information, not postal addresses in general. It is not a wrapper for any address-like text.

---

# Real-World Uses

A blog article combines headings, paragraphs, a quote, inline semantics, code, and a timestamp into a readable outline.

```html
<article>
  <h1>Why Semantic HTML Matters</h1>
  <p>
    Published <time datetime="2026-10-28">28 Oct 2026</time> by
    <strong>Kakade</strong>.
  </p>

  <p>
    <dfn>Semantic HTML</dfn> chooses elements by <em>meaning</em>, not looks.
  </p>

  <blockquote>
    <p>Structure is the interface between content and code.</p>
  </blockquote>

  <p>Use <code>&lt;strong&gt;</code> for importance, not for bold text.</p>
</article>
```

Output:

```text
Why Semantic HTML Matters
Published 28 Oct 2026 by Kakade.

Semantic HTML chooses elements by meaning, not looks.

    Structure is the interface between content and code.

Use <strong> for importance, not for bold text.
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **h1 to h6** | Six heading levels forming the outline |
| **Heading order** | Never skip levels for visual size |
| **p** | Paragraph of prose |
| **br and wbr** | Forced break; optional break opportunity |
| **hr** | Thematic break between sections |
| **pre** | Preserves whitespace and line breaks |
| **blockquote and q** | Block quote; inline quote with auto quotes |
| **cite and abbr** | Work title; abbreviation with `title` |
| **strong vs b** | Importance vs visual bold |
| **em vs i** | Stress emphasis vs alternate voice |
| **mark, small, sub, sup** | Highlight, fine print, sub/superscript |
| **code, kbd, samp, var** | Code, input, output, variable |
| **time and data** | Machine-readable date; text with machine value |
| **address and dfn** | Contact info; defining instance of a term |

## The Mental Model

```text
document outline
  │
  ├── h1 → main topic
  │     ├── h2 → section
  │     │     └── h3 → subsection
  │     │
  │     └── p → prose
  │           ├── strong / em → meaning
  │           ├── b / i → style only
  │           ├── code / kbd / samp / var → technical text
  │           └── mark / small / sub / sup → special treatment
  │
  └── blockquote / pre / hr → structural breaks
```

### The 3 things to remember

**1. Headings are an outline:**

```html
<h1>Kakade Notes</h1>
<h2>HTML</h2>
<h3>Text</h3>
```

**2. Meaning beats looks:**

```html
<p><strong>Important</strong> and <em>stressed</em> text.</p>
```

**3. Technical text has tags:**

```html
<p>Run <code>npm run dev</code> and press <kbd>Ctrl</kbd> + <kbd>C</kbd>.</p>
```

> **Choose text elements by meaning: headings build the outline, strong/em carry importance, and code/kbd/time make content machine-readable.**
