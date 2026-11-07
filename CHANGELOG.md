# Changelog

All notable changes to this repo are recorded here. Format follows Keep a Changelog; a release is a plugin version bump in `.claude-plugin/marketplace.json`.

## [0.1.10] - 2026-11-07

### Changed

- `diagnosing-bugs` (`SKILL.md`): the description now names its trigger as a
  failing or slow command, test, or service whose cause is still unknown, and
  says what it is not for (status updates, resume batons, reports that already
  name the cause). The old wording ("reports something broken") matched ordinary
  progress reports, so the skill was offered where no diagnosis was needed.

## [0.1.9] - 2026-11-07

### Changed

- `dynamic-workflows` (`authoring-workflows.md`): every script that reads `args`
  opens with an args prelude that accepts undefined, null, a JSON string or the
  wrong shape and returns a clear error instead of crashing (four crashes in two
  weeks, each before one agent ran: `args.chains.map is not a function`,
  `LANES.map` on undefined, `rows.length` on a string). A new section, "After a
  usage-limit hold or a crash", covers finding the lanes that never returned,
  confirming the run is over, reading each dead lane's worktree, and relaunching
  with `resumeFromRunId` so returned lanes replay from cache.
- `dynamic-workflows` (`SKILL.md`): the description and the pointer paragraph
  name the prelude and the recovery section.

## [0.1.8] - 2026-11-07

### Changed

- `worktree-isolation` (`SKILL.md`): a sixth failure mode, resuming a lane whose
  worktree is gone. An isolated lane that stops before writing anything has its
  worktree reclaimed as unchanged, and a `SendMessage` resume then lands in the
  shared checkout and moves it off main. Spawn a fresh lane instead, have the
  brief commit a placeholder before any yield, and check the primary checkout's
  branch after any resume. A new section covers cleanup after integration: audit
  each worktree, pin reflog-only commits under a ref, remove without `--force`,
  and why squash-landed lane worktrees survive branch cleanup tools.
- `worktree-isolation` (`edge-recovery.md`): after resolving a merge conflict
  that touches a shared file, `git diff HEAD -- <file>` must show only the
  incoming lane's own changes, since an exact-text resolver drops a row both
  sides edited; and the recovery for a branch still checked out by a finished
  lane's worktree (`git switch --detach`).
- `verifying-delegated-output` (`SKILL.md`): recovery when a lane ends on a
  waiting line instead of a report. Read the branch, run your own gates, write
  the report from the commits and say the builder ended without one, carry the
  unfinished check into the landing check, and do not attribute a running host
  process to the lane.

## [0.1.7] - 2026-10-22

### Fixed

- `briefing-subagents` (`brief-template.md` worked example): the heavy-build
  brief used names from an internal project (repo path, stakes line, table and
  branch names). It is now a neutral order-history retention example for a
  generic web service. Every instruction line is unchanged; only the names,
  paths, and stakes wording differ, so the example teaches the same brief shape.

## [0.1.6] - 2026-10-22

### Changed

- `dynamic-workflows` (`authoring-workflows.md` Resume, `SKILL.md`): never
  `SendMessage` an agent that is still running inside a workflow. The send
  resumes a second copy from the lane's transcript while the workflow's own
  instance keeps running, and both copies write one worktree. Put shared
  decisions in the brief before launch, or stop the workflow and relaunch with
  the decision; recovery is `TaskStop` on the workflow and each resumed agent id.
- `worktree-isolation` (Scratchpad bullet) and `briefing-subagents`
  (`brief-template.md` section 1 and the worked example): a lane's own temp
  directory comes from `mktemp -d` with a trap that removes exactly that path;
  never delete scratch by glob, since a parallel run's glob cleanup removed
  sibling lanes' live temp directories.

## [0.1.5] - 2026-10-19

### Fixed

- `tdd` and `diagnosing-bugs` now bundle the upstream MIT copyright and
  permission notice as `LICENSE.upstream`, as `grill-me` already did. Their
  READMEs pointed at this repository's LICENSE, which carries a different
  copyright, so a per-skill copy or mount dropped the notice MIT requires.
- `README.md` lists `tdd` and `diagnosing-bugs`, which shipped without an entry.

## [0.1.4] - 2026-09-10

### Changed

- `worktree-isolation`: broadened the outside-git collision section to cover
  host ports and container names, `git add` in a shared checkout, and host RAM
  under concurrent full test suites; added a section on the guard refusing
  git it cannot verify stays in the worktree and the session-level EnterWorktree
  edges; documented the pre-merge dirty-overlap check in `edge-recovery.md`.
- `briefing-subagents`: added briefing guidance for a lane that stops or
  restarts a live service, since a subagent can end between any two tool calls:
  lane-death-safe single-payload mutations and foreground long steps.
- `dynamic-workflows`: `authoring-workflows.md` now covers keeping structured
  `agent()` returns under the validation cap (bound counts with `maxItems`, long
  form to a file, and reject stub returns) and setting a per-lane WebSearch
  ceiling under the shared budget.

## [0.1.3] - 2026-09-10

### Added

- `grill-me`: intake interview skill adapted from mattpocock/skills `grilling`
  (MIT, pinned at 85f83d3fde1d): rounds over the decision tree, the whole
  frontier per round with a recommended answer each, facts gathered by the
  agent, model invocation enabled so it fires at intake.

## [0.1.0] - 2026-08-31

### Added

- Initial marketplace with the `agent-skills` plugin.
- Delegation and verification skills: `delegation-sizing`, `briefing-subagents`, `worktree-isolation`, `verifying-delegated-output`, `dynamic-workflows`.
- Authoring skill: `meta-skill-creator`.
- CI: `agentskills validate` on every skill, zip artifact per skill on tag.
