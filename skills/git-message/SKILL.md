---
name: git-message
description:
  Generate a commit message for changes already in the Git staging area,
  and await approval from the user. Use when the agent needs to generate
  a commit message for staged changes. Only generates the commit
  message, but does not perform the commit.
---

# Agent protocol: Generate commit message

**Goal:** Draft a commit message that adheres to Conventional Commits
specification and project standards, then present it for user approval.

**When:** Use when the agent needs to generate a commit message for
staged changes.

**`NOTE`:** _Use this skill only for generating a commit message, but
not for making commits._

## References

The following reference file serves as a strict guideline to study when
drafting and formatting the commit message.

Locate the reference file in the `references` folder.

- **`references/conventional-commit.md`**: _The abridged summary of the
  conventional commit specification._

**`ALWAYS`** study the reference file before crafting a commit message.

## Primary directives

Follow these rules when crafting, formatting, and presenting the commit
message:

### Content & style directives

- Aim for brevity and precision, but don't sacrifice clarity.
- Use bullet points to list changes.
- Optionally, precede the list of changes with a briefly summarized
  paragraph of "why" the changes occurred.

### Character limit directives

Strictly adhere to these limits to ensure readability in git logs:

- **Subject Line:** ≤50 characters recommended (Hard limit: 72)
- **Body Lines:** Wrap at 72 characters
- **Footer:** Wrap at 72 characters

### Formatting directives

- Use plain text, not markdown, for the commit messages.
- Don't use **`BACKTICKS`** or code blocks in the commit message.
- Don't use or show line numbers.
- Strictly follow the character limits per line, according to the
  `kbase` reference files.
- Respect `git` line length limits, according to the `kbase` reference
  files.

#### Avoid backticks

**`NOTE:`** _Avoid using **`BACKTICKS`** in commit messages. They can
cause shell errors. Use single or double quotes instead._

### Presentation directives

- **`NEVER`** use shell commands, like `echo`, `cat`, or similar, when
  presenting the commit message.
- Present the commit message directly as plain text in a formatted code
  block within the agent's response.

### Confirmation directives

_After_ presenting the commit message:

- Explicitly prompt for a `yes` or `no` confirmation from the user in
  the agent's direct response;
- Stop all processes and halt execution;
- If the user responds with `yes` or an affirmative, proceed with the
  commit;
- If the user responds with `no`, `cancel`, or a negative, halt the
  commit process and await further instructions;
- Ensure the prompt for confirmation concisely states that the next
  action performs the commit _if_ approved.

## Git directives

Use the `git` commands below to:

- Get a comprehensive understanding of the changes in the project;
- Get the status of the repository.

**`NOTE`:** _The `git diff` commands should always use the `--no-pager`
option and `--no-ext-diff` flag to ensure the use of the standard `diff`
and `pager` tools._

### For staged changes

```bash
git --no-pager diff --staged --no-ext-diff --stat --minimal --patience \
    --histogram --find-renames --summary --no-color -U10
```

### For `unstaged` changes

```bash
git --no-pager diff --no-ext-diff --stat --minimal --patience --histogram \
    --find-renames --summary --no-color -U10 <file_group>
```

### For repository status

```bash
git status --porcelain=v2 --branch
```

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

---

## Workflow

- **`References`:** Study the reference files.
- **`Analyze`:** Using the `git directives`, study the staged changes
  for summarization.
- **`Draft`:** Strictly following the guidelines in
  `references/conventional-commit.md` and the directives, write and
  format the complete commit message.
- **`VERIFY`:** Ensure the commit message adheres to formatting rules
  and the guidelines in the reference files. Also ensure **`NO`**
  `BACKTICKS` in commit message.
- **`Propose`:** Strictly following the directives, present the
  formatted commit message to the user.
- **`Confirm`:** Strictly following the directives, prompt for
  confirmation from the user.
- **`Halt`:** Stop all processes and halt execution;
- **`Await`:** Await the user's response before taking any further
  action.
