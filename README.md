# CaliberMind Skills for Agents

## Quick Start

```bash
npx skills add calibermind-official/skills
```

Select the skills you need from the wizard. Skills are installed to your
agent's directory (e.g., `.claude/skills/` for Claude Code).

## Claude.ai

Install the CaliberMind plugin in the Claude web and desktop apps. It adds both skills and
offers the CaliberMind MCP server as a connector.

- **For yourself:** go to **Customize → Plugins → Add → Add marketplace**, enter
  `https://github.com/calibermind-official/skills`, then install **calibermind**.
- **For your whole organization:** an Owner adds the same repo under
  **Organization settings → Plugins & skills → Marketplaces** and sets **calibermind** to
  **Available to install**, **Installed by default**, or **Required**.

When adding the repo URL, turn on **Sync automatically** so new versions are picked up whenever
this repo changes.

You can also download the
[latest zip](https://github.com/calibermind-official/skills/archive/refs/heads/main.zip)
and upload it in **Customize → Plugins → Add → Upload plugin**. Uploaded plugins don't sync, so
upload a new zip to update.

After installing, open the **CaliberMind** connector and click **Connect** to sign in.

### Upload individual skills

To upload skills without the plugin, zip each skill folder on its own:

```bash
git clone https://github.com/calibermind-official/skills.git
cd skills/skills
zip -r <skill-name>.zip <skill-name>
```

- **For yourself:** in Claude, go to **Customize → Skills**, click **+**, then
  **Create skill → Upload a skill**, and choose the zip.
- **For your whole organization:** an Owner uploads the zip in
  **Organization settings → Plugins & skills**. It then appears for every member.

If **Skills** is missing from Customize, an Owner needs to enable **Code execution and file
creation** and **Skills** under **Organization settings → Plugins & skills → Policy**.

## Claude Code Plugin

Add the CaliberMind marketplace and install the plugin:

```
/plugin marketplace add calibermind-official/skills
/plugin install calibermind@calibermind-skills
```

Run `/plugin marketplace update calibermind-skills` to pick up new versions.

You can also download the
[latest zip](https://github.com/calibermind-official/skills/archive/refs/heads/main.zip)
and upload it in **Claude Code → Settings → Plugins → Upload Plugin**.

### Alternative Installation

For organizations with strict security requirements or air-gapped environments,
the plugin can also be installed via private GitHub sync.

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

## Manual Installation

```bash
# Clone and copy the skill you need (replace <skill-name>, e.g. opportunity-buyer-chart)
git clone https://github.com/calibermind-official/skills.git
cp -r skills/.claude/skills/<skill-name> your-project/.claude/skills/

# Or use symlinks for multi-project setups (updates arrive with git pull)
ln -s /path/to/skills/.claude/skills/<skill-name> /path/to/your-project/.claude/skills/<skill-name>
```

To make a skill available in every project, copy or link it into `~/.claude/skills/` instead.


## Updating

Installed skills do not update on their own. To get new or changed skills:

| Installed with | How to update |
|---|---|
| `npx skills add` | Run `npx skills add calibermind-official/skills` again and select the skills you want. |
| Plugin zip upload (Claude Code or Claude.ai) | Download the [latest zip](https://github.com/calibermind-official/skills/archive/refs/heads/main.zip) and upload it again. |
| Private GitHub sync | Pull from this repo and push to your private repo: `git pull origin main && git push private main`. |
| Manual copy | `git pull` in your clone, then copy the skill folder again. |
| Manual symlink | `git pull` in your clone. |
| Claude.ai marketplace | Automatic if **Sync automatically** was turned on when adding the marketplace. Otherwise, sync the marketplace manually from the same settings page. |
| Claude.ai skill upload | Build a new zip and upload it again, replacing the old skill. An Owner does this for organization-provisioned skills. |

## Skills

| Skill                                                      | Description                                                                                                                                                                            |
| ---------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [calibermind-analyst](.claude/skills/calibermind-analyst/) | Answer B2B marketing and sales questions by querying CaliberMind's BigQuery data model. Covers attribution, pipeline, campaigns, ad ROI, leads, accounts, funnels, and buyer journeys. |
| [opportunity-buyer-chart](.claude/skills/opportunity-buyer-chart/) | Build an Opportunity Buyer Chart for one opportunity: buying group, marketing and sales touches, intent, milestones, and key moments. |

## Repository Structure

```
.claude-plugin/
  ├── plugin.json                   # Plugin manifest (Claude Code)
  └── marketplace.json              # Marketplace catalog
.claude/skills/                     # Canonical skill source (npx skills add)
  ├── calibermind-analyst/
  │   ├── SKILL.md                  # Skill instructions (frontmatter + markdown)
  │   └── references/
  │       └── sql-examples.md       # Worked BigQuery SQL patterns
  └── opportunity-buyer-chart/
      ├── SKILL.md
      └── references/               # Queries, classification, content, chart spec, theme
skills/                             # Plugin skill discovery (copies)
  ├── calibermind-analyst/
  └── opportunity-buyer-chart/
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
