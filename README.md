# Free QR Code Generator — No Signup, No Expiry

**👉 Just want to make a QR code? Use it here: [qrcodenosignup.com](https://qrcodenosignup.com/)**

**Source:** [github.com/vumaq/free-qr-code-generator_no-signup](https://github.com/vumaq/free-qr-code-generator_no-signup) — also linked from the site itself (top-right icon, and in the footer).

No account, no install, nothing to download — open the link and generate a
code. Everything below this point is for people who want to read the code,
fork it, or self-host their own copy. If that's not you, the link above is
all you need.

---

## What it is

A free, privacy-first QR code generator. No signup, no watermark, no
expiring codes — everything runs client-side in the browser, so nothing you
type is ever uploaded to a server.

## Features

- **No signup required** — open the page and start generating
- **Three content types** (Text · Wi-Fi · vCard), each with its own icon
  in the type selector: plain link/text, a **Wi-Fi** preset (network name,
  password, security type, hidden-network toggle), and a **vCard** preset
  (name, organization, phone, email, website) — both build the correct
  encoded format automatically, including proper escaping of special
  characters in the Wi-Fi payload
- **Switching content type clears the form and the preview** — each type
  starts fresh rather than silently remembering what you typed in a
  different mode
- **Settings persist across visits** via `localStorage` — content type,
  error correction level, export size, padding, and colors are remembered.
  What you actually *type* (the text, Wi-Fi password, contact details) is
  never persisted — only your preferences are
- **GA4 event tracking** for content-type switches, control changes, and
  downloads (by format and content type) — analytics itself is injected by
  Cloudflare, not this codebase; the page just fires `gtag('event', ...)`
  calls defensively (no-ops if `gtag` isn't present)
- **Static QR codes that never expire** — your content is encoded directly
  into the code, not behind a redirect/tracking service
- **Private by design** — QR generation happens entirely in the browser;
  nothing you type is sent anywhere
- **Custom colors** with a built-in contrast checker that warns you if your
  foreground/background combination is too low-contrast to scan reliably
- **Adjustable padding (quiet zone)**, **error correction level**, and
  **export size** (64–1024px)
- **Download as PNG or SVG**, or copy the code straight to your clipboard
- **Installable as a PWA** — add it to your home screen from the site itself
  (no app store needed) and it works offline afterward

---

## For developers: forking or self-hosting

The rest of this README is about the code, not the tool. Most people should
just use the live site above.

### Tech stack

- **[petite-vue](https://github.com/vuejs/petite-vue)** (~6KB, loaded via CDN,
  no build tooling) for reactivity — migrated from full Vue 3 to cut payload
  size. Same directive syntax (`v-if`, `@click`, `{{ }}`), but state is a
  plain reactive object instead of `ref()`/`computed()`, since petite-vue
  doesn't expose those — computed-style values are plain getters instead,
  and there's no `watch()`, so side effects (saving to localStorage,
  clearing fields on type switch) are called explicitly inside methods
  rather than reactively
- **[qrcode-generator](https://github.com/kazuhikoarase/qrcode-generator)**
  for QR matrix generation — rendered as hand-built SVG so colors, padding,
  and export size are fully controllable client-side
- Plain CSS with a small design-token system (spacing/radius/type scale via
  CSS custom properties) — no CSS framework
- No build step, no bundler, no package.json — it's a single HTML file plus
  a handful of static assets

### A note on form inputs

All text inputs (the content textarea, Wi-Fi fields, contact fields) are
intentionally **uncontrolled** — bound with `@input` only, never `v-model` /
`:value`. This is deliberate: since the whole app is one flat reactive scope,
any reactive state change anywhere (clicking a padding button, picking a
color) triggers a full re-render, and a controlled input's value binding can
race with fast typing and silently drop or duplicate characters. Keeping
inputs uncontrolled means the DOM is always the single source of truth for
what's typed. Don't reintroduce `v-model` on these without being aware of
this tradeoff.

### A note on petite-vue's getters

petite-vue has no `computed()` — derived values are plain JS getters, which
are **not memoized** the way Vue's `computed()` is. `qrResult` (the actual
QR-matrix generation) is manually memoized by cache key
(`payload + errorCorrectionLevel`) for exactly this reason: without it, the
getter reran on every property access within a single render (once each
from `matrix`, `svgMarkup`, `canDownload`, and the template itself), which
caused a real bug where typing anything immediately showed a false
"too much content" error. If you add more derived getters that do
real work (not just cheap lookups), consider whether they need the same
manual memoization pattern.

### Project structure

```
├── index.html                # the entire app — markup, styles, and script
├── manifest.json              # PWA manifest
├── sw.js                       # service worker (offline app-shell caching)
├── favicon.ico                  # legacy favicon (some browsers/tools
│                                  request this by default regardless of
│                                  <link rel="icon">)
├── icon-192.png                  # PWA icon
├── icon-512.png                   # PWA icon
├── icon-512-maskable.png           # PWA icon (maskable/adaptive variant)
├── og-image.png                     # Open Graph / Twitter share image (1200×630)
├── llms.txt                          # plain-text summary for LLM crawlers
├── robots.txt
└── sitemap.xml
```

### Running it locally

No build step, no dependencies to install. Just serve the folder:

```bash
python3 -m http.server 8000
# or
npx serve .
```

Then open `http://localhost:8000`.

> Opening `index.html` directly via `file://` will work for the QR generator
> itself, but the service worker and manifest require being served over
> `http://` or `https://` (browsers won't register a service worker on the
> `file://` protocol).

### Deploying your own copy

This is a static site — deploy it anywhere that serves static files
(Cloudflare Pages, Netlify, Vercel, GitHub Pages, S3, etc.).

**Important:** all internal references (manifest, icons, service worker
registration, and the service worker's own cached asset list) use
**relative paths** on purpose, so the site works correctly whether it's
deployed at a domain root (`https://example.com/`) or a subpath
(`https://example.com/some-repo-name/`), which matters in particular for
GitHub Pages project sites. Don't change these back to root-absolute
(`/manifest.json`) paths unless you're certain the site will only ever be
served from a domain root.

If you fork this and deploy to your own domain, update:
- `<link rel="canonical">` and the `og:url` / `twitter` meta tags
- `og:image` / `twitter:image` (currently pointing at
  `https://qrcodenosignup.com/og-image.png`)
- The JSON-LD `url` fields (`WebApplication` schema)
- `sitemap.xml`'s `<loc>` and `robots.txt`'s `Sitemap:` line
- `llms.txt`'s page URL

### Browser support

Targets modern evergreen browsers (Chrome, Firefox, Safari, Edge). Uses:
- `<input type="color">` for the color pickers
- The Clipboard API (`navigator.clipboard.write`) for the "Clipboard" export
  button — falls back to a visible error message in browsers that don't
  support it, rather than failing silently
- CSS `aspect-ratio` and `clamp()`

### Accessibility

- Single `<main>` landmark, proper heading hierarchy (one `<h1>`, `<h2>` per
  section, `<h3>` per card/FAQ item)
- Color inputs have both `aria-label` and an associated `<label for>`
- The generated QR preview has `role="img"` with a descriptive `aria-label`
- Text/background color combinations are checked against WCAG AA (4.5:1)
- Visible, high-contrast `:focus-visible` states on all interactive controls
- Respects `prefers-reduced-motion`

### SEO

- Full meta tags (title, description, canonical, Open Graph, Twitter Card)
- Three JSON-LD blocks: `WebApplication`, `HowTo` (mirrors the "How to make
  a QR code online" steps), and `FAQPage` — FAQ answers in the structured
  data are kept in sync word-for-word with the visible FAQ text on the page
  (required for Google to actually surface the rich result; the HowTo steps
  are kept in sync the same way)
- `sitemap.xml`, `robots.txt`, and `llms.txt` included

### License

Add a license of your choice (MIT is a common default for a project like
this) — none has been specified yet.
