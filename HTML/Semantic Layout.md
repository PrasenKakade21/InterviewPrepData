# Semantic Layout

Semantic layout is the difference between a pile of `div`s and a document that machines can understand. `header`, `nav`, `main`, `section`, `article`, `aside`, and `footer` describe *roles*, and those roles become landmarks that screen readers jump between.

The core idea:

```text
page
    ├── header   → intro / branding
    ├── nav      → navigation
    ├── main     → unique page content
    │     ├── section
    │     └── article
    ├── aside    → related content
    └── footer   → closing info
```

Semantic elements are not styling shortcuts — they carry meaning that `div` cannot.

---

# 1. Why Semantics Matter and the Landmarks They Create

A `div` says nothing. A `<nav>` says "navigation landmark", which lets a screen-reader user press a key and jump straight to it. Semantics also help SEO, readability, and long-term maintenance.

```html
<div class="top">
  <div class="menu">...</div>
</div>

<header>
  <nav aria-label="Primary">...</nav>
</header>
```

Output:

```text
div version: no landmarks, no shortcuts
header/nav version: "banner" and "navigation" landmarks
```

Think:

```text
div     → generic box
header  → banner landmark
nav     → navigation landmark
main    → main landmark (one per page)
aside   → complementary landmark
footer  → contentinfo landmark
```

Interview trap: semantics do not replace CSS. `<header>` is not automatically styled — you still style it.

---

# 2. header and nav

`header` introduces its nearest sectioning content or the page. `nav` groups major navigation links. Both can appear multiple times, but only the page-level ones become top-level landmarks.

```html
<header>
  <a href="/">Kakade.dev</a>
  <nav aria-label="Primary">
    <ul>
      <li><a href="/notes">Notes</a></li>
      <li><a href="/projects">Projects</a></li>
    </ul>
  </nav>
</header>

<article>
  <header>
    <h1>Why Semantic HTML Matters</h1>
    <p>By Kakade</p>
  </header>
</article>
```

Output:

```text
Page landmarks: banner (header), navigation (Primary)
Article header: intro for that article, not a page landmark
```

Notice: a `header` inside an `article` is the article's intro, not the site header. Nested semantics change the landmark scope.

---

# 3. The main Element

`main` holds the unique content of the page — the part that would not repeat across routes. There must be only one visible `main` per page, and it should not be inside `header`, `nav`, `aside`, or `footer`.

```html
<body>
  <header>...</header>
  <nav>...</nav>

  <main id="main">
    <h1>Kakade's HTML Notes</h1>
    <p>Everything unique to this page.</p>
  </main>

  <footer>...</footer>
</body>
```

Output:

```text
Screen reader "main" shortcut jumps directly to the notes.
Skip link target is #main.
```

Bad:

```html
<main>
  <div>...</div>
</main>
<main>
  <div>...</div>
</main>
```

Good:

```html
<main>
  <h1>Kakade's HTML Notes</h1>
</main>
```

Interview trap: two visible `main` elements break the main landmark. Multi-page apps must swap the content, not stack `main`s.

---

# 4. section and article

`article` is a self-contained composition that could be syndicated — a blog post, a comment, a product card. `section` is a thematic grouping inside a page, usually with a heading.

```html
<main>
  <section aria-labelledby="html-heading">
    <h2 id="html-heading">HTML</h2>
    <p>Markup fundamentals.</p>
  </section>

  <section aria-labelledby="articles-heading">
    <h2 id="articles-heading">Latest articles</h2>

    <article>
      <h3>Why Semantic HTML Matters</h3>
      <p>By Kakade</p>
    </article>
  </section>
</main>
```

Output:

```text
main
 ├── section "HTML"
 └── section "Latest articles"
       └── article "Why Semantic HTML Matters"
```

Rule of thumb: if the content makes sense on its own in an RSS feed, it is an `article`; if it is a chapter of the page, it is a `section`. A `section` without a heading is usually just a `div`.

---

# 5. aside, footer and address

`aside` holds tangential content — sidebars, pull quotes, related links. `footer` closes a page or section with metadata, links, or copyright. `address` supplies contact information for the nearest article or page.

```html
<main>
  <article>
    <h1>Kakade's Setup</h1>
    <p>Two monitors, one keyboard.</p>

    <aside>
      <h2>Related</h2>
      <a href="/notes">HTML notes</a>
    </aside>

    <footer>
      <p>Published <time datetime="2026-10-28">28 Oct 2026</time></p>
      <address>
        Contact <a href="mailto:kakade@example.com">Kakade</a>
      </address>
    </footer>
  </article>
</main>
```

Output:

```text
Article content
Related links (complementary landmark)
Published 28 Oct 2026
Contact Kakade
```

Notice: `footer` inside `article` is the article's footer. The page-level `footer` sits outside `main` and becomes the `contentinfo` landmark.

---

# 6. hgroup, search, div vs Semantic Elements and Landmarks

`hgroup` groups a heading with its subtitle. `search` marks a search landmark (newer HTML). `div` remains correct when there is no semantic meaning — it is a generic container, not a failure.

```html
<hgroup>
  <h1>Kakade's HTML Notes</h1>
  <p>From doctype to semantic layout</p>
</hgroup>

<search>
  <form action="/search" method="get">
    <label for="q">Search notes</label>
    <input id="q" name="q" type="search" />
  </form>
</search>

<div class="card-grid">
  <article>...</article>
</div>
```

Output:

```text
h1 + subtitle grouped
Search landmark around the search form
div used purely as a layout wrapper
```

Landmark map:

```text
header   → banner
nav      → navigation
main     → main
aside    → complementary
footer   → contentinfo
search   → search
form     → form (when named)
section  → region (only when named by a heading)
```

Interview trap: not every `section` becomes a landmark — an unnamed `section` is not exposed as a region. Use `aria-labelledby` or a visible heading.

---

# Real-World Uses

A blog layout assigns every major region a semantic role, with skip link, one main, article cards, and a page footer.

```html
<body>
  <a href="#main" class="skip-link">Skip to content</a>

  <header>
    <h1>Kakade.dev</h1>
    <nav aria-label="Primary">
      <ul>
        <li><a href="/notes">Notes</a></li>
        <li><a href="/about">About</a></li>
      </ul>
    </nav>
  </header>

  <main id="main">
    <h2>Latest notes</h2>
    <section aria-labelledby="posts">
      <h3 id="posts">Posts</h3>

      <article>
        <h4>Semantic Layout</h4>
        <p>Why landmarks matter.</p>
      </article>
    </section>
  </main>

  <aside aria-label="Related links">
    <a href="/html/forms">Forms guide</a>
  </aside>

  <footer>
    <p>© 2026 Kakade</p>
  </footer>
</body>
```

Output:

```text
Landmarks: banner, navigation, main, complementary, contentinfo
Skip link jumps to main.
Articles are self-contained and syndicatable.
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **Why semantics** | Meaning, landmarks, SEO, maintainability |
| **header** | Intro for page or section |
| **nav** | Major navigation links |
| **main** | Unique page content; exactly one visible |
| **section** | Thematic group, usually with a heading |
| **article** | Self-contained, syndicatable content |
| **aside** | Tangential/related content |
| **footer** | Closing metadata for page or section |
| **address** | Contact info for nearest article/page |
| **hgroup and search** | Heading + subtitle; search landmark |
| **div vs semantic** | `div` only when no meaning fits |
| **Landmarks** | banner, navigation, main, complementary, contentinfo |

## The Mental Model

```text
page
  ├── header → banner
  ├── nav → navigation
  ├── main → main (one)
  │     ├── section → region (named)
  │     │     └── article → self-contained
  │     └── aside → complementary
  └── footer → contentinfo
        │
        └── div = only when nothing semantic fits
```

### The 3 things to remember

**1. One main, many sections:**

```html
<main>
  <section aria-labelledby="html"><h2 id="html">HTML</h2></section>
</main>
```

**2. Article = portable:**

```html
<article>
  <h2>Why Semantic HTML Matters</h2>
  <p>By Kakade</p>
</article>
```

**3. div is a fallback, not a default:**

```html
<div class="card-grid">
  <article>...</article>
</div>
```

> **Semantic layout turns a page into a map — landmarks tell assistive tech and search engines where each region begins and ends.**
