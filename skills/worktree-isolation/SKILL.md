---
name: worktree-isolation
description: Use when spawning subagents that write files in parallel, so their branches, working directories, and scratch files do not collide. Covers worktree isolation and the five failure modes at its edges.
---

# Worktree isolation

For any subagent that writes files, spawn it with worktree isolation. The tool creates a git worktree from the parent commit, runs the subagent there, and surfaces the branch name back. Branches are visible globally because worktrees share the git directory. Worktree locks release at session end; if one will not release mid-session, defer the cleanup to the next restart rather than fighting it. The parent auto-names the branch unless the brief specifies one.

Isolation is a parameter on the spawn, not a sentence in the brief. Telling an agent "you are in an isolated worktree" does nothing; passing the isolation flag does. Read-only investigators do not need it. A file-writing background agent without it shares your working directory, so any checkout, merge, pull, or stash you run races its uncommitted edits.

## The five failure modes at the edges

Isolation protects against branch collisions. It does not protect against a drifting working directory, invisible files, or a contaminated base branch. Each of these cost a real rework cycle. Salvage recipes are in `edge-recovery.md`.

1. Spawn off the wrong directory. The worktree is created under whatever repo your shell is in at spawn time, not the repo the brief names, and it exists before the subagent reads the brief. The shell's directory persists across calls, so a stray earlier `cd` silently redirects every following spawn. A mis-pinned lane cannot recover from inside, because the guard refuses cross-repo git. Defense: run `cd <target repo> && pwd` as its own call immediately before every dispatch. Batch parallel spawns into one response only when they target the same repo; lanes for different repos spawn one at a time with a directory change between them. Add a brief gate: the repo root must match the expected repo, halt if not.
2. Escape into the shared checkout. An agent trained on absolute paths runs `cd /abs/path/to/repo` and escapes its worktree into the shared main checkout, where parallel agents are also writing. Defense: open every worktree brief with a discipline block: relative paths only, never `cd` to an absolute path under the main checkout, verify the directory and branch at start and end.
3. Gitignore blindness. A worktree branches from the committed tree, so gitignored or merely uncommitted files, such as local research notes, environment files, and design docs, are invisible to the subagent. Cite a gitignored spec path and the agent builds blind and improvises. Defense: before briefing, check whether the spec lives in a gitignored or uncommitted path; if so, paste its content inline rather than citing the path.
4. Shared-scaffolding merge conflicts. Three or more parallel agents that each branch from the same parent and touch the same shared files (a package index, a model file, a CHANGELOG, a counts table) guarantee three-way conflicts. These are structural, not agent bugs, since isolated agents cannot see each other's edits. Defense: merge serially, never batch. Pick the order up front (the agent extending the shared scaffolding first), and rebase each branch against the new main between merges. Two sharp edges live in `edge-recovery.md`: a folded shared signature that git splits wrong, and a merge tail that can dominate the whole round's budget.
5. Wrong base branch. Even a correctly isolated worktree is cut from the currently checked-out branch, not the remote main. Spawn writers while the main checkout sits on a feature branch and every worktree inherits that branch's commits, which then contaminate unrelated PRs. Defense: check out main synced to the remote before spawning writers, and before merging audit each branch with a name-only diff against the remote main. A branch carrying files outside its task is contaminated; resolve by merge order or rebase.

## Scratch files collide outside git too

Parallel lanes on one machine share more than a repo. A session scratchpad is shared by the main thread and every subagent, so a chained "run this then that" can proceed against a sibling lane's state, and one lane can overwrite another's scratch file. Give each lane its own scratch subdirectory, keep chained commands from assuming a sibling's output, and read back any scratch file before trusting it.

## Nesting depth

No hard cap; subagents may spawn their own. At three or more levels deep, ask whether the chain has degenerated into uncoordinated research that should have been one well-scoped round. Deep nesting costs tokens and audit clarity, so pay it on purpose, not by default.
