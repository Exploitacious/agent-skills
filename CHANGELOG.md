# Changelog

All notable changes to this repo are recorded here. Format follows Keep a Changelog; a release is a plugin version bump in `.claude-plugin/marketplace.json`.

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
