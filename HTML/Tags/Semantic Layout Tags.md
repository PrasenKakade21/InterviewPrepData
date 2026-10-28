# Semantic Layout Tags

These tags describe page regions. They are the difference between a document and a pile of divs, and they map directly to accessibility landmarks.

The core idea:

```text
page
  ├── header → banner
  ├── nav → navigation
  ├── main → main
  ├── section → named region
  ├── article → standalone content
  ├── aside → complementary
  └── footer → contentinfo
```

---

# `<header>`

Introductory content for the page or nearest sectioning element: logo, title, nav, search.

```html
<header>
  <h1>Kakade.dev</h1>
  <nav aria-label="Primary"><a href="/notes">Notes</a></nav>
</header>
```

Output:

```text
Banner landmark at page level; intro content inside an article otherwise.
```

Gotchas: a `header` inside an `article` is not the page banner; multiple headers are valid but scope matters.

---

# `<nav>`

A major navigation landmark containing links.

```html
<nav aria-label="Primary">
  <ul>
    <li><a href="/notes">Notes</a></li>
    <li><a href="/projects">Projects</a></li>
  </ul>
</nav>
```

Output:

```text
Navigation landmark named "Primary"; list of links.
```

Gotchas: not every link group needs `nav`; name multiple navs with `aria-label`; it is a landmark, not a menu widget.

---

# `<main>`

The unique content of the page, excluding repeated header/nav/footer.

```html
<main id="main">
  <h1>Kakade's HTML Notes</h1>
  <p>Everything unique to this page.</p>
</main>
```

Output:

```text
Main landmark; skip link target #main.
```

Gotchas: one visible `main` per page; do not nest it in header/nav/aside/footer.

---

# `<section>`

A thematic grouping, usually with a heading.

```html
<section aria-labelledby="forms-heading">
  <h2 id="forms-heading">Forms</h2>
  <p>Labels, validation, and submission.</p>
</section>
```

Output:

```text
Region landmark named "Forms" (because it has an accessible name).
```

Gotchas: an unnamed section is not a landmark; use `div` when there is no theme.

---

# `<article>`

Self-contained content that could be syndicated: post, comment, product, card.

```html
<article>
  <header>
    <h2>Why Semantic HTML Matters</h2>
    <p>By Kakade</p>
  </header>
  <p>Structure is meaning.</p>
</article>
```

Output:

```text
Article landmark; header belongs to the article.
```

Gotchas: nested articles are valid (comments inside a post); don't use it for arbitrary page sections.

---

# `<aside>`

Tangential content: sidebars, related links, pull quotes, ads.

```html
<aside aria-label="Related links">
  <h2>Related</h2>
  <a href="/html/forms">Forms guide</a>
</aside>
```

Output:

```text
Complementary landmark named "Related links".
```

Gotchas: if the content is essential to understanding the main content, it is not an aside.

---

# `<footer>`

Closing content for the page or nearest section: copyright, author, related links.

```html
<footer>
  <p>© 2026 Kakade</p>
  <address><a href="mailto:kakade@example.com">Contact</a></address>
</footer>
```

Output:

```text
Contentinfo landmark at page level; section footer otherwise.
```

Gotchas: page footer goes outside `main`; article footer stays inside the article.

---

# `<address>`

Contact information for the nearest article or page.

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

Gotchas: not for arbitrary postal addresses; it is specifically contact details.

---

# `<hgroup>`

Groups a heading with its subtitle or tagline.

```html
<hgroup>
  <h1>Kakade's HTML Notes</h1>
  <p>From doctype to semantic layout</p>
</hgroup>
```

Output:

```text
H1 with a subordinate subtitle treated as one heading group.
```

Gotchas: only headings and `p` belong in `hgroup`; don't use it to stack unrelated headings.

---

# `<search>`

A landmark for search functionality (newer HTML).

```html
<search>
  <form action="/search" method="get">
    <label for="q">Search notes</label>
    <input id="q" name="q" type="search" />
  </form>
</search>
```

Output:

```text
Search landmark around the search form.
```

Gotchas: support is still rolling out; a named `form` also exposes a form landmark as a fallback.

---

# `<div>`

A generic block container with no semantics.

```html
<div class="card-grid">
  <article class="card">...</article>
</div>
```

Output:

```text
Pure layout wrapper; no landmark or role.
```

Gotchas: correct for layout, wrong for content roles; div soup harms accessibility and CSS clarity.

---

# Quick Reference

| Tag | Landmark |
| --- | -------- |
| `header` | banner (page level) |
| `nav` | navigation |
| `main` | main |
| `section` | region (only when named) |
| `article` | article |
| `aside` | complementary |
| `footer` | contentinfo (page level) |
| `address` | — (contact info) |
| `hgroup` | — (heading + subtitle) |
| `search` | search |
| `div` | — (generic) |

> **Semantic layout turns a page into a map — landmarks tell assistive tech and search engines where each region begins and ends.**
