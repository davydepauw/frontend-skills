# CSS Conventions

## Naming

### Casing
| Context | Convention | Example |
|---------|------------|---------|
| Plain CSS / BEM | `kebab-case` | `.site-header`, `.card__title` |
| CSS Modules | `camelCase` | `styles.cardTitle` |
| Custom properties | `--kebab-case` | `--color-primary`, `--space-4` |
| `@keyframes` | `camelCase` or `kebab-case` | `fadeIn`, `slide-up` |

Pick one and be consistent within a project. Never mix within a file.

### Semantic vs presentational names
Prefer **what it is** over **what it looks like**:

```css
/* Good — semantic */
--s-color-primary: #0050e6;
.alert--warning { }

/* Bad — presentational */
--s-color-blue: #0050e6;
.alert--yellow { }
```

### State prefixes
See `references/stateful-selectors.md` for the full guide on state naming, ARIA attribute selectors, and `:has()` patterns.

---

## File Organisation

Organise CSS following **atomic design principles** — see `references/atomic-design.md` for the full stage definitions. The folder structure mirrors the hierarchy directly:

```
styles/
├── themes/                    ← design tokens (see design-tokens.md)
│   ├── _primitives.css
│   ├── _semantic.css
│   ├── _component.css
│   └── index.css
├── base/                      ← reset, element styles (Atomic: Generic + Elements)
│   ├── _reset.css
│   └── _typography.css
├── atoms/                     ← smallest indivisible components
│   ├── _button.css
│   ├── _input.css
│   └── _icon.css
├── molecules/                 ← simple groups of atoms
│   ├── _form-field.css
│   └── _search-bar.css
├── organisms/                 ← complex UI sections
│   ├── _site-header.css
│   └── _product-card.css
├── templates/                 ← page-level layout skeletons, no visual styling
│   └── _page-layout.css
└── utilities/                 ← exception classes only: .sr-only, .full-width
    └── _utilities.css
```

The entry file imports in this order:
```css
/* global.css */
@layer reset, base, atoms, molecules, organisms, templates, utilities;

@import './themes/index.css';
@import './base/_reset.css';
@import './base/_typography.css';
@import './atoms/_button.css';
/* … */
@import './utilities/_utilities.css';
```

### Co-located styles (component frameworks)
When using a component framework (React, Vue, Svelte), co-locate the CSS file with the component but keep the atomic design stage reflected in the directory structure:

```
src/
├── components/
│   ├── atoms/
│   │   ├── Button/
│   │   │   ├── Button.tsx
│   │   │   └── Button.module.css
│   ├── molecules/
│   │   └── FormField/
│   │       ├── FormField.tsx
│   │       └── FormField.module.css
│   └── organisms/
│       └── SiteHeader/
│           ├── SiteHeader.tsx
│           └── SiteHeader.module.css
└── styles/
    ├── themes/
    ├── base/
    └── utilities/
```

### Entry point rules
- `@layer` declaration order must be the first thing in the entry file
- No component styles directly in the entry file — only `@import` statements
- Tokens are always imported first, before any layer that consumes them

---

## Comment Standards

### Public API (design tokens)
Document intent and usage for anything a consumer will reference:

```css
/**
 * Primary action button.
 * Use for the single most important action on a surface.
 * Avoid using more than once per page section.
 */
.button--primary { … }
```

### Implementation notes
Use inline comments only for non-obvious choices:

```css
.modal {
  /* isolation creates a new stacking context without affecting layout */
  isolation: isolate;
}
```

Never comment on what the code obviously does:

```css
/* Bad */
.button {
  color: red; /* set color to red */
}
```

---

## Formatting Rules

- **One property per line** — no shorthand cramming multiple declarations on one line.
- **Space after colon**: `color: red` not `color:red`.
- **Include one space** before the opening brace.
- **Trailing semicolons** on all declarations, including the last one.
- **Group CSS variables** in a class at the top
- **Blank line below CSS variables** and the rest of the rules.
- **Blank line between rule blocks**.
- **No blank lines inside a rule block** (use comment headers to separate logical groups within a large block instead).
- **Closing brace on its own line**.
- **Selector per line** when a rule has multiple selectors.
- **Lowercase all hex values**
- **Use shorthand hex values where available**.
- **Avoid specifying units for zero values**

```css
h1,
h2,
h3 {
  --heading-font-family: var(--s-heading-font-family);
  --heading-font-weight: var(--s-heading-font-weight);

  font-family: var(--heading-font-family);
  font-weight: var(--heading-font-weight);
  line-height: 1.2;
}
```

---

## Formatting — Prettier

CSS formatting is enforced by [Prettier](https://prettier.io/) with [`prettier-plugin-css-order`](https://github.com/Siilwyn/prettier-plugin-css-order) for automatic property ordering. Do not manually reorder properties — let the plugin handle it.

Project config (`.prettierrc` or `prettier.config.js`):

```json
{
  "files": ["*.css", "*.scss"],
  "options": {
    "tabWidth": 4,
    "useTabs": true,
    "plugins": ["prettier-plugin-css-order"],
    "cssDeclarationSorterOrder": "smacss",
    "trailingComma": "all",
    "semi": true,
    "singleQuote": false
  }
}
```

Key settings:
- **Tabs, width 4** — indentation
- **SMACSS order** — properties sorted by: box model → border → background → text → other
- **Semicolons on** — always
- **Double quotes** — for any string values

When generating CSS, write properties in any reasonable order — Prettier will reorder on save. Focus on correctness and completeness, not property sequence.

---

## Things to Avoid

| Anti-pattern | Why | Instead |
|--------------|-----|---------|
| `* { margin: 0 }` in components | Resets all descendants, breaks third-party | Target specific elements |
| `!important` in component CSS | Blocks future overrides | Lower specificity or use `@layer` |
| Magic numbers (`margin-block-start: 13px`) | Untracked, breaks on resize | Use spacing tokens |
| Colour literals in components | Hard to theme | Use `var(--s-color-*)` tokens |
| Utility classes for layout/spacing | Couples structure to presentation; scattered responsibility | Write component-scoped CSS |
| Empty rule blocks | Dead code | Remove them |
| Specifying units for zero values | Unnecessary bytes | Omit units when value is 0 |
| Vendor prefixes by hand | Maintenance burden | Use PostCSS Autoprefixer |
