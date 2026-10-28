# HTML Fundamentals — Revision Notes (Beginner)
> One-line mental model: HTML declares meaning, the browser parses it into a DOM, and CSS/JS build on that structure.

## Core recall
- HTML (HyperText Markup Language) is markup, not programming: no variables, loops, or logic — just elements describing content.
- HTML5 is a living standard: semantic elements, native media, form validation, and browser APIs.
- Document anatomy: `<!DOCTYPE html>` → `<html lang="en">` → `<head>` (metadata) + `<body>` (visible content).
- Doctype is not a tag — it selects standards mode; missing doctype triggers quirks mode with legacy layout behavior.
- `head` holds title, charset, viewport, description, stylesheet/script links; `body` holds everything rendered.
- Comments: `<!-- ... -->`, never displayed, never a place to keep production code.
- Tags and attribute names are case-insensitive; lowercase is the convention. `id`/`class` values are case-sensitive.
- Render chain: HTML → DOM, CSS → CSSOM, combined into render tree → layout → paint.
- HTML5 vs XHTML: forgiving `text/html` vs strict `application/xhtml+xml` XML rules.
- Void elements (`img`, `br`, `input`, `meta`, `link`) cannot have children; the `/` is ignored by HTML5 parsers.

## Must-remember code
```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <title>Kakade Profile</title>
  </head>
  <body>
    <main>
      <h1>Kakade</h1>
      <p>Frontend learner</p>
    </main>
  </body>
</html>
```
```html
<!-- profile -->
<section>
  <h2>Kakade</h2>
  <img src="/kakade.png" alt="Kakade" width="120" height="120" />
</section>
```
```html
<meta name="viewport" content="width=device-width, initial-scale=1" />
```

## Interview traps
- Calling HTML a programming language — it is a markup language and is parsed, not compiled.
- Forgetting the doctype — the page silently renders in quirks mode.
- Putting visible content inside `head` — it is not rendered where you expect.
- Believing the `/` in `<img />` is required — it is ignored in HTML5, only meaningful in SVG/XHTML.
- Omitting image `width`/`height` — causes layout shift (CLS) when the image loads.

## 3 things to remember
1. Skeleton first: doctype, `html lang`, `head` metadata, `body` content.
2. Meaning before styling: choose the element that describes the content.
3. Keep the render path clean: dimensions on images, `defer` on scripts.
