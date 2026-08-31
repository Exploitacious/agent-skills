# Audit checklist

Read this for per-pattern audit depth, the triage when an audit fails, and the time it should take.

## Per-pattern audit specifics

### Parallel-research
Each report file exists and is non-empty. Reports cite files with line numbers, not vague references. Reports do not contradict each other on factual claims; if they do, find out which is right. Each report stayed in its assigned scope. Skip the test suite, since no code changed.

### Registry-driven generation
Every catalog item has a corresponding output file. Each output has all required sections. Cross-item references are explicit and consistent. No stub strings anywhere in the set. Run the full doc lint. Skip the code suite if the items are docs only.

### Surgical-pack
All named edits were made and their file and line ranges match the brief. No drift edits into unnamed files. The shared rationale is visible in the commit message and PR description. Full suite green, lints green, and each edit has a corresponding test.

### Heavy-build
Every deliverable maps to a real artifact, both a file and a behavior. The PR description is thorough, not generic. The decision record exists if the brief named one. The branch matches the brief. Full suite green, every named verification command ran and passed, and several outputs were spot-read for stubs.

### Reviewer-fix
You ran the relevant test before committing, the message is clear, and you filed a note if the fix was non-obvious.

## When the audit fails

Triage into one of three:

1. A regression in the agent's diff. Send it back with the specific failure and the check that caught it, or fix it inline if it is small.
2. A pre-existing failure surfaced by new code. Capture it as a follow-on. It is not the agent's problem; you triage its priority.
3. A weakness in the brief itself. You own it. Note it so the next brief is better.

Do not override a failed check with "good enough," do not skip a check because the round is small, and do not ship the round with a promise to fix it in a follow-up. Any of those defeats the audit.

## Audit time budget

| Pattern | Audit time |
|---|---|
| parallel-research | 10 to 20 min |
| registry-driven | 20 to 30 min |
| surgical-pack | 5 to 10 min |
| heavy-build | 10 to 20 min |
| reviewer-fix | n/a, you are the worker |

Consistently over budget means the briefs are too weak. Invest more in the style bar and the banned-shortcuts section up front. Audit time and brief time trade off: better briefs give faster audits.

## Capture the audit result

The audit itself produces an artifact; do not run it only in your head. Append a one-line verdict to your durable log or journal, note any brief weaknesses so future briefs improve, and commit the round's work after the audit passes. In a fleet, roll the result up to the coordinator with deliverable counts and PR references, and record an architectural decision if the round changed a lasting surface.
