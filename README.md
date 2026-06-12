# Agent Skills

## Quick Start

```bash
npx skills add calibermind-official/skills
```

Select the skills you need from the wizard. Skills are installed to your
agent's directory (e.g., `.claude/skills/` for Claude Code).

## Claude Code Plugin

The Claude community marketplace listing is coming soon.

In the meantime, install by downloading the
[latest zip](https://github.com/calibermind-official/skills/archive/refs/heads/main.zip)
and uploading it in **Claude Code → Settings → Plugins → Upload Plugin**.

### Alternative Installation

For organizations with strict security requirements or air-gapped environments,
the plugin can also be installed via private GitHub sync or zip upload.

<details>
<summary>Install via private GitHub sync</summary>

1. Create a **new private repo** under your GitHub organization
2. Clone this repository and push it to your private repo:
   ```bash
   git clone https://github.com/calibermind-official/skills.git
   cd skills
   git remote add private git@github.com:your-org/your-private-repo.git
   git push private main
   ```
3. Open **Claude Code → Settings → Plugins**
4. Click **Sync from GitHub**
5. Select your private repo from the repository list
6. Click **Create**

</details>



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
.claude-plugin/
  ├── plugin.json                   # Plugin manifest (Claude Code)
  └── marketplace.json              # Marketplace catalog
.claude/skills/                     # Canonical skill source (npx skills add)
  └── calibermind-analyst/
      ├── SKILL.md                  # Skill instructions (frontmatter + markdown)
      └── references/
          └── sql-examples.md       # Worked BigQuery SQL patterns
skills/                             # Plugin skill discovery (copies)
  └── calibermind-analyst/
.mcp.json                           # Bundled MCP server config
```

Each skill is a self-contained directory with a `SKILL.md` entry point. The
`references/` folder holds supplementary material the agent loads on demand.

## Prerequisites

All skills require a connected
[CaliberMind MCP server](https://docs.calibermind.com/agent-cal/calibermind-mcp-server)
with an active, AI-enabled CaliberMind account and a user with appropriate permissions.

## License

This project is licensed under the [MIT License](LICENSE).
