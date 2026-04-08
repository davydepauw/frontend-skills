# Design Tokens

## Token Taxonomy (Four-Level Model)

```
--p-  Primitive tokens    Raw values, no semantic meaning
        ↓
--s-  Semantic tokens     Purpose-named, reference primitives
        ↓
--c-  Component tokens    Per-theme overrides, defined in themes/<theme>/_component.css (optional)
        ↓
      Local variables     Scoped inside the component selector, no prefix
```

### Primitives (`--p-`)
```css
:root {
  --p-blue-500: #0050e6;
  --p-blue-600: #003db3;
  --p-spacing-1: 0.25rem;
  --p-spacing-4: 1rem;
  --p-radius-md: 0.375rem;
}
```

Never use primitive tokens directly in components. They exist to feed semantic tokens.

### Semantic tokens (`--s-`)
```css
:root {
  --s-color-primary: var(--p-blue-500);
  --s-color-primary-hover: var(--p-blue-600);
  --s-color-text: #1a1a1a;
  --s-color-surface: #ffffff;
  --s-color-border: #e2e2e2;

  --s-component-padding: var(--p-spacing-4);
  --s-component-border-radius: var(--p-radius-md);
}
```

### Component tokens (`--c-`) — theme files
Component tokens are defined per theme, not in the component itself. They live in `styles/themes/_component.css` (single theme) or `styles/themes/<theme-name>/_component.css` (multiple themes):

```css
/* styles/themes/brand/_component.css */
:root {
  --c-card-padding: var(--p-spacing-6);
  --c-card-border-radius: var(--p-radius-md);
  --c-card-compact-padding: var(--p-spacing-2);
}
```

They are **optional**. When absent, the component falls back to semantic tokens automatically (see local variables below).

### Local variables — inside the component
Inside the component selector, declare local variables that reference the component token with a semantic fallback:

```css
.card {
  --card-padding: var(--c-card-padding, var(--s-component-padding));
  --card-border-radius: var(--c-card-border-radius, var(--s-component-border-radius));

  padding: var(--card-padding);
  border-radius: var(--card-border-radius);
}
```

Local variables have no prefix — not even an underscore. `--card-padding` is correct; `--_card-padding` is not. They are scoped to the component selector and serve three purposes:

Local variables must reference `--c-` or `--s-` tokens — never `--p-` primitives directly:

```css
/* Bad — local variable referencing a primitive */
.card {
  --card-padding: var(--p-spacing-4);
}

/* Good — local variable referencing component token with semantic fallback */
.card {
  --card-padding: var(--c-card-padding, var(--s-component-padding));
}
```
1. **Single fallback declaration** — the `--c-` → `--s-` chain is defined once, used many times
2. **Readable aliases** — `var(--card-padding)` is clearer than repeating the full fallback chain on each property
3. **Consumer override point** — a parent can override `--card-padding` on a specific instance

Modifiers follow the same pattern — always `--c-` with a `--s-` fallback, never a raw value:

```css
.card--compact {
  --card-padding: var(--c-card-compact-padding, var(--s-component-padding-small));
}
```

---

## Theme File Structure

All design tokens live under `styles/themes/`.

### Single theme
```
styles/themes/
├── _primitives.css    --p- tokens
├── _semantic.css      --s- tokens
├── _component.css     --c- tokens
└── index.css          @import of all three
```

### Multiple themes
```
styles/themes/
├── default/
│   ├── _primitives.css
│   ├── _semantic.css
│   ├── _component.css
│   └── index.css
└── brand/
    ├── _primitives.css
    ├── _semantic.css
    ├── _component.css
    └── index.css
```

### `index.css` per theme
```css
/* styles/themes/brand/index.css */
@import './_primitives.css';
@import './_semantic.css';
@import './_component.css';
```

Import order is load order — primitives must load before semantics, semantics before components, since each tier references the tier above.

### What goes where

| File | Contains | References |
|------|----------|------------|
| `_primitives.css` | Raw values on `:root` | Nothing |
| `_semantic.css` | Purpose-named tokens on `:root` | `--p-` tokens |
| `_component.css` | Per-component overrides on `:root` | `--p-` or `--s-` tokens |

> **The `--p-` boundary:** `--c-` tokens in theme files may reference `--p-` primitives directly. Local variables *inside a component selector* may not — they must go through `--c-` or `--s-` tokens. This keeps raw values out of component CSS while still allowing theme files to express exact spacing or colour primitives.

---

## Token Naming Patterns

### Colour tokens
```
--s-color-{role}
--s-color-{role}-{variant}
--s-color-{role}-{state}

Examples:
--s-color-primary
--s-color-primary-subtle     (lighter tint)
--s-color-primary-emphasis   (darker shade)
--s-color-danger
--s-color-danger-hover
--s-color-text
--s-color-text-secondary
--s-color-surface
--s-color-surface-raised
--s-color-border
--s-color-border-strong
```

### Spacing tokens
Use a numeric scale tied to a base unit:
```css
--p-spacing-1: 0.25rem;   /*  4px */
--p-spacing-2: 0.5rem;    /*  8px */
--p-spacing-3: 0.75rem;   /* 12px */
--p-spacing-4: 1rem;      /* 16px */
--p-spacing-6: 1.5rem;    /* 24px */
--p-spacing-8: 2rem;      /* 32px */
--p-spacing-12: 3rem;     /* 48px */
--p-spacing-16: 4rem;     /* 64px */
```

### Typography tokens
```css
--p-font-size-sm: 0.875rem;
--p-font-size-base: 1rem;
--p-font-size-lg: 1.125rem;
--p-font-size-xl: 1.25rem;
--p-font-size-2xl: 1.5rem;

--p-font-weight-normal: 400;
--p-font-weight-medium: 500;
--p-font-weight-bold: 700;

--p-line-height-tight: 1.2;
--p-line-height-base: 1.5;
--p-line-height-loose: 1.75;
```

---

## Theming

### Light / dark with `color-scheme`
```css
:root {
  color-scheme: light dark;

  --s-color-text: #1a1a1a;
  --s-color-surface: #ffffff;
}

@media (prefers-color-scheme: dark) {
  :root {
    --s-color-text: #f0f0f0;
    --s-color-surface: #1a1a1a;
  }
}
```

### User-toggled themes via `data-theme`
```css
:root,
[data-theme="light"] {
  --s-color-text: #1a1a1a;
  --s-color-surface: #ffffff;
}

[data-theme="dark"] {
  --s-color-text: #f0f0f0;
  --s-color-surface: #1a1a1a;
}
```

Set `data-theme` on `<html>` via JS when the user toggles. This overrides the OS preference.

### Cascade Layers + theming
```css
@layer theme.base, theme.dark, theme.user;

@layer theme.base {
  :root { --s-color-surface: #fff; }
}

@layer theme.dark {
  @media (prefers-color-scheme: dark) {
    :root { --s-color-surface: #1a1a1a; }
  }
}

@layer theme.user {
  [data-theme="dark"] { --s-color-surface: #1a1a1a; }
  [data-theme="light"] { --s-color-surface: #fff; }
}
```

---

## DTCG Format (Design Token Community Group)

If exporting tokens to JSON for tool interop (Figma Tokens, Style Dictionary):

```json
{
  "color": {
    "primary": {
      "$value": "#0050e6",
      "$type": "color",
      "$description": "Primary brand action colour"
    }
  },
  "space": {
    "4": {
      "$value": "1rem",
      "$type": "dimension"
    }
  }
}
```

Use `$value`, `$type`, and optionally `$description`. Tools like Style Dictionary consume this format and output CSS custom properties, JS objects, or native platform tokens.

---

## Style Dictionary Integration

Style Dictionary transforms token JSON → platform outputs.

Basic config:
```js
// style-dictionary.config.js
export default {
  source: ['tokens/**/*.json'],
  platforms: {
    css: {
      transformGroup: 'css',
      prefix: 'p',
      buildPath: 'src/styles/theme/',
      files: [{ destination: '_primitives.css', format: 'css/variables' }],
    },
  },
}
```

Run: `npx style-dictionary build`

---

## Pitfalls

| Problem | Cause | Fix |
|---------|-------|-----|
| Token not updating in dark mode | Semantic token not re-declared in dark theme block | Re-declare `--s-` tokens, not `--p-` tokens |
| Component ignores theme | Component uses `--p-` token directly in a property | Define a local variable with `--c-` → `--s-` fallback; use the local variable on properties |
| Theme override has no effect | `--c-` token defined in theme but component uses local variable that was already overridden inline | Check specificity; inline `--card-*` overrides win over theme `--c-*` tokens |
| Token name collision | Two tokens with the same name in different scopes | Local variables are scoped to the component selector — no global collision risk |
| Circular reference | Token A references token B which references token A | Flatten — `--p-` tokens may not reference `--s-` tokens |
