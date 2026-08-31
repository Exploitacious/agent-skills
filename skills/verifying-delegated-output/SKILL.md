---
name: verifying-delegated-output
description: Use when a subagent has returned work and you are about to trust, land, or report it. Covers the two claim classes to verify before trust, the git-state checks before any merge, and the content audit that catches stubs and fabricated numbers.
---

# Verifying delegated output

A returned summary is a set of claims, not facts. Default to trust, but the safety net is the audit below, run after the subagent reports and before you declare the round complete or spawn the next one. Two claim classes fail in different ways: git and file-state claims, and content claims.

## Capture before you chain

Subagent output is not durable until you capture it. The result lives as a tool-result in your context, which context loss destroys. Before the next spawn: record what shipped in a durable log or decision record, commit any files the subagent created, and if you are in a fleet, roll up to the coordinator with deliverable counts and PR references, since transient messages do not survive either. Do not chain a second subagent before capturing the first.

## Git and file-state claims are the least trustworthy

Verify each in a separate call before the merge chain, never chained inside it. A chained check decorates, it does not gate, because the branch name prints after everything already ran.

1. Confirm the branch before the merge. A concurrent agent can move a shared checkout's HEAD out from under you. Start merge chains with an explicit change into the primary checkout; your own shell may be sitting inside an agent's worktree, where the whole dance no-ops against the wrong tree.
2. Verify the remote has the work before merging a PR. Local logs and diffs show the agent's commits even when they were never pushed, and the PR merges what is on the remote. Confirm the branch tip equals its remote tip and that the expected file is in the remote tree. Better handoff: brief the agent to commit but not push, then you push and merge.
3. Demand a real ref update on push. Output must show the old-to-new range for the branch. "Everything up-to-date" after a merge means the merge landed elsewhere.
4. "Already committed" can mean gitignored-invisible. A file that is both untracked and gitignored shows in neither status nor the file list, so the agent infers it was handled and the feature ships green without its data file. Confirm deliverables are actually staged, and for any new data file check whether it is ignored and force-add it with a breadcrumb if so.
5. Let gates show their real exit code. Never pipe a gate through a tail inside a chain that ends in a push; the pipe reports the tail's exit 0 and a red build ships. Run the gate to a log, read it, then commit in a separate call.

## Content claims: run the audit gates

- Full suite green, not just the tests the subagent added; new work regresses old. Triage a red result into the agent's diff versus a pre-existing failure. A pre-existing failure is a follow-on, not the agent's fault.
- Project lints and doc invariants green.
- Sample-load each claimed test module. A module that imports but collects zero tests, or all-skipped, is a stub that passes by being skipped.
- Spot-read a few outputs for stub strings and copied-in text: "TBD", "placeholder", "see source", empty bodies, or verbatim doctrine pasted into comments.
- If the agent added a lint, feed it a known-bad input and confirm it catches it. A lint that misses its violation is a no-op.
- Ground-truth every specific claim. Check line and file counts, cited line numbers, symbol existence, and any "no findings" or "all clean" on a deep audit, which is improbable at scale. Shortcut signs that make this mandatory: an opener like "Perfect" or a mid-thought continuation, vague line references, round-number totals with no source, and file sizes confused for line counts. When verification surfaces a fabrication, mark the round incomplete, redo the small surface inline or respawn with a brief that bans the observed shortcut, and never paste an unverified summary onward as findings.

Depth per pattern and the triage for a failed audit are in `audit-checklist.md`.

## Do not act on a line-specific finding without reading the source

A reviewer reading a PR diff sees diff line numbers, not file line numbers, and they do not map one-to-one. A line-specific critical finding looks precise but is the highest-risk hallucination class: a reported "variable undefined at line 702" was defined at the top of that function, and applying the fix would have broken correct code. Before acting on any line-specific critical finding, open the actual file at that line and confirm. Pattern-level findings such as "this branch lacks error handling" stand on their merits and need no line check. If you cannot reproduce a finding in the source, it is hallucinated: ignore it and merge.
