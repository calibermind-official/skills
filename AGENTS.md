# AGENTS.md

This repository contains reusable AI agent skills for CaliberMind.

## Repository structure

```
.claude-plugin/
  └── plugin.json          # Plugin manifest (Claude Code)
.claude/skills/            # Canonical skills (one subdirectory per skill)
  └── <skill-name>/
      ├── SKILL.md         # Required — frontmatter + instructions
      └── references/      # Optional — supplementary docs, examples
skills/                    # Plugin skill discovery (symlinks → .claude/skills/)
  └── <skill-name>/
.mcp.json                  # Bundled MCP server config
```

## Skills

- **calibermind-analyst** — Answer B2B marketing and sales questions by querying
  CaliberMind's BigQuery data model via the CaliberMind MCP connector.

## Usage

Install via `npx skills add calibermind-official/skills`, or load as a Claude
Code plugin with `claude --plugin-dir /path/to/skills`. Skills are
auto-discovered from `.claude/skills/*/SKILL.md` (npx) or `skills/*/SKILL.md`
(plugin mode).

## Conventions

- Each skill is self-contained in its own directory under `.claude/skills/`.
- `SKILL.md` frontmatter `description` must be ≤ 200 characters.
- Dense reference material goes in `references/`, not in the main `SKILL.md`.
- All SQL in skills targets Google BigQuery Standard SQL (read-only).
