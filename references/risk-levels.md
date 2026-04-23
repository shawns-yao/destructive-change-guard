# Risk Levels

Use these levels when deciding whether a destructive change can proceed.

## Low Risk

Typical examples:

- Agent-created temporary files from the current task
- Build artifacts, generated caches, or obvious throwaway outputs
- Tiny precise removals with no information loss
- Verified dead code removal with narrow scope
- Fast-exempt edits where only whitespace, duplicated blank lines, duplicated punctuation, or obvious typo tokens change

Default action:

- Proceed
- State what changed

Fast exemption only applies when the target is precise and visible, recovery is easy, and no user-authored meaning, logic, comments, tests, config, data, references, or structure is removed.

## Medium Risk

Typical examples:

- Removing or replacing a meaningful code block
- Rewriting a paragraph or section
- Bulk replacement across one or more files
- Changing existing config with semantic impact
- Cleanup with some uncertainty but bounded scope
- Existing-file edits that may silently remove comments, rationale, tests, references, or behavior as a side effect
- Rename or move operations that may break imports, links, routes, assets, tests, config, or external references
- Three or more low-risk destructive actions accumulated in the same task

Default action:

- Minimize the change
- Show the target clearly
- Prefer diff, preview, or dry-run first

## High Risk

Typical examples:

- Deleting user-authored files or text
- Clearing directories
- Discarding uncommitted work
- Git reset / rollback that can lose changes
- Database deletion or truncation
- Whole-file or whole-module rewrite with uncertain information loss
- Any change with recovery level `hard` or `none`
- Any high-risk change where recovery is only `partial`

Default action:

- Ask the user first
- Explain impact and recovery path

## Escalation Rule

If ownership is unclear, upgrade one risk level.

If recoverability is poor, upgrade one risk level.

If repeated low-risk destructive actions accumulate across multiple files, a directory, a module, or user-authored content, upgrade to medium risk and summarize before continuing.
