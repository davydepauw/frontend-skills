# Progressive Enhancement

## Core Principle

Write CSS that works in the oldest supported browser first, then layer improvements for browsers that support them. The baseline must be functional and usable — not merely "not broken".

```
Baseline CSS → works everywhere
  ↓
@supports layer → works in supporting browsers
  ↓
@media layer → adapts to device context
  ↓
@layer enhancement → highest capability browsers
```

---

## Baseline-First Authoring

Start with the simplest layout that works universally. Add enhancements on top.

```css
/* Baseline: block layout, works in any browser */
.card-grid > * {
  margin-block-end: 1rem;
}

/* Enhancement: CSS Grid in supporting browsers */
@supports (display: grid) {
  .card-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
    gap: 1rem;
  }

  /* Remove baseline margin now that grid gap handles spacing */
  .card-grid > * {
    margin-block-end: 0;
  }
}
```

---

## `@supports` — Feature Queries

### Basic syntax
```css
@supports (property: value) {
  /* styles for when property is supported */
}

@supports not (display: grid) {
  /* fallback for when grid is not supported */
}

@supports (display: grid) and (gap: 1rem) {
  /* styles when both features are supported */
}
```

### Testing modern features
```css
/* Container queries */
@supports (container-type: inline-size) {
  .wrapper {
    container-type: inline-size;
  }
}

/* :has() */
@supports selector(:has(+ *)) {
  form:has(:invalid) .submit {
    opacity: 0.5;
  }
}

/* color-mix() */
@supports (background: color-mix(in oklch, red, blue)) {
  .tint {
    background: color-mix(in oklch, var(--color-primary) 20%, transparent);
  }
}

/* CSS nesting */
@supports (selector(&)) {
  .card {
    & .card__title { font-size: 1.25rem; }
  }
}
```

### `@supports` vs `@layer`
Use `@supports` to **detect capability**. Use `@layer` to **manage specificity and load order**. They solve different problems and compose well.

---

## Feature Query Patterns

### Opt-in enhancement
```css
/* Baseline */
.layout {
  display: flex;
  flex-wrap: wrap;
}

/* Enhancement */
@supports (container-type: inline-size) {
  .layout {
    display: block;
  }

  .layout__item {
    container-type: inline-size;
  }

  @container (min-width: 400px) {
    .card { display: grid; }
  }
}
```

### Opt-out fallback (graceful degradation direction)
```css
/* Modern baseline */
.grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
}

/* Fallback for unsupported browsers */
@supports not (display: grid) {
  .grid {
    display: flex;
    flex-wrap: wrap;
  }
  .grid > * {
    flex: 0 1 200px;
  }
}
```

Prefer opt-in over opt-out — it's easier to reason about and layers don't fight each other.

---

## `@layer` for Third-Party Enhancement

Wrap third-party CSS in an anonymous layer so its specificity cannot override your styles:

```css
@import url('https://cdn.example.com/component.css') layer;

/* Your styles in a named layer always win */
@layer components {
  .component { color: var(--color-text); }
}
```

---

## Common Features and Their Fallbacks

### CSS Grid → Flexbox
```css
.layout {
  display: flex;
  flex-wrap: wrap;
  gap: 1rem;
}

@supports (display: grid) {
  .layout {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
  }
}
```

### Subgrid → explicit row heights or min-height alignment
```css
/* Fallback: equal-height cards via align-items: stretch */
.grid {
  display: grid;
  align-items: stretch;
}

@supports (grid-template-rows: subgrid) {
  .card {
    display: grid;
    grid-row: span 3;
    grid-template-rows: subgrid;
  }
}
```

### `color-mix()` → static fallback colour
```css
.tint {
  background: rgba(0, 80, 230, 0.2); /* fallback */
}

@supports (background: color-mix(in oklch, red, blue)) {
  .tint {
    background: color-mix(in oklch, var(--color-primary) 20%, transparent);
  }
}
```

### `:has()` → class-based JS fallback
```css
/* JS adds .has-error when any input is invalid */
.form.has-error .submit { opacity: 0.5; }

@supports selector(:has(+ *)) {
  /* Override with native :has() where supported */
  .form:has(:invalid) .submit { opacity: 0.5; }
}
```

### Custom properties → static values
```css
.button {
  background: #0050e6; /* hardcoded fallback */
  background: var(--color-primary, #0050e6); /* custom property with fallback */
}
```

---

## What Not to Do

**Don't feature-detect layout as a binary**
Some browsers support grid but not subgrid, or support container queries but not `:has()`. Test for the specific feature you're using.

**Don't use `@supports` to target specific browsers**
`@supports` is about capability, not identity. Browser targeting belongs in bug reports, not production CSS.

**Don't skip the baseline**
A page with zero CSS layout that collapses to a single column of stacked blocks is a valid, accessible baseline. A blank white page is not.
