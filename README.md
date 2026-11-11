# claude-toolkit

Personal Claude Code skills, shared by every project as one plugin.

## Skills
- `/toolkit:audit`: careful cleanup audit. Independent reviewers (dead code, duplication,
  clarity), a skeptic pass that tries to disprove each finding, a report you approve, then small
  commits checked one at a time. Reads each project's `CLAUDE.md` for protected content and checks.
  Args: path(s) to limit scope, `quick`, `report`.

## Add to a project
Put this in the project's `.claude/settings.json` and commit it. Claude Code (local or cloud)
then installs the plugin when a session starts in that repo.

```json
{
  "extraKnownMarketplaces": {
    "claude-toolkit": { "source": { "source": "github", "repo": "anshuldaga/claude-toolkit" } }
  },
  "enabledPlugins": { "toolkit@claude-toolkit": true }
}
```

## Change a skill
Edit `plugins/toolkit/skills/<name>/SKILL.md`, bump `version` in
`plugins/toolkit/.claude-plugin/plugin.json`, commit and push. New sessions pick it up.

## Add a skill
Create `plugins/toolkit/skills/<new-name>/SKILL.md` (frontmatter `name` + `description`).
