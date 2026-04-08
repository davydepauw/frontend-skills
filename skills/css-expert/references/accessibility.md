# CSS Accessibility

## Focus Styles

### Never remove focus outlines without a replacement
```css
/* Bad */
:focus { outline: none; }
*:focus { outline: 0; }

/* Good — custom outline that fits the design */
:focus-visible {
  outline: 2px solid var(--color-primary);
  outline-offset: 2px;
}
```

### `:focus` vs `:focus-visible`
| Selector | When it fires |
|----------|--------------|
| `:focus` | Any focus (mouse click, keyboard, programmatic) |
| `:focus-visible` | Only when the browser determines a visible indicator is needed (keyboard nav, programmatic focus on non-interactive elements) |

Use `:focus-visible` for custom outlines. Keep `:focus` only when you need to style all focus states (e.g., a text input that should always show its focus ring).

```css
/* Suppress mouse-click ring, keep keyboard ring */
.button:focus { outline: none; }
.button:focus-visible {
  outline: 2px solid var(--color-primary);
  outline-offset: 3px;
}
```

### High-contrast mode (Windows Forced Colors)
Custom outlines may disappear in Forced Colors mode. Use `outline` (not `box-shadow`) for focus styles — it's preserved.

```css
:focus-visible {
  outline: 2px solid var(--color-primary);
  /* box-shadow is ignored in Forced Colors mode */
}
```

---

## Motion and Animation

### `prefers-reduced-motion`
```css
@media (prefers-reduced-motion: reduce) {
  /* Slow down, don't necessarily remove */
  .animated {
    transition-duration: 0.01ms;
    animation-duration: 0.01ms;
    animation-iteration-count: 1;
  }
}
```

### The right approach: simplify, not eliminate
```css
.modal {
  transform: translateY(2rem);
  opacity: 0;
  transition:
    transform 300ms ease,
    opacity 300ms ease;
}

.modal.is-open {
  transform: translateY(0);
  opacity: 1;
}

@media (prefers-reduced-motion: reduce) {
  .modal {
    transform: none; /* Remove spatial movement */
    transition: opacity 150ms ease; /* Keep the fade — communicates state change */
  }
}
```

When to remove vs. reduce:
- **Remove**: decorative motion (parallax, background animations, auto-playing carousels)
- **Reduce**: functional motion that communicates state (modal open/close, accordion expand, toast appear)
- **Never remove**: motion that is the only indicator of state (a spinner; replace with a static indicator instead)

### `prefers-reduced-motion: no-preference`
Use this to only apply animations when the user has not requested reduced motion:

```css
@media (prefers-reduced-motion: no-preference) {
  .hero {
    animation: fadeIn 1s ease both;
  }
}
```

---

## Colour Contrast

### WCAG requirements
| Level | Normal text | Large text (18pt / 14pt bold) | UI components |
|-------|-------------|-------------------------------|---------------|
| AA | 4.5:1 | 3:1 | 3:1 |
| AAA | 7:1 | 4.5:1 | — |

### `color-contrast()` (future — limited support)
```css
.button {
  /* Pick the colour with highest contrast against the background */
  color: color-contrast(var(--color-primary) vs white, black);
}
```

Currently only behind flags. Use a design token system that enforces contrast at token creation time (e.g., in Figma, Style Dictionary with contrast checks).

### Contrast with `color-mix()` tokens
When generating tints, verify contrast doesn't drop below WCAG thresholds:
```css
/* Don't assume a tint has sufficient contrast */
--color-primary-subtle: color-mix(in oklch, var(--color-primary) 30%, white);
/* Check contrast of text on --color-primary-subtle before use */
```

---

## Visibility and Screen Readers

Different CSS properties have different effects on screen readers:

| Property | Visually hidden | From screen readers |
|----------|----------------|---------------------|
| `display: none` | Yes | Yes — completely removed |
| `visibility: hidden` | Yes | Yes — removed |
| `opacity: 0` | Yes | **No** — still announced |
| `clip-path: inset(100%)` | Yes | **No** — still announced |
| `.sr-only` pattern | Yes | **No** — still announced |

### `.sr-only` — visually hide, keep accessible
```css
.sr-only {
  position: absolute;
  inline-size: 1px;
  block-size: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border: 0;
}

/* Make focusable when needed (skip links, etc.) */
.sr-only:focus,
.sr-only:active {
  position: static;
  inline-size: auto;
  block-size: auto;
  overflow: visible;
  clip: auto;
  white-space: normal;
}
```

### Skip links
```css
.skip-link {
  position: absolute;
  transform: translateY(-100%);
  transition: transform 200ms ease;
}

.skip-link:focus {
  transform: translateY(0);
}
```

---

## Text and Spacing Accessibility

### Zoom to 200%
WCAG 1.4.4 requires no loss of content or functionality at 200% text size.

```css
/* Bad — fixed pixel font sizes break at 200% text zoom */
body { font-size: 16px; }

/* Good — respects user's browser font size setting */
body { font-size: 1rem; }
```

### Minimum tap target size
WCAG 2.5.5 (AAA) recommends 44×44px; WCAG 2.5.8 (AA, 2.2) requires 24×24px.

```css
.icon-button {
  min-inline-size: 44px;
  min-block-size: 44px;
  display: inline-flex;
  align-items: center;
  justify-content: center;
}
```

### Line spacing (WCAG 1.4.12)
Ensure the layout doesn't break when users apply these overrides:
- Line height: 1.5× font size
- Letter spacing: 0.12× font size
- Word spacing: 0.16× font size
- Paragraph spacing: 2× font size

Use relative units (`em`, `rem`, `lh`) for spacing, not `px`. Test by applying these values via a browser extension (e.g., Text Spacing Bookmarklet).

---

## `prefers-contrast`

```css
@media (prefers-contrast: more) {
  .card {
    border: 2px solid var(--color-text);
  }

  .button {
    outline: 2px solid currentColor;
  }
}

@media (prefers-contrast: less) {
  /* Reduce visual noise for users who find high contrast distracting */
  .divider {
    opacity: 0.3;
  }
}
```

---

## CSS That Affects ARIA

- `display: none` and `visibility: hidden` set `aria-hidden="true"` implicitly. Don't combine with an explicit `aria-hidden`.
- `content: "..."` in `::before`/`::after` **is** read by some screen readers. Never put meaningful text in `content`. For decorative icons, use `content: ""` or `aria-hidden="true"` on the element.
- `pointer-events: none` prevents mouse interaction but does **not** remove an element from tab order or screen reader access. Use `tabindex="-1"` and `aria-hidden` explicitly.

## Only Use Valid CSS Properties

Never write properties that do not exist in the CSS specification. HTML attributes, ARIA attributes, and JavaScript concepts are not CSS properties and will be silently ignored by the browser.

```css
/* Invalid — aria-hidden is an HTML attribute, not a CSS property */
.form-field__hint {
  aria-hidden: true;
}
```

To control ARIA state, set the attribute in HTML or JavaScript:
```html
<span aria-hidden="true">decorative text</span>
```
```js
element.setAttribute('aria-hidden', 'true')
```

ARIA attributes *can* be used as CSS **selectors** (e.g. `[aria-hidden="true"]`), but only where documented in `references/stateful-selectors.md` — as semantic style hooks, not as a substitute for proper visibility management.
