# Changelog

All notable changes to this repo are recorded here. Format follows Keep a Changelog; a release is a plugin version bump in `.claude-plugin/marketplace.json`.

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
