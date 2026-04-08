# CSS Architecture

## Choosing an Architecture

| Approach | Best for | Avoid when |
|----------|----------|------------|
| BEM | Any scale; great for teams | Utility-heavy designs (too verbose) |
| CSS Modules | Component-based frameworks (React, Vue, Svelte) | Global stylesheets or SSR without a bundler |
| Cascade Layers (`@layer`) | Any project wanting explicit specificity control | Needing IE/old-Edge support |

These approaches compose — BEM naming inside CSS Modules, atomic design stage names expressed as `@layer` declarations.

---

## BEM (Block Element Modifier)

### Naming contract
```css
/* Block */
.card { }

/* Element — double underscore */
.card__title { }
.card__image { }

/* Modifier — double hyphen */
.card--featured { }
.card__title--truncated { }
```

### Rules
- **One block per component.** The block name is the component's identity.
- **Elements belong to the block, not to each other.** `.card__title__icon` is wrong; use `.card__icon`.
- **Modifiers describe state or variation**, never layout. Layout is the parent's responsibility.
- **Avoid deeply nested selectors.** `.card .card__title` is redundant; `.card__title` is sufficient.
- **Never style bare HTML elements inside a component.** Every meaningful child gets a BEM element class. Style `.card__image`, not `img` nested inside `.card__media`.

```css
/* Bad — bare element selector inside a component */
.card__media {
  img {
    width: 100%;
    object-fit: cover;
  }
}

/* Good — explicit BEM element class */
.card__image {
  width: 100%;
  object-fit: cover;
}
```

### State classes
See `references/stateful-selectors.md` for the full guide on ARIA attributes, pseudo-classes, `is-`/`has-` prefixes, and state component tokens.

---

## CSS Modules

### What it does
CSS Modules scope class names locally by default. The bundler (Vite, webpack, Next.js) transforms `.card` into `.card_abc123` at build time.

### Naming inside CSS Modules
Use camelCase or PascalCase — it maps to JS property access:
```css
/* card.module.css */
.cardWrapper { }
.cardTitle { }
.cardTitle--featured { }   /* BEM modifiers still work */
```
```js
import styles from './card.module.css'
element.className = styles.cardWrapper
```

### `composes` for sharing
```css
.base { color: var(--color-text); }

.heading {
  composes: base;
  font-size: 1.5rem;
}
```

### Global escape hatch
```css
:global(.third-party-class) { … }
```

### Limitations
- No cascade across module files by design — this is a feature, not a bug.
- Animations and `@keyframes` names should be declared locally or use `:global`.

---

## Cascade Layers (`@layer`)

### Declaration order sets specificity
```css
/* Declare all layers upfront — order matters */
@layer reset, base, atoms, molecules, organisms, templates, utilities;
```
Later-declared layers win specificity battles regardless of selector weight.

### Layer anatomy
```css
@layer reset {
  * { margin: 0; padding: 0; }
}

@layer atoms {
  .button { background: var(--c-button-bg, var(--s-color-primary)); }
}

@layer utilities {
  /* Exception-only: .sr-only and grid spanning classes (.full-width) */
  .sr-only { position: absolute; inline-size: 1px; block-size: 1px; /* … */ }
  .full-width { grid-column: 1 / -1; }
}
```

### Anonymous layers
```css
@import url('third-party.css') layer;  /* anonymous layer, lowest priority */
```
Wrap third-party CSS in an anonymous layer to prevent its styles from leaking into your specificity budget.

### Unlayered styles
Styles outside any `@layer` have **higher specificity than all layers**. Be deliberate: either put everything in layers, or reserve unlayered styles as the intentional escape hatch.

---

## Mixing Approaches

A common production combination:

```
@layer reset, tokens, base, atoms, molecules, organisms, templates, utilities;

reset      → normalize / modern reset
tokens     → @layer with :root custom properties
base       → element styles (typography, links)
atoms      → smallest indivisible components
molecules  → simple groups of atoms
organisms  → complex UI sections
templates  → page-level layout skeletons
utilities  → exception classes only: .sr-only, grid spanning (.full-width)
```

If using CSS Modules: each `.module.css` file contributes to the `atoms`/`molecules`/`organisms` layer via a global layer declaration in your entry CSS.
