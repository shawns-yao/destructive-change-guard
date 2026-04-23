# Destructive Change Guard

`destructive-change-guard` is a Codex skill that adds safety checks before destructive work.

It helps agents avoid accidentally deleting, overwriting, shrinking, resetting, rewriting, renaming, moving, or silently damaging user-owned information.

## What It Protects

This skill treats the guarded object as information, not just files.

It protects:

- code logic
- comments and rationale
- documentation and prompts
- configuration meaning
- tests and examples
- user-authored wording
- data and records
- imports, links, routes, assets, and references
- repository state and uncommitted work

## When To Use

Use this skill when a task may remove or make existing information hard to recover.

Typical triggers include:

- deleting files, directories, code, docs, tests, config, or data
- cleanup requests with unclear scope
- whole-file rewrites when a targeted edit would be safer
- bulk search-and-replace
- git reset, checkout, clean, rollback, or discard operations
- database DELETE, DROP, or TRUNCATE operations
- rename or move operations that may break references
- repeated small removals that accumulate into meaningful information loss
- ordinary edits that may silently remove comments, tests, behavior, references, or user wording as a side effect

## Key Behaviors

- Uses light triggers for precise low-risk edits.
- Allows fast exemption for harmless edits such as removing one blank line.
- Requires stronger checks for medium-risk and high-risk destructive work.
- Checks target, change type, ownership, scope, information loss, and recoverability.
- Tracks cumulative destructive actions within the same task.
- Treats partial recovery as a real risk, not a safe baseline.
- Treats rename and move operations as potentially destructive when references may break.
- Avoids assuming Git is enough for recovery.
- Requires explicit user confirmation for high-risk destructive actions.

## Safety Model

Before medium-risk or high-risk destructive changes, the skill asks the agent to classify:

- target
- change type
- information loss risk
- recovery level
- ownership
- scope
- safer alternatives
- action decision

For medium-risk or high-risk work, it also requires a recovery gate:

- Git repository state
- recoverable baseline
- recovery completeness
- working tree state
- user-owned uncommitted changes
- backup method
- remote backup status

## Never Auto-Execute

The skill explicitly blocks automatic execution of commands such as:

```text
rm -rf
del /f /q
rmdir /s /q
git reset --hard
git checkout -- <path>
git clean -fd
git clean -fdx
truncate
drop table
delete from <table>
```

These require explicit user intent and confirmation.

## Files

```text
destructive-change-guard/
├── SKILL.md
└── references/
    ├── change-check-template.md
    ├── never-auto-execute.md
    ├── recovery-gate.md
    ├── risk-levels.md
    └── safer-alternatives.md
```

## Installation

Clone or copy this folder into your Codex skills directory.

Example:

```powershell
git clone https://github.com/shawns-yao/destructive-change-guard.git
```

Then place the skill folder where your Codex setup discovers local skills.

## Notes

This skill is intentionally conservative, but it includes fast-exemption rules to avoid slowing down obviously harmless edits.

The goal is not to prevent deletion forever. The goal is to prevent accidental information loss, especially when the change is hard to recover or ownership is unclear.
