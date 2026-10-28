# Projects

Projects turn syntax into muscle memory. Each build below reuses the whole roadmap: semantics, links, media, lists, tables, forms, accessibility, metadata, and loading. Build them in plain HTML first — no framework, no CSS reset, no build step.

The core idea:

```text
project
    ↓
sketch structure (what is it?)
    ↓
choose semantic elements
    ↓
add labels, alt, metadata
    ↓
validate + tab through it
```

If it works with CSS disabled and keyboard only, the HTML is solid.

---

# 1. Profile Page

A profile page is the smallest complete document: header, nav, main, article, image, contact details, and footer. It exercises metadata, semantic layout, and image alt rules.

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <title>Kakade — Frontend Learner</title>
    <meta name="description" content="Kakade's profile and HTML notes." />
  </head>
  <body>
    <header>
      <h1>Kakade</h1>
      <nav aria-label="Primary">
        <a href="#about">About</a>
        <a href="#contact">Contact</a>
      </nav>
    </header>

    <main>
      <article id="about">
        <img src="/img/kakade.jpg" alt="Kakade at a desk" width="240" height="240" />
        <h2>About</h2>
        <p>Frontend learner documenting HTML, CSS, and JavaScript.</p>
      </article>

      <section id="contact" aria-labelledby="contact-heading">
        <h2 id="contact-heading">Contact</h2>
        <address>
          <a href="mailto:kakade@example.com">kakade@example.com</a>
        </address>
      </section>
    </main>

    <footer><p>© 2026 Kakade</p></footer>
  </body>
</html>
```

Output:

```text
Semantic profile page with landmarks, labeled nav,
dimensioned image, contact address, and footer.
```

Stretch goal: add a `dl` with skills and a project list.

---

# 2. Accessible Signup Form

The signup form is the classic interview build: labels, fieldsets, validation attributes, autocomplete, errors, and a submit button with an explicit type.

```html
<form action="/signup" method="post">
  <fieldset>
    <legend>Create Kakade's account</legend>

    <label for="name">Full name</label>
    <input id="name" name="name" type="text" autocomplete="name" required />

    <label for="email">Email</label>
    <input id="email" name="email" type="email" autocomplete="email" required />

    <label for="password">Password</label>
    <input
      id="password"
      name="password"
      type="password"
      autocomplete="new-password"
      minlength="8"
      aria-describedby="password-help"
      required
    />
    <p id="password-help">At least 8 characters.</p>

    <label>
      <input type="checkbox" name="terms" required />
      I accept the terms
    </label>

    <button type="submit">Sign up</button>
  </fieldset>
</form>
```

Output:

```text
Tab order: name → email → password → terms → Sign up
Empty submit blocked by native validation
Password helper read with the field
```

Stretch goal: add a country `select`, a `datalist` for city, and a live error region.

---

# 3. Pricing Table

A pricing table exercises `table`, `caption`, `thead`/`tbody`/`tfoot`, `th scope`, and a scroll wrapper for mobile.

```html
<div class="table-scroll">
  <table>
    <caption>Kakade.dev plans</caption>
    <thead>
      <tr>
        <th scope="col">Plan</th>
        <th scope="col">Price</th>
        <th scope="col">Seats</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <th scope="row">Free</th>
        <td>₹0</td>
        <td>1</td>
      </tr>
      <tr>
        <th scope="row">Pro</th>
        <td>₹499</td>
        <td>5</td>
      </tr>
    </tbody>
    <tfoot>
      <tr>
        <th scope="row">All plans</th>
        <td colspan="2">Cancel anytime</td>
      </tr>
    </tfoot>
  </table>
</div>
```

Output:

```text
Accessible pricing grid with headers, a total row,
and horizontal scrolling on narrow screens.
```

Stretch goal: compare a semantic table with CSS-grid card pricing and note the tradeoff.

---

# 4. Media Gallery

A gallery combines `figure`/`figcaption`, responsive `srcset`, lazy loading, and a captioned video. It is the best practice for alt text and performance.

```html
<section aria-labelledby="gallery-heading">
  <h2 id="gallery-heading">Kakade's setup</h2>

  <figure>
    <img
      src="/img/desk-800.jpg"
      srcset="/img/desk-400.jpg 400w, /img/desk-800.jpg 800w"
      sizes="(max-width: 600px) 100vw, 50vw"
      alt="Kakade's desk with two monitors"
      width="800"
      height="450"
      loading="lazy"
      decoding="async"
    />
    <figcaption>The 2026 interview prep desk.</figcaption>
  </figure>

  <video controls width="640" height="360" poster="/img/tour-poster.jpg">
    <source src="/media/tour.mp4" type="video/mp4" />
    <track kind="captions" src="/media/tour.vtt" srclang="en" label="English" default />
    Your browser does not support video.
  </video>
</section>
```

Output:

```text
Responsive image with caption and reserved space,
plus a captioned video with a poster and fallback text.
```

Stretch goal: add `picture` with AVIF/WebP sources and measure the difference.

---

# 5. Blog Article Page

A blog article combines the document outline, `article`, `time`, a blockquote, code, an author footer, and SEO metadata. It is the closest to a real content site.

```html
<article>
  <header>
    <h1>Why Semantic HTML Matters</h1>
    <p>
      Published <time datetime="2026-10-28">28 Oct 2026</time> by
      <strong>Kakade</strong>
    </p>
  </header>

  <section aria-labelledby="why">
    <h2 id="why">Why it matters</h2>
    <p>Structure is the interface between content and code.</p>

    <blockquote>
      <p>Semantic markup is documentation the browser can read.</p>
    </blockquote>
  </section>

  <section aria-labelledby="code">
    <h2 id="code">Example</h2>
    <pre><code>&lt;article&gt;Kakade&lt;/article&gt;</code></pre>
  </section>

  <footer>
    <p>Written by Kakade. <a href="/notes">More notes</a></p>
  </footer>
</article>
```

Output:

```text
Logical outline: H1 → H2 Why it matters → H2 Example
Timestamp, quote, escaped code, and author footer.
```

Stretch goal: add JSON-LD `Article` and Open Graph tags, then check the social preview.

---

# 6. Dashboard, FAQ and Modal

Three small builds cover the remaining native features: a semantic dashboard shell, a `details` FAQ, and a `dialog` modal.

```html
<!-- Dashboard shell -->
<body>
  <header><h1>Kakade's Dashboard</h1></header>
  <nav aria-label="Sections"><a href="#stats">Stats</a></nav>
  <main id="main">
    <section id="stats" aria-labelledby="stats-heading">
      <h2 id="stats-heading">Stats</h2>
      <dl>
        <dt>Topics done</dt><dd>18</dd>
        <dt>Mock score</dt><dd>92%</dd>
      </dl>
    </section>
  </main>
  <aside aria-label="Tips"><p>Review traps daily.</p></aside>
  <footer><p>© 2026 Kakade</p></footer>
</body>

<!-- FAQ -->
<details>
  <summary>Is HTML a programming language?</summary>
  <p>No — it is a markup language.</p>
</details>

<!-- Modal -->
<button popovertarget="help">Help</button>
<div id="help" popover>
  <p>Email kakade@example.com</p>
</div>
```

Output:

```text
Dashboard landmarks + description list
Native FAQ accordion
Native popover help panel
```

Stretch goal: replace the popover with a `<dialog>` for a true modal and compare focus behavior.

---

# Real-World Uses

These builds map directly to real work: the profile and article pages become portfolio and blog routes, the signup form becomes an auth flow, the pricing table becomes a marketing page, the gallery becomes a docs/media section, and the dashboard/FAQ/modal patterns show up in every admin UI.

```text
profile page     → /about, portfolio
signup form      → /signup, /login
pricing table    → /pricing
media gallery    → /gallery, docs screenshots
blog article     → /blog/:slug
dashboard + FAQ  → /app, /help
modal            → settings, confirmations
```

Output:

```text
A single HTML foundation reused across marketing, auth, content, and app screens.
```

---

# Quick Revision

| Project | Skills exercised |
| ------- | ---------------- |
| **Profile page** | Document shell, semantics, image alt, address |
| **Signup form** | Labels, fieldsets, validation, autocomplete |
| **Pricing table** | caption, thead/tbody/tfoot, scope, scroll |
| **Media gallery** | figure, srcset, lazy loading, video captions |
| **Blog article** | Outline, time, quote, code, article footer |
| **Dashboard** | Landmarks, description list, aside |
| **FAQ** | details/summary, no JS |
| **Modal** | dialog/popover, focus behavior |

## The Mental Model

```text
build order
  │
  ├── 1. document shell → doctype, head, lang
  ├── 2. landmarks → header, nav, main, footer
  ├── 3. content → headings, text, lists, tables
  ├── 4. media → alt, dimensions, captions
  ├── 5. forms → labels, fieldsets, validation
  └── 6. verify → validator + keyboard + no-CSS pass
```

### The 3 things to remember

**1. Structure before styling:**

```html
<main>
  <h1>Kakade's Dashboard</h1>
  <section aria-labelledby="stats"><h2 id="stats">Stats</h2></section>
</main>
```

**2. Media and forms need metadata:**

```html
<img src="/desk.jpg" alt="Kakade's desk" width="800" height="450" loading="lazy" />
<label for="email">Email</label><input id="email" type="email" required />
```

**3. Verify like a user:**

```text
Validator → Tab through → disable CSS → screen reader spot-check
```

> **Build the structure, label everything, and verify with keyboard and no CSS — projects are where HTML becomes instinct.**
