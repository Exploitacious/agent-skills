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

## Host resources collide outside git too

Worktrees isolate git state and nothing else. Every other host resource lives in one namespace across every parallel lane, so isolation that stops branch collisions does nothing for these.

- Scratchpad. A session scratchpad is shared by the main thread and every subagent, so a chained "run this then that" can proceed against a sibling lane's state and one lane can overwrite another's scratch file. Give each lane its own scratch subdirectory and read back any scratch file before trusting it.
- Ports and containers. A throwaway container bound to a fixed host port collides with a sibling lane already on that port ("port already allocated"), and a client that connects to `localhost:<port>` reaches whatever container owns the mapping, not necessarily the one this lane meant to start. Give each lane a lane-unique container name and a reserved port range, and verify the container's actual port mapping with a `docker ps` name check before connecting.
- The shared checkout. Sessions in one main checkout share the index, so `git add -A` or `git add .` — and a plain `git commit` after another session has already staged its own files — sweeps that work into your commit under your message; explicit-path `git add` does not unstage what a sibling already staged. Commit from your own worktree where you can. Where a shared-checkout commit is unavoidable, name the paths on the commit itself (`git commit -- <paths>`) or read the full staged diff (`git diff --cached --stat`) before committing, so nothing else rides along.
- Host RAM. Lanes that each run the full test suite at once oversubscribe memory, and the OOM killer takes a test worker; the runner reports that crash as a failed gate on work that is actually fine. When more than one file-writing lane will run the full suite in the same window, cap the parallelism (at most two at once, or a lower per-run worker count) or split the gate so builds run the suite and reviewers run only the changed files. Treat a worker crash under contention as suspected infrastructure rather than a regression, but confirm it before dismissing it: look for the resource-exhaustion signature (an OOM-killed worker) and re-run the affected files at reduced concurrency. A clean re-run says infrastructure; a failure that repeats at low concurrency is a real defect to chase.

## When the guard blocks legitimate work

While a session is isolated in a worktree, the guard blocks git it cannot verify stays inside the worktree, and the same checks cover every subagent the session spawns. A `git -C <repo>` is blocked outright as one of the redirect forms, and a `cd` into another directory before running git can trip the working-directory and command-shape checks too. So a lane isolated in one repo's worktree cannot reliably run its own git in a second repo — not because the guard protects that repo, but because it cannot prove the command stays in the worktree. When a lane genuinely needs a second repo, either run it without worktree isolation (safe only when no parallel lane shares that repo) or have it return a verified patch you apply from the main checkout.

The guard keys off the session's cwd and the command text, whether the session entered its worktree with `--worktree`, with EnterWorktree, or on resume, so a few edges bite in practice:

- A lane spawned in the same batch as EnterWorktree keeps the shared checkout as its cwd, so every Bash call is refused, plain `ssh` and `echo` included, since the working-directory check does not single out git. Spawn lanes after the session is in the worktree, or from a session that is not isolated.
- A CLI agent launch that carries a free-text prompt, such as a headless `-p "<prompt>"` or `exec "<prompt>"`, trips the command-shape check, because the guard cannot prove from the text that the prompt is not a git command. Run those from the main checkout.
- A stray `cd` into a `.claude/worktrees/<name>` path leaves the session treated as isolated with no EnterWorktree, and ExitWorktree is then a no-op. A plain `cd` back to the repo root clears it.

For a doctrine or multi-repo pass, work from the main checkout and drive each branch with `git -C <worktree>`, using absolute paths.

## Nesting depth

No hard cap; subagents may spawn their own. At three or more levels deep, ask whether the chain has degenerated into uncoordinated research that should have been one well-scoped round. Deep nesting costs tokens and audit clarity, so pay it on purpose, not by default.
