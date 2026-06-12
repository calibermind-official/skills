# Agent Skills

## Quick Start

```bash
npx skills add calibermind-official/skills
```

Select the skills you need from the wizard. Skills are installed to your
agent's directory (e.g., `.claude/skills/` for Claude Code).

## Skills

| Skill                                                      | Description                                                                                                                                                                            |
| ---------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [calibermind-analyst](.claude/skills/calibermind-analyst/) | Answer B2B marketing and sales questions by querying CaliberMind's BigQuery data model. Covers attribution, pipeline, campaigns, ad ROI, leads, accounts, funnels, and buyer journeys. |

## Manual Installation

```bash
# Clone and copy the skill you need
git clone https://github.com/calibermind-official/skills.git
cp -r skills/.claude/skills/calibermind-analyst your-project/.claude/skills/

# Or use symlinks for multi-project setups
ln -s /path/to/skills/.claude/skills/calibermind-analyst /path/to/your-project/.claude/skills/calibermind-analyst
```

## Repository Structure

```
.claude/skills/                     # All skills live here
  └── calibermind-analyst/
      ├── SKILL.md                  # Skill instructions (frontmatter + markdown)
      └── references/
          └── sql-examples.md       # Worked BigQuery SQL patterns
```

Each skill is a self-contained directory with a `SKILL.md` entry point. The
`references/` folder holds supplementary material the agent loads on demand.

## Prerequisites

All skills require a connected
[CaliberMind MCP server](https://docs.calibermind.com/agent-cal/calibermind-mcp-server)
with an active, AI-enabled CaliberMind account and a user with appropriate permissions.

## License

This project is licensed under the [MIT License](LICENSE).
