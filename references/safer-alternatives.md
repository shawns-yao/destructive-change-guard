# Safer Alternatives

Use this order when destructive work is possible but not yet clearly necessary.

## Preferred Order

1. Narrow the scope.
2. Show the target or diff.
3. Disable, isolate, move, rename, or deprecate.
4. Replace only the necessary block.
5. Delete or rewrite only if smaller changes are not enough.

## Rename And Move Caution

Rename and move operations are lower-loss than deletion only when references remain intact.

Treat rename or move as medium risk when:

- imports, routes, links, assets, tests, config, or external references may point to the old path
- multiple files or directories are involved
- the target is part of a public API, CLI entry, package export, migration, or generated contract

Before medium-risk rename or move, identify likely references and update them in the same change when appropriate.

## Examples

- rename a conflicting config key before deleting a whole config section
- comment or mark a legacy path as deprecated before removing it entirely
- replace one paragraph instead of rewriting the full document
- remove one verified dead branch instead of rewriting the module for neatness

## Rule

If a lower-loss option can satisfy the task with acceptable clarity and maintenance cost, prefer it over deletion or full rewrite.
