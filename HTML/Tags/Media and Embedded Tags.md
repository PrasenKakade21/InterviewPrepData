# Media and Embedded Tags

Media tags bring images, audio, video, and external documents into the page. Embedded tags (`iframe`, `object`) are for other people's content. The rules are consistent: describe it, reserve its space, and provide a fallback.

The core idea:

```text
media
    ├── image → img, picture, source, figure, figcaption
    ├── time-based → audio, video, track
    ├── document/software → iframe, object, embed
    └── graphics → canvas, svg, math
```

---

# `<img>`

Embeds an image. Void element; requires `src` and `alt`.

| Attribute | Purpose |
| --------- | ------- |
| `src` | Image URL |
| `alt` | Accessible description |
| `width` / `height` | Intrinsic size (prevents CLS) |
| `srcset` / `sizes` | Responsive selection |
| `loading` | `lazy`/`eager` |
| `decoding` | `async`/`sync` |
| `fetchpriority` | `high`/`low`/`auto` |
| `usemap` | Client-side image map |

```html
<img
  src="/img/kakade.png"
  alt="Kakade presenting HTML notes"
  width="320"
  height="200"
  loading="lazy"
  decoding="async"
/>
```

Output:

```text
Image with reserved 320x200 space, loaded lazily.
```

Gotchas: `alt=""` for decorative images; `alt` is not a tooltip; don't lazy-load the LCP hero.

---

# `<picture>` and `<source>`

`picture` chooses among `source` elements based on format/conditions; the final `img` is the fallback.

```html
<picture>
  <source srcset="/img/hero.avif" type="image/avif" />
  <source srcset="/img/hero.webp" type="image/webp" />
  <img src="/img/hero.jpg" alt="Kakade's desk" width="1200" height="675" />
</picture>
```

Output:

```text
AVIF browser → hero.avif; WebP → hero.webp; else hero.jpg.
```

Gotchas: `source` is void; `picture` needs the fallback `img`; use `media` for art direction.

---

# `<figure>` and `<figcaption>`

Groups media with an associated caption.

```html
<figure>
  <img src="/img/setup.jpg" alt="Kakade's desk" width="800" height="450" />
  <figcaption>Kakade's interview prep setup.</figcaption>
</figure>
```

Output:

```text
[image]
Kakade's interview prep setup.
```

Gotchas: `figcaption` must be first or last child; figures can contain code/diagrams, not only images.

---

# `<audio>`

Embeds sound with native controls.

| Attribute | Purpose |
| --------- | ------- |
| `controls` | Show player UI |
| `src` or `<source>` | Audio file |
| `preload` | `none`/`metadata`/`auto` |
| `loop` | Repeat |
| `muted` | Start muted |
| `autoplay` | Play automatically (usually blocked/undesirable) |

```html
<audio controls preload="metadata">
  <source src="/media/intro.mp3" type="audio/mpeg" />
  Your browser does not support audio.
</audio>
```

Output:

```text
Player with play/pause, volume, and scrubber.
```

Gotchas: never autoplay narration; provide a transcript for speech content.

---

# `<video>` and `<track>`

Embeds video with controls, poster, and caption tracks.

```html
<video controls width="640" height="360" poster="/img/poster.jpg">
  <source src="/media/talk.mp4" type="video/mp4" />
  <track kind="captions" src="/media/talk.vtt" srclang="en" label="English" default />
  Your browser does not support video.
</video>
```

Output:

```text
Video player with poster and English captions enabled.
```

`track` kinds: `subtitles`, `captions`, `descriptions`, `chapters`, `metadata`. Gotchas: captions are required for speech; autoplay normally requires `muted`.

---

# `<iframe>`

Embeds another HTML document in a nested browsing context.

| Attribute | Purpose |
| --------- | ------- |
| `src` | Page URL |
| `title` | Accessible frame name (required) |
| `width` / `height` | Size |
| `loading` | `lazy` |
| `allow` | Permissions policy |
| `allowfullscreen` | Fullscreen permission |
| `sandbox` | Restrict capabilities |

```html
<iframe
  src="https://www.youtube.com/embed/VIDEO_ID"
  title="Kakade explains semantic HTML"
  width="560"
  height="315"
  loading="lazy"
  allowfullscreen
></iframe>
```

Output:

```text
Lazy-loaded video frame named "Kakade explains semantic HTML".
```

Gotchas: always set `title`; use `sandbox` for untrusted content; iframes are expensive — lazy-load them.

---

# `<object>` and `<embed>`

Legacy embeds for PDFs, SVG, and plugins. `object` supports fallback content; `embed` does not.

```html
<object data="/files/resume.pdf" type="application/pdf" width="600" height="400">
  <p><a href="/files/resume.pdf">Download Kakade's resume</a></p>
</object>

<embed src="/files/chart.svg" type="image/svg+xml" width="400" height="300" />
```

Output:

```text
PDF inline with a download fallback, or the SVG embedded directly.
```

Gotchas: plugin support is gone; prefer native PDF links or `<img>`/inline SVG; `embed` has no fallback.

---

# `<canvas>`

A bitmap drawing surface controlled by JavaScript.

```html
<canvas id="spark" width="240" height="60" aria-label="Study trend: rising"></canvas>
<script>
  const ctx = document.getElementById("spark").getContext("2d");
  ctx.fillStyle = "#0f172a";
  ctx.fillRect(0, 40, 240, 20);
</script>
```

Output:

```text
A dark bar drawn on a 240x60 canvas.
```

Gotchas: content is not in the accessibility tree; provide a text alternative or data table.

---

# `<svg>` and `<math>`

Inline SVG is vector markup in the DOM; MathML (`math`) renders mathematical notation.

```html
<svg viewBox="0 0 24 24" width="24" height="24" role="img" aria-label="Checkmark">
  <path d="M4 12l5 5L20 6" fill="none" stroke="currentColor" stroke-width="2" />
</svg>

<math>
  <mi>x</mi><mo>=</mo><mfrac><mn>1</mn><mn>2</mn></mfrac>
</math>
```

Output:

```text
Scalable checkmark icon; x = 1/2 rendered as math.
```

Gotchas: SVG needs `viewBox` for scaling and `role`/`aria-label` when meaningful; MathML support varies.

---

# `<map>` and `<area>`

Client-side image maps: `map` defines clickable regions, `area` defines each region.

```html
<img src="/img/office.png" alt="Kakade's office" usemap="#office" width="600" height="300" />
<map name="office">
  <area shape="rect" coords="0,0,200,150" href="/desk" alt="Desk" />
  <area shape="circle" coords="400,150,60" href="/library" alt="Library" />
</map>
```

Output:

```text
Clicking the desk region opens /desk; the circle opens /library.
```

Gotchas: image maps are hard to use on mobile and with keyboards; prefer separate links over a map when possible.

---

# Quick Reference

| Tag | Purpose |
| --- | ------- |
| `img` | Raster image |
| `picture` / `source` | Format/art-direction selection |
| `figure` / `figcaption` | Media + caption |
| `audio` | Sound player |
| `video` | Video player |
| `track` | Captions/subtitles |
| `iframe` | Embedded document |
| `object` / `embed` | Legacy embeds |
| `canvas` | Bitmap drawing |
| `svg` / `math` | Vector graphics / math |
| `map` / `area` | Client-side image map |

> **Media is content plus metadata — alt and dimensions for images, captions and controls for audio/video, titles for frames, fallbacks for everything.**
