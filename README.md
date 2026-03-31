# Front-end skills

A collection of front-end agent skills for Claude Code.

## Installation

Install all skills into the current project:
```
pnpx skills add davydepauw/frontend-skills --skill='*'
```

Install a specific skill:
```
pnpx skills add davydepauw/frontend-skills --skill='<skill-name>'
```

Install all skills globally (available across all projects):
```
pnpx skills add davydepauw/frontend-skills --skill='*' -g
```

## Skills

| Skill | Description |
|-------|-------------|
| [example](./skills/example/SKILL.md) | Example skill — replace with your own |

## Adding a new skill

1. Create a directory under `skills/` with your skill name (lowercase, hyphens only)
2. Add a `SKILL.md` file with the required frontmatter:

```markdown
---
name: your-skill-name
description: What this skill does and when Claude should use it
user-invocable: true
---

# Your Skill Title

Instructions for Claude...
```

3. Add an entry to the Skills table in this README
4. Bump the version in `package.json`

## Versioning

Releases are automated via [semantic-release](https://semantic-release.gitbook.io/) on every push to `main`. Versions are derived from [conventional commits](https://www.conventionalcommits.org/):

| Commit prefix | Release type |
|---------------|-------------|
| `fix:` | patch (`0.0.x`) |
| `feat:` | minor (`0.x.0`) |
| `feat!:` or `BREAKING CHANGE:` | major (`x.0.0`) |

To preview the next release locally:
```
pnpm run release:dry
```
