# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working
with code in this repository.

## Project overview

This repository demonstrates reusable and `composable` agent skills
following the **Agent Skills Protocol**. Skills work across both Gemini
CLI and Claude Code. The architecture emphasizes:

- **Single responsibility**: Each skill performs one task well
- **`Composability`**: Orchestrator skills coordinate sub-skills
- **Scripts for complexity**: Use shell scripts when prompt engineering
  lacks enough power
- **References for depth**: Include supporting documentation where
  needed

You can install skills in `~/.gemini` (Gemini CLI) or `~/.claude`
(Claude Code) for global use. Commands provide Gemini-specific wrappers
and remain optional - Claude Code uses skills directly.

## Skill architecture

All skills follow a standardized structure per
`docs/skill-reference-guide.md`:

```bash
skills/<skill-name>/
  ├── SKILL.md           # Skill definition with YAML frontmatter
  ├── references/        # Supporting documentation
  ├── scripts/          # Shell scripts for complex operations
  └── tests/            # Test scripts
```

Each `SKILL.md` contains:

- YAML `frontmatter` (`name`, `description`)
- `GOAL` and `WHEN` sections
- Efficiency directives (batching, parallel execution, token
  optimization)
- Workflow (step-by-step with status handling)
- Output (files modified, status communication with
  `SUCCESS`/`WARN`/`ERROR`)

### Key orchestration patterns

All orchestrator skills follow the patterns from
`docs/skill-reference-guide.md`:

**Sequential orchestration with looping** (`git-commit`):

- Pattern: Fixed order, status capture, automatic loop control
- Coordinates `git-add`, `git-message`, and `git-status` skills in
  sequence
- Each sub-skill provides single-responsibility, reusable functionality
- Automatically loops until repository becomes clean (no user prompt
  needed)
- Requires user approval for each commit message via `git-message` skill
- See: `skills/git-commit/SKILL.md:20-69`

**Conditional orchestration with branching** (`update-changelog`):

- Pattern: Dependencies, optional steps, smart initialization
- Orchestrates `init-changelog`, `edit-changelog`, and
  `cleanup-changelog` skills
- Uses status codes for conditional logic (skip cleanup if `WARN`, halt
  on `ERROR`)
- Smart initialization: creates `CHANGELOG.md` only if missing
- Tracks processed commits via `.last-aggregated-commit` pointer file
- Initialization strategy: ≤100 commits processes full history, >100
  commits uses recent 100
- See: `skills/update-changelog/SKILL.md`

## Development commands

### Prerequisites

- **Biome**: Code formatting and linting
- **Prettier**: Markdown formatting
- **Vale**: Markdown prose linting (configure with Google Style Guide)
- **Git**: Version control operations
- **Bash 4.0+**: Required for changelog scripts (uses associative
  arrays)

### Formatting

```bash
# Format code using Biome
biome format --write .

# Format markdown files
prettier --write <file_path>
```

### Linting

```bash
# Check code with Biome
biome check .

# Lint markdown with Vale (requires vale sync first)
vale sync
vale --no-wrap --output=JSON <file_path>
```

### Testing skills

Each skill with scripts includes test files in `tests/` directory. Run
tests directly:

```bash
bash skills/<skill-name>/tests/test-<script-name>.sh
```

### Creating new skills

Use `skills/new-agent-skill/SKILL.md` as a template when creating new
skills. This template provides the standard structure and sections
expected in all skills.

## Important script conventions

All scripts and skills follow the standardized conventions from
`docs/skill-reference-guide.md`.

### Status codes

**All skills** output status on the first line using the three-level
protocol:

- `SUCCESS`: Operation completed with changes (exit code 0)
- `WARN`: Operation completed but no changes needed (exit code 0)
- `ERROR`: Operation failed (exit code 1)

Extended domain-specific codes include: `APPROVED`,
`REJECTED_EDIT_FILES`, `REJECTED_REGENERATE`, `REJECTED_ABORT` (used by
`git-message`).

Orchestrator skills capture and handle these status codes for
conditional workflow logic.

### Changelog scripts

- `init-changelog.sh` (139 lines): Creates `CHANGELOG.md` structure and
  `.last-aggregated-commit` pointer
  - Detects version from `CHANGELOG.md` or defaults to `0.1.0`
  - Smart commit history processing: ≤100 commits `backfills` full
    history, >100 commits uses recent 100
- `edit-changelog.sh` (270 lines): Parses git commits using conventional
  commit format and updates changelog
  - Requires Bash 4.0+ for associative arrays
  - Filters commits by type: `feat`, `fix`, `perf`, `refactor`,
    `revert`, `build`
  - Skips non-user-facing types: `chore`, `ci`, `style`, `test`, `docs`
  - Detects breaking changes via `!` suffix or `BREAKING CHANGE:` in
    body
  - `Deduplicates` by issue references (`#123`, `GH-123`)
  - Maps types to Keep a Changelog categories:
    - `feat` → Added
    - `fix` → Fixed
    - `perf` → Changed (with "Improved" prefix)
    - `refactor` → Changed (if user-facing)
    - `revert` → Removed
    - `build` → Changed (for dependency updates)
- `remove-empty-headers.sh` (85 lines): Removes empty changelog sections
  using `AWK` (`Aho`, `Weinberger`, and `Kernighan` pattern scanning
  language)

### Git staging scripts

The `git-add` skill uses specific git diff commands:

```bash
# For unstaged changes
git --no-pager diff --no-ext-diff --stat --minimal --patience --histogram \
    --find-renames --summary --no-color -U10 <file_group>

# For staged changes
git --no-pager diff --staged --no-ext-diff --stat --minimal --patience \
    --histogram --find-renames --summary --no-color -U10
```

Never use `git add .` or `git add -A` - always stage files explicitly.

## Skill development guidelines

**All skills follow a standardized structure** according to
`docs/skill-reference-guide.md`, which provides the authoritative
reference guide for creating and updating agent skills.

The reference guide defines:

- **Core protocol**: Three-level status codes (`SUCCESS`, `WARN`,
  `ERROR`) with optional extended codes
- **Skill structure templates**: Complete templates for YAML
  `frontmatter`, sections, and formatting
- **Orchestration patterns**: Linear, sequential (looping), and
  conditional (branching)
- **Scripting standards**: `POSIX` compliance, portability requirements,
  best practices
- **Anti-patterns**: Seven strict prohibitions including monolithic
  skills, implicit dependencies, and silent failures

When creating or modifying skills:

1. **Follow the reference guide**: Use `docs/skill-reference-guide.md`
   templates and patterns
2. **Use skill template**: Start with `skills/new-agent-skill/SKILL.md`
   as a base
3. **Study existing skills**: Reference `skills/git-commit/SKILL.md` and
   `skills/update-changelog/SKILL.md` for orchestration examples
4. **Add references**: Place supporting documentation in `references/`
   folder when needed
5. **Write scripts**: Use shell scripts for complex logic to reduce
   token usage and improve reproducibility
6. **Include tests**: Provide test scripts in `tests/` subdirectory for
   validation

## Configuration files

- `biome.json`: Biome configuration for code formatting/linting
  - Git integration enabled (`vcs.enabled: true`)
  - Recommended linter rules enabled
  - Auto-organize imports enabled
  - Uses `.editorconfig` if present
- `settings.json`: Skills feature flag (requires `previewFeatures: true`
  and `experimental.skills: true`)
  - Install in `~/.gemini/settings.json` for Gemini CLI
  - Install in `~/.claude/settings.json` for Claude Code
- `.beads/`: Beads tool integration (version control and interaction
  tracking)

## Key constraints

### Git skills

- Never use `git checkout` or `git restore`
- Never stage everything with `git add .` or `git add -A`
- Always await user approval before committing
- Isolate `.gitignore` changes in separate commits

### Markdown skills

- Follow E-Prime directive (avoid forms of "`to be`")
- Format with Prettier before and after edits
- Wrap filenames, paths, `URIs`, and `URLs` in backticks
- Use sentence case for headings
- Fix Vale lint issues iteratively until none remain

### Changelog skills

- Use conventional commit format: `type(scope): description`
- Track position via `.last-aggregated-commit` file
- Filter out non-user-facing commit types (`chore`, `ci`, `style`,
  `test`, `docs`)
- Collect by issue references to avoid duplicates (`#123`, `GH-123`)
- Breaking changes detected via `!` suffix (for example, `feat!:`) or
  `BREAKING CHANGE:` in body

## Important architecture patterns

All skills follow the patterns defined in
`docs/skill-reference-guide.md`:

### Status code protocol

First-line status output enables orchestrator skills to make intelligent
decisions:

- `SUCCESS`: Operation completed with changes → proceed to next step
- `WARN`: Operation completed but no changes needed → skip optional
  steps
- `ERROR`: Operation failed → halt immediately

See `git-commit` and `update-changelog` for orchestration examples using
status codes.

### State management

The changelog system uses `.last-aggregated-commit` pointer file for
incremental processing:

- Stores `SHA` (Secure Hash Algorithm) of last processed commit
- Enables resilient tracking across rebases/cherry-picks
- Smart initialization based on commit history size

### Conventional commit type mapping

Types map to Keep a Changelog categories:

- `feat` → **Added** (new features)
- `fix` → **Fixed** (bug fixes)
- `perf` → **Changed** (performance improvements, prefixed with
  "Improved")
- `refactor` → **Changed** (user-facing refactors only)
- `revert` → **Removed** (reverted changes)
- `build` → **Changed** (dependency updates only)
- `docs`, `test`, `chore`, `ci`, `style` → **Skipped** (not user-facing)
