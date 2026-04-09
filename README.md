# Front-end skills

A collection of front-end agent skills compatible with any AI coding agent that supports the [Agent Skills open standard](https://agentskills.io).

## Installation & Updates

Re-running the same command installs or updates to the latest version.

All skills:
```
npx skills add davydepauw/frontend-skills --skill='*'
```

A specific skill:
```
npx skills add davydepauw/frontend-skills --skill='<skill-name>'
```

All skills globally (available across all projects):
```
npx skills add davydepauw/frontend-skills --skill='*' -g
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
| Install | ```npx skills add davydepauw/frontend-skills --skill='css-expert'``` |
| References | [SKILL.md](./skills/css-expert/SKILL.md) |
| Manual eval prompts | [evals/evals.json](./skills/css-expert/evals/evals.json) (for regression-style testing in agents that support it) |

**Cursor / editor metadata (optional):** [SKILL.md](./skills/css-expert/SKILL.md) may include extra YAML keys (`user-invocable`, `paths`, `allowed-tools`) for hosts that read them. The [Agent Skills](https://agentskills.io) open standard uses `name` and `description` in frontmatter; other keys are typically ignored by spec-only tooling and safe to leave for Cursor compatibility.
