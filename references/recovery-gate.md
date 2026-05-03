# Recovery Gate

Use this check before medium-risk or high-risk destructive changes.

## Required Questions

1. Is the target inside a Git repository?
2. Is there a recoverable baseline?
3. Is the working tree clean?
4. Are there user-owned or unclear uncommitted changes?
5. If Git is unavailable or insufficient, is there another approved backup path?

## Decision Rules

- No Git + medium/high risk -> do not proceed automatically
- Git exists + user-owned uncommitted changes -> ask the user first
- Git exists + no clear recovery point for high-risk work -> ask for checkpoint or explicit confirmation
- Low-risk cleanup of agent-created temporary files can proceed without Git if the scope is precise

## Important Constraint

Do not create commits, stash changes, reset changes, or push to remote on the user's behalf unless the user explicitly approves that path.

## Recovery Gate Template

```text
Recovery gate:
- Git repository: yes / no
- Recoverable baseline: yes / no / unclear
- Working tree state: clean / modified / unknown
- User-owned uncommitted changes: yes / no / unclear
- Backup method: existing commit / checkpoint commit / backup branch / manual backup / none
- Remote backup: available / not available / not required / unclear
- Action: proceed / ask user for checkpoint / do not proceed
```
