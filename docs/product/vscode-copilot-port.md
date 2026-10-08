## DoR: Requirements Extracted
**Source:** User conversation (2026-10-08), prior Cursor port spec (docs/product/cursor-port.md), VSCode Agent Customization docs (code.visualstudio.com)

| # | Requirement | Source |
|---|-------------|--------|
| 1 | Agent Plugins 1.0 format — `plugin.json` with `$schema`, client-specific content under `com.github.copilot/` | User decision |
| 2 | Third build target alongside Claude Code + Cursor — not replacing either | User decision |
| 3 | Same `source/shared/` content bodies, new `source/vscode/` overlay directory | Architecture pattern from Cursor port |
| 4 | Must work with VSCode native chat (Copilot Enterprise) — not Claude Code extension | User constraint (toolset limitation) |
| 5 | Phase 1: migration only — functional parity with cloud models, no optimization | User decision |
| 6 | Phase 2 (future, out of scope): local model adoption, context slimming | User decision |
| 7 | Agents use `.agent.md` extension with YAML frontmatter (`name`, `description`, `tools` as array, `model`, `agents`, `handoffs`) | VSCode docs |
| 8 | Rules convert from `.mdc` to `.instructions.md` with `applyTo:` glob frontmatter | VSCode docs |
| 9 | Hooks use PascalCase event keys; matcher values are ignored — scripts must self-filter tool names | VSCode docs |
| 10 | Commands map to skills (`SKILL.md` format) or user-invocable agents | VSCode docs |
| 11 | `${PLUGIN_ROOT}` env token replaces `${CURSOR_PLUGIN_ROOT}` and `${CLAUDE_PLUGIN_ROOT}` | VSCode docs |
| 12 | No breaking changes to existing Claude Code or Cursor files | Carried from Cursor port spec |
| 13 | Improvements to one target must carry to all three (shared `source/`) | Carried from Cursor port spec |
| 14 | Full agent roster — all 14 agents + discovered agents | User confirmed |
| 15 | Copilot Enterprise tier — agent mode, MCP, model picker, hooks all available | User confirmed |

**Retro risks:** From Cursor port — behavioral constraints ignored by LLMs; enforcement hooks critical, must port correctly. Hook matcher semantics differ (VSCode ignores matcher values) — higher risk than Cursor port.

---

# Feature Spec: VSCode/Copilot Port of Atelier Pipeline (Agent Plugins 1.0)
**Author:** Robert (CPO) | **Date:** 2026-10-08
**Status:** Draft — Pending Review

## The Problem

User's toolset is limited to VSCode. Atelier Pipeline ships to Claude Code and Cursor but not VSCode/GitHub Copilot. Without this port, the pipeline is unusable in the constrained environment. This is a compliance-driven need, not a market play.

## Who Is This For

1. **The user** — developer constrained to VSCode who needs the pipeline's multi-agent orchestration, quality gates, and institutional memory.
2. **Future VSCode-only users** — teams where Cursor and Claude Code are not approved tools.

## Business Value

- **Unblocks usage** in environments where VSCode is the only approved IDE
- **Platform reach** — VSCode + Copilot is the largest AI-assisted coding market; having a target there is strategic even if not the primary driver
- **Future-proofs for local models** — Phase 2 leverages Copilot's model picker for custom/local model assignment per agent

**KPIs:**

| KPI | Definition | Measurement | Timeframe | Acceptance |
|-----|-----------|-------------|-----------|------------|
| Functional parity | All 14 agents operational in VSCode Copilot | Count of agents that complete their role in a pipeline run | 60 days | 14/14 agents work |
| Hook enforcement | Path enforcement hooks block correctly despite no matcher filtering | Count of blocked violation attempts vs. total attempts | During testing | 100% violations blocked, 0% false denials |
| Pipeline completion | Full pipeline (Robert → Sable → Sarah → Colby → Poirot) completes | End-to-end pipeline run | During testing | 1+ successful full pipeline |
| Model routing | Agent-specified models are respected by Copilot | Verify model assignment per agent | During testing | Model picker shows correct model per agent |

## User Stories

### US-1: Plugin installation
As a VSCode user, I install atelier-pipeline as an Agent Plugin 1.0 and get all 14 agent personas, enforcement hooks, MCP integration, and pipeline skills — the same experience as Claude Code and Cursor users.

### US-2: Shared source, triple output
As a maintainer, when I update a file in `source/shared/`, the change is available to Claude Code, Cursor, and VSCode users. I maintain one copy of each agent persona, rule, and command body.

### US-3: Eva orchestration in Copilot
As a VSCode user, Eva orchestrates my pipeline — routing to agents, managing phase transitions, enforcing quality gates. Orchestration rules load as always-apply instructions.

### US-4: Enforcement hooks in Copilot
As a VSCode user, enforcement hooks block agents from writing outside their lanes. Hooks fire on all PreToolUse events; scripts self-filter by tool name from stdin JSON.

### US-5: Brain integration in Copilot
As a VSCode user, the brain MCP server connects via `mcp.json`. Brain captures, searches, and telemetry work identically.

### US-6: Agent invocation via Copilot chat
As a VSCode user, I invoke agents via `@agent-name` in Copilot chat. Agents appear in the agent picker with correct names and descriptions.

### US-7: Model assignment per agent
As a VSCode user, each agent uses the model specified in its frontmatter. Eva and Sarah use a capable cloud model; Colby and scouts can be assigned different models.

### US-8: Subagent orchestration
As a VSCode user, Eva can delegate to subagents (Colby, Sarah, Poirot, etc.) using Copilot's `agents` allow-list and `handoffs` mechanism, achieving the same multi-agent workflow as Claude Code's Agent tool.

## User Flow

### Installation
```
Clone/install atelier-pipeline plugin
  → VSCode reads plugin.json ($schema: Agent Plugins 1.0)
  → Discovers: agents/, skills/, mcp.json
  → Client-specific content from com.github.copilot/: hooks, instructions
  → Run /pipeline-setup skill to configure project-specific values
  → Pipeline ready
```

### Pipeline execution
```
User types in Copilot Chat → Eva (default agent or instructions-based) routes
  → @robert-spec for feature discovery
  → @sable-ux for UX design
  → @sarah for architecture
  → @colby for implementation (hooks enforce paths)
  → @poirot for blind review
  → @ellis for commit
```

## Edge Cases and Error Handling

| Case | Handling |
|------|---------|
| User has Claude Code, Cursor, AND VSCode in same project | `.claude/`, `.cursor-plugin/`, and Agent Plugins 1.0 structure coexist — different directories |
| Hook fires on non-Write tool (matcher ignored) | `enforce-paths.sh` reads tool name from stdin JSON; returns `allow` for non-Write tools. Already does this — verify. |
| Hook script fails | Fail-open by default; document how to configure fail-closed |
| Brain MCP server not configured | Pipeline runs in baseline mode (no brain) — same as other targets |
| Copilot model not available (e.g., specified model not in Enterprise tier) | `model` field supports priority array — fall back to next model |
| VSCode doesn't support `effort`, `color`, `maxTurns`, `permissionMode` | Omit from VSCode frontmatter — no functional impact (cosmetic + safety-net fields) |
| Subagent depth limits in Copilot | Eva delegates one level deep — same as current design. Test multi-agent chains. |
| `.mdc` rule glob patterns don't map cleanly to `.instructions.md` `applyTo:` | Map individually; some rules become always-apply (no glob) |

## Acceptance Criteria

| # | Criterion | Measurable |
|---|-----------|------------|
| AC-1 | `plugin.json` with `$schema` for Agent Plugins 1.0 format | File present, valid JSON, schema validates |
| AC-2 | All 14 agents discoverable as `.agent.md` files | All agents appear in Copilot agent picker |
| AC-3 | Instructions files load as always-apply or glob-scoped | Eva persona active in every chat; scoped rules active per glob |
| AC-4 | `hooks.json` under `com.github.copilot/` registers enforcement hooks | PreToolUse hooks fire on tool calls |
| AC-5 | `enforce-paths.sh` correctly self-filters tool names (no false denials) | Test: Read tool → allowed; Write outside lane → denied |
| AC-6 | `mcp.json` registers brain MCP server | Brain tools available in Copilot |
| AC-7 | Skills discoverable and invocable | `/pipeline-setup` works in Copilot |
| AC-8 | Model assignment per agent works | Agent frontmatter `model:` respected by Copilot |
| AC-9 | Eva routes to subagents via handoffs or agent allow-list | Multi-agent pipeline completes |
| AC-10 | Existing Claude Code files unchanged | `git diff .claude/` shows zero changes |
| AC-11 | Existing Cursor files unchanged | `git diff .cursor-plugin/` shows zero changes |
| AC-12 | `source/shared/` not duplicated | VSCode plugin references shared content bodies |
| AC-13 | `source/vscode/` overlay directory created | VSCode-specific frontmatter and config live here |
| AC-14 | Pipeline-setup skill handles VSCode target | Setup detects IDE and configures correctly |

## Scope

### In scope (Phase 1 — Migration)
- Agent Plugins 1.0 `plugin.json` manifest with `$schema`
- `source/vscode/` overlay directory (frontmatter for agents, hooks config, instruction frontmatter)
- 14 agent personas as `.agent.md` files with VSCode-compatible frontmatter
- 21 rules converted to `.instructions.md` format (always-apply or `applyTo:` glob-scoped)
- Hooks adapted: PascalCase format, `${PLUGIN_ROOT}` token, self-filtering scripts
- Commands mapped to skills or user-invocable agents
- `mcp.json` for brain MCP server
- Pipeline-setup skill adapted for VSCode
- Build/install script that assembles VSCode target from `source/vscode/` + `source/shared/`

### Out of scope (Phase 1)
- Local model optimization (Phase 2)
- Context slimming for smaller context windows (Phase 2)
- VSCode Marketplace publication (after validation)
- Modifying existing Claude Code or Cursor plugin structures
- Copilot-specific UI components or themes
- Agent Plugins 1.0 `automations/` (future consideration)
- Supporting VSCode versions older than current stable

### Out of scope (permanently)
- Replacing Claude Code or Cursor targets
- Breaking changes to `source/shared/`

## Non-Functional Requirements

| NFR | Requirement |
|-----|-------------|
| Zero duplication | Agent bodies exist once in `source/shared/` — all three plugins reference them |
| Additive only | No changes to `.claude/`, `.claude-plugin/`, `.cursor-plugin/`, or existing source overlays |
| Same shell scripts | Hooks call the same `enforce-*.sh` scripts — logic is not duplicated per platform |
| Platform detection | Hook scripts detect IDE via env vars (`PLUGIN_ROOT`, `CURSOR_PLUGIN_ROOT`, `CLAUDE_PLUGIN_ROOT`) |
| Graceful degradation | Missing VSCode-unsupported fields (effort, color, maxTurns, permissionMode) are silently omitted |
| Build reproducibility | `source/vscode/` + `source/shared/` assembly produces identical output given identical inputs |

## Dependencies

| Dependency | Required by | Risk |
|-----------|-------------|------|
| VSCode (current stable) | All | None — target platform |
| GitHub Copilot Enterprise | Agent mode, MCP, model picker, hooks | None — user confirmed |
| jq | Enforcement hooks | Low — already required |
| Node.js >= 18 | Brain MCP server | Low — already required |
| Agent Plugins 1.0 spec stability | Plugin format | Medium — spec may evolve; pin to current version |

## Risks and Open Questions

| # | Risk/Question | Severity | Mitigation |
|---|--------------|----------|------------|
| 1 | **Hook matcher ignored** — `enforce-paths.sh` fires on every tool call, not just Write/Edit/MultiEdit | High | Verify script already self-filters from stdin JSON; add early-exit for non-write tools if not |
| 2 | **Subagent orchestration model differs** — Claude Code uses explicit `Agent` tool; Copilot uses `agents` allow-list + `handoffs` | High | Map Eva's routing to handoffs; test multi-step pipeline chains; may need Eva-as-instructions rather than Eva-as-agent |
| 3 | **Rule conversion fidelity** — 21 `.mdc` files (4900+ lines) converting to `.instructions.md` may lose glob scoping or load ordering | Medium | Test each rule individually; accept some rules becoming always-apply if glob mapping is imprecise |
| 4 | **`model` field array support** — VSCode docs say `model` supports priority array; need to verify Enterprise tier has the models we specify | Low | Test with available models; document fallback chain |
| 5 | **Eva as orchestrator** — Eva's always-loaded persona may conflict with Copilot's own system behavior | Medium | Test Eva instructions don't get overridden or ignored; may need stronger instruction positioning |
| 6 | **Agent Plugins 1.0 spec maturity** — format may change | Medium | Pin `$schema` version; monitor spec updates |
| 7 | **Context window pressure** — 14 agents + 21 instruction files + hooks + MCP = heavy context | Low (Phase 1) | Phase 2 addresses this; Phase 1 uses cloud models with large context |

## Timeline Estimate

Medium pipeline. Estimated 4-5 ADR steps:

1. **Plugin structure** — Agent Plugins 1.0 manifest, directory layout, `source/vscode/` overlay scaffold
2. **Agent conversion** — 14 `.agent.md` files with VSCode frontmatter overlays, subagent wiring (handoffs + agents allow-list)
3. **Rules conversion** — 21 `.mdc` → `.instructions.md`, glob mapping, load order verification
4. **Hooks adaptation** — PascalCase format, `${PLUGIN_ROOT}` token, self-filter verification, `com.github.copilot/hooks/`
5. **Skills, MCP, and setup** — pipeline-setup for VSCode, mcp.json, build/install script

Steps 2 and 3 can potentially parallelize. Step 1 is prerequisite for all others.

## DoD: Verification

| # | Requirement | Status | Evidence |
|---|-------------|--------|----------|
| 1 | Agent Plugins 1.0 format | Pending | plugin.json with $schema |
| 2 | Third build target (additive) | Pending | source/vscode/ exists; .claude/ and .cursor-plugin/ untouched |
| 3 | Same source/shared/ bodies | Pending | No duplication in diff |
| 4 | VSCode native chat (Copilot) | Pending | Agents invocable in Copilot |
| 5 | .agent.md with correct frontmatter | Pending | All 14 agents in picker |
| 6 | .instructions.md rules | Pending | Eva persona loads; scoped rules activate |
| 7 | Hooks self-filter correctly | Pending | Write blocked, Read allowed |
| 8 | Commands → skills | Pending | /pipeline-setup works |
| 9 | Brain MCP | Pending | mcp.json, tools available |
| 10 | Model routing | Pending | model field respected |
| 11 | Subagent orchestration | Pending | Full pipeline completes |
| 12 | No Claude Code changes | Pending | git diff .claude/ clean |
| 13 | No Cursor changes | Pending | git diff .cursor-plugin/ clean |
| 14 | source/vscode/ overlays | Pending | Frontmatter files present |
| 15 | Build/install script | Pending | Assembly produces correct output |

**Grep check:** TODO/FIXME/HACK/XXX in output files → 0
**Template:** All sections filled — no TBD, no placeholders
