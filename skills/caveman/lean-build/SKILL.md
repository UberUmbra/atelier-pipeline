---
name: lean-build
description: Strict scope enforcement for build tasks. Convert feature request to narrow, complete outcome. Explicit stop condition required. No modes, config, extensibility, or polish unless acceptance criteria require them.
---

# Lean Build

Strict scope enforcement for build tasks. Converts a feature request into a narrow, complete outcome with an explicit stop condition.

## Before Building

- Establish observable acceptance criteria — what passes, what fails, what is out of scope.
- State explicit non-goals: what will not be built in this unit.

## Execution

- Trace entry points through layers that maintain system invariants.
- Deliver coherent end-to-end path across responsible layers — do not isolate work to one file when the intent requires multiple.
- Reuse existing seams; refactor only when needed to avoid duplicating behavior.

## Exclusions (unless acceptance explicitly requires them)

- Modes, providers, configuration options
- Extensibility hooks, plugin points
- Polish, formatting improvements, cosmetic changes

## Scope Gates

- Add a surface, dependency, service, configuration item, or migration only when necessary for the acceptance criteria.
- If a candidate addition cannot be traced to an acceptance criterion, exclude it.

## Stop Condition

Work is done when acceptance criteria pass — not when the code looks complete.

## Report

Report only: material omissions and scope decisions made. Not a summary of every file touched.
