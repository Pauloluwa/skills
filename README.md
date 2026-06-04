# skills

[![skills.sh](https://skills.sh/b/Pauloluwa/skills)](https://skills.sh/Pauloluwa/skills)

A collection of agent skills, installable into any agent (Cursor, Claude Code, Codex, and more) with the open [`skills`](https://www.skills.sh/) CLI.

## Install

Install every skill in this repo and pick your agent interactively:

```bash
npx skills add Pauloluwa/skills
```

Install a single skill:

```bash
npx skills add Pauloluwa/skills --skill frontend-architecture
```

Useful flags: `-g/--global` (user dir instead of project), `--copy` (copy files instead of symlinking), `-y` (skip prompts), `--all` (all skills to all agents).

## Skills

### frontend-architecture

Framework- and tool-agnostic frontend architecture pattern engine. It scans the open workspace first, then applies one uniform, domain-centric structure where every domain lives in `modules/<domain>/` (service, hook, store, types, utils). The detected state/data tool changes file contents, not file location.

Highlights:

- Discovery-first: detects framework, router, state/data/i18n/test tools, language, and existing conventions before writing anything.
- Domain modules: all logic for a domain in one folder; no separate `state/`, `stores/`, or `features/` layer.
- Hooks layer: `hooks/consolidated/` for single-domain composites (`useConsolidated<Domain>`) and `hooks/utility/` for generic reusable hooks.
- Config vs constants: runtime wiring in `config/`, static values in `constants/`.
- Minimal-delta scaffolding and clarification questions for new/empty repos or whole-project refactors.

See [`skills/frontend-architecture/SKILL.md`](skills/frontend-architecture/SKILL.md) and [`skills/frontend-architecture/reference.md`](skills/frontend-architecture/reference.md).

## License

[MIT](LICENSE)
