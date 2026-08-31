---
name: delegation-sizing
description: Use when deciding whether a task is worth handing to subagents, which delegation pattern fits, or how much wall-clock a delegated round saves. Covers the delegate-or-not gate, the five patterns and their decision tree, agent-type choice, and the conversion-factor estimate.
---

# Delegation sizing

You are a foreman, not an engineer. The main thread scopes, briefs, and audits; subagents do the heavy reading and writing in contexts you can throw away. This skill decides one thing: is this task worth delegating, and if so, in what shape.

## Delegate when all three hold

1. The work is more than about a day of solo effort.
2. It parallelizes across independent files or scopes.
3. You can scope each piece well enough to write a brief.

If any one fails, stay solo or use a single surgical subagent. Delegation adds brief-authoring and audit overhead. Below roughly six solo hours, that overhead beats the parallelism. Single-threaded synthesis, one architectural design, or a debug of unknown cause does not speed up by fanning out. One fully briefed subagent equals one mind, no faster than yours.

## The five patterns

Pick one before you spawn. Depth, brief size, and wall-clock live in `patterns-and-sizing.md`.

1. Parallel-research. Two or three read-only subagents write notes to a shared directory; you synthesize. For surveys and reconnaissance.
2. Registry-driven generation. One subagent per item in a known catalog. For bulk independent content.
3. Surgical-pack. Two or three small related edits sharing a rationale, one PR, reviewed inline.
4. Heavy-build. One subagent on one substantial isolated PR, worktree-isolated.
5. Reviewer-fix. You fix it inline. Use when the change is under five lines and a brief would cost more than the fix.

Decision tree: under a day and under five lines, reviewer-fix. Mostly read-only across sources, parallel-research. A catalog of independent items, registry-driven. Two to four related edits, surgical-pack. Otherwise heavy-build.

## Pick the agent type

Default to the general, full-tool agent type. Narrow or excerpt-reading types miss content past their read window and report confident wrong numbers on counts and audits, so reserve them for one-shot symbol lookups. An agent type's tools come from its definition, not your brief: a read-only type cannot run tools its definition withholds, no matter what you write. If a `tools.md` slot lists your host's agent types and their tool access, check it before briefing tool-dependent work; otherwise read the type's declared tools yourself.

## Estimate in deliverables, not hours

Under the three conditions, a foreman turns a solo estimate into roughly a fifth to a tenth of the wall-clock. The ceiling is your brief-and-audit throughput, not the subagent count: one foreman runs two or three subagents per parallel round, and rounds chain. Report the payoff as a deliverable count with a solo-versus-foreman line, since hours imply the sequential work delegation removes. Worked examples and the viability rules are in `patterns-and-sizing.md`.

Keep brief-authoring under a fifth of the expected subagent time. Past that, the scope is too small for the pattern: do it yourself, or combine items into one heavier lane.
