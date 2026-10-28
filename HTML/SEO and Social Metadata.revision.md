# SEO and Social Metadata — Revision Notes (Intermediate)
> One-line mental model: SEO is clarity — unique title, one canonical URL, descriptive metadata, and crawlable structure.

## Core recall
- `title` = strongest on-page signal + result heading; unique, ~50–60 characters.
- `meta description` = ~120–160 character snippet suggestion; engines may rewrite it.
- `rel="canonical"` names the preferred URL for duplicate content and consolidates signals (hint, not redirect).
- `meta name="robots"` controls `index`/`noindex` and `follow`/`nofollow`.
- Open Graph uses `property="og:*"` (title, description, image, url, type) for social previews.
- Twitter/X uses `twitter:card`, `twitter:title`, `twitter:image`, falling back to OG.
- JSON-LD (`application/ld+json`, schema.org) enables rich results and must match visible content.
- Heading structure: one `h1`, ordered `h2`/`h3`; `lang` targets language/region.
- Crawlers follow real `href` values; JS-only navigation is weak or invisible.
- Image `alt` supplies context; never keyword-stuff it.

## Must-remember code
```html
<title>HTML Forms for Interviews — Kakade.dev</title>
<meta name="description" content="Labels, validation, GET vs POST, and accessible form controls." />
<link rel="canonical" href="https://kakade.dev/html/forms" />
<meta name="robots" content="index, follow" />
```
```html
<meta property="og:title" content="HTML Forms for Interviews" />
<meta property="og:description" content="Labels, validation, and accessible controls." />
<meta property="og:image" content="https://kakade.dev/img/forms-cover.png" />
<meta property="og:type" content="article" />

<meta name="twitter:card" content="summary_large_image" />
```
```html
<script type="application/ld+json">
  {
    "@context": "https://schema.org",
    "@type": "Article",
    "headline": "HTML Forms for Interviews",
    "author": { "@type": "Person", "name": "Kakade" }
  }
</script>
```
```html
<a href="/html/forms">HTML forms guide</a>
```

## Interview traps
- Duplicate or missing titles/descriptions across pages.
- Canonical pointing to a different page than `noindex`.
- Using `name` instead of `property` for Open Graph.
- Relative OG image URLs (must be absolute).
- Structured data that does not match visible content.
- JS-only navigation instead of anchors.
- Keyword stuffing alt text/headings.
- Multiple `h1`s or skipped heading levels.

## 3 things to remember
1. Unique title + description + canonical per page.
2. OG uses `property`; Twitter uses `name="twitter:*"`.
3. Real links and meaningful alt keep content crawlable.
