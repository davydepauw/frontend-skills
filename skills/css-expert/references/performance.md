# CSS Performance

## Core Web Vitals Affected by CSS

| Metric | What CSS affects |
|--------|-----------------|
| **LCP** (Largest Contentful Paint) | Render-blocking stylesheets, large background images, font loading |
| **CLS** (Cumulative Layout Shift) | Missing `width`/`height` on images, late-loading fonts, dynamic content without reserved space |
| **INP** (Interaction to Next Paint) | Heavy `transition`/`animation` on main thread, layout-triggering properties in event handlers |

---

## Render-Blocking Stylesheets

Every `<link rel="stylesheet">` in `<head>` blocks rendering until it downloads and parses.

### Reduce blocking time
```html
<!-- Critical styles inline in <head> — never blocks rendering -->
<style>
  /* above-the-fold layout and typography only */
  body { margin: 0; font-family: system-ui; }
  .hero { min-block-size: 100svh; }
</style>

<!-- Non-critical styles loaded async -->
<link rel="stylesheet" href="/styles/non-critical.css" media="print" onload="this.media='all'">
<noscript><link rel="stylesheet" href="/styles/non-critical.css"></noscript>
```

### Critical CSS extraction
Tools for automating above-the-fold extraction:
- **Critters** (webpack/Vite plugin) — inlines critical CSS, loads rest async
- **critical** (npm) — CLI and Node API
- **Astro** — built-in critical CSS inlining per route

What belongs in critical CSS:
- Layout for above-the-fold content
- Typography for visible text
- Colour tokens used in the initial viewport
- Nothing from below the fold

---

## Font Loading

### `font-display` strategy

```css
@font-face {
  font-family: 'Brand';
  src: url('/fonts/brand.woff2') format('woff2');
  font-display: swap;          /* show fallback, swap when loaded */
  font-display: optional;      /* use font only if already cached */
  font-display: block;         /* invisible text until loaded (avoid) */
}
```

| Value | LCP | CLS | Use when |
|-------|-----|-----|----------|
| `swap` | Good | Risk | Brand fonts critical to design |
| `optional` | Best | None | Performance-first; font enhances but isn't required |
| `fallback` | Good | Minimal | Balanced — short block period, then swap |
| `block` | Bad | Risk | Never in production |

### Reduce CLS from font swap
Match the fallback font metrics to the web font using `size-adjust`, `ascent-override`, etc.:

```css
@font-face {
  font-family: 'Brand Fallback';
  src: local('Arial');
  size-adjust: 105%;
  ascent-override: 90%;
}
```

Tools: [Font Style Matcher](https://meowni.ca/font-style-matcher/), Next.js `next/font` (automatic), Astro `@astrojs/fonts`.

---

## CSS Containment

Tell the browser a subtree is independent, enabling optimisation of paint and layout.

```css
.card {
  contain: layout paint;
  /* layout: changes inside don't affect outside layout */
  /* paint: contents don't render outside the border box */
}

/* Full containment for off-screen components */
.sidebar {
  contain: strict;   /* = layout paint size style */
}
```

### `content-visibility`
Skip rendering of off-screen content entirely:

```css
.below-fold-section {
  content-visibility: auto;
  contain-intrinsic-size: 0 500px;  /* estimated height to prevent layout shift */
}
```

Browser skips layout and paint for off-screen sections. Re-renders on scroll. Large pages with many sections see 50–70% faster initial render.

---

## `will-change`

Hint to the browser to promote an element to its own compositor layer.

```css
/* Good — applied before animation starts, removed after */
.menu {
  will-change: transform;
}

/* Bad — always on; wastes GPU memory */
.every-card {
  will-change: transform;
}
```

### Rules
- Apply only to elements that **will** animate.
- Remove `will-change` after the animation ends via JS.
- Do not apply to more than a handful of elements at a time — each promoted layer consumes GPU memory.
- Never use `will-change: all`.

### Prefer `transform` and `opacity`
These properties animate on the compositor thread (no layout recalculation):

```css
/* Good — compositor only */
.modal { transform: translateY(100%); }
.modal.is-open { transform: translateY(0); }

/* Bad — triggers layout */
.modal { inset-block-end: 100%; }
.modal.is-open { inset-block-end: 0; }
```

Properties that trigger layout (avoid animating): `inline-size`, `block-size`, `inset-*`, `margin`, `padding`, `font-size`.

---

## Avoiding Layout Thrash

Layout thrash happens when JS reads layout properties (triggering a layout flush), then writes styles (invalidating layout), then reads again in a loop.

CSS can prevent the need for JS reads:

```css
/* CSS intrinsic sizing avoids JS height measurement */
.accordion__content {
  display: grid;
  grid-template-rows: 0fr;
  transition: grid-template-rows 300ms ease;
}

.accordion__content > div {
  overflow: hidden;
}

.accordion.is-open .accordion__content {
  grid-template-rows: 1fr;
}
```

---

## Bundle Splitting

### CSS Modules with lazy-loaded routes
When using a bundler (Vite, webpack, Next.js), CSS Modules are automatically split per component. CSS for lazily imported components is only loaded when the component mounts.

```js
// Lazy-loaded component — its CSS is only fetched when rendered
const HeavyChart = lazy(() => import('./HeavyChart'))
```

### Splitting global CSS
```html
<!-- Route-specific stylesheet -->
<link rel="stylesheet" href="/styles/dashboard.css" media="(min-width: 0)">
```

Or use PostCSS plugins (purgecss, Lightning CSS) to strip unused rules from the global bundle.

---

## Measurement

| Tool | What to measure |
|------|----------------|
| Chrome DevTools Coverage | Unused CSS bytes |
| Lighthouse | LCP, CLS, blocking resources |
| WebPageTest | Waterfall for stylesheet load order |
| `performance.mark()` + DevTools | Animation frame budget |
| `PerformanceObserver` (CLS) | Layout shift from font swap |

```js
// Observe CLS in production
new PerformanceObserver(list => {
  for (const entry of list.getEntries()) {
    if (!entry.hadRecentInput) console.log('CLS:', entry.value)
  }
}).observe({ type: 'layout-shift', buffered: true })
```
