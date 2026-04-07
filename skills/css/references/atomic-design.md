# Atomic Design

## The Five Stages

| Stage | Definition | CSS responsibility |
|-------|------------|-------------------|
| **Atoms** | Smallest indivisible UI elements | Intrinsic styles only: typography, colour, spacing on the element itself |
| **Molecules** | Simple groups of 2–3 atoms with a single, focused purpose | Relative layout between atoms (gap, flex); no external positioning |
| **Organisms** | Complex UI sections composed of multiple molecules or atoms | Section-level layout; may own a background, border, or padding |
| **Templates** | Page-level layout skeletons | Grid/flex containers, slot areas; no visual styling |
| **Pages** | Specific instances of templates with real content | Overrides for content-specific edge cases only |

---

## CSS at Each Stage

### Atoms
Atoms style themselves, never their surroundings. An atom should not know where it will be placed.

```css
/* Good — intrinsic */
.button {
  padding: 0.5em 1em;
  font: inherit;
  background: var(--color-primary);
  border-radius: var(--radius-sm);
}

/* Bad — atom controlling its own margin in context */
.button {
  margin-block-start: 1rem;   /* ← layout concern; belongs to parent */
}
```

### Molecules
Molecules own the **spacing between** their atoms, not the atoms themselves.

```css
.form-field {
  display: flex;
  flex-direction: column;
  gap: var(--space-1);   /* space between label and input atoms */
}
```

### Organisms
Organisms own their internal layout and may apply background/border/padding that visually frames them.

```css
.site-header {
  display: grid;
  grid-template-columns: auto 1fr auto;
  padding-inline: var(--space-4);
  background: var(--color-surface);
}
```

### Card: molecule or organism?

A card is almost always an **organism**. It typically composes multiple distinct molecules or atoms — media, a content block, a footer with actions — each with their own layout responsibility. That makes it complex enough to be an organism.

A card would only be a molecule if it contains just 2–3 atoms with a single purpose (e.g., an icon + label pairing). If it has sections — header, body, footer, or media — it is an organism.

```
.card              → organism
.card__media       → internal layout area (not a separate component)
.card__content     → internal layout area
.card__footer      → internal layout area
.card__title       → atom (heading)
.card__description → atom (text)
.card__image       → atom (image)
.button            → atom (reused inside .card__footer)
```

### Templates
Templates are layout-only. No colours, no borders, no visual treatment.

```css
.page-layout {
  display: grid;
  grid-template-areas:
    "header"
    "main"
    "footer";
  min-block-size: 100dvh;
}
```

### Pages
Pages contain real content instances. CSS at this level should be rare — only genuine content-specific overrides.

```css
/* Acceptable — this page has a unique hero */
.home-hero {
  background-image: url('/home-hero.avif');
}
```

---

## Mapping Atomic Design to CSS Patterns

### With BEM
- **Block** = Atom or Molecule
- **Organism** = BEM block with complex internal layout
- **Template** = layout-only block (`.page-grid`, `.sidebar-layout`)

### With CSS Modules
Each stage gets its own module file. The file name signals the stage:
```
button.module.css        → atom
form-field.module.css    → molecule
site-header.module.css   → organism
page-layout.module.css   → template
```

### With design tokens
Tokens belong at the atom level — atoms consume tokens directly. Molecules and organisms inherit through the cascade; they should rarely redeclare token values.

---

## Common Pitfalls

### Utility classes
Avoid utility classes for layout and spacing. Spacing, colour, and sizing decisions belong in the component's own CSS file, not scattered across markup. This keeps responsibility clear: the component owns its own appearance.

The only exceptions are `.sr-only` (accessibility) and grid spanning classes like `.full-width` — cases where the class must be applied from the outside and has no meaningful component to belong to.

### Atoms that break outside their context
If an atom only works in one specific organism, it isn't really an atom — it's a molecule element. Audit atoms by asking: "Can I drop this into any page and have it look correct?"

### Organisms that own page layout
An organism controls its internal layout, not its position on the page. If an organism is setting `position: fixed` or `margin: auto`, that responsibility likely belongs to a template.

### Skipping molecules
Jumping from atoms to organisms creates organisms with fragile internal layout. Build the molecule in between; it makes the organism easier to rearrange.

---

## When to Break the Rules

Atomic design is a mental model, not a strict enforcer. Valid exceptions:

- **Page-level one-offs**: A landing page with a unique hero section doesn't need to be an organism if it's never reused.
- **Theming**: A `data-theme` attribute on a template or organism is fine — theming is orthogonal to the hierarchy.
- **Third-party components**: Map them to the closest stage; don't force a third-party component through the full hierarchy.
