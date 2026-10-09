---
name: caveman-compress
description: Compress a markdown file by stripping ceremony while preserving all facts, decisions, code, URLs, and structure. Creates .original.md backup. Invoke: /caveman:compress <filepath>.
---

# Caveman Compress

Lossless compression of markdown files. Strips ceremony. Preserves facts.

## Invocation

```
/caveman:compress <filepath>
```

`filepath` is the absolute path to the markdown file to compress.

## Strip

- Filler phrases: "it is worth noting", "as mentioned above", "it should be said that", "needless to say"
- Redundant transitions: "moving on to", "with that in mind", "to summarize the above"
- Passive voice where active is shorter: "was updated by" → "updated"
- Repeated points: state each fact once, at its first occurrence
- Throat-clearing intros: "In this document, we will discuss…"
- Hedged conclusions restating the section

## Preserve — Never Touch

- Code blocks: verbatim, including whitespace
- URLs: verbatim
- File paths: verbatim
- Heading structure: same headers, same nesting
- Technical terms: exact spelling, casing, punctuation
- All decisions, constraints, and facts — none omitted
- Numbered lists where order matters

## Process

1. Read the file.
2. Check word count. If < 200 words: report "too short — no gain" and stop.
3. Compress in memory.
4. Write backup: `{filename}.original.md` in the same directory.
5. Write compressed version to original path (overwrite).
6. Report: original word count → compressed word count → reduction %.

## Target

- Files ≥ 500 words: target ≥25% word reduction.
- Files 200–499 words: compress opportunistically, no minimum target.

## Example Report

```
Compressed docs/pipeline/pipeline-state.md
Before: 847 words. After: 601 words. Reduction: 29%.
Backup: docs/pipeline/pipeline-state.original.md
```
