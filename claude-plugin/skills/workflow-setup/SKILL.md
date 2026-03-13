---
name: workflow-setup
description: >
  Interactive setup of custom workflow statuses and transition rules for beads.
  Generates .beads/WORKFLOWS.md and syncs status.custom config.
  Use when user wants to customize their kanban/workflow states.
allowed-tools: "Read,Write,Edit,Bash(bd:*)"
version: "0.60.0"
author: "Steve Yegge <https://github.com/steveyegge>"
license: "MIT"
---

# Workflow Setup

Configure custom workflow statuses and transition rules for a beads project.

## What this does

1. Asks the user about their workflow stages
2. Generates `.beads/WORKFLOWS.md` describing statuses and transitions
3. Runs `bd config set status.custom "..."` to register statuses with the CLI

## Step 1: Gather workflow from user

Ask the user to describe their workflow. Prompt with questions like:

- What stages does a task go through from creation to completion?
- Are there different paths for different work types? (bugs vs features vs research)
- Who or what decides when a task moves to the next stage? (author, reviewer, CI, manual testing)
- Are there any transitions that should be forbidden?

If the user gives a short answer like "open, research, dev, testing, review, done", expand it into full definitions by asking clarifying questions about each status.

## Step 2: Check for existing file

```bash
cat .beads/WORKFLOWS.md 2>/dev/null
```

If the file exists, show the user current config and ask if they want to replace or modify it.

## Step 3: Generate `.beads/WORKFLOWS.md`

Write the file using this exact format:

```markdown
# Workflow

## Statuses

| Status | Description |
|--------|-------------|
| open | Task created, not yet started |
| in_progress | Active work |
| review | Awaiting code review |
| blocked | Waiting on external dependency |
| closed | Done |

## Transitions

| From | To | Condition |
|------|-----|-----------|
| open | in_progress | Work begins |
| in_progress | review | Ready for review |
| review | closed | Review approved |
| review | in_progress | Changes requested |
| * | blocked | Blocker discovered |
| blocked | * | Blocker resolved |
```

### Format rules

- The `Statuses` table MUST include `open` and `closed` (required by beads)
- The `Description` column explains when a task is in this state
- The `Transitions` table uses `*` as wildcard for "any status"
- The `Condition` column explains what triggers the transition
- Keep descriptions concise (one sentence)
- Use snake_case for multi-word statuses (e.g. `in_progress`, `tech_audit`)

## Step 4: Sync with CLI

After writing the file, extract status names and register them:

```bash
bd config set status.custom "<comma-separated custom statuses>"
```

Only include statuses that are NOT built-in. Built-in statuses (do not include in config):
`open`, `in_progress`, `blocked`, `deferred`, `closed`, `pinned`, `hooked`

For example, if the workflow adds `research`, `testing`, `review`:
```bash
bd config set status.custom "research,testing,review"
```

## Step 5: Confirm

Show the user:
1. The generated file contents
2. The `bd config` command that was run
3. Remind them that agents will read `.beads/WORKFLOWS.md` to follow transition rules

## Examples

### Software team with code review
```
/workflow-setup
> We do: backlog, in progress, code review, QA testing, deployed
```

### Research-heavy project
```
/workflow-setup
> Stages: idea, literature review, experiment, analysis, write-up, peer review, published
```

### Simple solo workflow
```
/workflow-setup
> Just: todo, doing, done
```
Maps to: `open` (todo), `in_progress` (doing), `closed` (done) — no custom statuses needed.
