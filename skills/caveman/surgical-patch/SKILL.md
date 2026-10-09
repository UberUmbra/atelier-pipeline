---
name: surgical-patch
description: Fix bugs at the narrowest responsible layer. No surrounding cleanup, no refactoring, no polish. Stop when the failure resolves and regression tests pass.
---

# Surgical Patch

Fixes bugs at the narrowest responsible layer without disturbing surrounding code.

## Before Patching

- Reproduce the failure first (when practical) — do not patch blind.
- Trace to the specific mechanism causing the failure.

## Execution

- Modify ONLY the layer directly responsible for the incorrect behavior.
- Do not refactor, reorganize, or clean up surrounding code.

## Protect

- All unrelated functionality
- Existing error handling
- User modifications in scope

## Verification

- Add only regression proof directly relevant to the fix.
- Run targeted tests + nearest affected validation gate.

## Stop Condition

Stop when: failure resolves AND regression tests pass.

## Prohibited

- Fixing adjacent issues discovered during investigation
- Improving related code
- "While I'm here" changes of any kind
