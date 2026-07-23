---
title: "Web Performance Optimization: 15 Techniques to Speed Up Your Website"
slug: "web-performance-guide"
category: developer
tags:
  - web performance
  - optimization
  - Core Web Vitals
  - frontend
summary: "A slow website does not just frustrate users — it directly costs conversions and search ranking. This guide covers 15 proven optimization techniques organized by impact category, with concrete implementation details and the measurement tools to prove they are working."
coverImage: ""
status: published
scheduledAt: ""
---

## Introduction

Google's research established that as page load time increases from 1 second to 3 seconds, bounce rate increases by 32%. At 5 seconds, it jumps 90%. Amazon famously estimated that every 100ms of latency cost them 1% in revenue. These are not hypothetical numbers — they reflect real user behavior captured across billions of sessions.

Beyond user experience, Core Web Vitals are a confirmed Google ranking factor. A slow site costs you both users and organic search visibility simultaneously.

The good news: most performance wins come from a small set of techniques. This guide organizes the 15 most impactful by category, starting with the changes that move the needle most.

---

## Core Web Vitals: What You Are Optimizing For

Google's Core Web Vitals define three user-centric performance metrics. These are what Lighthouse, PageSpeed Insights, and Google Search Console measure and report.

| Metric | What It Measures | Good | Needs Work | Poor |
|---|---|---|---|---|
| **LCP** (Largest Contentful Paint) | How quickly the main content loads | ≤ 2.5s | 2.5–4.0s | > 4.0s |
| **INP** (Interaction to Next Paint) | Responsiveness to user interactions | ≤ 200ms | 200–500ms | > 500ms |
| **CLS** (Cumulative Layout Shift) | Visual stability (content jumping around) | ≤ 0.1 | 0.1–0.25 | > 0.25 |

INP replaced FID (First Input Delay) in March 2024. It measures the worst interaction latency across the full page visit, making it harder to game and more representative of real interactivity.

---

## Image Optimization (The Biggest Wins)

Images account for 50–70% of total page weight on most websites. This is where the largest gains live.

### Technique 1: Use Modern Image Formats

WebP delivers images roughly 25–30% smaller than JPEG at equivalent visual quality, and 30–40% smaller than PNG for photos. AVIF is even more aggressive — often 50% smaller than JPEG — though encoding is slower and browser support is slightly behind WebP.

```html
<picture>
  <source srcset="hero.avif" type="image/avif">
  <source srcset="hero.webp" type="image/webp">
  <img src="hero.jpg" alt="Hero image">
</picture>
```

The `<picture>` element lets browsers select the best format they support, with JPEG as a universal fallback. WebP is now supported in all major browsers including Safari 14+.

### Technique 2: Responsive Images with srcset and sizes

Serving a 1920px image to a 375px mobile screen wastes 5–10x the bandwidth needed.

```html
<img
  src="hero-800.jpg"
  srcset="hero-400.jpg 400w,
          hero-800.jpg 800w,
          hero-1200.jpg 1200w,
          hero-1920.jpg 1920w"
  sizes="(max-width: 600px) 100vw,
         (max-width: 1200px) 80vw,
         1200px"
  alt="Hero image"
>
```

The browser uses `sizes` to understand the rendered width in CSS pixels, then selects the appropriate source from `srcset` based on the device pixel ratio. This requires generating multiple sizes of each image, which image CDNs (Cloudinary, Imgix) or build tools (Sharp, next/image) handle automatically.

### Technique 3: Lazy Loading Below-the-Fold Images

```html
<img src="article-photo.jpg" loading="lazy" alt="Article photo">
```

The `loading="lazy"` attribute defers image loading until the image approaches the viewport. It requires zero JavaScript and is supported in all modern browsers. Apply it to every image that is not visible in the initial viewport. Never apply it to your LCP image — that counterproductively delays the metric you are trying to improve.

### Technique 4: Set Explicit Width and Height to Prevent CLS

```html
<img src="photo.jpg" width="800" height="600" alt="Photo">
```

Without explicit dimensions, the browser cannot reserve space for the image before it loads, causing content below to jump down — a CLS violation. Setting `width` and `height` attributes lets the browser calculate the aspect ratio and reserve space immediately. CSS `max-width: 100%` still makes it responsive.

---

## Resource Loading

### Technique 5: Minify and Bundle CSS/JS

Minification strips whitespace, comments, and shortens variable names. A typical 100KB JavaScript file minifies to 40–50KB. Bundling reduces the number of HTTP requests. Every major build tool handles this: webpack, Vite, esbuild, Rollup, Parcel.

Verify your production build is actually minified by opening DevTools Sources and checking that scripts are not human-readable.

### Technique 6: Code Splitting

Shipping your entire JavaScript bundle on every page loads code for features the current user may never trigger. Code splitting divides the bundle into chunks loaded on demand.

```javascript
// Dynamic import — chunk loaded only when user triggers this feature
const { Chart } = await import('./chart-library.js');
```

React supports this via `React.lazy()` and `Suspense`. Next.js does route-based splitting automatically. Aim for initial JS bundle under 100KB (gzipped) for fast mobile load.

### Technique 7: Preload Critical Resources

```html
<!-- Preload the hero image (your LCP candidate) -->
<link rel="preload" href="hero.webp" as="image">

<!-- Preload a critical web font -->
<link rel="preload" href="/fonts/body.woff2" as="font" type="font/woff2" crossorigin>

<!-- DNS prefetch for third-party domains -->
<link rel="dns-prefetch" href="https://analytics.example.com">
```

`preload` tells the browser to fetch the resource immediately, at high priority, even if it is not yet in the HTML parser's queue. Use sparingly — preloading too many resources defeats the purpose by competing for bandwidth. Prioritize your LCP element and critical fonts.

### Technique 8: Font Optimization

Web fonts are a common source of invisible latency. Two techniques matter most:

```css
@font-face {
  font-family: 'MyFont';
  src: url('/fonts/myfont.woff2') format('woff2');
  font-display: swap; /* Show fallback font immediately, swap when loaded */
}
```

`font-display: swap` eliminates invisible text during font load (FOIT — Flash of Invisible Text), replacing it with a fallback font immediately. The swap happens when the custom font is ready.

Font subsetting — including only the characters you actually use — reduces font files dramatically. A full Latin + extended font might be 300KB; a subset for English-only content might be 20KB. Tools: `pyftsubset`, Google Fonts `text=` parameter.

---

## Network Optimization

### Technique 9: CDN for Static Assets

A Content Delivery Network serves static files from edge nodes physically close to users. A visitor in Tokyo loading assets from a server in Virginia experiences 150–200ms of raw latency before a single byte arrives. With a CDN, that drops to under 10ms from a nearby edge node.

All major cloud providers offer CDN services: Cloudflare (exceptional free tier), AWS CloudFront, Fastly, Akamai. Configure your build output, images, and fonts to be served through CDN.

### Technique 10: HTTP/2 Multiplexing

HTTP/1.1 browsers open 6 parallel connections per domain. HTTP/2 sends all requests over a single connection simultaneously, eliminating head-of-line blocking. HTTP/2 is enabled at the server or CDN level — no application code changes required.

Verify: open Chrome DevTools → Network tab → right-click any request → add Protocol column. You should see `h2` for HTTP/2.

### Technique 11: Aggressive Cache Headers

```
Cache-Control: max-age=31536000, immutable
```

Fingerprinted assets (with content hash in filename like `app.3f4a8b.js`) can be cached indefinitely — if the content changes, the filename changes, busting the cache automatically. The `immutable` directive tells the browser not to revalidate even on hard refresh.

For HTML documents, use `Cache-Control: no-cache` (forces revalidation but still allows 304 Not Modified responses without retransferring the body).

### Technique 12: Brotli Compression

Brotli is a compression algorithm developed by Google that outperforms gzip by 15–25% for text assets (HTML, CSS, JS). Enable it at the server or CDN level. NGINX example:

```nginx
brotli on;
brotli_comp_level 6;
brotli_types text/html text/css application/javascript;
```

If your CDN is Cloudflare, Brotli is enabled automatically with zero configuration.

---

## Runtime Performance

### Technique 13: Avoid Layout Thrashing

Reading a DOM property like `offsetHeight` forces the browser to finish all pending style calculations. Alternating reads and writes in a loop causes "layout thrashing" — the browser recalculates layout dozens of times per second.

```javascript
// Bad: causes layout thrashing
elements.forEach(el => {
  const height = el.offsetHeight; // forces layout
  el.style.height = height * 2 + 'px'; // triggers style change
});

// Good: batch reads, then batch writes
const heights = elements.map(el => el.offsetHeight); // one layout pass
elements.forEach((el, i) => {
  el.style.height = heights[i] * 2 + 'px'; // one style update pass
});
```

### Technique 14: Virtual Scrolling for Long Lists

Rendering 10,000 DOM nodes simultaneously — a contact list, a log viewer, a data grid — creates significant memory pressure and paint cost. Virtual scrolling renders only the items currently visible in the viewport plus a small buffer, swapping nodes as the user scrolls.

Libraries: `react-virtual` (TanStack Virtual), `react-window`, `@angular/cdk/virtual-scroll`. Implementing from scratch is non-trivial; use a library.

### Technique 15: Web Workers for CPU-Intensive Tasks

JavaScript runs on the main thread. Heavy computation (parsing large CSVs, image processing, encryption, compression) blocks the main thread and makes the UI unresponsive — directly harming INP.

```javascript
// main.js
const worker = new Worker('./processor.worker.js');
worker.postMessage({ data: largeDataset });
worker.onmessage = ({ data: result }) => {
  updateUI(result);
};

// processor.worker.js
self.onmessage = ({ data: { data } }) => {
  const result = expensiveComputation(data); // runs off main thread
  self.postMessage(result);
};
```

Web Workers run on a separate thread with no access to the DOM. They communicate via message passing. Libraries like `comlink` make the API ergonomic.

---

## Measurement Tools

| Tool | Access | Best For |
|---|---|---|
| **Lighthouse** | Chrome DevTools → Lighthouse tab | Complete audit, development iteration |
| **PageSpeed Insights** | pagespeed.web.dev | Real user data (CrUX) + lab data |
| **WebPageTest** | webpagetest.org | Detailed waterfall, multi-location testing |
| **Chrome DevTools Performance tab** | F12 → Performance | Profiling runtime jank and long tasks |
| **Core Web Vitals report** | Google Search Console | Real user data at scale across your site |

Run Lighthouse in an incognito window to avoid extension interference. Run PageSpeed Insights on your production URL for data that includes real user measurements from the Chrome User Experience Report (CrUX).

---

## FAQ

**What is a good LCP score?**
Under 2.5 seconds for the 75th percentile of your users (meaning 75% of visits experience LCP under that threshold). Google scores pages as "Good," "Needs Improvement," or "Poor" — the Good threshold is the target. Focus first on your LCP element: it is almost always a hero image, a large heading, or a video thumbnail.

**Do JavaScript frameworks hurt performance?**
They can, but it is usually implementation quality rather than the framework itself. React, Vue, Angular, and Svelte all have production sites with excellent Core Web Vitals. The risks specific to frameworks: large initial bundle size, client-side rendering delaying LCP, and hydration causing INP spikes. Server-side rendering or static generation eliminates the LCP delay. Techniques 5, 6, and 13 address the rest.

**Should I prioritize mobile or desktop performance?**
Mobile. Google uses mobile-first indexing, meaning your mobile performance score is what affects search ranking. Mobile devices have slower CPUs, slower networks, and less memory than desktops. Achieving good Core Web Vitals on mobile typically ensures excellent desktop performance automatically. Test on a real mid-range Android device (or Lighthouse's mobile simulation) rather than your development machine.

---

## Conclusion

Performance optimization has a natural order of returns: image optimization delivers disproportionate gains for most sites because images dominate page weight. Network optimization (CDN, caching, HTTP/2) is often free with modern infrastructure. Runtime optimization matters most for interactive applications.

Start by running Lighthouse on your current production site. Look at the opportunities section — it prioritizes recommendations by estimated time savings and is almost always correct about where to focus first. Address the biggest opportunities, deploy, measure again, and repeat.

The 15 techniques in this guide are not exotic; they are industry standard. Most can be implemented in a day. The compounding effect of several improvements working together — modern image formats + lazy loading + CDN + Brotli — routinely cuts page weight by 60% and load time in half.
