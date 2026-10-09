---
name: caveman-review
description: Terse code review format. Each finding is one line: file:L{n}: {severity emoji}: {problem}. {fix}. Use for Poirot output and PR reviews.
---

# Caveman Review Mode

Terse code review format for Poirot findings and PR reviews.

## Line Format

```
path/to/file.ext:L{n}: {emoji} {problem}. {fix}.
```

For multi-line spans:
```
path/to/file.ext:L{n}-{m}: {emoji} {problem}. {fix}.
```

## Severity Emojis

| Emoji | Severity | Use for |
|-------|----------|---------|
| 🔴 | bug | Broken behavior — incorrect output, crash, data loss |
| 🟡 | risk | Fragile — will break under edge case or load |
| 🔵 | nit | Style — works but violates convention |
| ❓ | q | Question — unclear intent, needs clarification |

## Rules

- **Exact line numbers.** Only cite ranges you actually examined. Never fabricate.
- **Backtick symbols.** Identifiers, method names, config keys: `backtick_wrapped`.
- **Concrete fix.** State the fix, not "consider refactoring". "Replace X with Y."
- **Why when non-obvious.** Omit explanation when the problem is self-evident.
- **One finding per line.** Do not combine multiple issues.

## Strip

- Throat-clearing: "I noticed that…", "It seems like…"
- Hedging: "You might want to…", "It could be worth…"
- Praise: "Great job on…", "This is well-structured overall."
- Restating code logic back in prose
- Vague suggestions: "This could be improved."

## Exception

Security findings and architectural concerns may use a full sentence for the fix when one line is ambiguous or incomplete.

## Example

```
src/hooks/enforce-agent-paths.sh:L47: 🔴 Missing null check on $AGENT_NAME. Add `[[ -z "$AGENT_NAME" ]] && exit 1` before L47.
src/hooks/enforce-agent-paths.sh:L82: 🟡 `grep -r` without `--` treats leading-dash paths as flags. Use `grep -r -- "$pattern"`.
pipeline-config.json:L12: 🔵 `caveman_mode` key missing from template. Add after `brevity`.
```
