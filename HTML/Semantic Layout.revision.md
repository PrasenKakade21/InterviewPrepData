# Semantic Layout — Revision Notes (Beginner)
> One-line mental model: semantic elements are named regions — they tell assistive tech and search engines what each part of the page is.

## Core recall
- `div` is generic; `header`, `nav`, `main`, `section`, `article`, `aside`, `footer` carry meaning.
- Landmarks: `header` → banner, `nav` → navigation, `main` → main, `aside` → complementary, `footer` → contentinfo, `search` → search.
- One visible `main` per page; it holds unique content and is not nested in header/nav/aside/footer.
- `article` = self-contained and syndicatable; `section` = thematic part of the page, usually with a heading.
- A `section` becomes a region only when it has an accessible name (`aria-labelledby` or heading).
- `header`/`footer` inside an `article` belong to that article, not the page.
- `aside` is tangential content (related links, pull quotes), exposed as complementary.
- `address` is contact info for the nearest article/page, not any postal address.
- `hgroup` groups a heading with its subtitle; `search` wraps search UI.
- Semantics do not style anything — CSS is still required.

## Must-remember code
```html
<body>
  <a href="#main" class="skip-link">Skip to content</a>

  <header>
    <h1>Kakade.dev</h1>
    <nav aria-label="Primary">
      <ul>
        <li><a href="/notes">Notes</a></li>
      </ul>
    </nav>
  </header>

  <main id="main">
    <h2>Latest notes</h2>
  </main>

  <footer>
    <p>© 2026 Kakade</p>
  </footer>
</body>
```
```html
<section aria-labelledby="html-heading">
  <h2 id="html-heading">HTML</h2>
  <p>Markup fundamentals.</p>
</section>
```
```html
<article>
  <header>
    <h1>Why Semantic HTML Matters</h1>
    <p>By Kakade</p>
  </header>
  <p>Structure is meaning.</p>
</article>
```

## Interview traps
- Multiple visible `main` elements.
- `section` used for pure layout with no heading.
- Treating `header`/`footer` as page-only when they can belong to articles.
- Assuming semantic tags come with styling.
- Using `aside` for primary content.
- `address` misused for any address-like text.
- Forgetting that unnamed sections are not landmarks.

## 3 things to remember
1. One `main`; everything else is a named region.
2. `article` is portable, `section` is a page chapter.
3. `div` only when no semantic element fits.
