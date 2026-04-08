# Progressive Enhancement

## Core Principle

Write CSS that works in the oldest supported browser first, then layer improvements for browsers that support them. The baseline must be functional and usable — not merely "not broken".

Only use `@supports` for **Newly Available** features. Widely Available features have sufficient browser coverage to use directly — wrapping them in `@supports` adds noise without benefit. See `references/modern-css.md` for the support status of each feature.

```
Baseline CSS       → works everywhere
  ↓
Widely Available   → use directly, no @supports needed
  ↓
Newly Available    → wrap in @supports, provide a functional fallback
```

---

## Baseline-First Authoring

Start with the simplest layout that works universally. Add enhancements on top.

```css
/* Baseline: block layout, works in any browser */
.card-grid > * {
  margin-block-end: 1rem;
}

/* Enhancement: CSS Grid is widely available — use directly */
.card-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
  gap: 1rem;
}

.card-grid > * {
  margin-block-end: 0;
}
```

---

## `@supports` — Feature Queries

### Basic syntax
```css
@supports (property: value) {
  /* styles for when property is supported */
}

@supports not (property: value) {
  /* fallback for when property is not supported */
}

@supports (display: grid) and (gap: 1rem) {
  /* styles when both features are supported */
}
```

### When to use it
Only for Newly Available features where absence degrades the experience:

```css
/* interpolate-size — Newly Available */
@supports (interpolate-size: allow-keywords) {
  :root { interpolate-size: allow-keywords; }

  .accordion__content {
    block-size: 0;
    overflow: hidden;
    transition: block-size 300ms ease;
  }

  .accordion.is-open .accordion__content {
    block-size: auto;
  }
}

/* ::details-content — Newly Available */
@supports selector(::details-content) {
  details::details-content {
    block-size: 0;
    overflow: hidden;
    transition: block-size 300ms ease, content-visibility 300ms allow-discrete;
  }

  details[open]::details-content {
    block-size: auto;
  }
}

/* @scope — Newly Available */
@supports selector(@scope) {
  @scope (.card) to (.card__footer) {
    p { color: var(--s-color-text-secondary); }
  }
}

/* field-sizing — Newly Available */
@supports (field-sizing: content) {
  textarea {
    field-sizing: content;
    min-block-size: 3lh;
    max-block-size: 20lh;
  }
}
```

### `@supports` vs `@layer`
Use `@supports` to **detect capability**. Use `@layer` to **manage specificity and load order**. They solve different problems and compose well.

---

## `@layer` for Third-Party Enhancement

Wrap third-party CSS in an anonymous layer so its specificity cannot override your styles:

```css
@import url('https://cdn.example.com/component.css') layer;

/* Your styles in a named layer always win */
@layer atoms {
  .component { color: var(--s-color-text); }
}
```

---

## Common Features and Their Fallbacks

### CSS Grid, Subgrid, `:has()`, container queries, `color-mix()`
All Widely Available — use directly without `@supports`.

```css
/* Grid */
.layout {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
}

/* Subgrid */
.card {
  display: grid;
  grid-row: span 3;
  grid-template-rows: subgrid;
}

/* :has() */
form:has(:invalid) .submit {
  opacity: 0.5;
}

/* color-mix() */
.tint {
  background: color-mix(in oklch, var(--s-color-primary) 20%, transparent);
}
```

### `interpolate-size` → static show/hide fallback
```css
/* Fallback: instant open/close */
.accordion__content {
  display: none;
}

.accordion.is-open .accordion__content {
  display: block;
}

/* Enhancement: animated height */
@supports (interpolate-size: allow-keywords) {
  :root { interpolate-size: allow-keywords; }

  .accordion__content {
    display: block;
    block-size: 0;
    overflow: hidden;
    transition: block-size 300ms ease;
  }

  .accordion.is-open .accordion__content {
    block-size: auto;
  }
}
```

### `::details-content` → native `<details>` behaviour fallback
```css
/* Fallback: browser default open/close, no animation */

/* Enhancement: animated */
@supports selector(::details-content) {
  details::details-content {
    block-size: 0;
    overflow: hidden;
    transition: block-size 300ms ease, content-visibility 300ms allow-discrete;
  }

  details[open]::details-content {
    block-size: auto;
  }
}
```

### Custom properties → inline fallback value
```css
.button {
  background: var(--c-button-bg, var(--s-color-primary));
}
```

---

## What Not to Do

**Don't wrap Widely Available features in `@supports`**
`:has()`, `@container`, `color-mix()`, subgrid, and CSS nesting are in all modern browsers. The guard adds noise and a false impression that these features are fragile.

**Don't use `@supports` to target specific browsers**
`@supports` is about capability, not identity. Browser targeting belongs in bug reports, not production CSS.

**Don't skip the baseline**
A page with zero CSS layout that collapses to a single column of stacked blocks is a valid, accessible baseline. A blank white page is not.
