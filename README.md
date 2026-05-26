# skills

A collection of [agent skills](https://www.skills.sh/) for AI coding agents (Claude Code, Cursor, Codex, and others).

## Installation

Install any skill in this collection with the [`skills` CLI](https://www.skills.sh/docs/cli):

```bash
npx skills add isaias-alt/skills
```

This downloads the skills and configures them for your agent. No npm package, no global install: the CLI reads this repository directly.

To install into a specific project, run the command from that project's root. To install globally, follow your agent's convention for global skills (for Claude Code, that is `~/.claude/skills/`).

## Skills in this collection

### socratic-duck

A confrontational variant of rubber-duck debugging. It interrogates you about a single technical decision to expose flaws, hidden assumptions, and unresolved trade-offs before any code is written, then writes a decision log to `decisions/` in your project root.

**This skill is deliberately confrontational.** It does not validate your plan; it tries to break it. See [`socratic-duck/README.md`](./socratic-duck/README.md) for the full behavior description and the warning before you install.

## Repository structure

Each skill lives in its own directory at the repository root, following the layout expected by the `skills` CLI:

```
skills/
├── README.md              (this file: describes the collection)
└── socratic-duck/
    ├── SKILL.md           (the skill itself)
    ├── README.md          (skill-specific docs and warnings)
    └── examples/          (sample output)
```

New skills are added as sibling directories over time.

## License

See [LICENSE](./LICENSE).
