---
name: git-add
description:
  Autonomously select files to stage for an atomic Git commit. Use when
  the agent wants to stage selected files for a git commit. Only stages
  selected files, but does not commit them.
---

# Agent protocol: Stage atomic git changes

**Goal:** Stage a single, cohesive set of related file changes for an
atomic commit.

**When:** Use when the agent wants to stage selected files for a git
commit.

**`NOTE`:** _Use this skill only for staging selected files, but not
committing them._

## References

Study these guidelines to identify and select files for atomic, cohesive
changes.

Locate the reference files in the `references` folder.

- **`references/atomic-commits-best-practices.md`**: _Best practices for
  making atomic commits._
- **`references/autonomous-staging-patterns.md`**: _Patterns to use for
  autonomously staging atomic commits._
- **`references/ignore-patterns-reference.md`**: _Patterns to use for
  ignoring files._

**`NOTE`:** _Study the reference files `ONLY` if more than one file
exists to stage._

## Primary directives

- **`NEVER`** use `git add .` or `git add -A`.
- **`NEVER`** use `git checkout` or `git restore`.
- Assume a file in both `staged` and `unstaged` states needs staging.
- Stage files explicitly: `git add <file1> <file2> ...`.
- Leave unrelated changes `unstaged`.
- Isolate `.gitignore` changes; stage them in a separate commit.
- Except for `.gitignore`, **`DON'T`** edit any files

## Git directives

Use the `git` commands below to:

- Get a comprehensive understanding of the changes in the project;
- Get the status of the repository; and
- Stage files for the commit.

**`NOTE`:** _The `git diff` commands should always use the `--no-pager`
option and `--no-ext-diff` flag to ensure the use of the standard `diff`
and `pager` tools._

### For `unstaged` changes

```bash
git --no-pager diff --no-ext-diff --stat --minimal --patience --histogram \
    --find-renames --summary --no-color -U10 <file_group>
```

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

## Efficient analysis directives

Maximize efficiency and reduce token usage:

- **Single-file shortcut**: If exactly one tracked file remains
  `unstaged`, stage it immediately. Skip reference study and analysis.
- **Batch processing**: Execute git operations, like `git diff` or
  `git add`, on file groups simultaneously. Avoid individual file
  operations.
- **Parallel processing**: if possible, execute operations
  simultaneously and/or in parallel.
- **Targeted analysis**:
  - Analyze tracked files via `git diff`.
  - Read non-ignored `untracked` files to determine relevance.
  - Omit ignored files from analysis.
- Optimize all responses, outputs, communications, and operations for
  context size and token efficiency.

## Task management directives

For complex tasks, use your `todo` management system to:

- Break down, plan, revise, and streamline tasks;
- Maintain internal task state;
- Remove unnecessary and redundant operations;
- Optimize tool usage and workflow to fulfill the goal.

---

## Workflow

**Exception:** If exactly one _`tracked unstaged`_ file exists, stage it
immediately and skip the `References`, `Analyze`, and `Identify` steps
below.

- **`References`:** Study the `references` reference files.
- **`Analyze`:** Review the repository's status to identify all
  modified, new, and `untracked` files.
- **`Ignore`:** Strictly following the guidelines in the reference
  files, update and format `.gitignore`, ensuring each pattern per line.
- **`Identify`:** Strictly following the guidelines in the reference
  files, select the smallest group of `unstaged` files that form a
  single logical and cohesive change.
- **`Stage`:** Add the verified atomic group to the staging area.
- **`Report`:** Summarize a bullet point list of staged files.
- **`Halt`:** Done.
