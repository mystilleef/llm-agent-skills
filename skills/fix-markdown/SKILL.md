---
name: fix-markdown
description:
  Fix lint, formatting, and prose issues in markdown files using
  Prettier and Vale. Use when the user or agent needs to fix lint,
  formatting, and prose issues in markdown files.
---

# Agent protocol: Fix markdown file

**Goal:** Use `prettier` and `vale` to fix lint, formatting, and prose
issues with the target markdown files.

**When:** Use when the user or agent needs to fix lint, formatting, or
improve the prose of markdown files.

## References

The following reference files serve as strict guidelines when updating
prose.

Locate the reference files in the `references` folder.

- **`references/e-prime-directive.md`**: _The E-Prime Communication
  Protocol defining the rules for avoiding `to be` verbs._

## Primary directives

Follow these rules when processing the markdown file:

### Editing directives

- **`Format First & Last:`** Always format the document before starting
  analysis and after finishing all edits.
- **`Rewrite Freely:`** Feel free to rewrite phrases to simplify
  structure, fix style issues, or address grammatical violations.
- **`E-Prime Compliance:`** Strictly follow the E-Prime directive when
  writing, updating, or correcting prose.
- **`Research:`** Use the search tool or `Perplexity` to research
  solutions if you encounter a complex lint issue.

### Tool directives

Use the commands below to manage formatting and linting.

**To format the file:**

```bash
prettier --write <file_path>
```

**To sync lint rules:**

```bash
vale sync
```

**To find lint issues:**

```bash
vale --no-wrap --output=JSON <file_path>
```

### Vale directives

Repeat the following cycle to address prose issues:

1. Run `vale` against the file in question.
2. Select issues to address.
3. Analyze the issue and plan a fix.
4. Fix the issue in the file.
5. Run `vale` again to verify the fix or find the next issue.
6. Repeat steps 1-5 until **`NO`** issues remain.
7. Execute the _Vale path directives_.

#### Vale path directives

Always wrap the following inside backticks:

- `Filenames`;
- `Uris`;
- `Urls`; and
- `Paths`, both relative and full;

For example:

```markdown
- `filename.md`
- `https://example.com`
- `/path/to/file`
```

#### Addressing common lint issues

- **`Filenames and Paths:`** Wrap all filenames, `uris`, `urls`, and
  paths, both relative and full, inside backticks.
- **`Context Awareness:`** Check the reported line number. The flagged
  issue might represent part of a hyphenated word or a `substring`
  within a technical term (for example, `LLM` inside `vLLM`).
- **`False Positives:`** Wrap words in backticks to fix false-positive
  spelling issues, acronyms, abbreviations, names, and proper nouns.
  Treat tool names containing acronyms as proper nouns.
- **`Headings:`** Capitalize only the first character in a heading to
  fix standard capitalization issues (Sentence case).
- **`Passive Voice:`** Follow the E-Prime directive to fix `to be` and
  passive voice issues.
- **`Follow Hints`:** Optionally, vale provides a link associated with
  an issue. Follow the link if needed for hints on how to fix the issue.

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

## Task management directives

For complex tasks, use your `todo` management system to:

- Break down, plan, revise, and streamline tasks;
- Maintain internal task state;
- Remove unnecessary and redundant operations;
- Optimize tool usage and workflow to fulfill the goal.

---

## Workflow

- **`Sync`:** Run `vale sync` to update local lint rules.
- **`Format`:** Run `prettier --write` to ensure a clean baseline.
- **`Path Fixes`:** Study the file, then execute the _Vale path
  directives_.
- **`Lint & Fix`:** Iteratively run `vale`, analyze issues, and apply
  fixes according to the **Vale directives**.
- **`Final Format`:** Run `prettier --write` again to ensure consistent
  formatting after edits.
- **`Halt`:** Stop all processes and halt execution.
