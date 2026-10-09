---
name: verify-and-stop
description: Translate acceptance conditions into smallest sufficient proof set. Run focused checks before comprehensive test gates. Stop when acceptance criteria pass — no polish, no extras.
---

# Verify and Stop

Translates acceptance conditions into the smallest sufficient proof set, runs focused checks, and stops the moment criteria pass.

## Proof Set Construction

- Translate acceptance conditions into the smallest set of checks that constitute proof.
- Reuse validated results when repository state matches current conditions.

## Execution Order

1. Run focused checks first: targeted tests, grep verification, direct exercise.
2. Run comprehensive test gate only if focused checks are insufficient or reveal gaps.

## Reporting

Report outcomes using exactly these four states:

- `PASS` — criterion met
- `FAIL` — criterion not met (include evidence)
- `UNAVAILABLE` — cannot be verified in current environment
- `BLOCKED` — dependency or prerequisite missing

No other outcome language.

## Constraints

- Preserve product code — no modifications unless the verification request explicitly includes fixes.
- Skip enhancements: no polish, no cleanup, no supplementary tests once acceptance criteria pass.
- Stop immediately when acceptance proof is complete.

## Documentation

Record: commands run, results, unresolved risks. Nothing else.
