# Agent skills

This project highlights how to use skills in a `reusable` and
`composable` manner. The project uses orchestration skills to manage
sub-skills to run automated workflows.

**`NOTE`:** Skills work in both `Gemini CLI` and `Claude Code` and any
other agent environment that supports the `Agent Skills Protocol`.

---

## Prerequisites

### Enable skills in your configuration files

Locate your configuration file at (`~/.gemini/settings.json`):

```json
{
  "general": {
    "previewFeatures": true
  },
  "experimental": {
    "skills": true
  }
}
```

### Install required tools for markdown skills

Install and configure `prettier` and `vale` to use the `fix-markdown`
and `update-changelog` skills. If unsure, configure `vale` with the
`Google Style Guide` for it.

The Git skills require `git` installed on your machine.

---

## Using commands to invoke skills

Prefer `skills` over `commands`. Skills offer cross-platform, portable,
`reusable`, flexible, token-efficient, context-preserving, and
`composable` functionality.

Use commands as proxies to launch orchestration skills. Notice how the
custom commands in this project invoke skills directly.

---

## Designing `composable` skills

To prevent `context rot` and encourage skill composition, design
`reusable` skills that focus on a single responsibility.

When possible, use an orchestrator skill to manage sub-skills, enabling
complex workflows from straightforward, `reusable` skills.

### Examples

#### Git commit workflow

The `git-commit` skill orchestrates atomic commits:

```bash
git-commit (orchestrator)
  ├─► git-add       # Stages atomic changes
  ├─► git-message   # Generates conventional commit message
  └─► git-status    # Displays repository status
```

Repeats until the repository remains clean. Each sub-skill offers
`reusability` and focuses on a single responsibility.

**See:** [`skills/git-commit/SKILL.md`](skills/git-commit/SKILL.md)

#### Update changelog workflow

The `update-changelog` skill manages `CHANGELOG.md` generation:

```bash
update-changelog (orchestrator)
  ├─► init-changelog    # Creates CHANGELOG.md if absent (script: init-changelog.sh)
  ├─► edit-changelog    # Adds entries from git commits (script: edit-changelog.sh)
  └─► cleanup-changelog # Formats and cleans (invokes fix-markdown skill)
```

Uses conditional logic based on script status codes (`SUCCESS`, `WARN`,
`ERROR`) and state management via `.last-aggregated-commit` to track
processed commits.

**See:**
[`skills/update-changelog/SKILL.md`](skills/update-changelog/SKILL.md)

### Key principles

- **Single responsibility** - Each skill performs one task well
- **`Composability`** - `Orchestrators` coordinate sub-skills
- **Scripts for complexity** - Use scripts when prompt engineering lacks
  efficiency
- **References for depth** - Include supporting docs when needed

**Explore the existing skills** in `skills/` to see patterns in action.

---

## Resources

**Official documentation:**

- [`Gemini CLI Skills Documentation`](https://geminicli.com/docs/cli/skills/)
- [`Agent Skills Directory`](https://agentskills.io/home)
