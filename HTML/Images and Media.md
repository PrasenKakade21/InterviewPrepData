# Images and Media

Images and media make a page feel real, but they are also where performance and accessibility break first. `img` needs `alt` and dimensions, `srcset` picks the right file, `picture` changes art direction, and `video`/`audio` should never autoplay with sound.

The core idea:

```text
media element
    ↓
browser chooses a source
    ↓
reserves space → loads → paints
```

Reserve space and describe the content, and images stop causing layout shift and screen-reader silence.

---

# 1. The img Element and alt Text Rules

`img` embeds a raster image and is a void element — no closing tag. `alt` is required: it describes the image's purpose, not its pixels.

```html
<img src="/img/kakade.png" alt="Kakade presenting HTML notes" width="320" height="200" />

<!-- Decorative image: empty alt, not a missing one -->
<img src="/img/divider.svg" alt="" width="600" height="8" />

<!-- Icon inside a labeled button -->
<button aria-label="Close dialog">
  <img src="/icons/close.svg" alt="" width="16" height="16" />
</button>
```

Output:

```text
Screen reader: "Kakade presenting HTML notes, image"
Decorative divider: skipped entirely
Close button: announced as "Close dialog, button"
```

| Image type | `alt` value |
| ---------- | ----------- |
| Informative | Describe the content/purpose |
| Decorative | `alt=""` |
| Functional (in a link/button) | Describe the action |
| Complex chart | Short alt + longer description nearby |

Interview trap: `alt="image"` or `alt="kakade.png"` is worse than no alt — it adds noise without meaning.

---

# 2. Width, Height and CLS, loading and decoding

Always set `width` and `height` so the browser reserves space before the image loads. `loading="lazy"` defers offscreen images; `decoding="async"` lets the browser decode without blocking rendering.

```html
<img
  src="/img/kakade-notes.webp"
  alt="Kakade's handwritten HTML notes"
  width="800"
  height="450"
  loading="lazy"
  decoding="async"
/>
```

Output:

```text
Space for 800x450 is reserved immediately.
Image loads only when it nears the viewport.
No layout shift when it appears.
```

Think:

```text
no width/height → 0px box → image loads → content jumps  (bad CLS)
width/height    → box reserved → image loads → no jump    (good CLS)
```

Notice: CSS can still resize the image (`width: 100%; height: auto`), but the HTML attributes provide the aspect ratio the browser needs for reservation.

---

# 3. srcset, sizes and picture/source

`srcset` offers multiple resolutions so the browser can pick based on screen density and viewport. `sizes` tells it the rendered width. `picture` switches art direction or formats with `source` elements.

```html
<img
  src="/img/kakade-800.jpg"
  srcset="/img/kakade-400.jpg 400w, /img/kakade-800.jpg 800w, /img/kakade-1600.jpg 1600w"
  sizes="(max-width: 600px) 100vw, 50vw"
  alt="Kakade at a conference"
  width="800"
  height="450"
/>

<picture>
  <source srcset="/img/kakade.avif" type="image/avif" />
  <source srcset="/img/kakade.webp" type="image/webp" />
  <img src="/img/kakade.jpg" alt="Kakade at a conference" width="800" height="450" />
</picture>
```

Output:

```text
Phone on 2x DPR, 360px wide → browser picks kakade-800.jpg
Desktop at 50vw             → browser picks the best matching width
AVIF-capable browser        → uses kakade.avif
Fallback                    → kakade.jpg
```

Interview trap: `picture` is for art direction/format fallback; `srcset` on `img` is for resolution switching. `picture` still needs a final `<img>` as the fallback.

---

# 4. figure and figcaption

`figure` groups media with an optional caption. `figcaption` must be the first or last child, and the figure can be moved without breaking the article flow.

```html
<figure>
  <img
    src="/img/kakade-setup.jpg"
    alt="Kakade's desk with two monitors"
    width="800"
    height="450"
    loading="lazy"
  />
  <figcaption>Kakade's interview prep setup, 2026.</figcaption>
</figure>
```

Output:

```text
[image]
Kakade's interview prep setup, 2026.
```

A `figure` is not only for images — code samples, diagrams, and quotes can be figures too. The caption is associated with the figure for assistive tech.

---

# 5. audio, video and track

`audio` and `video` embed media natively. `controls` shows the browser UI, `preload` hints at loading strategy, and `track` adds captions/subtitles. Never autoplay audio.

```html
<audio controls preload="metadata">
  <source src="/media/kakade-intro.mp3" type="audio/mpeg" />
  Your browser does not support audio.
</audio>

<video controls width="640" height="360" poster="/img/kakade-poster.jpg">
  <source src="/media/kakade-talk.mp4" type="video/mp4" />
  <track
    kind="captions"
    src="/media/kakade-talk.vtt"
    srclang="en"
    label="English"
    default
  />
  Your browser does not support video.
</video>
```

Output:

```text
Audio player with play/pause and volume.
Video player with poster, captions menu, and English captions on by default.
```

Interview trap: `autoplay` usually requires `muted` in modern browsers, and autoplaying narration is an accessibility failure. Provide captions for any speech.

---

# 6. iframe, object, embed and Media Accessibility

`iframe` embeds another HTML page — maps, videos, widgets. Always add a descriptive `title`, because screen readers announce it as a frame. `object` and `embed` are legacy plugin/PDF embeds; `object` is preferred when you must use one.

```html
<iframe
  src="https://www.youtube.com/embed/VIDEO_ID"
  title="Kakade explains semantic HTML"
  width="560"
  height="315"
  loading="lazy"
  allowfullscreen
></iframe>

<object data="/files/kakade-resume.pdf" type="application/pdf" width="600" height="400">
  <p><a href="/files/kakade-resume.pdf">Download Kakade's resume</a></p>
</object>
```

Output:

```text
Screen reader: "Kakade explains semantic HTML, frame"
PDF renders inline, or the fallback link appears.
```

Media accessibility checklist:

- `alt` for images, `title` for iframes.
- Captions/transcripts for speech.
- No autoplay with sound.
- Controls visible for audio/video.
- Fallback content inside `video`, `audio`, `object`.

---

# Real-World Uses

A blog post hero combines a responsive image, explicit dimensions, lazy loading, and a caption — then embeds a captioned talk.

```html
<article>
  <h1>Kakade's 2026 Setup</h1>

  <figure>
    <picture>
      <source srcset="/img/setup.avif" type="image/avif" />
      <img
        src="/img/setup.jpg"
        alt="Kakade's desk with two monitors and a keyboard"
        width="1200"
        height="675"
        loading="lazy"
        decoding="async"
      />
    </picture>
    <figcaption>Kakade's interview prep setup.</figcaption>
  </figure>

  <iframe
    src="https://www.youtube.com/embed/VIDEO_ID"
    title="Kakade's desk tour"
    width="560"
    height="315"
    loading="lazy"
    allowfullscreen
  ></iframe>
</article>
```

Output:

```text
Responsive image with caption, no layout shift.
Lazy-loaded video embed with an accessible frame title.
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **img** | Void element; always needs `src` and `alt` |
| **alt rules** | Purpose, not pixels; `alt=""` when decorative |
| **width/height** | Reserve space, prevent CLS |
| **srcset/sizes** | Resolution switching per viewport/DPR |
| **loading/decoding** | `lazy` offscreen, `async` decode |
| **figure/figcaption** | Media plus associated caption |
| **picture/source** | Art direction and format fallback |
| **audio** | `controls`, no unsolicited autoplay |
| **video/track** | Poster, captions, fallback text |
| **iframe** | Descriptive `title`, `loading="lazy"` |
| **object/embed** | Legacy embeds; `object` preferred |
| **Media a11y** | Alt, captions, controls, no autoplay sound |

## The Mental Model

```text
media decision
  │
  ├── image?
  │     ├── informative → alt describes purpose
  │     ├── decorative → alt=""
  │     ├── responsive → srcset + sizes
  │     └── format/art → picture + source
  │
  ├── audio/video?
  │     ├── controls
  │     ├── captions/track
  │     └── no autoplay sound
  │
  └── embedded page?
        └── iframe + title + lazy
```

### The 3 things to remember

**1. Alt describes purpose:**

```html
<img src="/kakade.png" alt="Kakade presenting HTML notes" width="320" height="200" />
```

**2. Dimensions prevent shift:**

```html
<img src="/setup.jpg" alt="Kakade's desk" width="800" height="450" loading="lazy" />
```

**3. Captions and titles:**

```html
<video controls><track kind="captions" src="/talk.vtt" srclang="en" default /></video>
<iframe src="/map" title="Kakade's office location"></iframe>
```

> **Media is content plus metadata: alt and dimensions for images, captions and controls for audio/video, titles for iframes.**
