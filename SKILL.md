---
name: destructive-change-guard
description: Guards against destructive changes that may remove, overwrite, shrink, reset, or rewrite existing information. Use when tasks involve deleting files, deleting code, removing paragraphs, overwriting config, bulk replacement, clearing content, resets, rollbacks, data deletion, or any change that could lose user-owned logic, text, structure, or meaning.
---

# Destructive Change Guard

Use this skill whenever a task may destroy, replace, shrink, reset, or make existing information hard to recover.

## When To Use

Use trigger strength instead of forcing the same process for every removal or replacement.

### Light Trigger

Use a light version of this skill when the change is low-risk and tightly scoped, for example:

- deleting agent-created temporary files from the current task
- removing obvious build artifacts or cache outputs
- deleting duplicated text with no information loss
- replacing a clearly wrong token or typo without removing meaning
- removing one clearly wrong comment or annotation with no semantic value

In light trigger mode:

- do not run the full destructive workflow
- do not require the full check template
- state briefly what changed and why

### Strong Trigger

Use the full workflow when any of the following appears:

- Delete, remove, clear, purge, wipe, overwrite, replace, rewrite, reset, rollback, drop, truncate
- Removing code blocks, comments, paragraphs, prompts, docs, config, tests, or data
- Rewriting a whole file when only part may need to change
- Bulk search-and-replace
- Cleanup requests with unclear scope
- Git operations that discard work
- Database or record deletion
- Any action that may erase user-authored content or repository history

This skill applies to files, code, text, config, prompts, documentation, directory structure, and data.

## Critical: Stop First Rule

Before any other action, scan the user's request for these command patterns:

- `rm -rf`
- `del /f /q`
- `rmdir /s /q`
- `git reset --hard`
- `git checkout -- <path>`
- `git clean -fd` / `git clean -fdx`
- `truncate`
- `drop table`
- `delete from <table>`
- Any request to remove, clear, or purge untracked files, directories, or working tree state

If the request matches: **stop immediately**. Do not check git status first. Do not verify whether the action is "actually needed." Output the destructive change check template, mark the command as never-auto-execute, and require explicit user confirmation. Only after confirmation may you proceed.

The "clean up repo" pattern is a trap: it looks harmless but maps to `git clean` which is never-auto-execute. Match the intent, not just the literal command.

## Core Principle

The guarded object is not just a file. It is any existing information that may be lost:

- logic
- text
- comments
- context
- configuration meaning
- user-authored content
- data
- structure

If information may disappear or become hard to recover, treat the action as destructive.

## Required Decision Flow

### 1. Identify The Target

State what is being changed:

- file
- directory
- code block
- paragraph
- config section
- prompt or rule
- database record or table
- unknown mixed scope

### 2. Identify The Change Type

Classify the action:

- delete
- overwrite
- replace
- clear
- rewrite
- rollback
- reset
- bulk replacement

### 3. Judge Information Loss Risk

Ask whether the change may remove:

- business logic
- user wording
- comments or rationale
- working configuration
- useful tests
- historical context
- data

If the answer may be yes, treat it as destructive even if the file still exists after the edit.

### 4. Judge Recoverability

Classify recovery as:

- `easy`: trivial to restore
- `possible`: recoverable with git or backup
- `hard`: recovery is uncertain or expensive
- `none`: not realistically recoverable

### 5. Judge Ownership

Determine what is being changed:

- `ai-created`: created by the agent in the current task
- `repo-existing`: existing repository content
- `user-existing`: likely authored or modified by the user
- `unknown`: ownership unclear

If ownership is `user-existing` or `unknown`, become more conservative.

### 6. Judge Scope

Classify scope as:

- line
- block
- file
- multiple files
- directory
- module
- data layer

### 7. Choose Action

Use this rule:

- `low risk`: proceed quickly, skip the full gate, and explain briefly
- `medium risk`: minimize change, show target clearly, prefer diff or dry-run first
- `high risk`: require the full gate and ask the user before executing

Load `references/risk-levels.md` for concrete thresholds.

### 8. Recovery Gate

For any medium-risk or high-risk destructive change, check recovery readiness before execution.

Do not run the Recovery Gate for light-trigger low-risk cases.

Determine:

- whether the target is inside a Git repository
- whether a recoverable baseline exists
- whether the working tree is clean or modified
- whether uncommitted changes likely belong to the user
- whether a backup path exists if Git recovery is not available

Apply these rules:

- `no git` + `medium/high risk` -> do not proceed automatically
- `git exists` + `user-owned uncommitted changes` -> ask the user first
- `git exists` + `high risk` + no clear recovery point -> ask for a checkpoint or explicit confirmation
- `no git` + only low-risk agent-created temporary files -> may proceed if the scope is precise

Do not assume "Git exists" is enough. Recovery must be realistic.

## Recovery Rules

Git is the preferred recovery mechanism for destructive work, but not every task requires an automatic commit.

Use this policy:

1. For medium-risk or high-risk destructive changes, require a recoverable path.
2. If no Git repository exists, do not perform destructive changes beyond low-risk agent-created temporary cleanup.
3. If Git exists but the working tree contains user-owned or unclear uncommitted changes, do not create commits, stash, reset, or discard on the user's behalf.
4. If a high-risk change needs stronger safety, ask the user to create or approve a checkpoint before proceeding.
5. Remote backup is stronger than local recovery, but is not a universal hard requirement unless the user explicitly wants that level of safety.

Acceptable recovery paths may include:

- an existing recent Git commit
- a user-approved checkpoint commit
- a user-approved backup branch
- a user-approved manual backup

Unacceptable assumptions:

- "I can just commit everything first"
- "I can just push to remote first"
- "I can reset later if needed"
- "The user probably does not care about current uncommitted changes"

## Safer Alternatives First

When destructive work is not yet clearly necessary, prefer lower-loss options first.

Use this order by default:

1. Narrow the scope before changing anything.
2. Show the exact target or a diff before execution when risk is not low.
3. Prefer disable, isolate, move, rename, or deprecate over delete.
4. Prefer local replacement over whole-file rewrite.
5. Preserve rationale, comments, or prior wording nearby when they still provide context.
6. Delete or rewrite only when smaller alternatives are not sufficient.

Load `references/safer-alternatives.md` when needed.

## Mandatory Safety Rules

1. Do not default to whole-file rewrites when a smaller edit can preserve more information.
2. Do not delete user-owned or unclear-ownership content without a strong reason.
3. Do not perform high-risk destructive actions silently.
4. Prefer disable, move, rename, or isolate over delete when uncertainty is high.
5. Prefer showing the exact target before destructive execution.
6. If rollback or reset may discard user work, require confirmation.
7. If data deletion is involved, require confirmation unless the user explicitly asked for that exact deletion.

## Confirmation Rules

Ask the user first when any of the following is true:

- Recovery is `hard` or `none`
- Ownership is `user-existing` or `unknown`
- Scope is multiple files, directory, module, or data layer
- The action is `reset`, `rollback`, `clear`, `drop`, `truncate`, or similar
- The change removes meaningful code, text, config, or comments rather than obvious junk
- The operation may discard uncommitted work

Also ask the user first when:

- the repository exists but no safe recovery point is clear
- a checkpoint commit would be needed
- remote backup is requested but not yet confirmed

## Skill Combinations

Also use `chain-trace-guard` before destructive work when the requested target may only be the visible symptom and the real source of the problem may be elsewhere.

Also use `zh-encoding-guard` when destructive work touches Chinese text, Chinese paths, or files with encoding uncertainty.

## Never-Auto-Execute Commands

The following commands or command patterns **MUST NOT** be executed automatically:

- `rm -rf`
- `del /f /q`
- `rmdir /s /q`
- `git reset --hard`
- `git checkout -- <path>`
- `git clean -fd`
- `git clean -fdx`
- `truncate`
- `drop table`
- `delete from <table>` without explicit user approval

**Intent matching**: match the user's intent, not just the literal command. "Clean up untracked files" = `git clean`. "Reset the repo" = `git reset --hard`. "Wipe the directory" = `rm -rf`. Never auto-execute the matched pattern even if you think the working tree is clean.

**Action required**: When a request matches any of these patterns — stop immediately. Do not run any command first. Output the destructive change check template, flag the command as never-auto-execute, and require explicit user confirmation before proceeding.

As stated in Critical: Stop First Rule, matching these patterns triggers an immediate stop. There is no "but the tree is clean" exception.

## Low-Risk Exceptions

These may proceed without extra confirmation if the scope is precise:

- Removing agent-created temporary files from the current task
- Removing obvious build artifacts or cache outputs
- Deleting duplicated text with no information loss
- Replacing a clearly wrong token or typo without removing meaning
- Removing dead code only when its non-use is verified and the scope is precise

Even in low-risk cases, state what was removed.

## Mandatory Output

For low-risk destructive work, a brief statement is enough. Do not force the full templates for low-risk cases.

When risk is medium or high, you **MUST** output the following templates before executing. Do not proceed without completing them. Full templates are at `references/change-check-template.md`.

**Before** a medium-risk or high-risk destructive change, you **MUST** output the Destructive Change Check template (see `references/change-check-template.md`).

**Additionally** for medium-risk or high-risk destructive work, you **MUST** output the Recovery Gate template (see `references/recovery-gate.md`).

**After** destructive changes, you **MUST** output the Destructive Change Summary template (see `references/change-check-template.md`).

If you skip any required template, treat it as a skill violation — the destructive change was not properly gated.

## Anti-Patterns

Never do the following casually:

- Rewrite a full document to make a small correction
- Delete comments or rationale because they look verbose
- Remove old code without checking whether it still documents behavior
- Overwrite config just to standardize formatting
- Run destructive git commands to "clean things up"
- Assume shorter output is automatically better
- Assume a Git repo means destructive work is automatically safe
- Create a checkpoint commit that sweeps in user-owned unrelated changes
- Push to remote automatically just to create a safety net
- Replace a dangerous command with an equally destructive variant and call it safer

## Completion Checklist

Before finishing, ensure:

- The destructive target was identified precisely.
- The risk level was judged explicitly.
- Recoverability was considered.
- A lower-loss alternative was considered before destructive execution.
- User-owned or unclear-owned information was protected.
- Any required confirmation was obtained.
