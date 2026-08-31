# The eight-section brief template

Read this when writing a brief and you want the full template with examples. The sections are guideline-rigor: a missing section flags the audit, it does not abort the spawn. Tune section weight to the pattern.

## Section 1: working directory, commit, branch
So the subagent knows its starting state and what not to push. Include the repo or worktree path, the parent commit so it can verify state on entry, the target branch, and the do-not-push policy.

Give any lane that will commit its own git worktree. Every session and subagent in a repo share one checkout, one HEAD, and one index, so a lane that checks out its branch in the shared tree moves your HEAD with it. Three consequences all seen live: a foreman commit meant for the main branch landed on a worker's branch; a foreman push reported "everything up-to-date" at exit 0 because the local ref never moved, so the commit looked pushed for half an hour; a second worker branching from the polluted HEAD inherited foreign commits into its PR ancestry. Worktree isolation on the spawn handles tool-launched lanes; any lane doing its own git work needs the sentence in the brief. Repairs are ref-only or done in a temp worktree, never a checkout in the shared tree.

Give each lane its own scratch subdirectory, not just unique filenames. A session scratchpad is shared by the main thread and every subagent it spawns. Two lanes both wrote `scratchpad/pr-body.md` and one lane's in-place edits silently no-op'd against the other's content, nearly shipping the wrong PR body. Assign `scratchpad/<lane>/` per lane and require read-back before any scratch file is trusted as a source.

Skipped, the failure is: pushes to the wrong branch, premature merge, an unclean starting tree, a moved peer HEAD, or a clobbered scratch file.

## Section 2: files to read first
Lists paths in read order so the subagent does not rediscover structure. Three to ten paths, each with a one-line reason if non-obvious. Point at a distilled doctrine index plus the specific principle numbers this brief invokes, named by number, rather than a full-file doctrine read. A full doctrine file can run tens of KB; burying the six principles this brief needs inside it dilutes attention. Full reads are the escalation for genuinely doctrine-heavy lanes, not the default. Skipped, the subagent burns half an hour surveying the codebase before starting.

## Section 3: numbered deliverables
Verifiable artifacts, not vague goals. Numbered so report-back maps one-to-one. Each item names the files it produces and the behavior that changes. No "improve X"; write "X must do Y, tested by Z." Skipped, the subagent ships something that feels right but does not match the ask.

## Section 4: required tests and named principles
Cite the governing principles by number and name so they activate in context. State the test count expected per deliverable. Quote the specific clause when one applies. Skipped, the subagent picks generic practices that conflict with project rules.

## Section 5: verification commands
The exact commands the subagent runs before reporting, which are the same commands you run in audit: the test command, the lints, any doc-invariant or drift check. State that all must exit 0 and the output summaries go in the report. Skipped, the subagent reports done without the full suite and regressions ship.

## Section 6: style and accuracy bar
Specific requirements with the principle quoted: logging level, exception handling, whether the decision record lands in the same commit. Skipped, the subagent ships minimum-bar code that works but violates project rules.

## Section 7: banned shortcuts
Stop the cheapest shortcut at its source, each with the reason so judgment extrapolates. Typical: stub strings such as "TBD", "placeholder", "see source", or empty bodies (defer and document the blocker instead); bypassing commit or signing hooks; a silent swallow of an exception; and the word "just" in comments or PR text. Skipped, the subagent ships placeholders or defers work without surfacing the blocker.

## Section 8: report-back format
A literal template the subagent fills in, as headings and bullets. Name every field: working directory, branch, commit SHAs, per-deliverable status with files, verification output per command, blockers and deferrals with recommended follow-ons, and findings worth capturing that were not in the brief. Skipped, the subagent returns freeform prose you cannot audit or roll up.

## Worked example, heavy-build

```
BRIEF: signal-history retention env-var override

Working dir: /path/to/finance-report
Parent commit: 814db71 (main)
Target branch: feat/signal-history-retention
Do your branch work in your OWN git worktree; other sessions are live
  in the shared checkout. Scratch dir: scratchpad/signal-retention/,
  yours alone. Do NOT push. Do NOT merge.

Files to read first:
1. The distilled doctrine index (mandatory). This brief invokes the
   document-the-why, trust-and-audit, and best-effort-floor principles,
   plus the retention decision record.
2. app/signals/retention.py
3. tests/test_retention.py
4. docs/decisions/2026-05-18__retention-keep-forever.md

Deliverables:
1. retention.py: env var RETENTION_DAYS_OVERRIDE, default 0 (keep forever)
2. test_retention.py: 4 tests, default, override > 0, override 0 ignored,
   prune logging
3. docs/decisions/2026-05-21__retention-env-override.md: decision record
4. drift-check.sh: refuse ungated DELETE against signal_history

Required tests: the 4 above.
Governing principles:
- Document-the-why: the decision record lands in the same commit.
- Trust-and-audit: additive over destructive, env-var-gated, default preserve.
- Best-effort-floor: no swallowed exceptions, no silent degradation.
- Retention decision: pruning requires the env-var gate.

Verification commands:
  pytest tests/test_retention.py -v
  pytest
  ./scripts/drift-check.sh
  ./scripts/verify-docs.sh
All four must exit 0.

Style and accuracy bar:
- Failures loud: log at ERROR with exc_info=True.
- Decision record in the same commit as the code.
- Report back in the section-8 format below.

Banned:
- TBD, placeholder, see source, empty bodies.
- Bypassing commit hooks.
- Silent except: pass.
- Pruning DELETE without the env-var gate.
- "just" in comments or PR text.

Stakes: production trading platform. 25-plus traders consult
signal_history during market hours; a quiet prune would destroy the
multi-year base-rate history the matcher depends on. This is a
load-bearing safety mechanism.

Escalation grant: if a schema or environment gap would force a
degraded version, defer the item and document the blocker as a
follow-on. Do not ship the degradation.

Report back:
## Subagent report: signal-history retention override
Working dir / Branch / Commit SHAs
### Deliverables status (per item: done | partial | deferred, with files)
### Verification output (per command: PASS | FAIL with summary)
### Blockers and deferrals
### Findings to capture
```

## Brief sizing by pattern

| Pattern | Brief length | Time to write |
|---|---|---|
| parallel-research | 300 to 600 words | 5 to 10 min |
| registry-driven | 400 to 800 word template plus per-item params | 10 to 15 min |
| surgical-pack | 400 to 700 words | 10 to 15 min |
| heavy-build | 800 to 1500 words | 20 to 30 min |
| reviewer-fix | none, you do it | none |

If brief time passes half the expected subagent time, you are in reviewer-fix territory.

## Common brief failures
No principle citations, so output is vague. No stakes framing, so effort is low. No escalation grant, so a pressured subagent ships silent degradation. No report-back format, so you cannot audit. The word "just" anywhere. Open-ended deliverables: "improve retention" is not one, "add RETENTION_DAYS_OVERRIDE handling" is.
