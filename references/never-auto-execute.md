# Never Auto-Execute

These commands or patterns must not be executed automatically during destructive work:

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

## Rule

If a task appears to require one of these:

1. Stop.
2. Explain the risk.
3. Ask for explicit confirmation or propose a safer alternative.

## Why

These operations are too easy to misuse and can destroy user-owned work, repository state, or data in ways that are hard to recover.
