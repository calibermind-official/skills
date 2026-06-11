# AGENTS.md

This repository contains reusable AI agent skills for CaliberMind.

## Repository structure

```
.claude/skills/          # Claude Code skills (one subdirectory per skill)
  └── <skill-name>/
      ├── SKILL.md       # Required — frontmatter + instructions
      └── references/    # Optional — supplementary docs, examples
```

## Skills

- **calibermind-analyst** — Answer B2B marketing and sales questions by querying
  CaliberMind's BigQuery data model via the CaliberMind MCP connector.

## Usage

Add this repository to your Claude Code project as a skill source. The skills
are auto-discovered from `.claude/skills/*/SKILL.md`.

## Conventions

- Each skill is self-contained in its own directory under `.claude/skills/`.
- `SKILL.md` frontmatter `description` must be ≤ 200 characters.
- Dense reference material goes in `references/`, not in the main `SKILL.md`.
- All SQL in skills targets Google BigQuery Standard SQL (read-only).
