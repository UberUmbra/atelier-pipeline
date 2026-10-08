# ADR-0061: VSCode/Copilot Port of Atelier Pipeline

## DoR: Requirements Extracted
**Source:** `docs/product/vscode-copilot-port.md` (spec), `docs/product/cursor-port.md` (prior art), Sarah conversational phase (clarification decisions)

| # | Requirement | Source |
|---|-------------|--------|
| 1 | Third build target alongside Claude Code + Cursor -- not replacing either | Spec R2 |
| 2 | `.github/` layout: agents at `.github/agents/*.agent.md`, instructions at `.github/instructions/*.instructions.md`, hooks at `.github/hooks/hooks.json` | Clarification #1 |
| 3 | Same `source/shared/` content bodies, new `source/vscode/` overlay directory | Spec R3 |
| 4 | Eva: `.agent.md` with BOTH `handoffs:` (standard flow) AND `agent` tool + `agents: ["*"]` (dynamic routing), `disable-model-invocation: true` | Clarification #2 |
| 5 | Same Claude models via Copilot Enterprise -- Phase 1 only | Clarification #3 |
| 6 | Hook scripts self-filter tool names from stdin JSON -- already works, verify | Clarification #4 |
| 7 | VSCode tool names in frontmatter: `read`, `edit`, `execute`, `search`, `agent` | Spec R7, VSCode docs |
| 8 | `.instructions.md` with optional `applyTo:` globs -- always-apply when no `applyTo` | Spec R8 |
| 9 | Hooks: PascalCase events, matcher values IGNORED, scripts use project-relative paths | Spec R9 |
| 10 | No `effort`, `color`, `maxTurns`, `permissionMode`, per-agent `mcpServers`, per-agent inline `hooks` in VSCode frontmatter | Spec edge cases |
| 11 | All 14 agents + discovered agents | Spec R14 |
| 12 | Phase 1 scope: migration only, no context slimming or local model optimisation | Clarification #6 |
| 13 | Existing `.claude/` and `.cursor-plugin/` files unchanged | Spec R12 |
| 14 | Compliance-driven -- single user, VSCode-only toolset constraint | Clarification #7 |
| 15 | Pipeline-setup skill handles VSCode as a third target | Spec AC-14 |

**Retro risks:** From Cursor port -- behavioral constraints ignored by LLMs; enforcement hooks critical, must port correctly. Hook matcher semantics differ more sharply in VSCode (matcher values outright ignored) than in Cursor.

---

## Status
Proposed.

## Context

Atelier Pipeline ships to Claude Code (`.claude/`) and Cursor (`.cursor-plugin/`) but not VSCode/Copilot. The user's toolset is limited to VSCode. Without this port, the pipeline is unusable. This is a compliance-driven need, not a market play.

VSCode/Copilot uses a different file layout than both Claude Code and Cursor. Agents are `.agent.md` files with YAML frontmatter. Rules become `.instructions.md`. Hooks use the same PascalCase JSON format as Cursor but matcher values are ignored -- all hooks for an event fire unconditionally, and scripts must self-filter. Tool names differ from Claude Code (`read` vs `Read`, `edit` vs `Edit`, `execute` vs `Bash`, `search` vs `Glob`/`Grep`).

The Cursor port (ADR-0046 era) established the dual-target pattern: shared content bodies in `source/shared/`, platform-specific frontmatter overlays in `source/{platform}/`. This ADR extends that pattern to a third target.

## Options Considered

**Option A -- `.github/` layout with `source/vscode/` overlays.** Same assembly pattern as Cursor: `source/vscode/agents/*.frontmatter.yml` + `source/shared/agents/*.md` -> `.github/agents/*.agent.md`. Instructions assembled from shared rule content + VSCode `applyTo:` frontmatter. Hooks reuse the same shell scripts with a new `hooks.json` registration. This is what we are picking.

**Option B -- Agent Plugins 1.0 format with `plugin.json`.** The spec originally targeted this, but the conversational phase determined that `.github/` layout is the correct native convention for VSCode/Copilot and avoids plugin packaging overhead.

## Decision

Add a third build target using `.github/` layout. New `source/vscode/` overlay directory follows the same frontmatter-overlay assembly pattern as `source/claude/` and `source/cursor/`.

### Key Design Decisions

**1. Tool name mapping (frontmatter only).** VSCode uses lowercase tool names. Agent frontmatter `tools:` arrays use VSCode-native names. The shared body content references tool behavior ("read the file," "write to"), not tool names by capitalisation, so body content is unchanged.

| Claude Code | VSCode | Notes |
|-------------|--------|-------|
| `Read` | `read` | |
| `Write`, `Edit`, `MultiEdit` | `edit` | VSCode has a single edit tool |
| `Bash` | `execute` | |
| `Glob`, `Grep` | `search` | VSCode combines find + grep |
| `Agent` | `agent` | |
| `NotebookEdit` | *(omit)* | Not available in VSCode |

**2. Eva's dual-routing mechanism.** Eva is an `.agent.md` with `handoffs:` for standard pipeline transitions (user-visible buttons) AND `agents: ["*"]` + `agent` tool for dynamic routing (conditional logic, brain injection, retry loops). `disable-model-invocation: true` prevents Copilot from auto-delegating to Eva. Always-apply instructions set Eva as the default persona.

**3. Instructions organisation.** 5 core rules -> 5 `.instructions.md` files. 3 always-apply (no `applyTo:`): `default-persona`, `agent-system`, `branch-lifecycle`. 2 glob-scoped: `pipeline-orchestration` and `pipeline-models` with `applyTo: "docs/pipeline/**"`. 16 reference docs remain as plain `.md` in `.github/references/`, loaded by agents via the `read` tool on demand (same pattern as Claude Code's `.claude/references/`).

**4. Hooks: PascalCase + self-filter.** `.github/hooks/hooks.json` uses PascalCase event keys. Matcher values are documented but ignored at runtime. The existing `enforce-paths.sh` already reads `tool_name` from stdin JSON and exits 0 for non-Write tools -- no script changes needed. Hook paths are project-relative (`.github/hooks/enforce-paths.sh`), not plugin-relative.

**5. Command mapping.** Slash commands map to user-invocable agents (`user-invocable: true`). `/pm` -> `@robert-spec`, `/ux` -> `@sable-ux`, `/architect` -> `@sarah`, `/docs` -> `@agatha`. `/pipeline` and `/devops` -> Eva (default persona via instructions). No separate command files needed -- agents ARE the invocation targets.

**6. Assembly.** Same pattern as Cursor:
```
source/vscode/agents/{name}.frontmatter.yml  +  source/shared/agents/{name}.md
  -> .github/agents/{name}.agent.md

source/vscode/rules/{name}.frontmatter.yml  +  source/shared/rules/{name}.md
  -> .github/instructions/{name}.instructions.md
```
Extension changes: `.md` -> `.agent.md` for agents, `.md` -> `.instructions.md` for rules.

## Rationale

The `.github/` layout is VSCode/Copilot's native convention. Reusing the frontmatter-overlay assembly pattern from the Cursor port means one mechanical decision (tool name mapping) covers all agents, and the setup skill's assembly logic extends cleanly to a third target. Eva's dual-routing mechanism (handoffs + agent tool) is the proven pattern from existing VSCode orchestrator implementations -- it maps to both the user-facing handoff buttons and the programmatic delegation that Eva needs for conditional routing.

## Falsifiability

This decision is wrong if: (1) Copilot's `handoffs:` mechanism cannot express the conditional routing Eva needs (mitigation: `agent` tool + `agents: ["*"]` handles all dynamic cases), (2) always-apply instructions overwhelm Copilot's context window at 14 agents (mitigation: only 3 instructions are always-apply; Phase 2 addresses context slimming), (3) hook self-filtering produces false denials in VSCode (mitigation: self-filter logic is already proven in Cursor where the same "matcher ignored" pattern exists).

---

## Implementation Steps

### Step 1: Source overlay scaffold + Eva dual-routing agent + pattern agents

Create `source/vscode/` directory tree. Define the VSCode frontmatter format. Create Eva's agent file (new shared body + VSCode frontmatter with handoffs). Create 5 representative agent frontmatter files establishing the tool-mapping pattern.

**Files (7 create):**
- `source/shared/agents/eva.md` -- Eva agent body (orchestration persona for VSCode target)
- `source/vscode/agents/eva.frontmatter.yml`
- `source/vscode/agents/sarah.frontmatter.yml`
- `source/vscode/agents/colby.frontmatter.yml`
- `source/vscode/agents/investigator.frontmatter.yml`
- `source/vscode/agents/ellis.frontmatter.yml`
- `source/vscode/agents/robert-spec.frontmatter.yml`

**Code shape -- Eva frontmatter:**
```yaml
name: eva
description: >
  Pipeline Orchestrator. Routes work between agents, manages phase
  transitions, enforces quality gates. Default persona for all chats.
target: vscode
user-invocable: true
disable-model-invocation: true
tools: [agent, read, execute, search]
agents: ["*"]
model: claude-opus-4-7
handoffs:
  - label: "Start feature spec"
    agent: robert-spec
    prompt: "Run feature discovery and produce a product spec."
  - label: "Design UX"
    agent: sable-ux
    prompt: "Design user flows and interaction patterns from the spec."
  - label: "Architecture"
    agent: sarah
    prompt: "Produce an ADR from the spec and UX doc."
  - label: "Build"
    agent: colby
    prompt: "Implement the ADR steps."
  - label: "Review"
    agent: investigator
    prompt: "Blind-review the diff. No spec, no ADR -- diff only."
  - label: "Commit"
    agent: ellis
    prompt: "Commit and push the verified changes."
```

**Code shape -- Colby frontmatter (write-access agent):**
```yaml
name: colby
description: >
  Senior Software Engineer. Implements code, exercises what she shipped.
target: vscode
user-invocable: false
disable-model-invocation: true
tools: [read, edit, execute, search]
model: claude-sonnet-4-6
```

**Code shape -- Poirot frontmatter (read-only agent):**
```yaml
name: investigator
description: >
  Default post-build verifier. Blind code investigator. Diff-only review.
target: vscode
user-invocable: false
disable-model-invocation: true
tools: [read, execute, search]
model: claude-sonnet-4-6
```

**Acceptance criteria:**
- [ ] `source/vscode/` directory tree exists with `agents/` subdirectory
- [ ] Eva frontmatter has `handoffs:` array with 6 entries routing through the standard pipeline
- [ ] Eva frontmatter has `agents: ["*"]` and `tools:` includes `agent`
- [ ] Eva frontmatter has `disable-model-invocation: true`
- [ ] All 6 agent frontmatter files use VSCode tool names (`read`, `edit`, `execute`, `search`, `agent`)
- [ ] No unsupported fields (`effort`, `color`, `maxTurns`, `permissionMode`, `mcpServers`, inline `hooks`)
- [ ] `source/shared/agents/eva.md` exists with orchestration persona content
- [ ] Assembly of Eva produces valid `.agent.md` with YAML frontmatter + body

**After this step, I can** assemble Eva's `.agent.md` and see correct handoff routing through the full pipeline, plus verify 5 representative agents have VSCode-compatible frontmatter.

---

### Step 2: Complete agent roster

Create remaining agent frontmatter overlays following the pattern from Step 1.

**Files (8 create):**
- `source/vscode/agents/robert.frontmatter.yml`
- `source/vscode/agents/sable.frontmatter.yml`
- `source/vscode/agents/sable-ux.frontmatter.yml`
- `source/vscode/agents/agatha.frontmatter.yml`
- `source/vscode/agents/distillator.frontmatter.yml`
- `source/vscode/agents/scout.frontmatter.yml`
- `source/vscode/agents/synthesis.frontmatter.yml`
- `source/vscode/agents/sherlock.frontmatter.yml`

**Note:** Sentinel frontmatter (`source/vscode/agents/sentinel.frontmatter.yml`) is opt-in -- created by pipeline-setup Step 6a, same as Claude Code and Cursor. Not included in this step's count.

**Acceptance criteria:**
- [ ] All 14 agents (15 with Eva) have frontmatter overlays in `source/vscode/agents/`
- [ ] Read-only agents (robert, sable, investigator, distillator, scout, synthesis, sherlock) have NO `edit` in `tools:`
- [ ] Producer agents with write access (robert-spec, sable-ux, agatha) have `edit` in `tools:`
- [ ] `user-invocable: true` on agents that map to slash commands (robert-spec, sable-ux, sarah, agatha)
- [ ] `user-invocable: false` on subagent-only agents (colby, investigator, distillator, scout, synthesis, sherlock)
- [ ] Assembly of each frontmatter + shared body produces valid `.agent.md`

**After this step, I can** list all 15 assembled `.agent.md` files in `.github/agents/` and see the complete agent roster with correct tool assignments.

---

### Step 3: Instructions conversion + variant overlays

Convert 5 core rules to `.instructions.md` format. Create variant overlays for branching strategies. Reference docs copy as plain `.md` to `.github/references/`.

**Files (6 create):**
- `source/vscode/rules/pipeline-orchestration.frontmatter.yml` (`applyTo: "docs/pipeline/**"`)
- `source/vscode/rules/pipeline-models.frontmatter.yml` (`applyTo: "docs/pipeline/**"`)
- `source/vscode/variants/branch-lifecycle-trunk-based.frontmatter.yml`
- `source/vscode/variants/branch-lifecycle-github-flow.frontmatter.yml`
- `source/vscode/variants/branch-lifecycle-gitlab-flow.frontmatter.yml`
- `source/vscode/variants/branch-lifecycle-gitflow.frontmatter.yml`

**Assembly rule for always-apply instructions:** `default-persona`, `agent-system`, and `branch-lifecycle` have no `applyTo:` -- the shared content is installed with `.instructions.md` extension and no VSCode-specific frontmatter overlay. An empty frontmatter block (`---\n---\n`) is prepended by the assembly logic. No overlay file needed.

**Code shape -- glob-scoped instruction frontmatter:**
```yaml
applyTo: "docs/pipeline/**"
```

**Code shape -- assembled always-apply instruction:**
```markdown
---
---

# Default Persona: Eva

In this repository, you ARE **Eva** ...
```

**Code shape -- assembled glob-scoped instruction:**
```markdown
---
applyTo: "docs/pipeline/**"
---

# Pipeline Orchestration -- Operational Procedures
...
```

**Acceptance criteria:**
- [ ] 3 always-apply instructions assemble with empty frontmatter (no `applyTo:`)
- [ ] 2 glob-scoped instructions assemble with `applyTo: "docs/pipeline/**"`
- [ ] 4 variant overlays exist for all branching strategies
- [ ] 16 reference docs copy to `.github/references/` as plain `.md` (no frontmatter conversion)
- [ ] No always-apply instruction references VSCode-unsupported features

**After this step, I can** verify that `.github/instructions/` has 5 `.instructions.md` files -- 3 always-apply and 2 glob-scoped -- and `.github/references/` has all 16 reference docs.

---

### Step 4: Hooks adaptation + pipeline-setup third target

Create VSCode hooks.json with PascalCase events and project-relative paths. Verify self-filtering. Update pipeline-setup to handle VSCode target detection and assembly.

**Files (2 create, 3 modify):**
- `source/vscode/hooks/hooks.json` (NEW)
- `source/vscode/hooks/enforcement-config.json` (NEW)
- `skills/pipeline-setup/SKILL.md` (modify -- add VSCode target detection and assembly)
- `skills/pipeline-setup/directory-layout.md` (modify -- add `.github/` layout)
- `skills/pipeline-setup/hooks.md` (modify -- add VSCode hook registration)

**Code shape -- VSCode hooks.json:**
```json
{
  "hooks": [
    {
      "event": "PreToolUse",
      "matcher": "edit",
      "command": ".github/hooks/enforce-paths.sh"
    },
    {
      "event": "SessionStart",
      "command": ".github/hooks/session-boot.sh"
    }
  ]
}
```

Note: `matcher` is documented but ignored at runtime -- `enforce-paths.sh` self-filters by reading `tool_name` from stdin JSON.

**VSCode target detection:** The setup skill detects the IDE by checking environment variables: `VSCODE_PID` or `TERM_PROGRAM=vscode` for VSCode. Existing detection: `CURSOR_PROJECT_DIR` for Cursor, `CLAUDE_PROJECT_DIR` for Claude Code. When VSCode is detected, assembly writes to `.github/` instead of `.claude/` or `.cursor-plugin/`.

**Assembly output map (VSCode target):**

| Assembly Input | Output | Extension |
|---------------|--------|-----------|
| `source/vscode/agents/*.frontmatter.yml` + `source/shared/agents/*.md` | `.github/agents/*.agent.md` | `.agent.md` |
| `source/vscode/rules/*.frontmatter.yml` + `source/shared/rules/*.md` | `.github/instructions/*.instructions.md` | `.instructions.md` |
| `source/vscode/hooks/hooks.json` | `.github/hooks/hooks.json` | `.json` |
| `source/claude/hooks/enforce-*.sh` (shared scripts) | `.github/hooks/enforce-*.sh` | `.sh` |
| `source/shared/references/*.md` | `.github/references/*.md` | `.md` |
| `skills/*/SKILL.md` | `.github/skills/*/SKILL.md` | `.md` |
| existing `mcp.json` | `.github/mcp.json` | `.json` |

**Acceptance criteria:**
- [ ] `source/vscode/hooks/hooks.json` uses PascalCase event keys
- [ ] Hook paths are project-relative (`.github/hooks/...`)
- [ ] `enforce-paths.sh` self-filter passes: `echo '{"tool_name":"read"}' | .github/hooks/enforce-paths.sh` exits 0
- [ ] `enforce-paths.sh` self-filter blocks: `echo '{"tool_name":"edit","tool_input":{"file_path":"docs/architecture/test.md"},"agent_type":"colby"}' | .github/hooks/enforce-paths.sh` exits 2 for colby_blocked_paths
- [ ] Pipeline-setup detects VSCode and installs to `.github/`
- [ ] Pipeline-setup assembles `.agent.md` and `.instructions.md` with correct extensions
- [ ] Existing Claude Code (`.claude/`) and Cursor (`.cursor-plugin/`) targets unchanged
- [ ] `.github/hooks/enforce-paths.sh` reads `VSCODE_*` or falls back to `$SCRIPT_DIR/../..` for project root

**After this step, I can** run `/pipeline-setup` in VSCode and get the complete `.github/` directory populated with agents, instructions, hooks, references, and skills.

---

## Notes for Colby

- **Eva body (`source/shared/agents/eva.md`):** Extract from `source/shared/rules/default-persona.md` sections `routing-behavior`, `loaded-context`, and `no-code-writing`. Do NOT duplicate the full rules content -- the instructions files carry that. Eva's agent body should reference the instructions and focus on agent-specific concerns (handoffs, tool access, what Eva does that no other agent does).
- **Hook script reuse:** Copy the same enforcement scripts from `source/claude/hooks/` to `.github/hooks/` at install time. The scripts already handle platform detection via env var fallbacks. The `SCRIPT_DIR/../..` pattern resolves correctly for both `.claude/hooks/` and `.github/hooks/` directory depths.
- **`user-invocable` mapping:** robert-spec (`/pm`), sable-ux (`/ux`), sarah (`/architect`), agatha (`/docs`), eva (always available). All other agents: `user-invocable: false`.
- **No `disallowedTools` in VSCode.** Claude Code uses `disallowedTools` to restrict agents. VSCode uses only `tools:` (allow-list). Omit tools that the agent should not use rather than listing them in a deny-list.
- **Sentinel and discovered agents** follow the same pattern but are installed conditionally by pipeline-setup. The frontmatter overlays exist in `source/vscode/agents/` but are only assembled when the user opts in.

## DoD: Verification

| # | Requirement | Status | Evidence |
|---|-------------|--------|----------|
| 1 | Third build target (additive) | Pending | `source/vscode/` exists; `.claude/` and `.cursor-plugin/` untouched |
| 2 | `.github/` layout | Pending | agents at `.github/agents/*.agent.md`, instructions at `.github/instructions/*.instructions.md` |
| 3 | Same `source/shared/` bodies | Pending | No duplication -- diff shows no shared content copied into `source/vscode/` |
| 4 | Eva dual-routing | Pending | Eva frontmatter has `handoffs:` + `agents: ["*"]` + `agent` tool |
| 5 | VSCode tool names | Pending | All frontmatter uses `read`/`edit`/`execute`/`search`/`agent` |
| 6 | No unsupported fields | Pending | Grep for `effort:`, `color:`, `maxTurns:`, `permissionMode:` in `source/vscode/` returns 0 |
| 7 | Always-apply instructions | Pending | 3 instructions without `applyTo:` |
| 8 | Glob-scoped instructions | Pending | 2 instructions with `applyTo: "docs/pipeline/**"` |
| 9 | Hook self-filter works | Pending | `enforce-paths.sh` exits 0 for non-write tools, exits 2 for violations |
| 10 | Pipeline-setup handles VSCode | Pending | `/pipeline-setup` installs to `.github/` when VSCode detected |
| 11 | No Claude Code changes | Pending | `git diff .claude/` clean (excluding pipeline state files) |
| 12 | No Cursor changes | Pending | `git diff .cursor-plugin/` clean |
| 13 | All 14 agents + Eva have overlays | Pending | 15 `.frontmatter.yml` files in `source/vscode/agents/` |
| 14 | References as plain `.md` | Pending | `.github/references/` contains plain files, no `.instructions.md` conversion |
| 15 | Command mapping via user-invocable | Pending | No separate command files; agents with `user-invocable: true` handle invocation |

**Grep check:** TODO/FIXME/HACK/XXX in `source/vscode/` -> 0
**Template:** All sections filled -- no TBD, no placeholders
