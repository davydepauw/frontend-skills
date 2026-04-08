# Front-end skills

A collection of front-end agent skills compatible with any AI coding agent that supports the [Agent Skills open standard](https://agentskills.io).

## Installation & Updates

Re-running the same command installs or updates to the latest version.

All skills:
```
pnpx skills add davydepauw/frontend-skills --skill='*'
```

A specific skill:
```
pnpx skills add davydepauw/frontend-skills --skill='<skill-name>'
```

All skills globally (available across all projects):
```
pnpx skills add davydepauw/frontend-skills --skill='*' -g
```

## Available skills

### `css-expert`

An opinionated, modern CSS skill for writing scalable, maintainable stylesheets — without Tailwind or Sass.

This skill enforces a structured approach built on:

- **Atomic design** — CSS organised into `atoms/`, `molecules/`, `organisms/`, `templates/`, and `utilities/`, mirroring the component hierarchy
- **Native CSS** — leveraging modern features directly: `@layer`, `:has()`, container queries, CSS nesting, `@property`, and logical properties. No preprocessors in new projects.
- **Design token system** — a strict three-tier token architecture (`--p-` primitives → `--s-` semantic → `--c-` component) with per-theme override files
- **BEM** — explicit, flat class naming with no arbitrary abbreviations and no bare element selectors inside components
- **No utility classes** — all styling is component-scoped; the only exceptions are `.sr-only` and grid spanning classes like `.full-width`
- **Progressive enhancement** — `@supports` guards only for Newly Available features; Widely Available features are used directly

| | Note |
|---|---|
| Install | ```pnpx skills add davydepauw/frontend-skills --skill='css-expert'``` |
| References | [SKILL.md](./skills/css-expert/SKILL.md) |
