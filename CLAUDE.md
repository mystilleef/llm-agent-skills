# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This repository demonstrates reusable and composable agent skills following the **Agent Skills Protocol**. Skills work across both Gemini CLI and Claude Code. The architecture emphasizes:

- **Single responsibility**: Each skill performs one task well
- **Composability**: Orchestrator skills coordinate sub-skills
- **Scripts for complexity**: Use shell scripts when prompt engineering is insufficient
- **References for depth**: Include supporting documentation where needed

## Skill Architecture

Skills follow a consistent structure:
```
skills/<skill-name>/
  ├── SKILL.md           # Skill definition and protocol
  ├── references/        # Supporting documentation
  ├── scripts/          # Shell scripts for complex operations
  └── tests/            # Test scripts
```

### Key Orchestration Patterns

**Git Commit Workflow** (`git-commit`):
- Orchestrates atomic commits by coordinating `git-add`, `git-message`, and `git-status` skills
- Each sub-skill is reusable and focused on a single responsibility
- See: `skills/git-commit/SKILL.md`

**Changelog Workflow** (`update-changelog`):
- Orchestrates `init-changelog`, `edit-changelog`, and `cleanup-changelog` skills
- Uses conditional logic based on script status codes (`SUCCESS`, `WARN`, `ERROR`)
- Tracks processed commits via `.last-aggregated-commit` pointer file
- See: `skills/update-changelog/SKILL.md`

## Development Commands

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

### Testing Skills
Each skill with scripts includes test files in `tests/` directory. Run tests directly:
```bash
bash skills/<skill-name>/tests/test-<script-name>.sh
```

## Important Script Conventions

### Status Codes
Scripts use three-level status codes:
- `SUCCESS`: Operation completed with changes
- `WARN`: Operation completed but no changes needed
- `ERROR`: Operation failed

Status appears on the first line of script output. Orchestrator skills use these codes for conditional workflow logic.

### Changelog Scripts
- `init-changelog.sh`: Creates `CHANGELOG.md` structure and `.last-aggregated-commit` pointer
- `edit-changelog.sh`: Parses git commits using conventional commit format and updates changelog
  - Requires Bash 4.0+ for associative arrays
  - Filters commits by type: `feat`, `fix`, `perf`, `refactor`, `revert`, `build`
  - Skips non-user-facing types: `chore`, `ci`, `style`, `test`, `docs`
  - Detects breaking changes via `!` suffix or `BREAKING CHANGE:` in body
  - Deduplicates by issue references (`#123`, `GH-123`)
- `remove-empty-headers.sh`: Removes empty changelog sections

### Git Staging Scripts
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

## Skill Development Guidelines

When creating or modifying skills:

1. **SKILL.md format**: Use YAML frontmatter with `name` and `description` fields
2. **Clear protocols**: Define Goal, When, Workflow, and directives
3. **Efficiency directives**: All skills emphasize batch processing, parallel operations, and token optimization
4. **References**: Place supporting documentation in `references/` folder
5. **Scripts**: Use scripts for complex logic that's inefficient in prompts
6. **Tests**: Provide test scripts for validation

## Configuration Files

- `biome.json`: Biome configuration for code formatting/linting
- `settings.json`: Skills feature flag (requires `previewFeatures: true` and `experimental.skills: true`)
- `.beads/`: Beads tool integration (version control and interaction tracking)

## Key Constraints

### Git Skills
- Never use `git checkout` or `git restore`
- Never stage everything with `git add .` or `git add -A`
- Always await user approval before committing
- Isolate `.gitignore` changes in separate commits

### Markdown Skills
- Follow E-Prime directive (avoid "to be" verbs)
- Format with Prettier before and after edits
- Wrap filenames, paths, URIs, and URLs in backticks
- Use sentence case for headings
- Fix Vale lint issues iteratively until none remain

### Changelog Skills
- Use conventional commit format: `type(scope): description`
- Track position via `.last-aggregated-commit` file
- Filter out non-user-facing commit types
- Aggregate by issue references to avoid duplicates
