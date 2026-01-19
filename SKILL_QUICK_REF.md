# Skill quick reference

**Ultra-condensed lookup for skill development. Zero explanations -
templates and patterns only.**

**For full explanations**: see `SKILL_REFERENCE.md`

**Usage**:

- Find pattern in Section 1
- Copy template from Section 2
- Check anti-patterns in Section 6

## Contents

1. [Pattern lookup](#1-pattern-lookup) - Pattern selection guide
2. [Core templates](#2-core-templates) - YAML, headers, workflows
3. [Git commands](#3-git-commands) - Standard commands and constraints
4. [Status codes](#4-status-codes) - Status conventions and examples
5. [Script patterns](#5-script-patterns) - Scripts vs prompts guidance
6. [Anti-pattern rules](#6-anti-pattern-rules) - Common mistakes to avoid
7. [Domain cheat sheets](#7-domain-cheat-sheets) - Markdown, Changelog, Git
8. [Quick reference map](#8-quick-reference-map) - Cross-reference table

---

## 1. Pattern lookup

### 1.1 Pattern selection

| If You Need                 | Use Pattern        | Example Skill     | See Full Ref |
| --------------------------- | ------------------ | ----------------- | ------------ |
| Single task, no sub-skills  | Basic              | cleanup-changelog | Section 5.2  |
| Fixed sequence of skills    | Sequential         | git-commit        | Section 5.3  |
| Conditional skill execution | Conditional        | update-changelog  | Section 5.4  |
| Complex parsing/logic       | Script Integration | edit-changelog    | Section 7    |

### 1.2 Domain lookup

| Domain       | Skill Type           | Example        | See Full Ref  |
| ------------ | -------------------- | -------------- | ------------- |
| Git staging  | Atomic staging       | git-add        | Section 8.3.1 |
| Git messages | Message generation   | git-message    | Section 8.3.2 |
| Git status   | Status presentation  | git-status     | Section 8.3.3 |
| Markdown     | E-Prime, Vale        | fix-markdown   | Section 10.1  |
| Changelog    | Conventional commits | edit-changelog | Section 10.2  |

### 1.3 Status codes

| Status    | Meaning                | When             |
| --------- | ---------------------- | ---------------- |
| `SUCCESS` | Completed with changes | Work done        |
| `WARN`    | Completed, no changes  | Nothing to do    |
| `ERROR`   | Failed                 | Problem occurred |

**Extended (git-message)**:

| Status              | Meaning                           |
| ------------------- | --------------------------------- |
| APPROVED            | User approved message             |
| REJECTED_EDIT_FILES | User wants to change staged files |
| REJECTED_REGENERATE | User wants new message            |
| REJECTED_ABORT      | User cancelled commit             |

**See**: `SKILL_REFERENCE.md` section 6 for details

---

## 2. Core templates

### 2.1 `YAML` `frontmatter`

```yaml
---
name: skill-name
description:
  Brief description (1-3 sentences). Use when [context]. Employs
  [sub-skills if orchestrator].
---
```

### 2.2 Goal/When/Note header

```markdown
# [Readable skill title]

**`GOAL`**: [What the skill accomplishes]

**`WHEN`**: [When to invoke this skill]

**`NOTE`**: [Key constraint or warning] (optional)
```

### 2.3 Efficiency directives

```markdown
## Efficiency directives

- Batch operations on file groups, avoid individual file processing
- Use parallel execution when possible
- Target only relevant files
- Reduce token usage
```

### 2.4 Task management

```markdown
## Task management

For complex tasks: use `todo` system to break down, plan, and optimize
workflow.
```

### 2.5 Git directives (full)

````markdown
## Git directives

Git commands (use `--no-pager` and `--no-ext-diff` for diffs):

### For unstaged changes

```bash
git --no-pager diff --no-ext-diff --stat --minimal --patience --histogram \
    --find-renames --summary --no-color -U10 <file_group>
```
````

### For staged changes

```bash
git --no-pager diff --staged --no-ext-diff --stat --minimal --patience \
    --histogram --find-renames --summary --no-color -U10
```

### For repository status

```bash
git status --porcelain=v2 --branch
```

### For staging files

```bash
git add <file1> <file2> ...
```

````

### 2.6 Git directives (minimal)

```markdown
## Git directives

Use the `git` commands below to:

- Get a comprehensive understanding of the changes in the project;
- Get the status of the repository.

**`NOTE`:** _The `git diff` commands should always use the `--no-pager` option and `--no-ext-diff` flag to ensure the use of the standard `diff` and `pager` tools._

### For repository status

Use the following command to get the status of the repository.

```bash
git status --porcelain=v2 --branch
````

````

### 2.7 Git constraints

```markdown
## Primary directives

- Study `references/` files `ONLY` if more than one file exists to stage.
- **`NEVER`** use `git add .` or `git add -A`.
- **`NEVER`** use `git checkout` or `git restore`.
- Assume a file in both `staged` and `unstaged` states needs staging.
- Stage files explicitly: `git add <file1> <file2> ...`.
- Leave unrelated changes `unstaged`.
- Isolate `.gitignore` changes; stage them in a separate commit.
- Except for `.gitignore`, **`DON'T`** edit any files
````

### 2.8 References section

```markdown
## References

The following reference files serve as strict guidelines:

- **`references/filename.md`**: Brief description of what guidance this
  file provides
- **`references/another-file.md`**: Brief description
```

### 2.9 Output (basic)

```markdown
## Output

**Files created/modified:**

- `filename` - Description of what this file contains
- `another-file` - Description

**Status communication:**

Describes what the skill reports to the user or next orchestrator.
```

### 2.10 Output (script-based)

```markdown
## Output

**Files created/modified:**

- `CHANGELOG.md` - Created (if missing) or updated with new entries
- `.last-aggregated-commit` - Updated to `HEAD`

**Status communication:**

First line of output indicates status:

- `SUCCESS: [message]` - Operation completed with changes
- `WARN: [message]` - Operation completed but no changes needed
- `ERROR: [message]` - Operation failed

**Following lines (when SUCCESS):** Additional details about what
changed
```

### 2.11 Workflow (orchestrator)

```markdown
## Workflow

Follow these steps in sequence:

### Step 1: [First action description]

- Check/verify [prerequisite]
- If [condition]: Skip to Step N.
- If [other condition]: Continue to Step 2.

### Step 2: [Invoke first sub-skill]

- Invoke the `sub-skill-1` skill to [accomplish sub-goal].
- Capture the status from skill output (`SUCCESS`, `WARN`, or `ERROR`).
- Handle the status:
  - If `ERROR`: Halt and report the error to the user.
  - If `WARN`: [Context-specific action, may skip steps].
  - If `SUCCESS`: Continue to Step 3.

### Step 3: [Invoke second sub-skill]

- Invoke the `sub-skill-2` skill to [accomplish sub-goal].
- Capture the status from skill output.
- Handle the status:
  - If `ERROR`: Halt and report.
  - If `SUCCESS`: Continue to Step 4.
  - If [special status]: [Special handling, may loop back].

### Step 4: [Final action]

- Execute [final operation]
- If success: Continue to Step 5.
- If failure: Halt and report.

### Step 5: [Check for more work] (automatic loop control)

- Check for [remaining work]
- If no more work: Continue to Step 6.
- If work remains: automatically loop back to Step 2 (no user prompt
  needed).

### Step 6: [Present final status]

- Invoke [status presentation skill or report]
- Report summary: [what was accomplished]
- **`DONE`**
```

### 2.12 Workflow (basic)

```markdown
## Workflow

- Step 1 action
- Step 2 action
- Step 3 action
- **`DONE`**
```

**See**: `SKILL_REFERENCE.md` section 4 for details

---

## 3. Git commands

### 3.1 Standard commands

```bash
# Unstaged changes
git --no-pager diff --no-ext-diff --stat --minimal --patience --histogram \
    --find-renames --summary --no-color -U10 <file_group>

# Staged changes
git --no-pager diff --staged --no-ext-diff --stat --minimal --patience \
    --histogram --find-renames --summary --no-color -U10

# Repository status
git status --porcelain=v2 --branch

# Staging files
git add <file1> <file2> ...

# Commit history (for changelog)
git log "$LAST_COMMIT"..HEAD --reverse --pretty=format:%H
```

### 3.2 Constraints checklist

**`NEVER`**:

- ❌ Use `git add .` or `git add -A`
- ❌ Use `git checkout` or `git restore`
- ❌ Edit files except `.gitignore`
- ❌ Push to remote without explicit user request
- ❌ Use `git commit --amend` without explicit user request
- ❌ Skip commit message approval

**ALWAYS**:

- ✅ Stage files explicitly: `git add file1 file2 file3`
- ✅ Isolate `.gitignore` changes in separate commits
- ✅ Await user approval before committing
- ✅ Use atomic commits (single logical change per commit)
- ✅ Capture status from sub-skills and handle explicitly

**See**: `SKILL_REFERENCE.md` section 8 for details

---

## 4. Status codes

### 4.1 Status table

| Status    | Meaning                | When             | Exit Code |
| --------- | ---------------------- | ---------------- | --------- |
| `SUCCESS` | Completed with changes | Work done        | 0         |
| `WARN`    | Completed, no changes  | Nothing to do    | 0         |
| `ERROR`   | Failed                 | Problem occurred | 1         |

### 4.2 Protocol rules

- **First line contains status**: `STATUS: descriptive message`
- **Consistent format**: UPPERCASE status, colon + space, message
- **Extra details on following lines**: After first line, output details

### 4.3 Bash examples

```bash
# SUCCESS case
echo "SUCCESS: Operation completed with N changes"
exit 0

# WARN case
echo "WARN: No changes needed"
exit 0

# ERROR case
echo "ERROR: Operation failed - reason"
exit 1
```

**See**: `SKILL_REFERENCE.md` section 6 for details

---

## 5. Script patterns

### 5.1 Scripts vs prompts

| Use Scripts When                    | Use Prompts When            |
| ----------------------------------- | --------------------------- |
| Complex regex parsing               | `LLM` reasoning needed      |
| Associative arrays needed           | Context-aware decisions     |
| Stateful operations (pointer files) | User interaction required   |
| Repetitive logic (loops)            | Adapting to edge cases      |
| Performance-critical                | Handling ambiguity          |
| Deterministic algorithms            | Natural language processing |

### 5.2 Bash version check

```bash
# Check for Bash 4.0+ (required for associative arrays)
if ((BASH_VERSINFO[0] < 4)); then
  echo "ERROR: Bash 4.0 or higher is required."
  exit 1
fi
```

### 5.3 Prerequisite verification

```bash
# Verify prerequisites first
if [ ! -f "$CHANGELOG_FILE" ]; then
  echo "ERROR: $CHANGELOG_FILE doesn't exist. Please run 'init-changelog' first."
  exit 1
fi

# WARN for initialization (recoverable)
if [ ! -f "$POINTER_FILE" ]; then
  git rev-parse HEAD > "$POINTER_FILE"
  echo "WARN: Initialized pointer to HEAD. No changes processed."
  exit 0
fi
```

### 5.4 Temp file management

```bash
# Create secure temporary directory
TEMP_DIR=$(mktemp -d)
trap 'rm -rf "$TEMP_DIR"' EXIT
```

### 5.5 Debug mode

```bash
# Configuration
VERBOSE=${VERBOSE:-false}

# Debug function for verbose output
debug() {
  if [ "$VERBOSE" = "true" ]; then
    echo "DEBUG: $*" >&2
  fi
}

# Usage
debug "Processing commit: ${commit:0:7} - $SUBJECT"

# Enable with: VERBOSE=true bash script.sh
```

### 5.6 Pointer file pattern

```bash
# Read last aggregated commit
if [ ! -f "$POINTER_FILE" ]; then
  git rev-parse HEAD > "$POINTER_FILE"
  echo "WARN: Initialized pointer to HEAD. No changes processed."
  exit 0
fi

LAST_COMMIT=$(cat "$POINTER_FILE")

# Self-healing: reinitialize if corrupted
if ! git cat-file -e "$LAST_COMMIT" 2>/dev/null; then
  git rev-parse HEAD > "$POINTER_FILE"
  echo "WARN: Pointer corrupted. Reinitialized to HEAD."
  exit 0
fi

# Update pointer (only on success)
git rev-parse HEAD > "$POINTER_FILE"
```

### 5.7 Status propagation

```markdown
## Workflow

- Run `scripts/script-name.sh`
- Capture status from first line of output
- Handle the status:
  - If `ERROR`: Stop and report to user
  - If `WARN`: [Context-specific action]
  - If `SUCCESS`: Continue with next step
```

**See**: `SKILL_REFERENCE.md` section 7 for details

---

## 6. Anti-pattern rules

| Anti-Pattern           | Rule                         | Fix                              | Full Ref |
| ---------------------- | ---------------------------- | -------------------------------- | -------- |
| `God Skill`            | One responsibility per skill | Split to orchestrator + subs     | Sec 9.1  |
| Implicit Dependencies  | Verify prerequisites         | Check files exist first          | Sec 9.2  |
| Silent Failures        | Always output status         | Use `SUCCESS`/`WARN`/`ERROR`     | Sec 9.3  |
| Format Inconsistency   | Use `**\`KEYWORD\`:\*\*`     | Standardize headers              | Sec 9.4  |
| Duplication            | Use templates                | Copy from this guide             | Sec 9.5  |
| Token-Inefficient      | Batch operations             | `git diff f1 f2 f3` not separate | Sec 9.6  |
| Missing Status Capture | Capture first line           | Handle `ERROR`/`WARN`/`SUCCESS`  | Sec 9.7  |

**See**: `SKILL_REFERENCE.md` section 9 for details

---

## 7. Domain cheat sheets

### 7.1 Markdown skills

- Format before and after (prettier)
- Vale lint cycle until clean
- Wrap paths/filenames in backticks
- Sentence case headings
- No "`to be`" verbs (E-Prime)

### 7.2 Changelog skills

- Conventional commit format: `type(scope): description`
- User-facing types: `feat`, `fix`, `perf`, `refactor`\*, `build`\*,
  `revert`
- Non-user-facing types: `docs`, `style`, `test`, `chore`, `ci` (skip
  these)
- Remove duplicates by issue references (`#123`, `GH-123`)
- Update pointer to `HEAD` after success
- Keep a Changelog section order: Added, Changed, Deprecated, Removed,
  Fixed, Security
- Breaking changes: `!` suffix or `BREAKING CHANGE:` in body
- Filter by type, group by issue, sort by category

### 7.3 Git skills

- One commit = one logical change
- Never `git add .` or `git add -A`
- Never `git checkout` or `git restore`
- Always await approval before committing
- Isolate .gitignore changes
- Use porcelain v2 for status: `git status --porcelain=v2 --branch`
- Use `-U10` for better diff context
- Use `--patience --histogram` for better diffs
- Stage explicitly: `git add file1 file2 file3`

**See**: `SKILL_REFERENCE.md` section 10 for details

---

## 8. Quick reference map

| Need                  | See Section | Example File         |
| --------------------- | ----------- | -------------------- |
| Pattern selection     | Sec 1       | -                    |
| `YAML` `frontmatter`  | Sec 2.1     | any `SKILL.md`       |
| Goal/When/Note header | Sec 2.2     | any `SKILL.md`       |
| Efficiency directives | Sec 2.3     | git-add              |
| Task management       | Sec 2.4     | git-commit           |
| Git commands          | Sec 3       | git-add              |
| Status codes          | Sec 4       | edit-changelog.sh    |
| Script integration    | Sec 5       | edit-changelog       |
| Orchestrator workflow | Sec 2.11    | git-commit           |
| Basic workflow        | Sec 2.12    | cleanup-changelog    |
| Anti-patterns         | Sec 6       | `SKILL_REFERENCE.md` |
| Markdown rules        | Sec 7.1     | fix-markdown         |
| Changelog rules       | Sec 7.2     | edit-changelog       |
| Git rules             | Sec 7.3     | git-add              |

**Full documentation**: `SKILL_REFERENCE.md`

---

**End of SKILL_QUICK_REF.md**

For comprehensive explanations, examples, and detailed guidance, consult
`SKILL_REFERENCE.md`.
