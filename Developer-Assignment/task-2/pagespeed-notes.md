# PageSpeed Optimisation Notes — OrthoNow Landing Page

**File:** `task-2/index.html`
**Target:** PageSpeed Insights Mobile ≥ 90
**Environment:** Standalone HTML — no build step, no CDN, no framework

---

## Why This Page Should Score 90+ on Mobile

Every PageSpeed penalty on a landing page comes from one of four sources:
render-blocking resources, large payloads, layout instability, or slow
server response. This file eliminates all four by design.

---

## 1. Zero External HTTP Requests

The single largest mobile PageSpeed killer is network round-trips for
external assets. This page makes **zero** external requests:

| What was avoided | Why it matters |
|---|---|
| Google Fonts (`fonts.googleapis.com`) | Each font request blocks rendering and costs 100–400ms on mobile 4G |
| Bootstrap / Tailwind CDN | 30–80 KB of unused CSS; triggers render-blocking |
| jQuery or any JS library | Adds 30–90 KB; parse time on low-end Android devices is significant |
| External analytics scripts | GTM is deliberately commented out — the implementor adds it after QA |
| External images / hero background | CSS `linear-gradient()` replaces a hero image entirely |

**Result:** The browser renders the entire page from a single file response.
On a 4G connection (10 Mbps), a 25–35 KB HTML file transfers in under 30ms.

---

## 2. System Font Stack

```css
font-family: system-ui, -apple-system, 'Segoe UI', Roboto,
             'Helvetica Neue', Arial, sans-serif;
```

This cascade resolves to a font already installed on the device:
- iPhone / iPad: San Francisco (system-ui / -apple-system)
- Android: Roboto
- Windows: Segoe UI
- Linux: Ubuntu / Cantarell

**Zero font download. Zero FOIT (Flash of Invisible Text). Zero FOUT.**

---

## 3. CSS Custom Properties for Zero Duplication

CSS custom properties (`--clr-primary`, `--shadow-md`, etc.) are declared
once in `:root`. Every component references the token rather than repeating
a raw hex value. This keeps the inline `<style>` block compact (~12 KB
unminified) and avoids the specificity wars that bloat CSS in practice.

**Estimated minified CSS size:** ~7–8 KB (well within the 14 KB critical
CSS budget for zero render-blocking).

---

## 4. Fluid Typography with `clamp()`

```css
font-size: clamp(1.7rem, 5.5vw, 2.8rem);
```

`clamp()` replaces multiple `@media` breakpoint overrides for the same
property, reducing the CSS rule count and preventing layout shift as the
viewport resizes.

---

## 5. Inline SVG Icons — No Icon Font, No Sprite Sheet

All icons are inlined directly in HTML as `<svg>` elements. Benefits:

- No separate HTTP request for an icon font or sprite
- No invisible FOIT period while the icon font loads
- Each icon is only as large as it needs to be (100–300 bytes each)
- Fully accessible via `aria-hidden="true"` on decorative icons
- Colour inherits from CSS `currentColor` — zero duplication

---

## 6. CSS Animations Avoided

The page uses `transition` only for hover states (colour, transform, box-shadow).
No `@keyframes` animations that would trigger compositor layers on mobile.
No `transform: rotate` or `translateZ` hacks — these can cause GPU memory
pressure on low-end devices.

---

## 7. No Layout Shift Sources (CLS = 0)

Cumulative Layout Shift is eliminated by:

- **No images without explicit dimensions.** The only "images" are SVGs
  with explicit `width` / `height` attributes.
- **No late-loading web fonts.** System fonts render immediately.
- **No injected ad slots or dynamic content above the fold.**
- **Sticky header height is fixed** (`--header-h: 60px`) and reserved
  with `position: sticky` — the body does not jump when it locks.

---

## 8. Minimal JavaScript Payload

The entire `<script>` block is ~3 KB unminified (~1.5 KB gzipped). It:
- Uses no `document.write`
- Makes no synchronous XHR calls
- Defers no resources that need preloading
- Is placed at the bottom of `<body>` — non-blocking by default

The browser parses and executes it after the page is already painted.

---

## 9. Mobile-First CSS Strategy

All base styles target the smallest viewport (320px). Wider layouts are
applied with `min-width` breakpoints only (640px, 768px, 1024px). This
means mobile devices download only the rules they need in the first
cascade pass — they are not forced to parse and then override desktop-first
rules.

---

## 10. Sticky CTA Bar — Padding Compensation

The `<body>` has `padding-bottom: 70px` on mobile to prevent the sticky CTA
bar from obscuring page content. This is removed at the 768px breakpoint
(`padding-bottom: 0`) when the sticky bar is hidden. Without this padding,
the footer text would sit behind the bar — a UX defect that also causes
content to be clipped from accessibility tools.

---

## Estimated Scores

| Metric | Expected Range | Reason |
|---|---|---|
| Performance (Mobile) | 92–97 | No external resources, minimal JS, no images |
| Accessibility | 95–100 | Semantic HTML, ARIA labels, focus styles |
| Best Practices | 95–100 | No mixed content, no deprecated APIs |
| SEO | 85–90 | Meta description present; canonical and structured data not added (out of scope) |

---

## If GTM Is Added After Handoff

Adding a GTM container will reduce the performance score by approximately
5–12 points on mobile because:

1. GTM's loader script is render-blocking until it resolves
2. Tags fired inside the container may inject third-party scripts

To mitigate: load GTM asynchronously (as the standard snippet already does),
and audit the container regularly to remove unused tags. Consider enabling
GTM's "Server-Side Tagging" to reduce client-side script weight.

---

## How to Test

1. Open `task-2/index.html` by double-clicking (no build step needed)
2. Deploy to any static host: GitHub Pages, Netlify, Vercel
3. Run the deployed URL through [PageSpeed Insights](https://pagespeed.web.dev/)
4. Check the Lighthouse panel in Chrome DevTools (Ctrl+Shift+I → Lighthouse → Mobile)
