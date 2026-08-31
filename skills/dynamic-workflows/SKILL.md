---
name: dynamic-workflows
description: Use when authoring a Claude Code dynamic workflow, or deciding whether a task warrants a workflow over manual subagent delegation. Covers the escalation gate, the script primitives, the pipeline-versus-parallel rule, and the discipline that carries over from manual delegation.
---

# Dynamic workflows

A dynamic workflow is a JavaScript script the Claude Code runtime executes in the background while your session stays responsive. You or the model write the script; the runtime fans out subagents and the plan lives in code, in loops, branches, and script variables. Only the final return value lands back in your context. With the Agent tool you hold the plan turn by turn and every result hits your context; with a workflow the script holds the plan and only the answer returns. That is what lets a workflow run at a scale one conversation cannot coordinate, and makes the orchestration repeatable.

## When to reach for one

Escalate from manual delegation to a workflow when any of these hold: the work needs more than about ten agents; the orchestration is worth codifying and rerunning, such as a review you run on every branch, so save it as a command; you want refutation verification baked in, where independent agents refute each other's findings before anything is reported; or the sweep is too large for one context to hold.

Stay at manual delegation for fewer than ten agents, when you need to think between rounds, or when the shape is not known yet. Use a persistent fleet instead when the work spans sessions and needs human-async peers. A workflow costs meaningfully more than a normal session, so the gate is the value of the answer, not the novelty of the tool. When unsure, the cheaper tier wins.

## The primitives

A script opens with a pure-literal `meta` block (name, description, phase titles), then an async body. Standard JavaScript built-ins work except the time and randomness calls that break resume; pass time in through args instead.

- `agent(prompt, opts?)` spawns one subagent and returns its text, or a validated object when you pass a `schema`. It returns null if the user skips it, so filter before use. Key options: `label`, `phase`, `schema`, `model` (omit to inherit the session model, almost always right), `agentType`, and the worktree isolation flag.
- `pipeline(items, stage1, stage2, ...)` flows each item through all stages independently with no barrier between stages, so item A can be in stage 3 while item B is still in stage 1. Wall-clock is the slowest single chain. This is the default for multi-stage work.
- `parallel(thunks)` runs all concurrently and awaits all, a barrier. A failing thunk resolves to null, so filter before use. Use it only when a stage genuinely needs the whole previous set: a dedup or merge across everything, an early exit on the total, or a cross-item comparison.
- `phase(title)` starts a progress group; inside pipeline or parallel stages, pass the phase option instead to avoid racing the global state.
- `log(msg)` narrates to the user; use it to surface what was dropped or capped, since silent truncation reads as full coverage.
- `budget` exposes the token budget; scale depth to it and guard loops with it.
- `workflow(name, args?)` runs another saved workflow inline, one level deep.

## Pipeline versus parallel

Default to `pipeline`. Three or more agents touching shared scaffolding always conflict at a barrier merge, which is the same lesson as serializing shared-scaffolding merges. A barrier is justified only by a real cross-item dependency, not by wanting to map or filter first (do that inside a stage) and not by "it is cleaner." Barrier latency is real: if the slowest finder takes three times the fastest, a barrier wastes two-thirds of the fast finders' time.

The canonical shape is find then verify, pipelined so each finding verifies the moment its review completes. Compose freely from there: refutation verify (independent skeptics prompted to refute, killed on a majority), a judge panel over independent attempts, loop-until-dry for unknown-size discovery (dedup against a seen set, not the confirmed set), a multi-modal sweep where each agent searches a different way, and a completeness critic whose output is the next round.

## The discipline carries over

A workflow is the same foreman discipline at scale, and the rules do not relax. Every `agent()` prompt is a stakes-mode brief: real user, named principles, verifiable done, banned shortcuts, escalation grant. Never compress an `agent()` prompt into shorthand; a compressed brief is a degraded brief. Returned findings are still claims, and the script's verify stage is the first defense, not the last; ground-truth the headline numbers before reporting. Default to the general, full-tool agent type. Use worktree isolation only when agents mutate files in parallel, and remember that a worktree cannot see gitignored or uncommitted context, so paste it inline. Workflows cannot ask the user mid-run, so for sign-off between stages, run each stage as its own workflow.

Authoring detail, the args gotcha, resume, and worked patterns are in `authoring-workflows.md`.
