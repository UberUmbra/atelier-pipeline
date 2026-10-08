## DoR: Diff Metadata
**Files:** 25 | **Added:** ~1350 | **Removed:** ~50
**Functions modified:** pipeline-setup SKILL.md (platform detection, assembly manifest, overlay procedures), hooks.md (Step 3d VSCode installation), directory-layout.md (VSCode section)
**New files:** source/shared/agents/eva.md, source/vscode/agents/* (14 frontmatter overlays), source/vscode/hooks/hooks.json, source/vscode/hooks/enforcement-config.json, source/vscode/rules/* (2), source/vscode/variants/* (4), docs/architecture/ADR-0061, docs/product/vscode-copilot-port.md
**New dependencies:** none

## Exercised

Read `source/cursor/hooks/enforce-paths.sh` (the file the VSCode `hooks.json` references) lines 26–29; confirmed tool-name case filter is `case "$TOOL_NAME" in Write|Edit|MultiEdit)`. Grepped `source/claude/hooks/` for `enforce-paths.sh` — not present; the file exists only in `source/cursor/hooks/`. Verified the diff's install manifest copies `source/claude/hooks/enforce-*.sh` (all matching) to `.github/hooks/` — `enforce-paths.sh` is not in that source tree and is not installed. Read `source/claude/hooks/enforce-colby-paths.sh` for independent confirmation of the PascalCase filter (line 15). Checked `source/vscode/agents/eva.frontmatter.yml` model field against `source/claude/agents/` equivalents — `claude-opus-4-7` and `claude-sonnet-4-6` match the existing Claude Code overlays.

Exercise impractical for live VSCode hook execution (no Copilot runtime available); relied on static analysis of scripts and the assembly manifest.

## DoD: Verification
**Findings:** 2 | **Categories checked:** logic, security, resources, cross-layer wiring, dead code | **Grep verified:** enforce-paths.sh location, TOOL_NAME case filter, colby_blocked_paths .github/ entry, model IDs | **Exercised:** cursor enforce-paths.sh case logic, claude enforce-colby-paths.sh case logic, VSCode manifest copy source vs. hooks.json reference

## Findings

| # | Location | Severity | Category | Description | Suggested Fix |
|---|----------|----------|----------|-------------|---------------|
| 1 | `source/cursor/hooks/enforce-paths.sh:26–29`, all `source/claude/hooks/enforce-*-paths.sh:15` | BLOCKER | Logic / Security | Tool name case mismatch silently disables all path enforcement in VSCode. Scripts filter with `case "$TOOL_NAME" in Write\|Edit\|MultiEdit)` (PascalCase). VSCode/Copilot emits lowercase tool names (`edit`, `execute`). When the hook fires, `TOOL_NAME=edit` does not match any branch; the `*) exit 0` default fires and every call passes through harmlessly. Path enforcement is completely bypassed at runtime. The diff's claim "No changes to the script are needed" is incorrect. | Add lowercase arms to the case statement in every enforcement script: `Write\|Edit\|MultiEdit\|write\|edit\|multiedit)` — or normalise with `TOOL_NAME_LOWER=$(echo "$TOOL_NAME" \| tr '[:upper:]' '[:lower:]')` before the case. |
| 2 | `source/vscode/hooks/hooks.json:7`, assembly manifest in `skills/pipeline-setup/hooks.md:1118` | FIX-REQUIRED | Cross-layer wiring (missing file) | `hooks.json` registers `.github/hooks/enforce-paths.sh` as the sole PreToolUse enforcement command, but that file is never installed. The manifest copies `source/claude/hooks/enforce-*.sh` to `.github/hooks/` — `enforce-paths.sh` exists only in `source/cursor/hooks/`, outside the glob. At install time the hook destination is a missing script; Copilot fires the hook against a path that does not exist, producing a silent failure or runtime error rather than enforcement. | Add an explicit copy step: `source/cursor/hooks/enforce-paths.sh` → `.github/hooks/enforce-paths.sh`. Alternatively, register the per-agent scripts directly in `hooks.json` with per-agent conditions (as Claude Code does in `settings.json`), eliminating the dispatcher dependency. |
