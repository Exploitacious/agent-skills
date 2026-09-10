---
name: briefing-subagents
description: Use when writing the brief for a subagent or workflow lane. Covers stakes-mode framing (real user, named principles, escalation grant, verbatim API shapes) and the eight-section brief shape that makes returned work auditable.
---

# Briefing subagents

Your job here is the brief, not the code. A subagent delivers work into a real system, not output for grading. What you write into the brief is the only context it has, so a vague brief gets vague work back. Write the brief, then delegate; do not drift into doing the work yourself.

## Stakes-mode framing

The framing decides the quality more than the section list does. Every brief names:

- The real user and the real consequence. "The on-call engineer reading this at a 3 AM page" beats "the team." "25 traders size positions off this during market hours; a stale value misleads them" beats "this is important."
- The principles that govern the work, by name. A named rule activates in the subagent's context where "follow best practices" does not. If your project has a written engineering doctrine, cite its rules by their stable number and name; otherwise name the exact behavior you require, such as no swallowed exceptions or tests in the same commit. A `doctrine.md` slot, if present, holds the registry to cite from.
- An escalation grant. "If you hit a schema gap that would force a degraded version, defer the item and document the blocker as a follow-on. Do not ship the degradation." Without this, a subagent under brief pressure ships the quiet degradation.
- Done criteria an auditor can check: a test count, a green suite, no stub strings. Not self-report.
- Precise verbs, never "just." "Just implement X" calibrates effort downward.
- Vendor and API shapes pasted verbatim in a code fence, never paraphrased. "Returns serial, model, friendlyName" is true at the meaning level and false at the key-name level; the subagent codes against your words and breaks on the first live call. Key names like `serial` versus `serialOrLicense` silently skip every record. Tell the agent the keys are exact and to rename only at the mapping layer, not the read boundary.

## The eight sections

Guideline shape, not a gate. A missing section flags the audit, it does not abort the spawn. Tune section weight to the pattern: a research brief is short, a heavy-build brief is long. Full template, examples, and common failures are in `brief-template.md`.

1. Working directory, parent commit, target branch, and the do-not-push policy. Give any committing lane its own worktree and its own scratch subdirectory, since sessions and subagents share one checkout and one scratchpad.
2. Files to read first, in order, with a one-line reason each. Point at a distilled index plus the specific principles this brief invokes, not a full doctrine read.
3. Numbered deliverables, each a verifiable artifact. Replace "improve X" with "X must do Y, tested by Z."
4. Required tests and the governing principles by name.
5. Verification commands, the exact commands the subagent runs before reporting, which are the same commands you run in audit.
6. Style and accuracy bar, with the principles quoted where a clause applies.
7. Banned shortcuts, stated with the reason so judgment extrapolates: no stub strings, no bypassing commit hooks, no silent swallow, no "just."
8. Report-back format, every field named, as headings and bullets rather than prose, so the result maps one-to-one to your audit.

Keep brief-authoring under a fifth of the expected subagent time. Past that you are in reviewer-fix territory, so do it yourself.

## Briefing a lane that stops or restarts a live service

A subagent can end between any two tool calls: it hits its turn cap, or a rate limit, with no chance to clean up. Two failure shapes follow from that, and both are the brief's job to prevent.

- A stop, mutate, restart sequence split across calls is an outage waiting for the cap. If the lane ends after the stop, the service stays down until someone notices. Brief the lane to do the stop, the change, and the restart in one foreground payload, behind a trap that restores service on every exit, not only on error. Any single call that mutates live infrastructure should leave the system safe if the lane ends right after it.
- A long step in the background is a lane parked forever. The background-Bash completion notice fires only for the main session, so a subagent that backgrounds a long command and ends its turn waiting on it is never re-invoked and idles with the service down. Brief every long step to run in the foreground with an adequate timeout, and to not end the turn between a stop and its restart. If a job must run in the background, keep it with the parent rather than a lane.

## On Codex and OpenCode

The eight-section brief shape and the stakes-mode framing are provider-agnostic
prose; only the delivery differs. On Codex, the brief is the `message` argument
to `spawn_agent` (which takes only `task_name` and `message`), so everything the
lane needs must be in that one string; keep a per-type brief scaffold on disk and paste it in. On OpenCode, the
brief is the prompt handed to a typed subagent (an agent definition file). The verbatim-API-shape rule, the named-principle rule, and
the escalation grant apply identically on all three; the doctrine you cite is
whatever shared instructions every provider reads through `AGENTS.md`.
