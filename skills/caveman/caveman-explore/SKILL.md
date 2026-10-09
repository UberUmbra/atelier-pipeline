---
name: caveman-explore
description: FastContext: parallel citation-only code search. Issue multiple searches in parallel, return path:START-END citations only. No prose, no synthesis. For fast codebase orientation.
---

# Caveman Explore (FastContext)

Parallel citation-only code search. Locates relevant files fast. No synthesis.

## Behavior

1. **First turn: all searches in parallel.** Issue all glob patterns, grep queries, and direct reads simultaneously. Do not sequence.
2. **Return citations only.** One per line, format below.
3. **No prose.** No preamble. No explanation. No synthesis.
4. **Follow evidence 1-2 turns.** If a citation points to another file, read it in the next turn. Max 2 follow-up turns.
5. **Halt when located.** Once relevant locations identified, stop. Do not continue searching.

## Citation Format

```
path/to/file.ext:START-END  reason it is relevant
```

- `START-END`: actual line range examined (e.g. `L12-L47`)
- `reason`: one phrase, no sentence, no period
- Never fabricate line ranges — only cite ranges actually read

## Multiple Citations

```
source/shared/pipeline/pipeline-config.json:L1-L44  config schema with brevity/caveman_mode fields
source/claude/hooks/enforce-agent-paths.sh:L1-L120  agent path enforcement hook
.claude/rules/agent-system.md:L1-L60  Eva routing rules and agent table
```

## Halt Conditions

- Return "no relevant locations found" if genuinely unable after 2 follow-up turns.
- Do not keep searching to appear thorough.

## Not For

- Code review
- Design proposals
- Cross-file consistency analysis
- Explanations of how code works

Use the investigator (Poirot) for those tasks.
