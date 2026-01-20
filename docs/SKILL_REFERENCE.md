# Skill Development Reference

**Purpose**: This reference guide provides patterns, protocols, and optimizations for agents creating or updating SKILL.md files in this repository. Use this document to find copy-paste ready templates, understand orchestration patterns, and avoid common mistakes.

**Target audience**: AI agents developing new skills or updating existing ones.

**How to use this guide**:
- Start with the [Quick Start Decision Tree](#2-quick-start-decision-tree) to identify the pattern you need
- Jump to relevant sections using the table of contents
- Copy templates directly from [Section 4](#4-copy-paste-templates-for-common-sections)
- Study orchestration patterns in [Section 5](#5-orchestration-patterns)
- Review anti-patterns in [Section 9](#9-anti-patterns) to avoid mistakes

**Important constraint**: Skills can only access files in their own `skills/<skill-name>/` folder at runtime. This guide serves agents during skill development, not execution. Individual skills cannot reference this document during execution.

**Related documentation**: See `CLAUDE.md` for project-level context, development commands, and repository conventions.

---

## Table of Contents

2. [Quick Start Decision Tree](#2-quick-start-decision-tree)
3. [Standard SKILL.md Structure](#3-standard-skillmd-structure)
4. [Copy-Paste Templates](#4-copy-paste-templates-for-common-sections)
5. [Orchestration Patterns](#5-orchestration-patterns)
6. [Status Code Conventions](#6-status-code-conventions)
7. [Script Integration Patterns](#7-script-integration-patterns)
8. [Git-specific Patterns](#8-git-specific-patterns)
9. [Anti-Patterns](#9-anti-patterns)
10. [Domain-Specific Guidelines](#10-domain-specific-guidelines)
11. [Token Optimization Strategies](#11-token-optimization-strategies)
12. [Testing Guidelines](#12-testing-guidelines)
13. [Quick Reference Table](#13-quick-reference-table)

---

## 2. Quick Start Decision Tree

Use this decision tree to quickly identify the correct pattern for your skill:

```
Are you creating a new skill or updating an existing one?
├─ UPDATING → Read existing skill first, identify pattern from Section 13
└─ CREATING → Continue below

Does your skill call other skills?
├─ NO → Simple Skill Pattern (Section 5.2)
│   Example: cleanup-changelog (26 lines)
│   Template: Direct workflow, 3-5 steps
│
└─ YES → Continue below

    Does execution order depend on previous results?
    ├─ NO → Sequential Orchestration (Section 5.3)
    │   Example: git-commit
    │   Features: Fixed steps, status capture, auto-loop
    │
    └─ YES → Conditional Orchestration (Section 5.4)
        Example: update-changelog
        Features: Prerequisite checks, skip steps, branching

Does your skill need complex logic (parsing, state tracking)?
├─ YES → Script Integration Pattern (Section 7)
│   Example: edit-changelog
│   Place script in scripts/ folder
│
└─ NO → Prompt-based skill
    Use workflows and directives only

Is your skill git-related?
├─ YES → Use Git-Specific Patterns (Section 8)
│   Include git commands template
│   Add git constraints
│
└─ NO → Use domain-specific guidelines (Section 10)
    Markdown: E-Prime, Vale cycles
    Changelog: Conventional commits
```

**Quick pattern selection**:
- **Simple task, no sub-skills** → Section 5.2 (Simple Skill)
- **Multiple skills, fixed order** → Section 5.3 (Sequential)
- **Multiple skills, conditional execution** → Section 5.4 (Conditional)
- **Complex parsing/logic** → Section 7 (Script Integration)

---

## 3. Standard SKILL.md Structure

All SKILL.md files follow this structure for consistency and clarity.

### 3.1 YAML Frontmatter Format

**Required format** (use exactly this structure):

```yaml
---
name: skill-name
description:
  Brief description (1-3 sentences). Use when [context]. Employs [sub-skills if orchestrator].
---
```

**Example from git-commit**:

```yaml
---
name: git-commit
description:
  Commit changes in the repository. Use when the user or agent needs to
  commit changes in the repository. Employs the "git-add" and
  "git-message" skills to perform the commit.
---
```

**Why**: YAML frontmatter provides machine-readable metadata for skill discovery and invocation.

### 3.2 Header Format (Goal/When/Note)

**Standardized format** (use backtick-wrapped keywords):

```markdown
# [Readable skill title]

**`GOAL`**: [What the skill accomplishes]

**`WHEN`**: [When to invoke this skill]

**`NOTE`**: [Key constraint or warning] (optional)
```

**Example from git-commit**:

```markdown
# Commit changes in the git repository

**`GOAL`**: use the `git-add` and `git-message` skills to commit changes
in the repository and clean it.

**`WHEN`**: use when the user or agent needs to commit changes in the
repository.

**`NOTE`:** _Await user approval of commit messages before committing.
Make commits only after the user has approved the commit message._
```

**Why backticks**: Visual distinction makes keywords scannable. Distinguishes protocol keywords from descriptive text.

### 3.3 Standard Section Order

All skills should include sections in this order (omit if not applicable):

1. **YAML Frontmatter** (required)
2. **Title with Goal/When/Note** (required)
3. **Purpose** (optional, for orchestrators or complex skills)
4. **Prerequisites** (optional, if skill has dependencies)
5. **Primary Directives** (optional, core rules)
6. **Git Directives** (if git-related)
7. **Efficiency Directives** (recommended for complex skills)
8. **Task Management** (recommended for complex skills)
9. **Workflow** (required)
10. **Output** (required)
11. **References** (if references/ folder exists)
12. **Success Criteria** (optional, for script-based skills)

**Workflow section numbering**:
- **Orchestrators**: Use `### Step N` for main steps
- **Simple skills**: Use bullet points for brevity
- **Script-based**: Describe script invocation and status handling

---

## 4. Copy-Paste Templates for Common Sections

These templates eliminate duplication and ensure consistency across skills.

### 4.1 Efficiency Directives Template

**When to use**: Any skill processing multiple files or performing operations that could be batched.

**Used in**: git-commit, git-add, git-message, git-status, fix-markdown (5 skills)

**Copy-paste template**:

```markdown
## Efficiency directives

- Batch operations on file groups, avoid individual file processing
- Use parallel execution when possible
- Target only relevant files
- Reduce token usage
```

**Why**: Batching operations reduces token usage and API calls. Processing files individually is inefficient for operations like `git diff` where multiple files can be analyzed in one command.

**Example in practice**:
- **Bad**: `git diff file1`, `git diff file2`, `git diff file3` (3 commands)
- **Good**: `git diff file1 file2 file3` (1 command)

### 4.2 Task Management Template

**When to use**: Skills involving complex multi-step workflows or decision-making.

**Used in**: git-commit, git-add, git-message, git-status, fix-markdown (5 skills)

**Copy-paste template**:

```markdown
## Task management

For complex tasks: use `todo` system to break down, plan, and optimize
workflow.
```

**Expanded version** (use for skills with complex internal state):

```markdown
## Task management directives

For complex tasks, use your `todo` management system to:

- Break down, plan, revise, and streamline tasks;
- Maintain internal task state;
- Remove unnecessary and redundant operations;
- Optimize tool usage and workflow to fulfill the goal.
```

**Why**: Explicit task management directive ensures agents use the TodoWrite tool for complex operations, improving visibility and preventing forgotten steps.

### 4.3 Git Directives Templates

Three variants depending on skill needs.

#### 4.3.1 Full Git Directives (for staging/analysis skills)

**When to use**: Skills that analyze changes, stage files, or generate commit messages.

**Used in**: git-add, git-message

**Copy-paste template**:

```markdown
## Git directives

Git commands (use `--no-pager` and `--no-ext-diff` for diffs):

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
```

**Why these flags**:
- `--no-pager`: Prevents interactive paging
- `--no-ext-diff`: Uses standard diff format
- `--minimal`: Minimizes diff output
- `--patience --histogram`: Better diff algorithms for readability
- `--find-renames`: Detects file renames
- `-U10`: Shows 10 lines of context (better than default 3 for LLMs)
- `--no-color`: Prevents ANSI color codes

#### 4.3.2 Minimal Git Directives (for status-only skills)

**When to use**: Skills that only check repository status.

**Used in**: git-status

**Copy-paste template**:

```markdown
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
```

**Why minimal**: Status-only skills don't need diff commands, keeping the skill file concise.

#### 4.3.3 Primary Git Constraints (for staging skills)

**When to use**: Skills that modify git state (staging, committing).

**Used in**: git-add

**Copy-paste template**:

```markdown
## Primary directives

- Study `references/` files `ONLY` if more than one file exists to
  stage.

- **`NEVER`** use `git add .` or `git add -A`.
- **`NEVER`** use `git checkout` or `git restore`.
- Assume a file in both `staged` and `unstaged` states needs staging.
- Stage files explicitly: `git add <file1> <file2> ...`.
- Leave unrelated changes `unstaged`.
- Isolate `.gitignore` changes; stage them in a separate commit.
- Except for `.gitignore`, **`DON'T`** edit any files
```

**Why these constraints**:
- **Never `git add .`**: Prevents staging unrelated changes, maintains atomic commits
- **Never `git checkout/restore`**: Prevents accidental data loss
- **Explicit staging**: Ensures visibility and control over what gets committed
- **Isolate .gitignore**: Keeps config changes separate from content changes

### 4.4 References Section Template

**When to use**: Skills with references/ folder containing supporting documentation.

**Copy-paste template**:

```markdown
## References

The following reference files serve as strict guidelines:

- **`references/filename.md`**: Brief description of what guidance this file provides
- **`references/another-file.md`**: Brief description
```

**Example from git-status**:

```markdown
## References

The following reference files serve as strict guidelines to study for
presenting the repository status in a visually appealing way to the
user.

Locate the reference files in the `references` folder.

- **`references/git-status-codes.md`**: Complete reference for parsing
  git status output
- **`references/git-status-presentation.md`**: Git status presentation
  guidelines, examples, and templates
```

**Why**: Separates detailed specifications from the main SKILL.md, keeping the skill file concise while providing depth when needed.

### 4.5 Output Section Template

**When to use**: All skills (required section).

**Standard format for simple skills**:

```markdown
## Output

**Files created/modified:**

- `filename` - Description of what this file contains
- `another-file` - Description

**Status communication:**

Describes what the skill reports to the user or next orchestrator.
```

**Standard format for script-based skills**:

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

**Following lines (when SUCCESS):** Additional details about what changed
```

**Example from git-add**:

```markdown
## Output

**Files modified:**

- Staging area - Files added to git staging area

**Status communication:**

First line of output indicates status:

- `SUCCESS: staged N files for atomic commit` - Files staged
  successfully
- `WARN: no files available to stage` - No `unstaged` changes found
- `ERROR: [message]` - Failed to stage files

**Following lines (when SUCCESS):** bullet point list of staged files
```

**Why explicit status codes**: Enables orchestrators to make decisions based on outcomes. The "first line" convention provides a parseable protocol.

---

## 5. Orchestration Patterns

This section documents the three primary patterns for skill composition.

### 5.1 Pattern Selection Decision Tree

```
Does your skill call other skills?
├─ NO → Simple Skill Pattern (Section 5.2)
│   Example: cleanup-changelog
│   Lines: 20-30
│   Structure: Direct workflow
│
└─ YES → Orchestrator Pattern

    Does execution order depend on previous results?
    ├─ NO → Sequential Orchestration (Section 5.3)
    │   Example: git-commit
    │   Features: Fixed steps, status capture, auto-loop
    │   Lines: 100-120
    │
    └─ YES → Conditional Orchestration (Section 5.4)
        Example: update-changelog
        Features: Prerequisite checks, conditional steps
        Lines: 100-120
```

### 5.2 Simple Skill Pattern

**Definition**: A skill that performs a single responsibility without calling other skills.

**When to use**:
- Skill performs one cohesive task
- No sub-skill coordination needed
- Workflow has 3-5 clear steps
- No complex branching logic

**Example**: cleanup-changelog (26 lines total)

**Characteristics**:
- Direct, linear workflow
- Bullet points for steps (not numbered headers)
- Minimal sections (Goal/When, Workflow, Output)
- Can invoke scripts but doesn't orchestrate skills

**Template**:

```markdown
---
name: skill-name
description:
  Brief description. Use when [context].
---

# [Readable title]

**`GOAL`**: [What the skill accomplishes]

**`WHEN`**: [When to invoke this skill]

## Workflow

- Step 1 action
- Step 2 action
- Step 3 action
- **`DONE`**

## Output

**Files created/modified:**

- `filename` - Description

**Status communication:**

Brief description of what the skill reports.
```

**Complete example from cleanup-changelog** (`skills/cleanup-changelog/SKILL.md`):

```markdown
---
name: cleanup-changelog
description:
  Use the `fix-markdown` skill to clean up CHANGELOG.md, then remove
  headers with empty sections. Use after the agent has finished
  generating or updating the changelog.
---

# Cleanup changelog

**`GOAL`**: fix lint, formatting, and prose issues in `CHANGELOG.md`.

**`WHEN`**: use after the agent has finished generating or updating the
changelog.

## Remove empty headers script

Run the `scripts/remove-empty-headers.sh` script against `CHANGELOG.md`
to remove headers with empty sections.

## Workflow

- Use the `fix-markdown` skill to fix `CHANGELOG.md`
- Run the script to remove headers with empty sections in `CHANGELOG.md`
- Run `fix-markdown` skill again to ensure proper formatting
- **`DONE`**
```

**Why this pattern works**:
- Brevity: Only 26 lines for complete skill definition
- Clarity: Simple workflow is easy to understand and execute
- Token-efficient: Minimal context needed for execution

### 5.3 Sequential Orchestration Pattern

**Definition**: An orchestrator that invokes multiple sub-skills in a fixed sequence, capturing status at each step but continuing regardless (unless ERROR).

**When to use**:
- Multiple skills must execute in a specific order
- Each skill's status informs next actions but doesn't block
- Automatic loop control needed
- Some steps have conditional logic within them

**Example**: git-commit (103 lines)

**Characteristics**:
- Uses `### Step N` headers for main workflow steps
- Captures status after each sub-skill invocation
- Fixed sequence but conditional branching within steps
- Automatic loop detection (Step 5 in git-commit)
- Status-aware error handling at each step

**Template**:

```markdown
---
name: skill-name
description:
  Brief description. Employs the "sub-skill-1" and "sub-skill-2" skills
  to [accomplish goal].
---

# [Readable title]

**`GOAL`**: use the `sub-skill-1` and `sub-skill-2` skills to
[accomplish goal].

**`WHEN`**: use when [context].

**`NOTE`:** [Any critical constraints]

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

## Output

**Files created:**

- [Description of artifacts]

**Status communication:**

Reports the following at each step:

- **Step 1:** [What gets reported]
- **Step 2:** Status from sub-skill-1
- **Step 3:** Status from sub-skill-2
- **Step 4:** [Operation result]
- **Step 5:** [Loop detection info]
- **Step 6:** [Final summary]

## Efficiency directives

[Copy from Section 4.1]

## Task management

[Copy from Section 4.2]
```

**Complete example from git-commit** (`skills/git-commit/SKILL.md:20-92`):

```markdown
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
```

**Key features explained**:

1. **Fixed sequence with branching**: Steps execute in order (1→2→3→4→5→6), but internal logic allows skipping (Step 1 can jump to Step 6) or looping (Step 3 can loop to Step 2 or itself).

2. **Status capture pattern**: Every sub-skill invocation explicitly captures status: "Capture the status from skill output (`SUCCESS`, `WARN`, or `ERROR`)."

3. **Explicit status handling**: Three-way branching on status:
   - `ERROR`: Always halts
   - `WARN`: Context-specific (may skip steps)
   - `SUCCESS`: Continues to next step

4. **Automatic loop control**: Step 5 automatically loops back to Step 2 without user prompt. This is **automatic loop control** - the skill detects remaining work and continues.

5. **Extended status codes**: Step 3 shows extended status codes beyond the standard three (`APPROVED`, `REJECTED_*`), demonstrating how skills can define domain-specific statuses.

**Why numbered steps**: The `### Step N` headers provide clear checkpoints for status capture and make the flow easy to follow despite complexity.

### 5.4 Conditional Orchestration Pattern

**Definition**: An orchestrator that invokes sub-skills based on prerequisites, skipping steps conditionally based on status or state.

**When to use**:
- Execution depends on prerequisites or current state
- Some steps should only run if previous steps succeeded
- Graceful degradation needed (skip optional steps)
- Branching logic at orchestrator level, not just within steps

**Example**: update-changelog (105 lines)

**Characteristics**:
- Prerequisite checks before invoking sub-skills
- Conditional step execution ("Only run this step if...")
- Graceful handling of WARN status (skip optional cleanup)
- Status-driven decision making at orchestrator level

**Template**:

```markdown
---
name: skill-name
description:
  Orchestrate [workflow]. Invoke `sub-skill-1` (if needed),
  `sub-skill-2`, and `sub-skill-3` skills in sequence.
---

# [Readable title]

**`GOAL`**: orchestrate [workflow] by invoking [sub-skills] in sequence.

**`WHEN`**: use when [context].

## Purpose

This skill provides [high-level purpose]:

- [Capability 1] (via `sub-skill-1`)
- [Capability 2] (via `sub-skill-2`)
- [Capability 3] (via `sub-skill-3`)
- Handles dependencies and conditional logic intelligently

## Prerequisites

- [Prerequisite 1]
- [Prerequisite 2]

## Workflow

Follow these steps in sequence:

### Step 1: [Check prerequisite]

- Check if [prerequisite exists].
- If missing: Invoke the `sub-skill-init` skill to create it.
  - Verify the output status (`SUCCESS`, `WARN`, or `ERROR`).
  - If `ERROR`: Stop and report the error to the user.
  - If `SUCCESS` or `WARN`: Continue to Step 2.
- If exists: Continue directly to Step 2.

### Step 2: [Main operation]

- Invoke the `sub-skill-main` skill to [accomplish main goal].
- Capture the status from the first line of output (`SUCCESS`, `WARN`,
  or `ERROR`).
- Handle the status:
  - If `ERROR`: Stop and report the error to the user.
  - If `WARN` ([reason]): Report to user. Skip Step 3 and proceed to
    Step 4.
  - If `SUCCESS`: Continue to Step 3.

### Step 3: [Optional cleanup] (conditional)

- Only run this step if [previous step] reported `SUCCESS`.
- Invoke the `sub-skill-cleanup` skill to [accomplish cleanup goal].
- This step [describe what it does].

### Step 4: [Report completion]

- Communicate a summary to the user indicating:
  - What actions the skill performed
  - [Key outcomes]
  - Any warnings or errors encountered during the process
- **`DONE`**

## Behavior

**Smart initialization:**

- Automatically invokes `sub-skill-init` if [prerequisite missing]
- Skips initialization if [prerequisite exists]

**Conditional cleanup:**

- Only runs `sub-skill-cleanup` if [previous step] made changes
  (`SUCCESS` status)
- Skips cleanup if [previous step] requires no changes (`WARN` status)

**Error handling:**

- Stops immediately on `ERROR` from any sub-skill
- Reports errors to the user

## Output

**Files created/modified:**

- `file1` - Created (if missing) or updated
- `file2` - Updated to [state]

**Status communication:**

- Reports initialization status when invoking `sub-skill-init`
- Reports operation status from `sub-skill-main`
- Reports cleanup status when invoking `sub-skill-cleanup`
- Provides final summary of all actions taken
```

**Complete example from update-changelog** (`skills/update-changelog/SKILL.md:37-72`):

```markdown
## Workflow

Follow these steps in sequence:

### Step 1: Check for `CHANGELOG.md`

- Check if `CHANGELOG.md` exists in the repository root.
- If missing: Invoke the `init-changelog` skill to create it.
  - Verify the output status (`SUCCESS`, `WARN`, or `ERROR`).
  - If `ERROR`: Stop and report the error to the user.
  - If `SUCCESS` or `WARN`: Continue to Step 2.
- If exists: Continue directly to Step 2.

### Step 2: Update from git commits

- Invoke the `edit-changelog` skill to update from git commits.
- Capture the status from the first line of output (`SUCCESS`, `WARN`,
  or `ERROR`).
- Handle the status:
  - If `ERROR`: Stop and report the error to the user.
  - If `WARN` (no new commits): Report to user that changelog already
    reflects latest commits. Skip Step 3 and proceed to Step 4.
  - If `SUCCESS` (changes made): Continue to Step 3.

### Step 3: Cleanup formatting (conditional)

- Only run this step if `edit-changelog` reported `SUCCESS`.
- Invoke the `cleanup-changelog` skill to clean up formatting.
- This step runs: `fix-markdown` → `remove-empty-headers.sh` →
  `fix-markdown`

### Step 4: Report completion

- Communicate a summary to the user indicating:
  - What actions the skill performed
  - Whether the changelog received updates
  - Any warnings or errors encountered during the process
- **`DONE`**
```

**Key features explained**:

1. **Prerequisite verification**: Step 1 checks if CHANGELOG.md exists before proceeding. If missing, invokes initialization sub-skill conditionally.

2. **Smart initialization**: Only initializes if needed, avoiding unnecessary operations.

3. **Conditional step execution**: Step 3 explicitly states "Only run this step if `edit-changelog` reported `SUCCESS`". This is the defining characteristic of conditional orchestration.

4. **Graceful degradation**: If Step 2 reports WARN (no new commits), the workflow skips cleanup and jumps to completion. No error, just intelligent skipping.

5. **Status-driven flow**: The `WARN` status in Step 2 triggers different behavior than `SUCCESS` - it causes Step 3 to be skipped entirely.

**Behavior section**: Conditional orchestrators often include a "Behavior" section that explicitly documents the smart logic:

```markdown
## Behavior

**Smart initialization:**
- Automatically invokes `init-changelog` if `CHANGELOG.md` does not exist
- Skips initialization if `CHANGELOG.md` already exists

**Conditional cleanup:**
- Only runs `cleanup-changelog` if `edit-changelog` made changes (`SUCCESS` status)
- Skips cleanup if the update requires no changes (`WARN` status)
```

**Why this pattern works**:
- **Efficiency**: Skips unnecessary operations based on state
- **Robustness**: Handles missing prerequisites gracefully
- **Flexibility**: Adapts workflow to actual needs rather than always running all steps

**Comparison with Sequential**:
- **Sequential**: All steps run in fixed order (unless ERROR halts)
- **Conditional**: Steps may be skipped based on status or prerequisites

---

## 6. Status Code Conventions

Status codes provide a standardized protocol for communication between skills and orchestrators.

### 6.1 Three-Level Status Codes (Standard)

The repository uses a three-level status system for all skills and scripts.

**Status codes**:

1. **`SUCCESS`**: Operation completed successfully and made changes
2. **`WARN`**: Operation completed successfully but no changes were needed
3. **`ERROR`**: Operation failed, user intervention required

**Format**: Always output status on the **first line** of output, followed by descriptive message:

```
STATUS: descriptive message explaining what happened
```

**Examples**:

```bash
SUCCESS: Staged 5 files for atomic commit
SUCCESS: Changelog updated with 12 entries (3 Added, 7 Fixed, 2 Changed).
WARN: No files available to stage
WARN: No new commits to process. Changelog is already up to date.
ERROR: CHANGELOG.md doesn't exist. Please run 'init-changelog' first.
ERROR: Failed to stage files - git command failed
```

**Why three levels**:
- **SUCCESS vs WARN**: Distinguishes "work done" from "nothing to do". Orchestrators can skip optional cleanup steps when WARN is received.
- **ERROR**: Unambiguous failure signal. Orchestrators always halt on ERROR.

**Implementation in bash scripts**:

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

**Why exit 0 for WARN**: WARN is not a failure. The script completed successfully, it just had no work to do. Exit code 1 is reserved for actual errors.

### 6.2 Extended Status Codes (Domain-Specific)

Skills can define extended status codes beyond the standard three when domain logic requires it.

**Example from git-message** (user approval workflow):

```
APPROVED: [commit message follows]
REJECTED_EDIT_FILES: user wants to modify staged files
REJECTED_REGENERATE: user wants new message
REJECTED_ABORT: user cancelled commit
ERROR: [message]
```

**Why extended codes**: The approval workflow has four distinct outcomes after user interaction:
1. **APPROVED**: User accepts message, proceed with commit
2. **REJECTED_EDIT_FILES**: User wants to change staged files, unstage and return to staging
3. **REJECTED_REGENERATE**: User wants a different message, regenerate
4. **REJECTED_ABORT**: User cancels entire workflow, halt

**From git-message SKILL.md** (`skills/git-message/SKILL.md:108-118`):

```markdown
## Output

**Status communication:**

First line of output indicates user's decision:

- `APPROVED: [commit message follows]` - user approved message
- `REJECTED_EDIT_FILES: user wants to modify staged files` - User wants
  to re-stage
- `REJECTED_REGENERATE: user wants new message` - User wants message
  regenerated
- `REJECTED_ABORT: user cancelled commit` - User aborted process
- `ERROR: [message]` - Failed to generate message
```

**How orchestrators use extended codes** (from git-commit):

```markdown
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
```

**When to define extended codes**:
- User interaction with multiple outcomes
- Complex state transitions
- Domain-specific workflows that don't map to SUCCESS/WARN/ERROR

**Naming convention**: Use descriptive names with context:
- **Good**: `REJECTED_EDIT_FILES`, `REJECTED_REGENERATE`
- **Bad**: `REJECTED_1`, `REJECTED_2`

### 6.3 Status Communication Protocol

All skills must follow this protocol for status communication:

**Rule 1: First line contains status**

The **first line** of output MUST be the status code followed by descriptive message:

```
STATUS: descriptive message
```

**Why**: Enables orchestrators to parse status reliably. Scripts and skills can output additional detail on subsequent lines, but the first line is sacred.

**Rule 2: Consistent format**

Always use the format `STATUS: message` with:
- Status code in UPPERCASE
- Colon and space after status
- Descriptive message explaining the status

**Rule 3: Additional detail on subsequent lines**

After the first line, skills can output additional details:

```
SUCCESS: Staged 5 files for atomic commit
- skills/git-add/SKILL.md
- skills/git-message/SKILL.md
- skills/git-status/SKILL.md
- skills/git-commit/SKILL.md
- SKILL_REFERENCE.md
```

**How orchestrators capture status** (pattern from all orchestrators):

```markdown
- Invoke the `sub-skill` skill to accomplish goal.
- Capture the status from the first line of output.
- Handle the status:
  - If `ERROR`: Halt and report the error.
  - If `WARN`: [Context-specific handling].
  - If `SUCCESS`: Continue to next step.
```

**Example implementation** (from edit-changelog script):

```bash
# Success with detail
echo "SUCCESS: Changelog updated with $TOTAL_ENTRIES entries ($CATEGORIES)."

# Output format when script runs:
# SUCCESS: Changelog updated with 12 entries (3 Added, 7 Fixed, 2 Changed).
```

### 6.4 Status Code Examples from Scripts

Real examples from `skills/edit-changelog/scripts/edit-changelog.sh`:

**WARN - No work to do**:

```bash
if [ -z "$NEW_COMMITS" ]; then
  echo "WARN: No new commits to process. Changelog is already up to date."
  exit 0
fi
```

**WARN - Pointer initialization**:

```bash
if [ ! -f "$POINTER_FILE" ]; then
  git rev-parse HEAD > "$POINTER_FILE"
  echo "WARN: Initialized pointer to HEAD. No changes processed."
  exit 0
fi
```

**ERROR - Missing prerequisite**:

```bash
if [ ! -f "$CHANGELOG_FILE" ]; then
  echo "ERROR: $CHANGELOG_FILE doesn't exist. Please run 'init-changelog' first."
  exit 1
fi
```

**SUCCESS - Work completed**:

```bash
echo "SUCCESS: Changelog updated with $TOTAL_ENTRIES entries ($CATEGORIES)."
```

**Why these examples matter**: They show how WARN is used for "graceful no-op" situations (nothing to do, already initialized), while ERROR is reserved for actual problems (missing files, broken state).

---

## 7. Script Integration Patterns

Some skills delegate complex logic to shell scripts. This section documents when and how to integrate scripts.

### 7.1 When to Use Scripts vs Prompts

**Decision criteria**:

| Use Scripts When | Use Prompts When |
|------------------|------------------|
| Complex regex parsing | LLM reasoning needed |
| Associative arrays needed | Context-aware decisions |
| Stateful operations (pointer files) | User interaction required |
| Repetitive logic (loops over data structures) | Adapting to edge cases |
| Performance-critical operations | Handling ambiguity |
| Deterministic algorithms | Natural language processing |

**Examples from repository**:

**Script-based** (edit-changelog):
- Parsing conventional commit messages (regex)
- Deduplicating by issue references (associative arrays)
- Tracking pointer files (stateful)
- Categorizing commits (deterministic logic)

**Prompt-based** (git-message):
- Drafting commit message (LLM reasoning)
- Analyzing changes for context (understanding intent)
- Awaiting user approval (interaction)
- Handling user edits (adapting to feedback)

**Why this matters**: Scripts are ~10x faster for deterministic operations and use zero tokens. Prompts are essential for tasks requiring understanding, reasoning, or user interaction.

### 7.2 Script Status Propagation Pattern

When a skill invokes a script, it must capture and propagate the status to orchestrators.

**Standard pattern**:

```markdown
## Workflow

1. Run `scripts/script-name.sh`
2. Capture status from first line of output
3. Handle the status:
   - If `ERROR`: Stop and report to user
   - If `WARN`: [Context-specific action]
   - If `SUCCESS`: Continue with next step
```

**Example from edit-changelog** (`skills/edit-changelog/SKILL.md:45-47`):

```markdown
## Workflow

- Run `scripts/edit-changelog.sh`
- Verify `CHANGELOG.md` section updates
- Communicate success or failure to user
```

**Simplified because**: The skill is thin wrapper around script. The script handles all logic and status communication. The skill just invokes it.

**Example from update-changelog** (orchestrator handling script-based skill):

```markdown
### Step 2: Update from git commits

- Invoke the `edit-changelog` skill to update from git commits.
- Capture the status from the first line of output (`SUCCESS`, `WARN`,
  or `ERROR`).
- Handle the status:
  - If `ERROR`: Stop and report the error to the user.
  - If `WARN` (no new commits): Report to user that changelog already
    reflects latest commits. Skip Step 3 and proceed to Step 4.
  - If `SUCCESS` (changes made): Continue to Step 3.
```

**Why explicit status handling**: Even though edit-changelog is script-based, the orchestrator (update-changelog) treats it the same as any other skill - capture status, handle accordingly.

### 7.3 Essential Script Patterns

Key patterns found in all script-based skills.

#### 7.3.1 Bash Version Checking

**When to use**: Scripts using associative arrays (requires Bash 4.0+).

**Pattern** (from edit-changelog.sh:3-7):

```bash
# Check for Bash 4.0+ (required for associative arrays)
if ((BASH_VERSINFO[0] < 4)); then
  echo "ERROR: Bash 4.0 or higher is required."
  exit 1
fi
```

**Why**: Associative arrays (`declare -A`) don't exist in Bash 3.x. Failing early with clear message prevents cryptic errors.

#### 7.3.2 Prerequisite Verification (Early Exit)

**When to use**: All scripts. Verify prerequisites before doing any work.

**Pattern** (from edit-changelog.sh:26-45):

```bash
# 1. Verify prerequisites
if [ ! -f "$CHANGELOG_FILE" ]; then
  echo "ERROR: $CHANGELOG_FILE doesn't exist. Please run 'init-changelog' first."
  exit 1
fi

# Validate CHANGELOG.md structure
if ! grep -q '^# Changelog' "$CHANGELOG_FILE"; then
  echo "ERROR: $CHANGELOG_FILE appears malformed (missing '# Changelog' header)"
  exit 1
fi

if ! git rev-parse --git-dir > /dev/null 2>&1; then
  echo "ERROR: Current directory is not a git repository."
  exit 1
fi
```

**Why early exit**: Fail fast with clear error message. Don't waste time on operations that will fail anyway.

**WARN for initialization** (from edit-changelog.sh:48-52):

```bash
# 2. Read last aggregated commit
if [ ! -f "$POINTER_FILE" ]; then
  git rev-parse HEAD > "$POINTER_FILE"
  echo "WARN: Initialized pointer to HEAD. No changes processed."
  exit 0
fi
```

**Why WARN not ERROR**: Missing pointer file is recoverable - initialize and exit gracefully. Next run will process commits.

#### 7.3.3 Temporary File Management

**When to use**: Scripts creating intermediate files during processing.

**Pattern** (from edit-changelog.sh:22-23):

```bash
# Create secure temporary directory
TEMP_DIR=$(mktemp -d)
trap 'rm -rf "$TEMP_DIR"' EXIT
```

**Why**:
- **`mktemp -d`**: Creates unique temp directory, avoids collisions
- **`trap ... EXIT`**: Ensures cleanup even if script fails or is interrupted
- **Security**: Prevents temp file race conditions

**Usage in script**:

```bash
# Later in script...
sed -n '/^## /q;p' "$CHANGELOG_FILE" > "$TEMP_DIR/changelog_header.txt"
sed -n '/^## \[[0-9]/,$p' "$CHANGELOG_FILE" > "$TEMP_DIR/changelog_versions.txt"
```

#### 7.3.4 Debug Mode

**When to use**: Scripts with complex logic that may need troubleshooting.

**Pattern** (from edit-changelog.sh:12-19):

```bash
# Configuration
VERBOSE=${VERBOSE:-false}

# Debug function for verbose output
debug() {
  if [ "$VERBOSE" = "true" ]; then
    echo "DEBUG: $*" >&2
  fi
}
```

**Usage**:

```bash
debug "Processing commit: ${commit:0:7} - $SUBJECT"
debug "  Matched type: $TYPE, breaking: ${BREAKING:-no}"
```

**How to enable**:

```bash
VERBOSE=true bash scripts/edit-changelog.sh
```

**Why stderr**: Debug output goes to stderr (`>&2`), keeping stdout clean for status protocol. Orchestrators only capture stdout.

### 7.4 Pointer File Pattern

A specialized pattern for tracking incremental processing state.

**Definition**: A pointer file stores the last processed commit SHA, enabling incremental updates.

**Example**: `.last-aggregated-commit` in changelog skills

**Purpose**: Track which commits have been processed to avoid reprocessing entire history.

**Initialization** (smart detection from edit-changelog.sh:48-60):

```bash
# 2. Read last aggregated commit
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
```

**Why this works**:
1. **First run**: Pointer doesn't exist, initialize to HEAD, exit with WARN
2. **Second run**: Pointer exists, process commits since HEAD (first run)
3. **Corrupted pointer**: SHA invalid, reinitialize and exit gracefully
4. **Self-healing**: Script recovers from corruption automatically

**Update pattern** (edit-changelog.sh:256):

```bash
# 9. Update pointer (only on success)
git rev-parse HEAD > "$POINTER_FILE"
```

**Always update to HEAD**: After successful processing, update pointer to current HEAD so next run processes from here.

**Initial pointer placement** (from init-changelog script logic):

```bash
# Smart pointer initialization
COMMIT_COUNT=$(git rev-list --count HEAD)
if [ "$COMMIT_COUNT" -le 100 ]; then
  # Small repo: set to first commit (enables full backfill)
  git rev-list --max-parents=0 HEAD > "$POINTER_FILE"
else
  # Large repo: set to HEAD~100 (recent backfill, avoid slow first run)
  git rev-parse HEAD~100 > "$POINTER_FILE"
fi
```

**Why smart initialization**: Balances between full history (desirable for new changelogs) and performance (don't process 10,000 commits on first run).

---

## 8. Git-Specific Patterns

Skills interacting with git repositories follow these patterns and constraints.

### 8.1 Standard Git Commands (Copy-Paste Ready)

These commands appear across multiple git skills. Copy directly.

**For unstaged changes** (from git-add.sh:39-41):

```bash
git --no-pager diff --no-ext-diff --stat --minimal --patience --histogram \
    --find-renames --summary --no-color -U10 <file_group>
```

**For staged changes** (from git-add.sh:46-48):

```bash
git --no-pager diff --staged --no-ext-diff --stat --minimal --patience \
    --histogram --find-renames --summary --no-color -U10
```

**For repository status** (from git-add.sh:53):

```bash
git status --porcelain=v2 --branch
```

**For staging files** (from git-add.sh:58):

```bash
git add <file1> <file2> ...
```

**For commit history** (for changelog scripts):

```bash
git log "$LAST_COMMIT"..HEAD --reverse --pretty=format:%H
```

**Flag explanations**:

| Flag | Purpose | Why |
|------|---------|-----|
| `--no-pager` | Disables interactive paging | Prevents hanging on large outputs |
| `--no-ext-diff` | Uses standard diff format | Prevents custom diff tools from interfering |
| `--stat` | Shows change statistics | Provides summary of changes |
| `--minimal` | Minimizes diff output | Reduces token usage |
| `--patience` | Better diff algorithm | More readable diffs for LLMs |
| `--histogram` | Improved diff algorithm | Complements patience algorithm |
| `--find-renames` | Detects file renames | Shows renames instead of delete+add |
| `--summary` | Shows file creation/deletion | Additional context for changes |
| `--no-color` | Removes ANSI color codes | Prevents color codes in LLM context |
| `-U10` | Shows 10 lines of context | More context than default 3 lines |
| `--porcelain=v2` | Machine-readable format | Parseable output for scripts |

### 8.2 Git Constraints Checklist

All git skills MUST follow these constraints. Copy this checklist:

**NEVER**:
- ❌ Use `git add .` or `git add -A` (stages everything indiscriminately)
- ❌ Use `git checkout` or `git restore` (risk of data loss)
- ❌ Edit files except `.gitignore` (staging skills should only stage, not modify)
- ❌ Push to remote without explicit user request
- ❌ Use `git commit --amend` without explicit user request
- ❌ Skip commit message approval (user must approve before committing)

**ALWAYS**:
- ✅ Stage files explicitly: `git add file1 file2 file3`
- ✅ Isolate `.gitignore` changes in separate commits
- ✅ Await user approval before committing
- ✅ Use atomic commits (single logical change per commit)
- ✅ Capture status from sub-skills and handle explicitly

**Why these constraints**:

**Never `git add .` or `git add -A`**:
- **Problem**: Stages all changes including unrelated files
- **Impact**: Violates atomic commit principle
- **Solution**: Always stage explicitly by filename

**Never `git checkout/restore`**:
- **Problem**: Can discard uncommitted changes permanently
- **Impact**: Data loss for user
- **Solution**: Use `git restore --staged` only for unstaging (never restore working tree)

**Always await approval**:
- **Problem**: LLM-generated commit messages may not match user intent
- **Impact**: Poor git history, requires amending commits
- **Solution**: Present message, offer 4 options (approve/edit/regenerate/abort)

**Always isolate .gitignore**:
- **Problem**: .gitignore changes are meta-changes, not content changes
- **Impact**: Pollutes atomic commits with configuration changes
- **Solution**: Stage .gitignore separately, commit before content changes

### 8.3 Git Skill Sub-patterns

Three specialized patterns for git skills.

#### 8.3.1 Atomic Staging Pattern

**Definition**: Stage the smallest cohesive group of related changes for one commit.

**Example from git-add** (`skills/git-add/SKILL.md:80-92`):

```markdown
## Workflow

**Exception:** If exactly one tracked `unstaged` file exists, stage it
immediately and skip steps below.

1. Study references files
2. Review repository status to identify all modified/new/`untracked`
   files
3. Update `.gitignore` following reference guidelines
4. Select smallest group of `unstaged` files forming single logical
   change
5. Stage the atomic group
6. Output status as first line, then bullet list of staged files
```

**Single-file shortcut** (line 82-83):

```markdown
**Exception:** If exactly one tracked `unstaged` file exists, stage it
immediately and skip steps below.
```

**Why**: When only one file is changed, no grouping logic needed. Stage it and continue. Saves tokens.

**Atomic grouping** (line 89):

```markdown
4. Select smallest group of `unstaged` files forming single logical
   change
```

**How to identify atomic groups**:
- Files changed together for same feature
- Files that would be described in single commit message
- Files in same module/component
- Files with same conventional commit type

**Non-atomic example** (bad):

```
- src/auth/login.js (feat: add OAuth)
- src/ui/button.css (fix: button alignment)
- README.md (docs: update install instructions)
```

**Atomic example** (good - first commit):

```
- src/auth/login.js (feat: add OAuth)
- src/auth/oauth-provider.js (feat: add OAuth)
- src/auth/types.ts (feat: add OAuth)
```

#### 8.3.2 Commit Message Generation Pattern

**Definition**: Analyze staged changes, draft conventional commit message, present for approval.

**From git-message** (`skills/git-message/SKILL.md:90-99`):

```markdown
## Workflow

1. Study `references/conventional-commit.md`
2. Analyze staged changes via git directives
3. Draft and format complete commit message
4. Verify formatting rules and **NO BACKTICKS**
5. Present message and prompt for confirmation with 4 options
6. Await user response before further action
```

**Key requirements**:

**Study reference first** (line 1): Don't guess conventional commit format, read the specification.

**Analyze via git directives** (line 2): Use `git diff --staged` to see what's being committed.

**Verify NO BACKTICKS** (line 4): Markdown backticks cause shell errors in commit messages. Plain text only.

**4-option approval** (line 5):

From skills/git-message/SKILL.md:46-51:

```markdown
_After_ presenting the commit message, offer 4 options:

1. **Approve and commit** - Proceed with commit
2. **Edit staged files** - `Unstage` and return to staging
3. **Regenerate message** - Generate new message
4. **Abort commit process** - Cancel workflow
```

**Why 4 options**: Covers all user intentions:
1. Message is good → commit
2. I staged wrong files → go back to staging
3. Message is bad → try again
4. Cancel everything → abort

**Conventional commit format** (from references):

```
type(scope): subject

Body paragraph explaining the "why" not the "what".

Footer with issue references or breaking changes.
```

#### 8.3.3 Status Presentation Pattern

**Definition**: Present repository state to user in human-readable format.

**From git-status** (`skills/git-status/SKILL.md:22-33`):

```markdown
## References

The following reference files serve as strict guidelines to study for
presenting the repository status in a visually appealing way to the
user.

Locate the reference files in the `references` folder.

- **`references/git-status-codes.md`**: Complete reference for parsing
  git status output
- **`references/git-status-presentation.md`**: Git status presentation
  guidelines, examples, and templates
```

**User-facing only** (skills/git-status/SKILL.md:18-20):

```markdown
**`NOTE`:** _Use this skill **only** to present the status of the
repository to the user. The agent **shouldn't** use this skill for
internal use._
```

**Why user-facing only**: Agents should use `git status --porcelain=v2` directly for internal logic (faster, machine-readable). The git-status skill formats output for human consumption, adding visual presentation overhead.

**When to use**:
- Final step of git-commit workflow (show final state)
- User explicitly requests status check
- After batch operations (show what changed)

**When NOT to use**:
- Internal checks (use git commands directly)
- Condition evaluation (parse porcelain output)
- Loop control (checking if more work remains)

---

## 9. Anti-Patterns

Learn from common mistakes. Each anti-pattern includes why it's problematic and how to fix it.

### 9.1 The God Skill

**Problem**: Single skill handling multiple concerns that should be separate.

**Example** (hypothetical bad skill):

```markdown
# Git Workflow Skill

**GOAL**: Stage files, generate commit message, commit, and update changelog.

## Workflow

1. Stage all files with git add .
2. Generate commit message
3. Commit changes
4. Update CHANGELOG.md
5. Done
```

**Why it's bad**:
- Violates single responsibility principle
- Can't reuse staging logic without committing
- Can't reuse commit message generation independently
- No flexibility in workflow order
- Difficult to test individual pieces

**Solution**: Break into orchestrator + focused sub-skills.

**Good design** (actual repository structure):

```
git-commit (orchestrator)
  ├── git-add (atomic staging)
  ├── git-message (message generation)
  └── git-status (presentation)

update-changelog (orchestrator)
  ├── init-changelog (initialization)
  ├── edit-changelog (update from commits)
  └── cleanup-changelog (formatting)
```

**Why this works**:
- Each sub-skill has one job
- Sub-skills are reusable independently
- Orchestrator provides workflow flexibility
- Easy to test each piece

**Rule of thumb**: If your skill does more than one verb (stage AND commit AND update), split it.

### 9.2 Implicit Dependencies

**Problem**: Assuming state without verification.

**Example** (from hypothetical bad edit-changelog):

```bash
# Bad: assume CHANGELOG.md exists
LAST_COMMIT=$(cat .last-aggregated-commit)
git log "$LAST_COMMIT"..HEAD > /tmp/commits
# Process commits...
```

**Why it's bad**:
- Crashes if CHANGELOG.md doesn't exist
- Cryptic error if pointer file missing
- No guidance for user on how to fix
- Wastes time before failing

**Solution**: Explicit prerequisite checks with clear errors.

**Good implementation** (from edit-changelog.sh:26-29):

```bash
# Good: verify prerequisites first
if [ ! -f "$CHANGELOG_FILE" ]; then
  echo "ERROR: $CHANGELOG_FILE doesn't exist. Please run 'init-changelog' first."
  exit 1
fi
```

**Why this works**:
- Fails fast with clear message
- Tells user exactly what to do
- Exits before wasting time on doomed operations

**Another example** - self-healing for pointer file (edit-changelog.sh:48-60):

```bash
# Read pointer file
if [ ! -f "$POINTER_FILE" ]; then
  # Initialize gracefully
  git rev-parse HEAD > "$POINTER_FILE"
  echo "WARN: Initialized pointer to HEAD. No changes processed."
  exit 0
fi

LAST_COMMIT=$(cat "$POINTER_FILE")

# Validate pointer
if ! git cat-file -e "$LAST_COMMIT" 2>/dev/null; then
  # Self-heal if corrupted
  git rev-parse HEAD > "$POINTER_FILE"
  echo "WARN: Pointer corrupted. Reinitialized to HEAD."
  exit 0
fi
```

**Why this is better than ERROR**: Missing or corrupted pointer is recoverable. Initialize gracefully and exit with WARN. Next run will work.

**Rule**: Always verify prerequisites explicitly. Exit early with clear guidance.

### 9.3 Silent Failures

**Problem**: Scripts exit 0 when operations fail, or don't communicate status clearly.

**Example** (hypothetical bad script):

```bash
# Bad: no status communication
git add $FILES
git commit -m "$MESSAGE"
# Script exits without telling caller what happened
```

**Why it's bad**:
- Orchestrator can't tell if operation succeeded
- Failures are invisible
- User gets no feedback
- Debugging is impossible

**Solution**: Always output status on first line, use appropriate status code.

**Good implementation** (multiple examples):

```bash
# No files to stage (not an error)
if [ -z "$FILES_TO_STAGE" ]; then
  echo "WARN: No files available to stage"
  exit 0
fi

# Successfully staged files
echo "SUCCESS: Staged 5 files for atomic commit"
for file in $FILES; do
  echo "- $file"
done
exit 0

# Operation failed
if ! git add $FILES; then
  echo "ERROR: Failed to stage files - git command failed"
  exit 1
fi
```

**Rule**: Three status codes (SUCCESS/WARN/ERROR), first line always status, descriptive messages.

### 9.4 Format Inconsistency

**Problem**: Different Goal/When/Note capitalization and formatting across skills.

**Example** (actual inconsistencies found):

```markdown
# Bad: Mix of formats
**Goal:** Stage files  # No backticks
**`WHEN`**: Use when...  # Backticks
**NOTE:** Important  # Italics in some, plain in others
```

**Why it's bad**:
- Harder to scan
- Looks unprofessional
- No clear protocol standard
- Confuses agents learning from examples

**Solution**: Standardize on backtick-wrapped keywords.

**Good format** (consistent across all new skills):

```markdown
**`GOAL`**: [what the skill accomplishes]

**`WHEN`**: [when to invoke this skill]

**`NOTE`**: [key constraint] (optional, use italics for emphasis if needed)
```

**Why backticks**: Visual distinction, clearly marks protocol keywords vs regular text.

**Rule**: Always use `**\`KEYWORD\`:**` format for Goal/When/Note headers.

### 9.5 Duplication Over Reference

**Problem**: Copying entire sections verbatim across multiple skills instead of using templates.

**Example** (found in repository before this guide):

Efficiency directives duplicated in:
- git-commit (lines 93-98)
- git-add (lines 63-72)
- git-message (lines 77-82)
- git-status (lines 36-44)
- fix-markdown (lines 110-121)

Total duplication: ~75 lines

**Why it's bad**:
- Updating one requires updating five
- Inconsistencies creep in
- Wastes token budget in skill files
- Harder to maintain

**Solution**: Use templates from this guide.

**Good approach**:

Instead of copying 15 lines of efficiency directives, write:

```markdown
## Efficiency directives

- Batch operations on file groups, avoid individual file processing
- Use parallel execution when possible
- Target only relevant files
- Reduce token usage
```

**From this guide** (Section 4.1), not copied from another skill.

**Rule**: Use templates from this guide, not copy-paste from other skills.

### 9.6 Token-Inefficient Workflows

**Problem**: Sequential file processing when batch operations are available.

**Example** (hypothetical bad git-add):

```markdown
## Workflow

1. Run git status to get unstaged files
2. For each unstaged file:
   - Run git diff on the file
   - Determine if it should be staged
   - Stage it if appropriate
3. Done
```

**Why it's bad**:
- Runs `git diff` N times for N files
- Each git command uses tokens for prompt + response
- Scales linearly with file count
- Wastes time and tokens

**Solution**: Batch operations.

**Good implementation** (from git-add.sh:40-41):

```bash
# Good: analyze multiple files in one command
git --no-pager diff --no-ext-diff --stat --minimal --patience --histogram \
    --find-renames --summary --no-color -U10 file1 file2 file3
```

**One command, multiple files**. Git outputs diffs for all files at once.

**Rule**: Always batch file operations. Never process files individually unless truly necessary.

**More examples**:

**Bad**:
```bash
for file in $FILES; do
  prettier --write $file
done
```

**Good**:
```bash
prettier --write $FILES
```

**Bad**:
```bash
for file in $FILES; do
  git add $file
done
```

**Good**:
```bash
git add $FILES
```

### 9.7 Missing Status Codes

**Problem**: Orchestrator doesn't capture status from sub-skill invocations.

**Example** (hypothetical bad orchestrator):

```markdown
### Step 2: Stage files

- Invoke the git-add skill
- Continue to Step 3
```

**Why it's bad**:
- Can't detect if staging failed (ERROR)
- Can't handle "nothing to stage" case (WARN)
- Continues blindly regardless of outcome
- May try to commit with nothing staged

**Solution**: Always capture and handle status explicitly.

**Good implementation** (from git-commit):

```markdown
### Step 2: Stage atomic changes

- Invoke the `git-add` skill to stage a cohesive set of files.
- Capture the status from skill output (`SUCCESS`, `WARN`, or `ERROR`).
- Handle the status:
  - If `ERROR`: Halt and report the error to the user.
  - If `WARN` (no files available): Skip to Step 6.
  - If `SUCCESS` (files staged): Continue to Step 3.
```

**Three explicit outcomes**: Each status code has explicit handling.

**Rule**: Every sub-skill invocation must capture status and handle all possible codes.

---

## 10. Domain-Specific Guidelines

Different domains (markdown, git, changelog) have specialized conventions.

### 10.1 Markdown Skills

Skills that edit markdown files follow these conventions.

**E-Prime Directive** (from fix-markdown references):

```markdown
**`E-Prime Compliance:`** Strictly follow the E-Prime directive when
writing, updating, or correcting prose.
```

**What is E-Prime**: English without "to be" verbs (is, are, was, were, be, being, been).

**Why**: Forces active voice and clearer writing.

**Examples**:

| Standard English | E-Prime |
|-----------------|---------|
| "The file is ready" | "The file reached ready state" |
| "This was implemented" | "We implemented this" |
| "The skill will be invoked" | "The orchestrator invokes the skill" |
| "It is important to..." | "Users should..." or "Developers must..." |

**Format First & Last** (from fix-markdown.sh:32-34):

```markdown
**`Format First & Last:`** Always format the document before starting
analysis and after finishing all edits.
```

**Why**: Prettier ensures consistent formatting. Format before to establish baseline, format after to clean up edits.

**Commands**:

```bash
# Before editing
prettier --write <file_path>

# After editing
prettier --write <file_path>
```

**Vale Lint Cycle** (from fix-markdown.sh:63-75):

```markdown
### Vale directives

Repeat the following cycle to address prose issues:

1. Run `vale` against the file in question.
2. Select issues to address.
3. Analyze the issue and plan a fix.
4. Fix the issue in the file.
5. Run `vale` again to verify the fix or find the next issue.
6. Repeat steps 1-5 until **`NO`** issues remain.
7. Execute the _Vale path directives_.
```

**Why iterative**: Vale finds new issues after edits. Must loop until clean.

**Path Wrapping** (from fix-markdown.sh:76-91):

```markdown
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
```

**Why**: Vale flags unwrapped paths as spelling errors. Backticks mark them as code.

**Heading Capitalization** (from fix-markdown.sh:103-104):

```markdown
- **`Headings:`** Capitalize only the first character in a heading to
  fix standard capitalization issues (Sentence case).
```

**Examples**:

| Bad | Good |
|-----|------|
| "Create SKILL.md Files" | "Create SKILL.md files" |
| "Quick Start Decision Tree" | "Quick start decision tree" |

**Exception**: Preserve proper nouns, acronyms, code terms.

### 10.2 Changelog Skills

Skills that manage CHANGELOG.md follow Keep a Changelog format and conventional commits.

**Conventional Commit Format** (from edit-changelog.sh comments):

```
type(scope): description

Optional body with details.

Optional footer with BREAKING CHANGE: description
```

**Types**:

| Type | User-Facing? | Changelog Section |
|------|-------------|-------------------|
| `feat` | ✅ Yes | Added |
| `fix` | ✅ Yes | Fixed |
| `perf` | ✅ Yes | Changed (performance) |
| `refactor` | ⚠️ Conditional | Changed (if body mentions "user-facing") |
| `revert` | ✅ Yes | Removed |
| `build` | ⚠️ Conditional | Changed (if related to deps) |
| `docs` | ❌ No | (skip) |
| `style` | ❌ No | (skip) |
| `test` | ❌ No | (skip) |
| `chore` | ❌ No | (skip) |
| `ci` | ❌ No | (skip) |

**Why filter**: Changelog is for users, not developers. Users don't care about test refactoring or CI config.

**Breaking Changes** (edit-changelog.sh:124-132):

```bash
if [ -n "$BREAKING" ] || echo "$BODY" | grep -qi "^BREAKING[- ]CHANGE:"; then
  BREAKING_DESC=$(echo "$BODY" | grep -i "^BREAKING[- ]CHANGE:" | sed 's/^BREAKING[- ]CHANGE: //')
  if [ -z "$BREAKING_DESC" ]; then
    BREAKING_DESC="$DESC"
  fi
  entries_changed["$commit"]="- **BREAKING**: $BREAKING_DESC"
  continue
fi
```

**Two detection methods**:
1. `!` after type: `feat!: new API`
2. Footer: `BREAKING CHANGE: description`

**Deduplication by Issue** (edit-changelog.sh:103-122):

```bash
# Extract issue references in formats: #123, GH-123
ISSUES=$(echo "$SUBJECT $BODY" | grep -oE '#[0-9]+|GH-[0-9]+' | sed -E 's/^GH-/#/' | sort -u)

if [ -n "$ISSUES" ]; then
  duplicate=false
  for issue in $ISSUES; do
    if [[ -n "${seen_issues[$issue]}" ]]; then
      duplicate=true
      break
    fi
  done
  if [ "$duplicate" = true ]; then
    debug "  Skipped: Duplicate issue reference"
    continue
  fi
  for issue in $ISSUES; do
    seen_issues[$issue]=1
  done
fi
```

**Why deduplication**: Multiple commits for same issue (e.g., "feat: add OAuth #123", "fix: OAuth bug #123"). Should only appear once in changelog.

**Keep a Changelog Format**:

```markdown
# Changelog

## [Unreleased]

### Added
- New feature 1
- New feature 2

### Changed
- **BREAKING**: API redesign
- Performance improvement

### Fixed
- Bug fix 1

## [1.0.0] - 2025-01-15

### Added
- Initial release feature
```

**Section order** (from Keep a Changelog spec):
1. Added
2. Changed
3. Deprecated
4. Removed
5. Fixed
6. Security

**Pointer File** (from edit-changelog):

```
.last-aggregated-commit file contains:
abc123def456...  (commit SHA)
```

**Purpose**: Tracks last processed commit. Next run processes from this point to HEAD.

**Update timing**: Always update to HEAD after successful processing (edit-changelog.sh:256).

### 10.3 Git Skills

Git-specific skills follow additional conventions beyond general patterns.

**Atomic Commit Philosophy**:

```
One commit = One logical change
```

**What is atomic**:
- All changes serve single purpose
- Can be described in one commit message
- Could be reverted independently
- Makes sense as standalone unit

**What is NOT atomic**:
- "Fixed 5 bugs" (should be 5 commits)
- "Added feature and updated docs" (should be 2 commits)
- "Refactored code and added tests" (should be separate)

**Porcelain v2 Format** (from git commands):

```bash
git status --porcelain=v2 --branch
```

**Why v2**: Machine-readable, stable format. v1 may change, v2 is guaranteed stable.

**Example output**:

```
# branch.oid abc123def456...
# branch.head main
1 M. N... 100644 100644 100644 sha1 sha2 file1.txt
1 .M N... 100644 100644 100644 sha1 sha2 file2.txt
? untracked.txt
```

**Format**:
- `1 M.`: Staged modification
- `1 .M`: Unstaged modification
- `?`: Untracked file

**Specific Diff Flags** (why each flag matters):

**`--patience` and `--histogram`** (from git commands):

Standard diff:
```diff
-function old() {
-  return 1;
+function new() {
+  return 2;
}
```

With patience/histogram:
```diff
-function old() {
+function new() {
-  return 1;
+  return 2;
}
```

**Why better**: Patience algorithm finds better matching points, producing more intuitive diffs for LLMs.

**User Approval Workflow** (from git-message):

```
1. Generate commit message
2. Present to user with 4 options:
   - Approve and commit
   - Edit staged files
   - Regenerate message
   - Abort
3. Await user response
4. Take action based on response
```

**Why 4 options**: Covers all user intentions without forcing them to manually intervene.

---

## 11. Token Optimization Strategies

Strategies for minimizing token usage while maintaining clarity and functionality.

### 11.1 Reference File Strategy

**Problem**: Detailed specifications consume tokens in every skill execution.

**Solution**: Move detailed specs to references/ folder, reference conditionally.

**Example from git-add** (skills/git-add/SKILL.md:22-23):

```markdown
- Study `references/` files `ONLY` if more than one file exists to
  stage.
```

**Why this works**:
- Single-file case: Skip reference reading (saves ~1000 tokens)
- Multi-file case: Read references only when logic is needed
- References contain detailed atomic commit guidelines

**Reference file contents** (not loaded unless needed):
- Atomic commit philosophy
- Example groupings
- Edge case handling
- Detailed rules for .gitignore

**Pattern**:

```markdown
## Primary directives

- Study `references/` files `ONLY` if [condition].
```

**Another example from git-message** (skills/git-message/SKILL.md:26-27):

```markdown
- **`ALWAYS`** study `references/conventional-commit.md` before
  drafting.
```

**Why ALWAYS**: Commit message format is critical. Can't skip reference, but it's only loaded once per message generation.

**Token savings**: References folder = 2000-5000 tokens. Conditional reading saves these tokens when not needed.

### 11.2 Batch Processing

**Problem**: Sequential file operations scale linearly with file count.

**Solution**: Process multiple files in single commands.

**Examples**:

**Git diff** (good):
```bash
git diff file1 file2 file3  # One command, all files
```

**Git diff** (bad):
```bash
git diff file1  # Three commands
git diff file2
git diff file3
```

**Token impact**:
- **Good**: ~1000 tokens (one prompt + one response)
- **Bad**: ~3000 tokens (three prompts + three responses)

**Prettier** (good):
```bash
prettier --write file1.md file2.md file3.md
```

**Prettier** (bad):
```bash
prettier --write file1.md
prettier --write file2.md
prettier --write file3.md
```

**Vale** (good):
```bash
vale --no-wrap --output=JSON file1.md file2.md
```

**Git add** (good):
```bash
git add src/component.js src/component.test.js src/types.ts
```

**General rule**: If a command accepts multiple file arguments, always use it.

### 11.3 Template Efficiency

**Problem**: Verbose explanations in SKILL.md consume tokens on every execution.

**Solution**: Use condensed templates from this guide, link to this guide for details.

**Example - Efficiency directives**:

**Before** (hypothetical verbose version):

```markdown
## Efficiency directives

When processing files, always batch operations together instead of
processing files individually. This is important because:
- Reduces the number of API calls
- Minimizes token usage across multiple operations
- Improves performance by parallelizing when possible

Additionally, make sure to:
- Target only relevant files instead of processing everything
- Use parallel execution patterns wherever feasible
- Optimize for token efficiency in all operations
```

**After** (condensed template):

```markdown
## Efficiency directives

- Batch operations on file groups, avoid individual file processing
- Use parallel execution when possible
- Target only relevant files
- Reduce token usage
```

**Token savings**: ~150 tokens → ~30 tokens = 80% reduction.

**Why it works**: Directives are clear and actionable without explanation. Agents understand "batch operations" without needing rationale.

**Another example - Task management**:

**Before** (hypothetical verbose):

```markdown
## Task management

For complex multi-step operations, you should utilize the built-in
todo system that allows you to break down large tasks into smaller,
manageable pieces. This helps you to:
- Plan your approach before executing
- Track progress as you work
- Optimize the workflow by identifying redundancies
- Maintain state across multiple steps
```

**After** (condensed template):

```markdown
## Task management

For complex tasks: use `todo` system to break down, plan, and optimize
workflow.
```

**Token savings**: ~80 tokens → ~15 tokens = 81% reduction.

**General rule**: Templates should be directive, not explanatory. Action-oriented, not educational.

### 11.4 Workflow Optimization

**Problem**: Unclear workflow structure causes agents to re-read and re-parse.

**Solution**: Use consistent numbering and structure.

**Orchestrators** - Use ### Step N:

```markdown
## Workflow

Follow these steps in sequence:

### Step 1: Check for changes
[details]

### Step 2: Stage atomic changes
[details]

### Step 3: Generate commit message
[details]
```

**Why**: Step numbers provide clear checkpoints. Agents can reference "Step 2" unambiguously.

**Simple skills** - Use bullet points:

```markdown
## Workflow

- Action 1
- Action 2
- Action 3
- **`DONE`**
```

**Why**: Bullets are more concise than numbered steps for simple sequences.

**Conditional logic** - Use nested bullets:

```markdown
### Step 2: Update from commits

- Invoke the `edit-changelog` skill
- Capture status
- Handle status:
  - If `ERROR`: Stop
  - If `WARN`: Skip Step 3
  - If `SUCCESS`: Continue to Step 3
```

**Why**: Nesting shows logical structure. Clear branching paths.

**Token impact**:

| Structure Type | Tokens (approx) | Use Case |
|---------------|-----------------|----------|
| Numbered steps | ~500 | Orchestrators with 5-6 steps |
| Bullet points | ~200 | Simple skills with 3-4 steps |
| Nested bullets | ~300 | Conditional logic within steps |

**General rule**: Match structure to complexity. Don't over-structure simple workflows.

---

## 12. Testing Guidelines

Testing ensures skills work correctly and handle edge cases.

### 12.1 Script Testing

All scripts in `scripts/` folder should have corresponding tests in `tests/` folder.

**Standard structure**:

```
skills/skill-name/
├── scripts/
│   └── script-name.sh
└── tests/
    └── test-script-name.sh
```

**Test requirements**:

1. **Test all status paths**: SUCCESS, WARN, ERROR
2. **Test prerequisites**: Missing files, corrupted state
3. **Test edge cases**: Empty repo, large history, etc.

**Example test structure** (hypothetical test-edit-changelog.sh):

```bash
#!/bin/bash

# Test edit-changelog.sh

SCRIPT_DIR="$(dirname "$0")/../scripts"
SCRIPT="$SCRIPT_DIR/edit-changelog.sh"

# Test 1: ERROR - CHANGELOG.md missing
echo "Test 1: ERROR when CHANGELOG.md missing"
rm -f CHANGELOG.md .last-aggregated-commit
OUTPUT=$($SCRIPT 2>&1)
if [[ "$OUTPUT" =~ ^ERROR ]]; then
  echo "✓ Test 1 passed"
else
  echo "✗ Test 1 failed"
fi

# Test 2: WARN - No new commits
echo "Test 2: WARN when no new commits"
# Setup: create CHANGELOG.md, initialize pointer to HEAD
./skills/init-changelog/scripts/init-changelog.sh
OUTPUT=$($SCRIPT 2>&1)
if [[ "$OUTPUT" =~ ^WARN ]]; then
  echo "✓ Test 2 passed"
else
  echo "✗ Test 2 failed"
fi

# Test 3: SUCCESS - Process commits
echo "Test 3: SUCCESS when commits exist"
# Setup: create test commit
echo "test" > test.txt
git add test.txt
git commit -m "feat: test feature"
OUTPUT=$($SCRIPT 2>&1)
if [[ "$OUTPUT" =~ ^SUCCESS ]]; then
  echo "✓ Test 3 passed"
else
  echo "✗ Test 3 failed"
fi

# Cleanup
rm -f test.txt CHANGELOG.md .last-aggregated-commit
```

**Why test all paths**: Scripts have complex branching logic. Must verify each path works.

**Running tests**:

```bash
bash skills/edit-changelog/tests/test-edit-changelog.sh
```

### 12.2 Skill Testing Approach

Skills (SKILL.md files) are tested through execution, not unit tests.

**What to verify**:

1. **Status propagation**: Orchestrators correctly capture and handle sub-skill status
2. **Prerequisite verification**: Skills check prerequisites before proceeding
3. **Output format**: First line contains status code
4. **Workflow correctness**: Steps execute in correct order

**Manual testing checklist**:

For **orchestrators**:
- [ ] Each sub-skill invocation captures status
- [ ] ERROR status halts execution
- [ ] WARN status triggers appropriate handling
- [ ] SUCCESS status continues to next step
- [ ] Loop control works (if applicable)

For **simple skills**:
- [ ] Workflow steps execute in order
- [ ] Output format matches specification
- [ ] Final status is communicated

For **script-based skills**:
- [ ] Script invocation succeeds
- [ ] Status from script is captured
- [ ] Status is communicated to user or orchestrator

**Example verification** (git-commit):

```
1. Repository has changes → Step 1 detects them
2. git-add stages files → Step 2 captures SUCCESS
3. git-message generates message → Step 3 captures APPROVED
4. Commit executes → Step 4 reports SHA
5. More changes exist → Step 5 loops back to Step 2
6. No more changes → Step 6 presents status
```

**Test with edge cases**:

- Empty repository (no commits yet)
- Clean repository (no changes)
- Partial operations (interrupt between steps)
- Error conditions (git commands fail)

---

## 13. Quick Reference Table

Use this table to quickly find the right example skill for your needs.

| Pattern | Example Skill | Location | Lines | Key Features |
|---------|--------------|----------|-------|-------------|
| **Orchestration Patterns** |
| Sequential Orchestration | git-commit | `skills/git-commit/SKILL.md` | 103 | Fixed steps, auto-loop control, status capture |
| Conditional Orchestration | update-changelog | `skills/update-changelog/SKILL.md` | 105 | Prerequisite checks, conditional steps, skip logic |
| Simple Skill | cleanup-changelog | `skills/cleanup-changelog/SKILL.md` | 26 | Single responsibility, direct workflow |
| **Script Integration** |
| Full Script Logic | edit-changelog | `skills/edit-changelog/SKILL.md` | 107 | Status codes, pointer file, parsing |
| Script Wrapper | init-changelog | `skills/init-changelog/SKILL.md` | 97 | Script invocation, prerequisite check |
| **Git Patterns** |
| Atomic Staging | git-add | `skills/git-add/SKILL.md` | 109 | Atomic grouping, single-file shortcut |
| Message Generation | git-message | `skills/git-message/SKILL.md` | 118 | User approval, 4-option workflow, extended status |
| Status Presentation | git-status | `skills/git-status/SKILL.md` | 80 | User-facing only, reference-based presentation |
| **Domain-Specific** |
| Markdown Processing | fix-markdown | `skills/fix-markdown/SKILL.md` | 144 | E-Prime, Vale cycles, iterative fixing |
| Changelog Update | edit-changelog | `skills/edit-changelog/SKILL.md` | 107 | Conventional commits, deduplication, Keep a Changelog |
| Changelog Init | init-changelog | `skills/init-changelog/SKILL.md` | 97 | Structure creation, version detection, pointer init |

**How to use this table**:

1. **Find your pattern**: Look in "Pattern" column for what you're building
2. **Study the example**: Read the example skill at the location provided
3. **Copy relevant sections**: Use as template for your skill
4. **Adapt**: Modify for your specific use case

**Pattern selection guide**:

- **Need to orchestrate multiple skills?** → Sequential or Conditional
  - **Fixed order, always run?** → Sequential (git-commit)
  - **Skip steps based on state?** → Conditional (update-changelog)

- **Single focused task?** → Simple Skill (cleanup-changelog)

- **Complex parsing/logic needed?** → Script Integration (edit-changelog)

- **Git-related?** → Git Patterns
  - **Staging files?** → git-add
  - **Commit messages?** → git-message
  - **Show status?** → git-status

- **Markdown editing?** → fix-markdown

- **Changelog management?** → init-changelog, edit-changelog, update-changelog

**Common sections to reference**:

| Need | See Skill | Section |
|------|-----------|---------|
| Efficiency directives template | Any git skill | Section near end |
| Task management template | Any git skill | Section near end |
| Git commands (copy-paste) | git-add | Git directives section |
| Status code output format | git-add | Output section |
| User approval workflow | git-message | Workflow section |
| Prerequisite checking | edit-changelog | Script logic |
| Reference file usage | git-status | References section |

---

## Appendix: Maintenance Notes

**When to update this guide**:

1. **New pattern emerges**: A skill introduces a pattern not documented here
2. **Template evolution**: Common sections evolve (e.g., new directive template)
3. **Anti-pattern discovered**: A mistake is made repeatedly
4. **Section length changes**: Skill files grow/shrink significantly

**How to maintain**:

1. **Extract novel patterns**: When creating new skills, identify any novel orchestration or integration patterns
2. **Update line counts**: Section 13 table should reflect current line counts (±5 lines is acceptable)
3. **Add examples**: New anti-patterns or best practices should include concrete examples
4. **Keep templates current**: If common sections evolve, update templates in Section 4

**Philosophy**: This guide documents what exists and suggests best practices, but doesn't mandate rigid structures. Flexibility for novel designs is important.

---

**End of SKILL_REFERENCE.md**

This guide contains copy-paste ready templates for all common patterns found in the 10 existing skills. Use decision trees to quickly find the right pattern, copy templates directly, and reference orchestration examples for complex workflows.
