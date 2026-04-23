# Change Check Template

Use this template before medium-risk or high-risk destructive work.

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

Use this template when repeated low-risk destructive actions are upgraded.

```text
Cumulative destructive change check:
- Actions so far: <count and type>
- Affected scope: <files / blocks / directory / module>
- Information removed: none / minor / meaningful / unclear
- Recovery path: git / backup / manual / none
- Next action: proceed / show diff first / ask user first
```

Use this template after destructive work.

```text
Destructive change summary:
- Changed: <what was removed or replaced>
- Scope verified: yes / partial / no
- Recovery path: git / backup / manual / none
- Safer alternative used: yes / no
- User confirmation required: yes / no
- Residual risk: none / low / medium / high
```

## Rules

- Be specific about the target.
- Do not hide uncertainty.
- If ownership is unclear, say `unknown`.
- If recovery is not realistic, say `none`.
- If recovery may not restore the latest state completely, say `partial`.
