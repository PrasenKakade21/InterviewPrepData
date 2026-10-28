# Links and Navigation

Links are the "hypertext" in HTML: `a` turns text or images into jumps between pages, sections, emails, or files. Interviewers care about the details — relative paths, `target="_blank"` security, skip links, and why a link is not a button.

The core idea:

```text
<a href="destination">label</a>
    ↓
browser resolves URL
    ↓
navigates, downloads, opens app, or jumps in-page
```

Everything else in this chapter is about resolving the destination correctly and making the link usable for keyboard and screen-reader users.

---

# 1. The a Element and Absolute vs Relative URLs

`a` needs an `href` to be a link. An absolute URL includes the protocol and domain; a relative URL resolves against the current document's URL.

```html
<a href="https://kakade.dev/notes">Absolute</a>
<a href="notes/html">Relative</a>
<a href="../css/style.css">Parent-relative</a>
```

Output:

```text
Absolute  → https://kakade.dev/notes
Relative  → https://kakade.dev/blog/notes/html   (if page is /blog/post)
Parent    → https://kakade.dev/blog/css/style.css
```

| Form | Example | Resolves against |
| ---- | ------- | ---------------- |
| Absolute | `https://kakade.dev/notes` | Nothing — used as-is |
| Document-relative | `notes/html` | Current page URL |
| Root-relative | `/notes/html` | Domain root |
| Protocol-relative | `//cdn.kakade.dev/x.js` | Current protocol |

Interview trap: a link without `href` is not focusable and not announced as a link. `<a>Click</a>` is a span with extra steps.

---

# 2. Root-Relative Paths

A root-relative path starts with `/` and always resolves from the site root, regardless of how deep the current page is. This is why asset links stay stable across routes.

```html
<!-- Page: https://kakade.dev/blog/2026/html -->
<a href="/notes">Notes</a>
<img src="/img/kakade.png" alt="Kakade" />
```

Output:

```text
<a> → https://kakade.dev/notes
<img> → https://kakade.dev/img/kakade.png
```

Bad:

```html
<!-- Breaks when the page moves deeper -->
<a href="../../notes">Notes</a>
```

Good:

```html
<a href="/notes">Notes</a>
```

Root-relative paths break only when the site is deployed under a subpath (for example GitHub Pages project sites). In that case, either include the base path or use a `<base>` tag deliberately.

---

# 3. target and rel

`target="_blank"` opens a link in a new tab. Modern browsers imply `noopener`, but adding `rel="noopener noreferrer"` is still the explicit, safe pattern. `rel` also communicates relationships to search engines.

```html
<a href="https://kakade.dev" target="_blank" rel="noopener noreferrer">
  Kakade.dev
</a>
<a href="/sponsors" rel="sponsored">Our sponsor</a>
<a href="/old-post" rel="nofollow">Archived post</a>
```

Output:

```text
Kakade.dev opens in a new tab, with no window.opener access.
Sponsored/nofollow links carry SEO hints.
```

Think:

```text
target="_blank" → new tab
rel="noopener"  → new page cannot control window.opener
rel="noreferrer"→ also hides the Referer header
```

Interview trap: `target="_blank"` without `noopener` used to allow reverse-tabnabbing. Browsers now default to `noopener`, but explicit is better and expected in code review.

---

# 4. download, mailto and tel

`download` tells the browser to save the target instead of navigating to it. `mailto:` opens the mail client, `tel:` dials on mobile — both are links, not forms.

```html
<a href="/files/kakade-resume.pdf" download="Kakade-Resume.pdf">
  Download resume
</a>
<a href="mailto:kakade@example.com?subject=Interview">Email Kakade</a>
<a href="tel:+911234567890">Call Kakade</a>
```

Output:

```text
Download resume → saves Kakade-Resume.pdf
Email Kakade    → opens mail app with subject "Interview"
Call Kakade     → opens dialer on mobile
```

Notice: `download` only works for same-origin URLs. Cross-origin files ignore the filename and may open in a tab instead.

---

# 5. Fragment Links and Skip Links

A fragment link (`#id`) jumps to an element with that `id` on the same page. A skip link is a fragment link placed first in the tab order so keyboard users can jump past repeated navigation.

```html
<a href="#main-content" class="skip-link">Skip to content</a>

<nav>
  <a href="#html">HTML</a>
  <a href="#css">CSS</a>
</nav>

<main id="main-content">
  <h2 id="html">HTML</h2>
  <p>Markup notes.</p>
  <h2 id="css">CSS</h2>
  <p>Styling notes.</p>
</main>
```

Output:

```text
First Tab press → "Skip to content" link appears
Activating it → focus jumps to <main id="main-content">
Clicking #html → scrolls to the HTML heading
```

The skip link is usually hidden until focused with CSS (`.skip-link:focus { ... }`). Interview trap: fragment links need a matching `id`, not a `name`. Duplicate `id`s make the jump land on the first match.

---

# 6. Link vs Anchor, Accessibility and SEO

A **link** navigates to a URL; an **anchor** is the target inside the page (`id`). In HTML the `<a>` element covers both, which is why the terminology is confusing. For accessibility, link text must make sense out of context.

```html
<!-- Bad: meaningless out of context -->
<a href="/notes/html">Click here</a>

<!-- Good: descriptive -->
<a href="/notes/html">Read Kakade's HTML notes</a>

<!-- Image link needs alt text -->
<a href="/">
  <img src="/logo.svg" alt="Kakade.dev home" width="120" height="32" />
</a>
```

Output:

```text
Screen reader link list:
  "Read Kakade's HTML notes"
  "Kakade.dev home"
```

SEO basics: crawlers follow `href` values, so real links beat JavaScript click handlers. Use `rel="nofollow"`/`sponsored` where appropriate and keep link text descriptive. Never use a link for an in-page action — that is a `button`.

---

# Real-World Uses

A site header combines a logo link, primary navigation, skip link, a new-tab external link, and a download.

```html
<a href="#main" class="skip-link">Skip to content</a>

<header>
  <a href="/">
    <img src="/logo.svg" alt="Kakade.dev home" width="120" height="32" />
  </a>

  <nav aria-label="Primary">
    <a href="/notes">Notes</a>
    <a href="/projects">Projects</a>
    <a href="https://github.com/Kakade" target="_blank" rel="noopener noreferrer">
      GitHub
    </a>
  </nav>
</header>

<main id="main">
  <a href="/files/kakade-resume.pdf" download>Download resume</a>
</main>
```

Output:

```text
Tab order: Skip link → logo → Notes → Projects → GitHub → Download resume
External GitHub link opens a new tab safely.
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **a element** | Needs `href` to be a real link |
| **Absolute vs relative** | Full URL vs resolved against current page |
| **Root-relative** | Starts with `/`, resolves from domain root |
| **target and rel** | New tab needs `noopener noreferrer` |
| **download** | Saves same-origin files |
| **mailto and tel** | Open mail client / dialer |
| **Fragment links** | `#id` jumps within the page |
| **nav and skip links** | Landmark navigation; first focusable link |
| **link vs anchor** | Navigation vs in-page target |
| **A11y and SEO** | Descriptive text, real hrefs, image alt |

## The Mental Model

```text
<a href="...">
  │
  ├── https://...        → absolute
  ├── /path              → root-relative
  ├── path               → document-relative
  ├── #id                → in-page fragment
  ├── mailto: / tel:     → external app
  └── download           → save file
        │
        └── label must make sense out of context
```

### The 3 things to remember

**1. Always give a real href:**

```html
<a href="/notes">Read Kakade's notes</a>
```

**2. New tab = explicit safety:**

```html
<a href="https://kakade.dev" target="_blank" rel="noopener noreferrer">Kakade.dev</a>
```

**3. Skip link first:**

```html
<a href="#main" class="skip-link">Skip to content</a>
```

> **A link is navigation with a real destination — resolve the URL correctly, secure new tabs, and write link text that stands alone.**
