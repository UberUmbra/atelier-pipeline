---
name: caveman-commit
description: Terse commit messages. Conventional Commits format, subject ≤50 chars. Body only when WHY is non-obvious. For Ellis.
---

# Caveman Commit Mode

Terse commit messages for Ellis. Conventional Commits format.

## Subject Format

```
type(scope): subject
```

- Subject: ≤50 chars, imperative mood, no period at end
- Scope: optional, lowercase, matches changed subsystem

## Types

| Type | Use for |
|------|---------|
| `feat` | New capability |
| `fix` | Bug fix |
| `chore` | Maintenance, dependency updates |
| `docs` | Documentation only |
| `refactor` | Code change, no behavior change |
| `test` | Tests only |
| `perf` | Performance improvement |

## Subject Verb Rules

Prefer short verbs:
- "add" not "add support for" or "implement"
- "fix" not "fix the issue with"
- "use" not "update to use" or "switch to using"
- "remove" not "remove the unnecessary"
- "rename" not "rename X to Y for clarity"

## Body

Include body only when WHY is non-obvious from the diff and subject.

Good body candidate: architectural decision, workaround for upstream bug, intentional tradeoff.
Skip body: mechanical change, obvious fix, renaming.

Body format: one paragraph, plain prose, ≤72 chars per line.

## Breaking Changes

Append `!` after type: `feat!: change config key name`

Or footer: `BREAKING CHANGE: description of what breaks and migration path.`

## Examples

```
feat(pipeline-config): add caveman_mode field
fix(hooks): null-check AGENT_NAME before path enforcement
chore: remove stale pipeline-state entries
refactor(ellis): extract commit-body logic to helper
```
