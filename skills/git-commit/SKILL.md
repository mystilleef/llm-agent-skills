---
name: git-commit
description:
  Commit changes in the repository. Use when the user or agent needs to
  commit changes in the repository. Employs the "git-add" and
  "git-message" skills to perform the commit.
---

# Agent protocol: Commit changes in the git repository

**Goal:** Use the `git-add` and `git-message` skills to commit changes
in the repository and clean it.

**When:** Use when the user or agent needs to commit changes in the
repository.

**`NOTE`:** _Await user approval of commit messages before committing.
Make commits only after the user has approved the commit message._

## Commit process

1. Use the `git-add` skill to autonomously stage selected files.
2. Use the `git-message` skill to generate a commit message.
3. Halt and await user approval for the commit message.
4. Upon approval, commit staged changes using the commit message.

### Commit process loop

- Repeat the commit process, steps 1-4, until you commit all changes,
  and clean the repository.
- Immediately halt and abort all processes if the user doesn't approve a
  commit message.

## Repository status

After committing all changes and cleaning the repository, use the
`git-status` skill to present the status of the repository to the user.

## Efficient analysis directives

Maximize efficiency and reduce token usage:

- **Batch processing**: Execute operations on file groups
  simultaneously. Avoid individual file operations.
- **Parallel processing**: if possible, execute operations
  simultaneously and/or in parallel.
- **Targeted analysis**: Focus only on the specific file or files
  requested.
- Optimize all responses, outputs, communications, and operations for
  context size and token efficiency.

## Workflow

- Perform the commit process:
  - Stage files using the `git-add` skill.
  - Generate a commit message using the `git-message` skill.
  - Halt and await user approval for the commit message.
  - Commit staged changes using the commit message.
- Repeat the commit process until you clean the repository.
- Halt and abort all processes if the user doesn't approve the commit
  message.
- Use the `git-status` skill to present the status of the repository
- **`DONE`**
