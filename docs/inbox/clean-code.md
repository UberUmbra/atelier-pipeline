### Executive Summary

#### Overview

While autonomous coding agents drastically accelerate initial feature delivery, long-horizon automated coding introduces compounding structural technical debt—frequently termed **code slop**—that severely degrades codebase maintainability. Because AI agents prioritize local task completion within immediate context windows, unconstrained agentic pipelines systematically sacrifice architectural integrity, code reuse, and non-functional quality characteristics.

---

#### Key Maintainability Issues in Agentic Codebases

1. **Proliferation of Duplicate Code & Low Reuse**: Code duplication has increased by 81%, while structural code reuse (moving/refactoring existing code) has fallen by 70%. Agents routinely write bespoke helper functions rather than exploring the repository to import pre-existing abstractions.


2. **The 80/20 Non-Functional Quality Gap**: Agents rapidly deliver the functional core (~80% of the task) but omit the critical 20% covering error handling, input validation, rate limiting, and security controls. Because standard unit tests focus on nominal functional paths, this non-functional deficit bypasses CI and accumulates as invisible shadow debt.


3. **Defensive Error Suppression**: Defensive error suppression (catching exceptions without evaluating or logging root causes) has increased by 47%. To avoid execution failures, agents wrap brittle logic in generic try-catch blocks, leading to silent runtime failures in production.


4. **Comprehension Debt & Legacy Decay**: Human developers increasingly accept working agent patches without fully understanding the design logic, leading to comprehension debt. Furthermore, updates to legacy code (>12 months old) have fallen by 74%, as agents prefer creating parallel, isolated code silos rather than refactoring legacy modules.



---

#### Core Architectural Guardrails & Best Practices

* **Spec-Driven Development (SDD)**: Require human engineers or Architect Agents to write formal, machine-readable specifications defining functional boundaries, error propagation, and security requirements prior to code generation.


* **Deterministic Boundary Enforcement**: Replace natural language context files (e.g., `AGENTS.md`) with executable architectural fitness functions. Tools like `import-linter` (Python), `dependency-cruiser` (JS/TS), and `Semgrep` deterministically block prohibited cross-layer imports and code smells.


* **Machine-Readable ADRs & AgDRs**: Capture architecture decisions in structured Architecture Decision Records (ADRs) or real-time Agent Decision Records (AgDRs) containing explicit file-glob scopes, mandatory directives, and CLI verification checks.


* **AST-Based Repository Maps**: Provide agents with `tree-sitter` AST maps ranked via PageRank (e.g., Aider's repo map model) to expose global function signatures and existing utility abstractions, driving code reuse.



---

#### Target Pipeline Architecture for "Senior / Clean Coder" Behavior

To force an agentic pipeline to behave like a Senior Clean Coder, engineering teams must implement a decoupled multi-agent topology supported by trajectory-time feedback loops:

1. **Role Decoupling**:
* **Architect / Planner Agent**: Reads user intent, evaluates AST repository maps and ADRs, defines interface contracts, and outputs a step-by-step implementation plan.


* **Generator / Executor Agent**: Writes code incrementally following strict Test-Driven Development (TDD) principles (failing test first, then minimal passing implementation).


* **Evaluator / Reviewer Agent**: Evaluates diffs against clean architecture rules, complexity metrics (e.g., CRAP scores), and static analysis before allowing commits.




2. **Quiet-Mode Verification Loops**:
* Wire local completion hooks and pre-commit triggers to run a combined verification task chain (`ruff check`, `mypy`, `import-linter`, `pytest -q`).


* Running tests and linters in quiet/silent modes minimizes context-window bloat on success, while routing exact failure stack traces back to the agent for immediate self-correction before any Git commit occurs.