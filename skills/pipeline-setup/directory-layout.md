<section id="directory-map">

## What Each Directory Does

### Claude Code (`.claude/`)

| Directory | Loaded By | Purpose |
|-----------|-----------|---------|
| `.claude/rules/` | Claude Code automatically (every conversation) | Eva persona and orchestration rules -- always active |
| `.claude/agents/` | Claude Code when subagents are invoked | Agent personas for execution tasks |
| `.claude/commands/` | Claude Code when user types a slash command | Manual agent invocation overrides |
| `.claude/references/` | Agents when they need shared knowledge | Quality framework, lessons, templates |
| `.claude/hooks/` | Claude Code on every tool call (PreToolUse) | Mechanical enforcement of agent boundaries, sequencing, and git operations |
| `docs/pipeline/` | Eva at session start | State recovery, context preservation, error tracking |

### Cursor (`.cursor-plugin/`)

| Directory | Loaded By | Purpose |
|-----------|-----------|---------|
| `.cursor-plugin/agents/` | Cursor when subagents are invoked | Agent persona files (`.md` with YAML frontmatter) |
| `.cursor-plugin/rules/` | Cursor automatically or via `alwaysApply:` | Rules files (`.mdc` with YAML frontmatter) |
| `.cursor-plugin/hooks/` | Cursor hook runner | `hooks.json` + enforcement scripts |
| `docs/pipeline/` | Eva at session start | State recovery, context preservation, error tracking (shared with Claude Code) |

### VSCode/Copilot (`.github/`)

| Directory | Loaded By | Purpose |
|-----------|-----------|---------|
| `.github/agents/` | Copilot when agents are invoked | Agent persona files (`*.agent.md` with YAML frontmatter) |
| `.github/instructions/` | Copilot automatically (no `applyTo:`) or by glob pattern | Rules as `*.instructions.md`; always-apply have no `applyTo:`, glob-scoped have `applyTo: "pattern"` |
| `.github/hooks/` | Copilot hook runner on each event | `hooks.json` + enforcement scripts; PascalCase event keys, project-relative paths |
| `.github/references/` | Agents via `read` tool on demand | Plain `.md` reference docs -- quality framework, templates, lessons |
| `.github/skills/` | Agents via `read` tool on demand | Skill definition directories (`SKILL.md`) |
| `docs/pipeline/` | Eva at session start | State recovery, context preservation, error tracking (shared across all targets) |

**VSCode file layout detail:**

```
.github/
  agents/          # *.agent.md -- assembled from source/vscode/agents/ + source/shared/agents/
  instructions/    # *.instructions.md -- assembled from source/vscode/rules/ + source/shared/rules/
  hooks/
    hooks.json           # PascalCase events, project-relative paths (.github/hooks/...)
    enforce-paths.sh     # Path enforcement (self-filters tool_name from stdin JSON)
    hook-lib.sh          # Shared hook utility library
    enforcement-config.json
    session-boot.sh
    pipeline-state-path.sh
  references/      # *.md -- copied from source/shared/references/ (no conversion)
  skills/          # SKILL.md directories -- copied from skills/
  pipeline-config.json   # Branching strategy, feature flags
  .atelier-version       # Plugin version marker
  mcp.json         # MCP server config (when brain enabled)
```

</section>
