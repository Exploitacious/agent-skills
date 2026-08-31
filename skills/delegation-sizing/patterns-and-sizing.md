# Patterns and sizing

Read this when scoping a specific delegation round: which pattern, how long, whether it pays.

## The five patterns in depth

### Parallel-research
Two or three read-only subagents, each given the same question and a different source, writing one markdown file each to a shared directory. You synthesize after all return. Tools restricted to read, grep, glob. About 30 to 60 minutes total. Audit is light: read each report, spot-check a few cited files. Common failures: subagents duplicate effort because you did not partition cleanly; reports land in chat instead of files; you skip synthesis and treat raw reports as decisions.

### Registry-driven generation
One subagent per item in a catalog that defines the items in advance (one article per topic, one doc per command, one test file per module). Write the brief once as a template, inject per-item parameters, and have every subagent read the same governing rules and the same canonical reference. Roughly 15 to 30 minutes per item in parallel. Audit is medium: confirm every catalog item produced a file, spot-read individual items for stubs, run the doc lint over the set. Watch for template drift across items and silent cross-references between items that were meant to be independent.

### Surgical-pack
Two to four small related edits (five to thirty lines each) that share one rationale, bundled into a single branch and PR. The style section of the brief carries the shared rationale; deliverables list each edit with its file and line range. About 20 to 40 minutes. Audit is light and inline. The failure that hurts is missing one call site: grep the symbol to confirm coverage, and run the full suite, not just touched files.

### Heavy-build
One subagent on one substantial isolated PR: a feature, a self-contained refactor, a significant new surface. Worktree isolation is mandatory. The brief is the longest variant, with comprehensive governing rules, a full style bar, and a detailed report-back format. One to three hours of subagent work plus a heavy audit: run the full suite and every named verification command, read the whole diff, spot-read several outputs for stubs, confirm each deliverable maps to a real artifact. This pattern ships the most doctrine violations when the brief is thin, so do not shorten it.

### Reviewer-fix
You do it. Read the context, edit, run the relevant test, commit, note anything non-obvious. Two to fifteen minutes. Use when the change is under five lines and a brief would cost more than the fix. If you stack more than three of these in one turn, ask whether it should have been a surgical-pack. Always run the test even when the fix is obvious.

## Composing patterns
Real rounds mix them: research feeds a heavy-build; research identifies a catalog for registry-driven; a heavy-build feature gets its call sites wired by a surgical-pack. Keep one pattern per spawn. If a brief implies two, split into two spawns.

## The conversion factor
Under the three conditions (more than a day solo, parallelizes, briefable), a solo estimate divides by five to ten for the foreman estimate. Why not more: the bottleneck is brief-authoring plus audit, not subagent count. A foreman runs two or three subagents per parallel round, and rounds chain serially. Chained rounds and per-item parameter injection pull the realistic factor toward four or five for large registry work and two to three when rounds depend on each other. Ten is the ceiling for highly parallel work with a shared template.

When it does not pay: sub-day work (overhead alone is thirty-plus minutes per subagent), tightly sequential steps (no fan-out to gain), and single-threaded synthesis (one mind, and the subagent is not faster than you).

## Report viability, honestly
For each scoped item, state whether delegation is viable (yes if all three conditions hold, no if any fails materially, partial if some sub-tasks delegate and others do not), name the pattern, and give the foreman estimate. Partial is the common real case: name which sub-tasks delegate. Inflated delegation math costs trust when the conditions did not actually hold, so keep the accounting honest.

## Brief-time budget
Brief-authoring should stay under a fifth of the expected subagent time.

| Pattern | Subagent work | Max brief time |
|---|---|---|
| reviewer-fix | n/a | n/a, you do it |
| surgical-pack | 15 to 30 min | 5 to 10 min |
| parallel-research | 20 to 40 min | 5 to 10 min |
| registry-driven | 20 to 40 min per item | 10 to 20 min template plus 2 min per item |
| heavy-build | 60 to 180 min | 20 to 30 min |

Over budget on a brief means the scope is too small for the pattern. Switch to reviewer-fix, combine items into one heavier lane, or run a short research round first to clarify scope, then re-brief.

## Right-size the brief: context is headroom, not a target
Modern harnesses give workers a very large context window. A bigger window is not better work. Scope every brief as tightly as the task allows; most lanes fit well under a couple hundred thousand tokens and should stay there.

- Reserve the fixed overhead. Every spawn reloads the full system prompt and all active tool schemas before your brief lands.
- Count the reading the brief demands. Instructions, every file the worker opens, and the output it produces all share one window. A vague "review these 300 files" brief fails even at maximum context: the worker skims, runs past what it can hold, and fabricates specifics for the rest. That is silent data loss, not a slow worker.
- The lever is decomposition: more, smaller, sharply-scoped subagents. Split before you spawn. Two workers over 150 files each, a registry loop one item per agent, a research round to map a surface before a heavy build touches it.
- Reserve the largest-context lanes for work that genuinely needs it: a large codebase slice, a long document. Do it deliberately, not to rescue a lazy brief.

Pick the model tier by task complexity: a capable build-and-review model for heavy lanes, a lighter model for routine or mechanical ones, rotating between them as the work warrants. If a slot names your host's model tiers, follow it; otherwise use the strongest model you have for build and review and a cheaper one for mechanical lanes. Right-sizing the chunk is the foreman's job, not something the worker discovers mid-task.
