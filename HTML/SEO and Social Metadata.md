# SEO and Social Metadata

SEO in HTML is mostly clarity: a unique title, a useful description, one canonical URL, semantic headings, and links/image text that crawlers can read. Social metadata (Open Graph, Twitter cards) decides how the page looks when shared.

The core idea:

```text
crawler / social bot
    ↓
reads head metadata + body structure
    ↓
builds snippet, preview, and ranking signals
```

You are not "optimizing for algorithms" — you are making the page unambiguous.

---

# 1. title and meta description

`title` is the strongest on-page signal and the default search-result heading. `meta description` is the snippet suggestion. Both should be unique per page and written for humans.

```html
<title>HTML Forms for Interviews — Kakade.dev</title>
<meta
  name="description"
  content="Labels, validation, GET vs POST, and accessible form controls explained with examples."
/>
```

Output:

```text
Search result:
  HTML Forms for Interviews — Kakade.dev
  Labels, validation, GET vs POST, and accessible form controls explained with examples.
```

| Element | Length target | Purpose |
| ------- | ------------- | ------- |
| `title` | ~50–60 characters | Ranking + result heading |
| `meta description` | ~120–160 characters | Snippet suggestion |

Interview trap: Google may rewrite the snippet from page content. A description is a suggestion, not a guarantee — but a missing one gives the engine full control.

---

# 2. canonical and robots

`rel="canonical"` names the preferred URL when the same content is reachable at several addresses. `meta name="robots"` controls indexing and link following.

```html
<link rel="canonical" href="https://kakade.dev/html/forms" />
<meta name="robots" content="index, follow" />
<meta name="robots" content="noindex, nofollow" />
```

Output:

```text
Canonical → search engines consolidate signals on /html/forms
index, follow → default: index the page and follow links
noindex, nofollow → keep out of the index, don't follow links
```

Think:

```text
example.com/html/forms
example.com/html/forms?utm=abc
example.com/html/forms/
        ↓ canonical
https://kakade.dev/html/forms
```

Use `noindex` for private, duplicate, or thin pages. Never combine `noindex` with a canonical pointing elsewhere and expect predictable results.

---

# 3. Open Graph and Twitter Cards

Open Graph tags control the title, description, image, and type shown by Facebook, LinkedIn, Slack, and others. Twitter/X uses its own `twitter:` tags, falling back to Open Graph.

```html
<meta property="og:title" content="HTML Forms for Interviews" />
<meta property="og:description" content="Labels, validation, and accessible controls." />
<meta property="og:image" content="https://kakade.dev/img/forms-cover.png" />
<meta property="og:url" content="https://kakade.dev/html/forms" />
<meta property="og:type" content="article" />

<meta name="twitter:card" content="summary_large_image" />
<meta name="twitter:title" content="HTML Forms for Interviews" />
<meta name="twitter:image" content="https://kakade.dev/img/forms-cover.png" />
```

Output:

```text
Shared link preview:
  [cover image]
  HTML Forms for Interviews
  Labels, validation, and accessible controls.
  kakade.dev
```

Interview trap: Open Graph uses `property`, not `name`. Image URLs should be absolute and at least 1200×630 for large previews.

---

# 4. JSON-LD Structured Data

JSON-LD describes the page as structured data (Article, Person, FAQ, Product) so search engines can build rich results. It lives in a `script type="application/ld+json"` block.

```html
<script type="application/ld+json">
  {
    "@context": "https://schema.org",
    "@type": "Article",
    "headline": "HTML Forms for Interviews",
    "author": {
      "@type": "Person",
      "name": "Kakade"
    },
    "datePublished": "2026-10-28",
    "image": "https://kakade.dev/img/forms-cover.png"
  }
</script>
```

Output:

```text
Search engines can associate the page with Kakade,
the publish date, and the cover image for rich results.
```

Structured data must match visible content. Marking up a rating or FAQ that is not on the page is a spam signal.

---

# 5. Heading Structure and lang

Crawlers use headings to understand hierarchy and `lang` to serve the right language and pronunciation. One `h1`, ordered `h2`/`h3`, and a correct `lang` are cheap wins.

```html
<html lang="en">
  <body>
    <main>
      <h1>HTML Forms for Interviews</h1>
      <h2>Labels and inputs</h2>
      <h3>Why labels matter</h3>
    </main>
  </body>
</html>
```

Output:

```text
Outline:
  H1 HTML Forms for Interviews
    H2 Labels and inputs
      H3 Why labels matter
```

`lang` also helps search engines target the right audience (for example `en-IN` vs `en-US`). Do not stuff keywords into headings — clarity ranks better than repetition.

---

# 6. Crawlable Links and Images

Crawlers follow real `href` values and read image `alt`. JavaScript-only navigation and background-image text are invisible or weak signals.

```html
<a href="/html/forms">HTML forms guide</a>

<img
  src="/img/kakade-forms.png"
  alt="Form with labeled email and password fields"
  width="640"
  height="360"
/>
```

Output:

```text
Crawler discovers /html/forms via href
Image alt contributes to image search and context
```

Bad:

```html
<div onclick="location.href='/html/forms'">HTML forms guide</div>
```

Good:

```html
<a href="/html/forms">HTML forms guide</a>
```

Interview trap: `alt` is not keyword stuffing space. Describe the image; relevance follows.

---

# Real-World Uses

An article head combines title, description, canonical, robots, Open Graph, Twitter cards, and JSON-LD.

```html
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>HTML Forms for Interviews — Kakade.dev</title>
  <meta name="description" content="Labels, validation, GET vs POST, and accessible form controls." />
  <link rel="canonical" href="https://kakade.dev/html/forms" />
  <meta name="robots" content="index, follow" />

  <meta property="og:title" content="HTML Forms for Interviews" />
  <meta property="og:description" content="Labels, validation, and accessible controls." />
  <meta property="og:image" content="https://kakade.dev/img/forms-cover.png" />
  <meta property="og:type" content="article" />

  <meta name="twitter:card" content="summary_large_image" />
</head>
```

Output:

```text
Search snippet, canonical consolidation, social preview,
and rich-result eligibility from one head block.
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **title** | Unique, ~50–60 chars, ranking signal |
| **meta description** | ~120–160 chars, snippet suggestion |
| **canonical** | Preferred URL for duplicate content |
| **robots meta** | `index/follow`, `noindex`, `nofollow` |
| **Open Graph** | `property="og:*"` for social previews |
| **Twitter cards** | `twitter:card`, title, image |
| **JSON-LD** | Structured data matching visible content |
| **Heading structure** | One `h1`, ordered `h2`/`h3` |
| **lang** | Language and regional targeting |
| **Crawlable links/images** | Real `href`, meaningful `alt` |

## The Mental Model

```text
head
  ├── title → result heading
  ├── description → snippet
  ├── canonical → preferred URL
  ├── robots → index/follow policy
  ├── og:* / twitter:* → social preview
  └── application/ld+json → structured data
body
  ├── h1 → h2 → h3 outline
  ├── <a href> → crawlable paths
  └── img alt → image context
```

### The 3 things to remember

**1. Unique title + description:**

```html
<title>HTML Forms for Interviews — Kakade.dev</title>
<meta name="description" content="Labels, validation, and accessible controls." />
```

**2. One canonical URL:**

```html
<link rel="canonical" href="https://kakade.dev/html/forms" />
```

**3. Social tags use property:**

```html
<meta property="og:title" content="HTML Forms for Interviews" />
```

> **SEO is clarity for machines: unique titles, one canonical URL, descriptive metadata, and real links with meaningful text.**
