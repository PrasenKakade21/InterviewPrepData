# Document Structure and Metadata — Revision Notes (Beginner)
> One-line mental model: html is the root, head configures the browser, body carries the content.

## Core recall
- `html` is the single root and carries `lang` (and optionally `dir`); `head` = metadata, `body` = visible content.
- `title` sets the tab text and the default search-result heading; keep it unique per page.
- `<meta charset="UTF-8" />` must be the first thing in `head` (within the first 1024 bytes) so bytes decode correctly.
- `<meta name="viewport" content="width=device-width, initial-scale=1" />` makes mobile render at device width.
- `<meta name="description" content="...">` supplies a 120–160 character search/social snippet.
- `link` attaches stylesheets, icons, canonical URLs, preloads; `rel` defines the relationship.
- `base href` sets a default URL for all relative links — powerful and easy to misuse; omit unless needed.
- `lang` on `html` drives screen-reader pronunciation and spellcheck; `dir="rtl"` flips text direction.
- Never block zoom with `user-scalable=no` or `maximum-scale=1`.

## Must-remember code
```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <title>Kakade — HTML Notes</title>
    <meta name="description" content="Beginner-friendly HTML interview notes." />
    <link rel="canonical" href="https://kakade.dev/html" />
    <link rel="stylesheet" href="/css/style.css" />
  </head>
  <body>
    <h1>HTML Notes</h1>
  </body>
</html>
```
```html
<head>
  <base href="https://kakade.dev/" />
</head>
```
```html
<html lang="en" dir="ltr">
```

## Interview traps
- Missing/late charset → mojibake such as â‚¹ instead of ₹.
- `user-scalable=no` → blocks pinch-zoom, an accessibility failure.
- Multiple `head`/`body` tags → parser merges them unpredictably.
- Using `base` casually → breaks relative links and in-page anchors.
- Confusing `title` (tab/search heading) with `h1` (page heading).

## 3 things to remember
1. Charset + viewport first, always.
2. title and meta description are SEO surfaces, not decoration.
3. One html root, one head, one body, correct `lang`.
