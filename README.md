# Agent skills

This project demonstrates a standardized approach to building `reusable`
and `composable` agent skills. All skills follow the **Agent Skills
Protocol** and adhere to the patterns defined in
`docs/skill-reference-guide.md`.

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

### Install skills

Locate your configuration folder. For `Gemini CLI`, use `~/.gemini`. For
`Claude Code`, use `~/.claude`.

Then place the `skills` folder from this project in your configuration
folder.

### Optionally, install custom commands

The custom commands in this project only work for `Gemini CLI`. Place
the `commands` folder in your configuration folder for `Gemini CLI`. In
`Claude Code`, skills behave just like commands, eliminating the need to
install custom commands.

---

## Using commands to invoke skills

Prefer `skills` over `commands`. Skills offer cross-platform, portable,
`reusable`, flexible, token-efficient, context-preserving, and
`composable` functionality.

Use commands as proxies to launch orchestration skills. Notice how the
custom commands in this project invoke skills directly.

---

## Skill standardization

All skills in this project follow the standardized structure and
conventions defined in `docs/skill-reference-guide.md`. This ensures
consistency, maintainability, and predictable behavior across all
skills.

### Key standardization features

**Consistent structure:**

- YAML `frontmatter` with `name` and `description`
- Required sections: `GOAL`, `WHEN`, Efficiency directives, Workflow,
  Output
- Status codes on first line: `SUCCESS`, `WARN`, `ERROR`

**Orchestration patterns:**

- **Linear**: 3-5 step workflows for single-responsibility tasks
- **Sequential (looping)**: fixed order with automatic loop control (see
  `git-commit`)
- **Conditional (branching)**: smart initialization and optional steps
  (see `update-changelog`)

**Best practices:**

- `POSIX`-compliant shell scripts for portability
- Batch operations and parallel execution for efficiency
- Explicit error handling with status codes
- Test scripts in `tests/` subdirectories

**See:**
[`docs/skill-reference-guide.md`](docs/skill-reference-guide.md) for the
complete specification.

---

## Designing `composable` skills

To prevent `context rot` and encourage skill composition, design
`reusable` skills that focus on a single responsibility.

When possible, use an orchestrator skill to manage sub-skills, enabling
complex workflows from straightforward, `reusable` skills.

### Examples

#### Git commit workflow (sequential with looping)

The `git-commit` skill demonstrates the **sequential orchestration**
pattern with automatic loop control:

```bash
git-commit (orchestrator)
  ├─► git-add       # Stages atomic changes
  ├─► git-message   # Generates conventional commit message
  └─► git-status    # Displays repository status
```

Automatically loops until the repository remains clean. Uses status
codes (`SUCCESS`, `WARN`, `ERROR`) for flow control. Each sub-skill
provides single-responsibility, `reusable` functionality.

**See:** [`skills/git-commit/SKILL.md`](skills/git-commit/SKILL.md)

#### Update changelog workflow (conditional with branching)

The `update-changelog` skill demonstrates the **conditional
orchestration** pattern with smart initialization:

```bash
update-changelog (orchestrator)
  ├─► init-changelog    # Creates CHANGELOG.md if absent (script: init-changelog.sh)
  ├─► edit-changelog    # Adds entries from git commits (script: edit-changelog.sh)
  └─► cleanup-changelog # Formats and cleans (invokes fix-markdown skill)
```

Uses status codes for conditional logic: skips cleanup if `WARN` (no
changes), halts on `ERROR`. Smart initialization creates `CHANGELOG.md`
only when missing. State management via `.last-aggregated-commit` tracks
processed commits.

**See:**
[`skills/update-changelog/SKILL.md`](skills/update-changelog/SKILL.md)

---

## Key principles

All skills follow these core principles from the Agent Skills Protocol:

- **Single responsibility** - Each skill performs one task well
- **`Composability`** - `Orchestrators` coordinate sub-skills with
  status codes
- **Standardization** - Consistent structure, status codes, and patterns
  across all skills
- **Scripts for complexity** - Use `POSIX`-compliant scripts when prompt
  engineering lacks efficiency
- **References for depth** - Include supporting docs when needed
- **Efficiency first** - Batch operations, parallel execution, token
  optimization

**Explore the existing skills** in `skills/` to see patterns in action.
All skills show the standardized structure from
`docs/skill-reference-guide.md`.

---

## Resources

**Project documentation:**

- [`docs/skill-reference-guide.md`](docs/skill-reference-guide.md) -
  Master guide for skill structure, patterns, and conventions
- [`CLAUDE.md`](CLAUDE.md) - Repository-specific guidance for Claude
  Code

**Official documentation:**

- [`Gemini CLI Skills Documentation`](https://geminicli.com/docs/cli/skills/)
- [`Agent Skills Directory`](https://agentskills.io/home)
