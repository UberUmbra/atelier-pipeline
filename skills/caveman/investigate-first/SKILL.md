---
name: investigate-first
description: Separate observed symptom from inferred cause before editing. No code changes until root cause is established with evidence. For ambiguous failures, intermittent issues, and performance problems.
---

# Investigate First

Separates observed symptom from inferred cause before any code changes are made.

## Separation Rule

State the observed symptom and the inferred cause as two distinct things. Do not conflate them.

## Investigation Process

1. Trace system behavior methodically from symptom to mechanism.
2. Evaluate hypotheses by: available evidence, testability, falsifiability.
3. Do not edit until you have a credible explanation for what you observe.

## Root Cause Documentation

Before proposing a fix, document root cause with supporting proof:
- Grep results, log lines, or test output that confirm the mechanism
- Why alternative hypotheses were ruled out

## Implementation Gate

Implementation is only appropriate when:
- (a) Root cause is established with evidence
- (b) Fix scope is confirmed against that root cause

## Suitable For

- Ambiguous failures
- Intermittent issues
- Performance problems
- Systematic investigation requiring evidence before action

## Not Suitable For

- Obvious typos
- Known bugs with confirmed location
- Mechanical changes with no diagnostic ambiguity
