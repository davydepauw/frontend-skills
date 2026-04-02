# Modern CSS

## Browser Support Overview

Features below are **Baseline Widely Available** unless noted. Check [caniuse.com](https://caniuse.com) for exact versions.

| Feature | Baseline | Notes |
|---------|----------|-------|
| `:has()` | Widely Available | No Firefox < 121 |
| Container queries (`@container`) | Widely Available | — |
| `@layer` | Widely Available | — |
| CSS nesting | Widely Available | — |
| Subgrid | Widely Available | — |
| `color-mix()` | Widely Available | — |
| `@scope` | Newly Available | Chrome 118+, Firefox 128+, Safari 17.4+ |
| `field-sizing` | Newly Available | Chrome 123+, not Safari/Firefox yet |
| Logical properties | Widely Available | — |

---

## `:has()` — Relational Selector

Select a parent or sibling based on its descendants.

```css
/* Card with an image gets different padding */
.card:has(img) {
  padding: 0;
}

/* Label after a required input */
input:required + label {
  color: var(--color-danger);
}

/* Form with any invalid field */
form:has(:invalid) .submit-button {
  opacity: 0.5;
  pointer-events: none;
}

/* Navigation item that contains the current page link */
.nav-item:has([aria-current="page"]) {
  background: var(--color-surface-raised);
}
```

### Limitations
- `:has()` cannot be used inside another `:has()`.
- `:has()` with complex pseudo-elements (`:has(::before)`) behaves inconsistently — avoid.
- Performance: avoid broad `:has()` selectors on frequently repainted elements.

---

## Container Queries

Style elements based on their **container's size**, not the viewport.

### Setup
```css
/* Establish a containment context */
.card-wrapper {
  container-type: inline-size;
  container-name: card;   /* optional — enables named queries */
}

/* Query the container */
@container card (min-width: 400px) {
  .card {
    display: grid;
    grid-template-columns: 1fr 2fr;
  }
}
```

### `container-type` values
| Value | What it tracks |
|-------|---------------|
| `inline-size` | Width only (most common) |
| `size` | Width and height (creates block formatting context) |
| `normal` | Default; enables style queries only, no size queries |

### Style queries
```css
@container style(--variant: featured) {
  .card__title {
    font-size: var(--font-size-xl);
  }
}
```

### Pitfall: stacking context
`container-type: inline-size` and `container-type: size` both create a new stacking context. `position: fixed` inside will be clipped. Use a wrapper outside the container for fixed-position children.

---

## Subgrid

Allow a nested grid to adopt its parent's track definitions.

```css
.grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  grid-template-rows: auto 1fr auto;
  gap: 1rem;
}

.card {
  display: grid;
  grid-column: span 1;
  grid-row: span 3;

  /* adopt parent's row tracks */
  grid-template-rows: subgrid;
}

/* Now card's children align to the parent grid's row lines */
.card__header { grid-row: 1; }
.card__body   { grid-row: 2; }
.card__footer { grid-row: 3; }
```

Use case: cards in a grid where header, body, and footer must align across cards regardless of content height.

---

## CSS Nesting

Native nesting without a preprocessor.

```css
.card {
  padding: var(--space-4);

  & .card__title {
    font-size: var(--font-size-lg);
  }

  &:hover {
    background: var(--color-surface-raised);
  }

  @media (min-width: 640px) {
    display: grid;
  }
}
```

### Rules
- The `&` refers to the parent selector. It is **required** when the nested selector starts with a letter (to avoid ambiguity with element names): `& p {}` not `p {}`.
- `@media` inside a rule automatically inherits the parent selector.
- Nesting depth: keep to 2–3 levels max; deep nesting makes specificity hard to reason about.

---

## `@scope`

Limit styles to a subtree without increasing specificity.

```css
@scope (.card) to (.card__footer) {
  /* Applies to .card and descendants, but stops at .card__footer */
  p {
    color: var(--color-text-secondary);
  }
}
```

### Use cases
- Scoping third-party content without high-specificity selectors.
- Donut scopes: "style everything in `.card` except inside `.ad-slot`".

### Current support
Newly available — use `@supports` or progressive enhancement for production (see `progressive-enhancement.md`).

---

## `color-mix()`

Mix two colours in a given colour space.

```css
.button--subtle {
  background: color-mix(in oklch, var(--color-primary) 20%, transparent);
}

.divider {
  border-color: color-mix(in srgb, var(--color-text) 15%, transparent);
}
```

### Colour spaces
- `oklch` — perceptually uniform; best for creating harmonious tints/shades.
- `srgb` — standard; matches what you'd expect from CSS `rgba()`.
- `hsl`, `hwb`, `lab`, `lch`, `oklab` — all supported.

---

## Logical Properties

Write direction-agnostic CSS that works in LTR and RTL layouts.

| Physical | Logical |
|----------|---------|
| `margin-left` / `margin-right` | `margin-inline-start` / `margin-inline-end` |
| `margin-top` / `margin-bottom` | `margin-block-start` / `margin-block-end` |
| `padding-left` | `padding-inline-start` |
| `width` | `inline-size` |
| `height` | `block-size` |
| `top` / `bottom` / `left` / `right` | `inset-block-start` / `inset-block-end` / `inset-inline-start` / `inset-inline-end` |
| `inset: 0` (shorthand) | `inset: 0` (already logical) |

```css
/* Before */
.button {
  padding-left: 1rem;
  padding-right: 1rem;
  border-left: 3px solid var(--color-primary);
}

/* After — works in RTL too */
.button {
  padding-inline: 1rem;
  border-inline-start: 3px solid var(--color-primary);
}
```

---

## `field-sizing: content`

Auto-resize `<textarea>` and `<input>` to fit their content.

```css
textarea {
  field-sizing: content;
  min-height: 3lh;   /* minimum 3 line heights */
  max-height: 20lh;
}
```

Currently Chrome 123+ only. Use a JS resize observer as a fallback.
