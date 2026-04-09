---
name: css-expert
description: "Opinionated design-system CSS: use when writing or reviewing plain CSS, CSS Modules, or legacy SCSS for codebases that use (or should use) BEM, atomic-design-style folders, strict --p- / --s- / --c- tokens, native CSS or PostCSS, and component-scoped styling—not utility-first or Tailwind-by-default workflows unless the user is migrating or explicitly adopting this stack. Covers @layer, :has(), container queries, subgrid, progressive enhancement, critical CSS, and accessible focus/motion/contrast. Use whenever the user asks for scalable component CSS, token theming, architecture (BEM, layers, atomic design), or modern native CSS in that style. If the project already centers Tailwind, CSS-in-JS, or another system, follow the existing codebase first; apply this methodology only where it matches or when the user clearly wants this approach."
user-invocable: true
paths:
  - "**/*.css"
  - "**/*.scss"
  - "**/*.module.css"
  - "**/*.module.scss"
allowed-tools:
  - Read
  - Edit
  - Write
---

# CSS Expert

This skill encodes an **opinionated** methodology (BEM, atomic folders, token tiers, native CSS on greenfield). The root [README.md](../../README.md) summarizes it. When the repo you are editing already follows different rules, **match the project** unless the user asks to adopt this stack.

## Quick Router — read only the file relevant to your task

| What you are doing | Read this file |
|--------------------|----------------|
| Structuring a codebase (BEM, CSS Modules, `@layer`, atomic design) | `references/architecture.md` |
| Styling UI state (ARIA attributes, `:has()`, `is-`/`has-` prefixes, state tokens) | `references/stateful-selectors.md` |
| Applying atomic design to a component system | `references/atomic-design.md` |
| Naming, formatting, and comment conventions | `references/conventions.md` |
| File and folder structure (where files live in the project) | `references/architecture.md` |
| CSS custom properties, token taxonomies, theme systems | `references/design-tokens.md` (TOC at top) |
| `:has()`, container queries, subgrid, `@scope`, nesting, `color-mix()` | `references/modern-css.md` (TOC at top) |
| Supporting browsers that lack feature support | `references/progressive-enhancement.md` |
| Critical CSS, render-blocking, paint metrics, bundle size | `references/performance.md` |
| Focus styles, reduced-motion, contrast, screen reader behaviour | `references/accessibility.md` |

**Load only the file you need. Never preload all.**

---

## Agent Guardrails

### Universal CSS traps

These are easy to get wrong in generated CSS regardless of methodology. Check them before shipping rules.

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

**6. Only use valid CSS properties**  
Never write properties that do not exist in CSS. ARIA attributes (`aria-hidden`, `aria-expanded`), HTML attributes (`disabled`, `hidden`), and JavaScript-only concepts are not CSS properties. If it isn't in the CSS specification, it does not belong in a ruleset.

### This skill's methodology (defer to the project when it differs)

Naming, tokens, atomic stages, Sass policy, BEM element rules, and “no bare elements inside components” are defined in the references—not repeated here (avoids duplicate context when you load both this file and a reference).

| Topic | Reference |
|-------|-----------|
| Tooling (native CSS / PostCSS vs Sass on greenfield), naming, abbreviations | `references/conventions.md` |
| `--p-` / `--s-` / `--c-`, local variables, theme file layout | `references/design-tokens.md` |
| Atoms vs molecules vs organisms (e.g. composite cards) | `references/atomic-design.md` |
| BEM, CSS Modules, layers, file layout, bare-element rule | `references/architecture.md` |

**If the codebase already uses another system** (Tailwind or other utilities, CSS-in-JS as primary, different class naming, or Sass as the team standard), follow that system for edits unless the user explicitly wants to migrate toward this skill's approach.

---

## Workflow

1. Read existing CSS files before writing or reviewing — understand which architecture (BEM, CSS Modules, or both) and token naming the project already uses
2. If the project does not match this skill's methodology, prioritize consistency with existing code
3. Identify which subtopic the task touches (use the router above)
4. Read the relevant reference file (or the sections linked in its TOC)
5. Check the universal guardrails for the feature being written
6. Apply the pattern and note any browser-support caveats from the reference
