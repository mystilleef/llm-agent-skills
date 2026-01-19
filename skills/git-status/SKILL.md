---
name: git-status
description:
  Present the status of the git repository to the user. Use when the
  agent needs to display the status of the git repository to the user.
  Use ONLY for presentation to the user. The agent SHOULD NOT use this
  for it's own, or internal, use.
---

# Agent Protocol: Present git status

Follow all the instructions in this file to present the git status in a
visually appealing and consistent way to the user.

Perform this skill and the operations within it using the least
verbosity possible.

**`NOTE`:** _Use this skill **only** to present the status of the
repository to the user. The agent **shouldn't** use this skill for
internal use._

## References

The following reference files serve as strict guidelines to study for
presenting the repository status in a visually appealing way to the
user.

Locate the reference files in the `references` folder.

- **`references/git-status-codes.md`**: Complete reference for parsing
  git status output
- **`references/git-status-presentation.md`**: Git status presentation
  guidelines, examples, and templates

## Efficient analysis directives

Maximize efficiency and reduce token usage:

- **Batch processing**: execute git operations, like `git diff`, on all
  file simultaneously. Avoid individual file operations.
- **Parallel processing**: if possible, execute operations
  simultaneously and/or in parallel.
- Optimize all responses, outputs, communications, and operations for
  context size and token efficiency.

## Task management directives

For complex tasks, use your `todo` management system to:

- Break down, plan, revise, and streamline tasks;
- Maintain internal task state;
- Remove unnecessary and redundant operations;
- Optimize tool usage and workflow to fulfill the goal.

## Git directives

Use the `git` commands below to:

- Get a comprehensive understanding of the changes in the project;
- Get the status of the repository.

**`NOTE`:** _The `git diff` commands should always use the `--no-pager`
option and `--no-ext-diff` flag to ensure the use of the standard `diff`
and `pager` tools._

### For repository status

Use the following command to get the status of the repository.

```bash
git status --porcelain=v2 --branch
```

## Final presentation

- Study the reference files; then
- Using the reference files as strict guidelines, present the current
  git status.
- **`DONE`**
