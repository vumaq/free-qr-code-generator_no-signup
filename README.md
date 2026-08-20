# Free QR Code Generator — No Signup, No Expiry

A free, privacy-first QR code generator. No account, no watermark, no expiring
codes — everything runs client-side in the browser, so nothing you type is
ever uploaded to a server.

**Live site:** [qrcodenosignup.com](https://qrcodenosignup.com/)

## Features

- **No signup required** — open the page and start generating
- **Static QR codes that never expire** — your text is encoded directly into
  the code, not behind a redirect/tracking service
- **Private by design** — QR generation happens entirely in the browser;
  nothing you type is sent anywhere
- **Custom colors** with a built-in contrast checker that warns you if your
  foreground/background combination is too low-contrast to scan reliably
- **Adjustable padding (quiet zone)**, **error correction level**, and
  **export size** (64–1024px)
- **Download as PNG or SVG**, or copy the code straight to your clipboard
- **Installable as a PWA** — works offline once installed, thanks to a
  service worker caching the app shell
- Fully static, no backend, no database, no build step

## Tech stack

- **[Vue 3](https://vuejs.org/)** (loaded via CDN, no build tooling) for
  reactivity
- **[qrcode-generator](https://github.com/kazuhikoarase/qrcode-generator)**
  for QR matrix generation — rendered as hand-built SVG so colors, padding,
  and export size are fully controllable client-side
- Plain CSS with a small design-token system (spacing/radius/type scale via
  CSS custom properties) — no CSS framework
- No build step, no bundler, no package.json — it's a single HTML file plus
  a handful of static assets

## Project structure

```
├── index.html              # the entire app — markup, styles, and script
├── manifest.json            # PWA manifest
├── sw.js                     # service worker (offline app-shell caching)
├── icon-192.png              # PWA icon
├── icon-512.png               # PWA icon
├── icon-512-maskable.png       # PWA icon (maskable/adaptive variant)
├── og-image.png                # Open Graph / Twitter share image (1200×630)
├── robots.txt
└── sitemap.xml
```

## Running locally

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

## Deployment

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

## Browser support

Targets modern evergreen browsers (Chrome, Firefox, Safari, Edge). Uses:
- `<input type="color">` for the color pickers
- The Clipboard API (`navigator.clipboard.write`) for the "Clipboard" export
  button — falls back to a visible error message in browsers that don't
  support it, rather than failing silently
- CSS `aspect-ratio` and `clamp()`

## Accessibility

- Single `<main>` landmark, proper heading hierarchy (one `<h1>`, `<h2>` per
  section, `<h3>` per card/FAQ item)
- Color inputs have both `aria-label` and an associated `<label for>`
- The generated QR preview has `role="img"` with a descriptive `aria-label`
- Text/background color combinations are checked against WCAG AA (4.5:1)
- Visible, high-contrast `:focus-visible` states on all interactive controls
- Respects `prefers-reduced-motion`

## SEO

- Full meta tags (title, description, canonical, Open Graph, Twitter Card)
- `WebApplication` and `FAQPage` JSON-LD structured data — FAQ answers in
  the structured data are kept in sync word-for-word with the visible FAQ
  text on the page (required for Google to actually surface the rich result)
- `sitemap.xml` and `robots.txt` included

## License

Add a license of your choice (MIT is a common default for a project like
this) — none has been specified yet.
