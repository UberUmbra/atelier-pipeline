## DoR: Diff Metadata
**Files:** 27 | **Added:** ~750 | **Removed:** ~15
**Functions modified:** session-boot.sh (CAVEMAN_MODE variable + jq read + JSON output), eva.md (caveman_mode behavior block), default-persona.md (caveman-mode section), agent-system.md (cave agent table rows), pipeline-config.json (caveman_mode field)
**New dependencies:** none (jq already required)
**New files:** skills/caveman/* (10 skill files), source/claude/agents/colby-cave.frontmatter.yml, source/claude/agents/poirot-cave.frontmatter.yml, source/cursor/agents/colby-cave.frontmatter.yml, source/cursor/agents/poirot-cave.frontmatter.yml, source/vscode/agents/colby-cave.frontmatter.yml, source/vscode/agents/poirot-cave.frontmatter.yml, source/shared/agents/colby-cave.md, source/shared/agents/poirot-cave.md

## Exercised
- session-boot.sh: exercised against pipeline-config.json with `caveman_mode: true`. Output confirmed `caveman_mode: True` in JSON. Matches the diff's intent.
- enforce-colby-paths.sh: ran existing test suite (`tests/hooks/test_enforce_colby_paths.py`) — 3 tests passed. The script itself is correct; the wiring issue is in the frontmatter, not the script.
- pipeline-config.json: verified `caveman_mode: true` is present in the template via `jq -r '.caveman_mode'`.

## DoD: Verification
**Findings:** 2 | **Categories checked:** logic, security, error handling, dead code, type safety, cross-layer wiring | **Grep verified:** both findings confirmed via grep | **Exercised:** session-boot.sh CAVEMAN_MODE read, enforce-colby-paths.sh test suite

## Findings
| # | Location | Severity | Category | Description | Suggested Fix |
|---|----------|----------|----------|-------------|---------------|
| 1 | `source/claude/agents/colby.frontmatter.yml` | FIX-REQUIRED | Cross-layer wiring / security | The diff removes the `hooks:` block that wired `enforce-colby-paths.sh` to Colby. This was the **only** mechanical path-enforcement trigger for Colby (confirmed: not present in `settings.json` global hooks, not in any global PreToolUse chain — `docs/guide/technical-reference.md` L1256 states per-agent enforcement runs through frontmatter only). After the next install, Colby can write to `docs/`, `.github/workflows/`, and all other `colby_blocked_paths` without a block. The `colby-cave.frontmatter.yml` also has no `hooks:` entry, so both standard and cave Colby lose mechanical path enforcement. `skills/pipeline-setup/hooks.md` still documents the hook as a per-agent install requirement (L11), making the source and install docs inconsistent. | Re-add the hooks block to `colby.frontmatter.yml` (and add it to `colby-cave.frontmatter.yml`), or move `enforce-colby-paths.sh` into the global `settings.json` PreToolUse chain gated by `tool_input.subagent_type == 'colby' \|\| tool_input.subagent_type == 'colby-cave'`. Update `hooks.md` and `technical-reference.md` to match whichever approach is chosen. |
| 2 | `source/claude/hooks/enforce-pipeline-activation.sh` L39-42, `source/shared/pipeline/pipeline-config.json` L27 | FIX-REQUIRED | Cross-layer wiring / logic | `enforce-pipeline-activation.sh` gates only `subagent_type == 'colby'`. The new `pipeline-config.json` template ships with `caveman_mode: true`, which causes Eva to route Medium+ pipelines to `colby-cave` instead of `colby`. `colby-cave` uses a different subagent type and is never matched by the activation gate — Colby-cave can be invoked without an active pipeline, bypassing the telemetry-capture and quality-gate requirement. Grep confirms: `enforce-pipeline-activation.sh` has a single `colby)` case with no `colby-cave` branch. | Add `colby-cave)` to the case statement in `enforce-pipeline-activation.sh` alongside `colby)`. |
