---
name: git-status
description:
  Present the status of the git repository to the user. Use when the
  agent needs to display the status of the git repository to the user.
---

# Present git status

**`GOAL`**: present the git status in a visually appealing and
consistent way.

**`WHEN`**: the user requests the repository status or the agent needs
to show it.

**`NOTE`**: use this skill **only** to present the status to the user.
Avoid internal state analysis.

## Efficiency directives

- Batch operations on file groups, avoid individual file processing
- Use parallel execution when possible
- Target only relevant files
- Reduce token usage

## Task management

For complex tasks: use `todo` system to break down, plan, and optimize
workflow.

## Git directives

### For repository status

```bash
git status --porcelain=v2 --branch
```

## References

The following reference files serve as strict guidelines:

- **`references/git-status-codes.md`**: complete reference for parsing
  git status output
- **`references/git-status-presentation.md`**: git status presentation
  guidelines, examples, and templates

## Workflow

- Execute `git status` command
- Parse output using `git-status-codes.md`
- Format output using `git-status-presentation.md`
- Present final status to user
- **`DONE`**
