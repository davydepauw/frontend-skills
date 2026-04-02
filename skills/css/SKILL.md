---
name: css
description: "Use when writing or reviewing CSS/SCSS/CSS Modules files, designing token systems, choosing architecture (BEM, Cascade Layers, atomic design), auditing performance (critical CSS, paint metrics), applying modern CSS (:has(), container queries, @layer, subgrid), ensuring accessible focus/motion/contrast, or establishing naming and file-organisation conventions. Provides expert-level CSS guidance and catches common anti-patterns."
user-invocable: true
paths:
  - "**/*.css"
  - "**/*.scss"
  - "**/*.module.css"
  - "**/*.module.scss"
allowed-tools:
  - Read
---

# CSS Expert

## Quick Router — read only the file relevant to your task

| What you are doing | Read this file |
|--------------------|----------------|
| Structuring a codebase (BEM, CSS Modules, `@layer`, atomic design) | `references/architecture.md` |
| Applying atomic design to a component system | `references/atomic-design.md` |
| Naming, formatting, and file-organisation conventions | `references/conventions.md` |
| CSS custom properties, token taxonomies, theme systems | `references/design-tokens.md` |
| `:has()`, container queries, subgrid, `@scope`, nesting, `color-mix()` | `references/modern-css.md` |
| Supporting browsers that lack feature support | `references/progressive-enhancement.md` |
| Critical CSS, render-blocking, paint metrics, bundle size | `references/performance.md` |
| Focus styles, reduced-motion, contrast, screen reader behaviour | `references/accessibility.md` |

**Load only the file you need. Never preload all.**

---

## Agent Guardrails

Check these before generating CSS:

**1. `@layer` order determines specificity — not source order**
Layers declared later in the `@layer` statement win regardless of where the rule appears in the file. Establish the full layer order at the entry point.

**2. CSS custom properties are not scoped by default**
Properties defined on `:root` are global. Use component-level scope (`.component { --token: … }`) to prevent token bleed across components.

**3. `container-type: inline-size` creates a new stacking context**
This can trap `position: fixed` descendants. Verify stacking behaviour after adding container queries.

**4. `prefers-reduced-motion` should slow or simplify — not always remove**
Removing motion entirely can break comprehension (e.g., collapsed accordions with no visual cue). Reduce intensity; don't blindly set `animation: none`.

**5. High specificity blocks the cascade — check before reaching for `!important`**
Prefer lower-specificity selectors or cascade layers. `!important` is a smell — use component-scoped CSS with proper `@layer` ordering instead.

---

## Workflow

1. Identify which subtopic the task touches (use the router above)
2. Read the relevant reference file
3. Check the guardrails for the specific feature being written
4. Apply the pattern and note any browser-support caveats from the reference
