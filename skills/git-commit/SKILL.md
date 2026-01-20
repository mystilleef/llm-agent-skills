---
name: git-commit
description:
  Commit changes in the repository. Use when the user or agent needs to
  commit changes in the repository. Employs the "git-add",
  "git-message", and "git-status" skills to perform the commit.
---

# Commit changes in the git repository

**`GOAL`**: use the `git-add`, `git-message` and `git-status` skills to
commit changes in the repository and clean it.

**`WHEN`**: use when the user or agent needs to commit changes in the
repository.

**`NOTE`:** _Await user approval of commit messages before committing.
Make commits only after the user has approved the commit message._

## Workflow

Follow these steps in sequence:

### Step 1: Check for changes

- Run `git status --porcelain=v2` to verify repository has changes.
- If no changes: Skip to Step 6 (report clean repository).
- If changes exist: Continue to Step 2.

### Step 2: Stage atomic changes

- Invoke the `git-add` skill to stage a cohesive set of files.
- Capture the status from skill output (`SUCCESS`, `WARN`, or `ERROR`).
- Handle the status:
  - If `ERROR`: Halt and report the error to the user.
  - If `WARN` (no files available): Skip to Step 6.
  - If `SUCCESS` (files staged): Continue to Step 3.

### Step 3: Generate commit message and await approval

- Invoke the `git-message` skill to generate a commit message.
- Capture the status from skill output (`APPROVED`,
  `REJECTED_EDIT_FILES`, `REJECTED_REGENERATE`, `REJECTED_ABORT`, or
  `ERROR`).
- Handle the status:
  - If `ERROR`: Halt and report the error to the user.
  - If `APPROVED`: Continue to Step 4.
  - If `REJECTED_EDIT_FILES`: `Unstage` all files with
    `git restore --staged .`, then loop back to Step 2.
  - If `REJECTED_REGENERATE`: Loop back to Step 3.
  - If `REJECTED_ABORT`: Halt and report abort to the user.

### Step 4: Commit changes

- Execute `git commit` with the approved message.
- If success: report the commit `SHA` and continue to Step 5.
- If failure: Halt and report the error to the user.

### Step 5: Check for more commits (automatic loop control)

- Run `git status --porcelain=v2` to check for remaining changes.
- If no changes: Continue to Step 6.
- If changes remain: automatically loop back to Step 2 (no user prompt
  needed).

### Step 6: Present final status

- Invoke the `git-status` skill to present repository state.
- Report summary: number of commits created and final repository state.
- **`DONE`**

## Efficiency directives

- Batch operations on file groups, avoid individual file processing
- Use parallel execution when possible
- Target only relevant files
- Reduce token usage

## Task management

For complex tasks: use `todo` system to break down, plan, and optimize
workflow.
