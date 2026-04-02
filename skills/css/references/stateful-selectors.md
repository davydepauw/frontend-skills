# Stateful Selectors

## Prefer HTML Attributes Over Classes

Reach for semantic HTML attributes and pseudo-classes first. Only use class names for state when no native attribute or pseudo-class fits.

```html
<nav>
  <a href="/about" class="nav-link" aria-current="page">About</a>
  <a href="/blog" class="nav-link">Blog</a>
  <a href="/contact" class="nav-link">Contact Me</a>
</nav>
```

```css
.nav-link[aria-current="page"] {
  border-block-end: 3px solid var(--s-color-primary);
}
```

### Common semantic state hooks

| State | Prefer |
|-------|--------|
| Current page / step | `[aria-current="page"]`, `[aria-current="step"]` |
| Active tab | `[aria-selected="true"]` |
| Disabled field | `:disabled`, `[disabled]` |
| Invalid field | `:invalid`, `[aria-invalid="true"]` |
| Expanded accordion / menu | `[aria-expanded="true"]` |
| Pressed toggle button | `[aria-pressed="true"]` |
| ARIA tabs | `[role="tab"]` |

Using ARIA attributes as style hooks means your CSS and your accessibility semantics stay in sync — a single source of truth.

---

## JS-Driven State Classes

When no semantic attribute fits, use a class with a consistent prefix to distinguish state from BEM modifiers:

```css
.card__button.is-loading { }   /* transient UI state — JS toggles this */
.card__button--primary { }     /* design variation — defined at authoring time */
```

### Prefix conventions

| Prefix | Use for | Example |
|--------|---------|---------|
| `is-` | Transient or boolean UI state | `.is-loading`, `.is-open`, `.is-active` |
| `has-` | Content-dependent state | `.has-error`, `.has-notification` |
| `js-` | JS-only hooks, never styled directly | `.js-toggle-target` |

- `is-` and `has-` classes are styled in CSS.
- `js-` classes are selection hooks only — never attach visual styles to them.
- Never use these prefixes for design variations; those are BEM modifiers (`--primary`, `--compact`).

---

## `:has()` for Relational State

Style a parent or sibling based on its descendants' state, without JS:

```css
/* Form with any invalid field */
form:has(:invalid) .form__submit {
  opacity: 0.5;
  pointer-events: none;
}

/* Label when its input is focused */
.form-field:has(input:focus) .form-field__label {
  color: var(--s-color-primary);
}

/* Card in a loading state set via child attribute */
.card:has([aria-busy="true"]) {
  opacity: 0.6;
}
```

`:has()` is Widely Available — use directly without `@supports`.

---

## Pseudo-Classes for Native State

Prefer pseudo-classes over JS-toggled classes wherever the browser tracks the state natively:

```css
/* Disabled */
.button:disabled,
.button[disabled] {
  --button-bg: var(--c-button-bg-disabled, var(--s-color-surface-muted));
  cursor: not-allowed;
}

/* Invalid */
.input:invalid,
.input[aria-invalid="true"] {
  --input-border-color: var(--c-input-border-invalid, var(--s-color-danger));
}

/* Focus */
.input:focus-visible {
  --input-border-color: var(--c-input-border-focus, var(--s-color-primary));
  outline: 2px solid var(--s-color-primary);
  outline-offset: 2px;
}
```

---

## State and Component Tokens

Stateful variations should use component tokens, following the same `--c-` → `--s-` fallback pattern as the base component:

```css
.button {
  --button-bg: var(--c-button-bg, var(--s-color-primary));
  background: var(--button-bg);
}

.button:hover {
  --button-bg: var(--c-button-bg-hover, var(--s-color-primary-hover));
}

.button:disabled {
  --button-bg: var(--c-button-bg-disabled, var(--s-color-surface-muted));
}
```

Each state gets its own component token with a semantic fallback — never hardcode state colours directly on properties.
