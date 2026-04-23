---
name: destructive-change-guard
description: Guards against destructive changes that may remove, overwrite, shrink, reset, rewrite, discard, purge, clean, rename, move, truncate, drop, or bulk-replace existing information. Use when tasks involve deleting files/code/docs/tests/config/data, cleanup requests, refactors that remove old paths, git reset/checkout/clean, database DELETE/DROP/TRUNCATE, overwriting config, whole-file rewrites, repeated small removals, silent side-effect loss during ordinary edits, or any change that could lose user-owned logic, text, structure, history, references, or meaning.
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

### Fast Exemption

Skip the full destructive workflow when all conditions are true:

- the target is precise and visible
- the change affects only whitespace, formatting, duplicate empty lines, obvious typo tokens, or agent-created temporary artifacts
- no user-authored meaning, logic, comments, tests, config, data, references, or structure is removed
- the scope is a line or tiny block
- recovery is easy

Examples:

- removing one extra blank line
- fixing one misspelled variable name when meaning is unchanged
- removing duplicated punctuation
- deleting a temporary file created by the agent in the current task

Even when exempted, briefly state what was removed or changed.

### Strong Trigger

Use the full workflow when any of the following appears:

- Delete, remove, clear, purge, wipe, overwrite, replace, rewrite, reset, rollback, drop, truncate
- Removing code blocks, comments, paragraphs, prompts, docs, config, tests, or data
- Rewriting a whole file when only part may need to change
- Bulk search-and-replace
- Cleanup requests with unclear scope
- Git operations that discard work
- Database or record deletion
- Existing-file edits that may silently remove comments, rationale, tests, user wording, references, or behavior as a side effect
- Repeated low-risk removals in the same task that may accumulate into meaningful information loss
- Rename or move operations that may break imports, links, routes, assets, config, or external references
- Any action that may erase user-authored content or repository history

This skill applies to files, code, text, config, prompts, documentation, directory structure, and data.

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

## Silent Destruction Guard

Destructive risk can appear as a side effect, even when the user did not request deletion.

Treat existing-file edits as potentially destructive when the change may:

- remove comments, rationale, examples, tests, prompts, or user wording
- replace existing logic while implementing new behavior
- reformat code in a way that removes semantic structure
- move files and break relative imports, links, assets, routes, or references
- simplify UI or docs by removing user-visible meaning
- regenerate or overwrite content that was partially user-authored

For ordinary edits to existing files:

- preserve unrelated content by default
- prefer targeted patches over full rewrites
- check whether any information disappeared outside the requested scope
- if meaningful content is removed as a side effect, classify the edit using the normal destructive flow

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
- rename
- move
- bulk replacement

### 3. Judge Information Loss Risk

Ask whether the change may remove:

- business logic
- user wording
- comments or rationale
- working configuration
- useful tests
- historical context
- imports, links, routes, assets, or references
- data

If the answer may be yes, treat it as destructive even if the file still exists after the edit.

### 4. Judge Recoverability

Classify recovery as:

- `easy`: trivial to restore
- `possible`: recoverable with git or backup
- `partial`: recovery exists but may not restore the latest state completely
- `hard`: recovery is uncertain or expensive
- `none`: not realistically recoverable

### 5. Judge Ownership

Determine what is being changed:

- `ai-created`: created by the agent in the current task
- `repo-existing`: existing repository content
- `user-existing`: likely authored or modified by the user
- `unknown`: ownership unclear

`ai-created` only means created by the agent in the current task and not yet relied on by the user. Files created by previous sessions, other agents, scripts, or unknown automation are not automatically `ai-created`.

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
- whether the recoverable baseline is complete or only partial
- whether the working tree is clean or modified
- whether uncommitted changes likely belong to the user
- whether a backup path exists if Git recovery is not available

Apply these rules:

- `no git` + `medium/high risk` -> do not proceed automatically
- `git exists` + `user-owned uncommitted changes` -> ask the user first
- `git exists` + `high risk` + no clear recovery point -> ask for a checkpoint or explicit confirmation
- `no git` + only low-risk agent-created temporary files -> may proceed if the scope is precise

Do not assume "Git exists" is enough. Recovery must be realistic.

## Cumulative Risk Rule

Small destructive edits can become risky when repeated in the same task.

Track destructive actions within the current user request, including:

- deleting files
- deleting code blocks
- removing comments, tests, docs, prompts, config, or data
- replacing meaningful existing content
- renaming or moving files
- bulk formatting that removes or rewrites existing structure

If the same task accumulates 3 or more low-risk destructive actions, upgrade to medium risk.

If the cumulative scope reaches multiple files, a directory, a module, or user-authored content, pause and summarize before continuing.

Do not split a larger destructive operation into many "low-risk" edits to avoid the gate.

## Recovery Rules

Git is the preferred recovery mechanism for destructive work, but not every task requires an automatic commit.

Use this policy:

1. For medium-risk or high-risk destructive changes, require a recoverable path.
2. If no Git repository exists, do not perform destructive changes beyond low-risk agent-created temporary cleanup.
3. If Git exists but the working tree contains user-owned or unclear uncommitted changes, do not create commits, stash, reset, or discard on the user's behalf.
4. If a high-risk change needs stronger safety, ask the user to create or approve a checkpoint before proceeding.
5. Remote backup is stronger than local recovery, but is not a universal hard requirement unless the user explicitly wants that level of safety.
6. If recovery is `partial`, explain what can and cannot be restored before any high-risk destructive work.

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

## Rename And Move Risk

Rename and move operations are not automatically safe alternatives.

Treat rename or move as medium risk when:

- references, imports, routes, links, assets, tests, or config may point to the old path
- the target is used outside the edited file
- multiple files or directories are involved
- the file is part of a public API, CLI entry, route, package export, migration, or generated contract

Before medium-risk rename or move:

- identify likely references
- update references in the same change when appropriate
- prefer showing the affected paths before execution

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

- Recovery is `hard` or `none`, or recovery is `partial` for high-risk work
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

The following commands or command patterns must never be executed automatically by the agent as part of destructive work:

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

These require explicit user intent and confirmation, even if the agent believes the action is correct.

If a task seems to call for one of these, stop and present the risk instead of executing it.

## Low-Risk Exceptions

These may proceed without extra confirmation if the scope is precise:

- Removing agent-created temporary files from the current task
- Removing obvious build artifacts or cache outputs
- Deleting duplicated text with no information loss
- Replacing a clearly wrong token or typo without removing meaning
- Removing dead code only when its non-use is verified and the scope is precise

Even in low-risk cases, state what was removed.

## Required Output

For low-risk destructive work, a brief statement is enough. Do not force the full templates for low-risk cases.

Before a medium-risk or high-risk destructive change, include:

```text
Destructive change check:
- Target: <file / code block / paragraph / config / data>
- Change type: delete / replace / overwrite / rewrite / reset / clear / rename / move
- Information loss risk: low / medium / high
- Recovery: easy / possible / partial / hard / none
- Ownership: ai-created / repo-existing / user-existing / unknown
- Scope: line / block / file / multiple files / directory / module / data
- Safer alternative considered: yes / no
- Action: proceed / minimize change / show diff first / ask user first
```

For medium-risk or high-risk destructive work, also include:

```text
Recovery gate:
- Git repository: yes / no
- Recoverable baseline: yes / no / unclear
- Recovery completeness: complete / partial / unclear
- Working tree state: clean / modified / unknown
- User-owned uncommitted changes: yes / no / unclear
- Backup method: existing commit / checkpoint commit / backup branch / manual backup / none
- Remote backup: available / not available / not required / unclear
- Action: proceed / ask user for checkpoint / do not proceed
```

When cumulative low-risk destructive actions are upgraded, include:

```text
Cumulative destructive change check:
- Actions so far: <count and type>
- Affected scope: <files / blocks / directory / module>
- Information removed: none / minor / meaningful / unclear
- Recovery path: git / backup / manual / none
- Next action: proceed / show diff first / ask user first
```

After destructive changes, include:

```text
Destructive change summary:
- Changed: <what was removed or replaced>
- Scope verified: yes / partial / no
- Recovery path: git / backup / manual / none
- Safer alternative used: yes / no
- User confirmation required: yes / no
- Residual risk: none / low / medium / high
```

## Anti-Patterns

Never do the following casually:

- Rewrite a full document to make a small correction
- Delete comments or rationale because they look verbose
- Remove old code without checking whether it still documents behavior
- Overwrite config just to standardize formatting
- Run destructive git commands to "clean things up"
- Assume shorter output is automatically better
- Assume a Git repo means destructive work is automatically safe
- Treat rename or move as automatically safe because it preserves file content
- Ignore side-effect information loss during formatting, regeneration, or feature edits
- Split a destructive cleanup into many tiny edits to avoid a risk gate
- Create a checkpoint commit that sweeps in user-owned unrelated changes
- Push to remote automatically just to create a safety net
- Replace a dangerous command with an equally destructive variant and call it safer

## Completion Checklist

Before finishing, ensure:

- The destructive target was identified precisely.
- The risk level was judged explicitly.
- Recoverability was considered.
- Silent side-effect loss was checked for existing-file edits.
- Cumulative destructive actions were tracked within the current task.
- A lower-loss alternative was considered before destructive execution.
- User-owned or unclear-owned information was protected.
- Any required confirmation was obtained.
