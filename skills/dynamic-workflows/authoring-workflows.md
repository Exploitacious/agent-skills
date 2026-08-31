# Authoring workflows

Read this when writing a Claude Code dynamic workflow script and you want the invocation paths, the args gotcha, resume, and the quality-pattern code shapes.

## How to invoke
- One-off: include the word workflow in your prompt, and the model writes a script for the task instead of working turn by turn.
- Session-wide: set the ultra effort level, which plans a workflow for every substantive task. It is session-only and resets next launch, a deliberate cost guard.
- Saved: run an existing command. Save any run's script from the workflows list and it becomes a command, project-scoped or personal.

## The canonical find-then-verify shape

```javascript
const results = await pipeline(
  DIMENSIONS,
  d => agent(d.prompt, { label: `review:${d.key}`, phase: 'Review', schema: FINDINGS }),
  review => parallel(review.findings.map(f => () =>
    agent(`Independently verify, default to refuted if uncertain: ${f.title}`,
          { label: `verify:${f.file}`, phase: 'Verify', schema: VERDICT })
      .then(v => ({ ...f, verdict: v })))),
)
const confirmed = results.flat().filter(Boolean).filter(f => f.verdict?.isReal)
```

- Refutation verify: several independent skeptics per finding, each prompted to refute, killed when a majority refute. This stops plausible-but-wrong findings. Give each verifier a distinct lens (correctness, security, does-it-reproduce) when a finding can fail multiple ways.
- Judge panel: generate several independent attempts from different angles, score them with parallel judges, synthesize from the winner.
- Loop-until-dry: for unknown-size discovery, keep spawning finders until a few consecutive rounds return nothing new. Dedup against a seen set, not the confirmed set, or rejected findings reappear forever.
- Multi-modal sweep: agents each search a different way, by container, by content, by entity, by time; one angle will not find everything.
- Completeness critic: a final agent asking what is missing, whose output is the next round of work.

## The args gotcha

Args can arrive undefined or stringified at the script even when the call passed a proper JSON object, and `args.lanes.map(...)` then throws before a single agent runs. Rules:

- For a one-shot script, hardcode the inputs as a const literal in the body. Reserve args for genuinely parameterized saved workflows.
- When you do use args, guard every access with a usable fallback, and have big payloads read from a file on disk by the agents (pass the path), not through args.
- On failure, resume with the script path and the run id after patching the persisted script. Completed agents replay from cache, so a pre-flight args crash costs nothing.

## Resume

Runs are resumable within the same session: completed agent calls return cached results, edited or new calls run live. The tool persists each run's script to a file and returns the path. To iterate, edit that file and re-invoke with the path and the run id rather than resending the whole script. The same script and same args give a full cache hit.

## Deep research

Claude Code ships a built-in deep-research workflow: it fans out web search, cross-checks each claim, and returns a cited report with unsupported claims filtered out. Use it for live web research. Reach for a custom workflow only when the research shape needs something it does not cover.
