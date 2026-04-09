# Modern CSS

**On this page:** [Browser support](#browser-support-overview) · [@supports policy](#when-to-use-supports) · [`:has()`](#has-relational-selector) · [Container queries](#container-queries) · [Subgrid](#subgrid) · [Nesting](#css-nesting) · [`@scope`](#scope) · [`color-mix()`](#color-mix) · [Logical properties](#logical-properties) · [`@starting-style`](#starting-style-entry-animations) · [`interpolate-size`](#interpolate-size-allow-keywords-animating-to-auto) · [`@property`](#property-registered-custom-properties) · [`::details-content`](#details-content-styling-details-content) · [`field-sizing`](#field-sizing-content)

## Browser Support Overview

Features below are **Baseline Widely Available** unless noted. Check [caniuse.com](https://caniuse.com) for exact versions.

| Feature | Baseline | `@supports` needed? |
|---------|----------|---------------------|
| `:has()` | Widely Available | No |
| Container queries (`@container`) | Widely Available | No |
| `@starting-style` | Widely Available | No |
| `@layer` | Widely Available | No |
| CSS nesting | Widely Available | No |
| Subgrid | Widely Available | No |
| `color-mix()` | Widely Available | No |
| `@property` | Widely Available | No — but use as enhancement |
| `@scope` | Newly Available | Yes — Chrome 118+, Firefox 128+, Safari 17.4+ |
| `field-sizing` | Newly Available | Yes — Chrome 123+, not Safari/Firefox yet |
| `interpolate-size` | Newly Available | Yes — Chrome 129+, not Safari/Firefox yet |
| `::details-content` | Newly Available | Yes — Chrome 131+, Firefox 131+ |
| Logical properties | Widely Available | No |

## When to use `@supports`

Use `@supports` only for **Newly Available** features where the absence of the feature would degrade the experience. Widely Available features have sufficient browser coverage to use directly.

```css
/* No @supports needed — widely available */
.card:has(img) { padding: 0; }

/* @supports needed — not yet in all browsers */
@supports (interpolate-size: allow-keywords) {
  :root { interpolate-size: allow-keywords; }
}
```

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
Newly available — wrap in `@supports selector(@scope)` for production use.

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

## `@starting-style` — Entry Animations

Animate elements when they first appear in the DOM or transition from `display: none`.

```css
.dialog {
  opacity: 1;
  translate: 0 0;
  transition: opacity 300ms ease, translate 300ms ease;
}

@starting-style {
  .dialog {
    opacity: 0;
    translate: 0 1rem;
  }
}
```

Also works with `display: none` transitions (requires `transition-behavior: allow-discrete`):

```css
.toast {
  display: block;
  opacity: 1;
  transition: opacity 200ms ease, display 200ms allow-discrete;
}

.toast.is-hidden {
  display: none;
  opacity: 0;
}

@starting-style {
  .toast {
    opacity: 0;
  }
}
```

Use directly — no `@supports` needed.

---

## `interpolate-size: allow-keywords` — Animating to `auto`

Enables transitions and animations to and from intrinsic size keywords (`auto`, `min-content`, `max-content`, `fit-content`).

```css
@supports (interpolate-size: allow-keywords) {
  :root {
    interpolate-size: allow-keywords;
  }

  .accordion__content {
    block-size: 0;
    overflow: hidden;
    transition: block-size 300ms ease;
  }

  .accordion.is-open .accordion__content {
    block-size: auto;
  }
}
```

Set on `:root` to enable globally. Without `@supports`, the transition simply won't animate — the element will still show/hide correctly, so no additional fallback is needed.

---

## `@property` — Registered Custom Properties

Register a custom property with a type, initial value, and inheritance behaviour. Enables animating custom properties.

```css
@property --card-highlight {
  syntax: '<color>';
  inherits: false;
  initial-value: transparent;
}

.card {
  --card-highlight: transparent;
  background: var(--card-highlight);
  transition: --card-highlight 300ms ease;
}

.card:hover {
  --card-highlight: color-mix(in oklch, var(--s-color-primary) 10%, transparent);
}
```

Widely available — use directly. Falls back gracefully: unregistered custom properties are not animatable, so the hover state will simply snap without a transition in unsupporting browsers.

---

## `::details-content` — Styling `<details>` Content

Style and animate the expandable content of a `<details>` element.

```css
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

Requires `interpolate-size: allow-keywords` on `:root` for the `block-size: auto` transition to animate. Without it, the content still shows/hides correctly but without animation.

---

## `field-sizing: content`

Auto-resize `<textarea>` and `<input>` to fit their content.

```css
@supports (field-sizing: content) {
  textarea {
    field-sizing: content;
    min-block-size: 3lh;
    max-block-size: 20lh;
  }
}
```

Use a JS resize observer as a fallback where needed.
