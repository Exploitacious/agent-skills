# Edge recovery

Read this when a worktree edge case has already bitten and you need the salvage steps, or when planning a merge of several parallel branches.

## Salvage an agent that escaped into the main checkout
When an agent wrote uncommitted edits into the shared main checkout:

1. Stop the agent.
2. Stash its files with the include-untracked flag: `git stash push -u <its files>`.
3. Move the checkout back to a clean main: `git checkout main && git pull --ff-only`.
4. Create the branch it should have used: `git checkout -b <branch>`.
5. Restore its work: `git stash pop`.
6. Verify before trusting every file it wrote. Escape correlates with improvisation, so audit the content, do not just merge it.

If a post-salvage `git pull --ff-only` fails with "not possible to fast-forward," leaked commits are the cause. `git reset --hard origin/main` is safe when the remote squash carries the same diff.

## Before merging parallel branches: the dirty-overlap check
A worktree branch lands only as a commit, so brief every lane to make its own commit on its branch (one clean conventional commit) and report the branch and path. A "do not commit, the parent merges" brief just leaves every worktree dirty for the parent to commit by hand.

A shared checkout with concurrent sessions always has someone else's dirty files, and `git merge` in that checkout refuses with "local changes would be overwritten" when the branch touches one of them. Intersect the branch's changed files with the main checkout's uncommitted set — `git diff --name-only main...<branch>` against `git status --short` — as a detection step, not a fix: a file in both is one another session is editing, so do not overwrite or discard it. Merge somewhere that never touches the other session's uncommitted work: cut a named integration branch from the remote main in its own worktree (`git worktree add <tmp> -b integrate/<topic> origin/main`), then `git -C <tmp> merge <branch>`, resolve any conflict in `<tmp>`, and land the integration branch through the remote (`git -C <tmp> push -u origin integrate/<topic>`, then a PR, or a push straight to main when the repo allows it and the branch fast-forwards). Merging inside a detached worktree updates only that HEAD, and checking out main there fails while the main checkout owns it, so the dirty local main stays untouched until its owner runs `git pull --ff-only`. Or wait for that session to commit first. A purely derived file such as a generated index or a lockfile is the exception: park it and regenerate it after the merge rather than merging it.

## The folded shared signature
When both sides of a conflict contain an identical line block, such as a common function signature or a `PRIMARY KEY (...)\n);` tail, git folds that block into the common region and splits one side's definition from its body. A naive "keep both" then yields broken code: a definition with no body, or a table that never closes. Physically reorder and rewrite the whole region; do not just delete the conflict markers.

## After resolving a conflict: diff against HEAD
A lane is cut from an older integration branch, so inside one conflict block the integration side may have deleted some rows, edited others, and kept the rest, while the lane edits different rows of its own. A quick resolver that keeps the lane's side minus the rows the integration side deleted matches rows by exact line equality, and a row both sides touched equals neither the base copy nor the lane's copy, so it silently vanishes. This happened in a shared backlog file: three rows sat in one conflict block, the integration side had also edited one of them, and the resolution dropped it. Nothing failed; only a grep for the row found it.

Resolve from HEAD's block (the integration side) and apply the lane's per-row edits to it, matching rows by what identifies them (a date and topic, a key, an id) rather than by exact text. Before `git add`, run `git diff HEAD -- <file>`. It must show exactly the lane's intended row changes and nothing else; a removed line the lane did not mean to remove is a dropped row. This holds for any file many lanes edit, such as a backlog, an index, or a README. If the first pass was wrong, `git checkout -m <file>` recreates the conflict markers for another try.

## When the merge tail dominates the budget
Parallel builds are cheap; landing them can be the expensive part. One round of five parallel workers built in about 90 minutes and took about five hours of serial rebases across roughly 50 conflict hunks to land. When features stack on the same anchors, either serialize the builds, or give each worker its own new module and defer the central wiring, such as schema registration and orchestration call sites, to one thin integration commit you write after all lanes land. Disjoint lanes parallelize for free; shared anchors do not.

## Recover a hijacked HEAD
When HEAD moved in a checkout another agent is also using:

1. Clean the contested index for the other agent: `git reset`, then `git checkout -- <shared files>`.
2. Do your own commit in an isolated throwaway worktree: `git worktree add -b <branch> <scratch> origin/main`.
3. Redo your tracked edits there and copy the untracked new files over. Untracked files follow branch switches, so they are safe to move.

Base the worktree on the remote main, not local main. Local main is often stale behind merged PRs.

## A branch still checked out by a finished lane
An isolated lane's branch stays checked out in its worktree after the lane ends, so `git switch <branch>` in any other worktree fails with "already used by worktree". When the lane has pushed and its tree is clean (check `git -C <lane-worktree> status --short`), free the branch with `git -C <lane-worktree> switch --detach`, then switch to it where you want it. A dirty tree means the lane did not finish; read what it left before you touch it.
