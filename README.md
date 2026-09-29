# Mariusz's Agent Skills

A personal collection of [agent skills](https://www.npmjs.com/package/skills) — modular instruction sets that teach AI coding assistants (OpenCode, Claude Code, Codex, Cursor, …) how to handle specific tasks the way I want them handled.

Each top-level directory is a single skill, with a `SKILL.md` describing when to trigger it and what to do.

## Installation

Install everything in this repo into all detected agents:

```bash
npx skills add mariusz/skills
```

Install globally (available across all projects):

```bash
npx skills add mariusz/skills -g
```

Install a single skill only:

```bash
npx skills add mariusz/skills --skill mariusz-git-worktrees-setup -g
```

Target a specific agent:

```bash
npx skills add mariusz/skills -a opencode -g
```

Non-interactive (CI / scripts):

```bash
npx skills add mariusz/skills -g -y --all
```

> Requires Node.js. The `skills` CLI auto-detects which agents you have installed and places skills in the correct directory for each one.

## Skills

| Skill | Description |
|-------|-------------|
| [`mariusz-git-worktrees-setup`](./mariusz-git-worktrees-setup/SKILL.md) | Bare-repo + git worktrees workflow for cloning any repository, plus auto-generated `AGENTS.md` documenting the layout. |
| [`pixel-perfect`](./pixel-perfect/SKILL.md) | Audits a rendered page or component against Figma or a supplied design spec, then writes a verified diff table and JSON handoff. |
| [`react-code-smells-review`](./react-code-smells-review/SKILL.md) | Review React components for maintainability anti-patterns — force-updates, direct DOM manipulation, props in state, uncontrolled inputs, prop drilling, inheritance, duplicated/oversized components, and low cohesion. |
| [`json-response`](./json-response/SKILL.md) | Serialize audit/verification findings into a structured JSON blob (`issues.json`) ready for an implementer agent loop. |

## Adding a new skill

1. Create a new directory at the repo root: `<skill-name>/`
2. Add a `SKILL.md` with frontmatter (`name`, `description`) and the instructions.
3. Commit and push.
4. Users (including future-you on a new machine) get it via `npx skills add mariusz/skills`.

See the [`skills` package docs](https://github.com/vercel-labs/skills) for the SKILL.md format and frontmatter conventions.

## License

Personal use, no warranty. Fork freely.
