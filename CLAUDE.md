# Claude.md

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

You can install skills in `~/.gemini` (Gemini CLI) or `~/.claudie`
(Claude Code) for global use. Commands provide Gemini-specific wrappers
and remain optional - Claude Code uses skills directly.

## Skill architecture

Skills follow a consistent structure:

```bash
skills/<skill-name>/
  ├── SKILL.md           # Skill definition and protocol
  ├── references/        # Supporting documentation
  ├── scripts/          # Shell scripts for complex operations
  └── tests/            # Test scripts
```

### Key orchestration patterns

**Git commit workflow** (`git-commit`):

- Orchestrates atomic commits by coordinating `git-add`, `git-message`,
  and `git-status` skills
- Each sub-skill provides reusable functionality focused on a single
  responsibility
- Automatically loops through commits until repository becomes clean
- Requires user approval for each commit message via `git-message` skill
- See: `skills/git-commit/SKILL.md:20-69`

**Changelog workflow** (`update-changelog`):

- Orchestrates `init-changelog`, `edit-changelog`, and
  `cleanup-changelog` skills
- Uses conditional logic based on script status codes (`SUCCESS`,
  `WARN`, `ERROR`)
- Tracks processed commits via `.last-aggregated-commit` pointer file
- Smart initialization: ≤100 commits processes full history, >100
  commits processes recent 100
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

## Important script conventions

### Status codes

Scripts use three-level status codes:

- `SUCCESS`: Operation completed with changes
- `WARN`: Operation completed but no changes needed
- `ERROR`: Operation failed

Status appears on the first line of script output. Orchestrator skills
use these codes for conditional workflow logic.

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

When creating or modifying skills:

1. **`SKILL.md` format**: Use YAML `frontmatter` with `name` and
   `description` fields
2. **Clear protocols**: Define Goal, When, Workflow, and directives (see
   `skills/git-commit/SKILL.md` for reference)
3. **Required sections**:
   - `GOAL`: What the skill accomplishes
   - `WHEN`: When to invoke the skill
   - `Workflow`: Step-by-step execution with status handling
   - `Directives`: Primary rules, git-specific commands, efficiency
     requirements
   - `Task Management`: When to use `todo` system
   - `Output`: Files modified and status format
4. **Efficiency directives**: All skills emphasize batch processing,
   parallel operations, and token optimization
5. **References**: Place supporting documentation in `references/`
   folder (see existing skills for examples)
6. **Scripts**: Use scripts for complex logic with low prompt efficiency
   (rationale: reduces token usage, improves reproducibility)
7. **Tests**: Provide test scripts in `tests/` subdirectory for
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
  - Install in `~/.claudie/settings.json` for Claude Code
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

### Status code protocol

All scripts use three-level status codes on the first line of output:

- `SUCCESS`: Operation completed with changes
- `WARN`: Operation completed but no changes needed
- `ERROR`: Operation failed

Orchestrator skills use these codes for conditional workflow logic (see
`git-commit` and `update-changelog`).

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
