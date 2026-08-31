# agent-skills

> **This is a public repository.** Do not commit API keys, passwords, tokens,
> internal URLs, or any credentials. Use `.env` for secrets and verify
> `.gitignore` is working before every commit.

Portable agent skills for delegating work to subagents, verifying what they return, and authoring skills of your own. Each skill is a folder with a `SKILL.md` and its reference files. They load unmodified in any harness that scans a skills directory; the `name` + `description` frontmatter is the only contract a consumer reads.

## What's here

Delegation and verification:

- `delegation-sizing`: decide whether a task is worth handing to subagents, which pattern fits, and how much wall-clock a round saves.
- `briefing-subagents`: write a brief that makes returned work auditable.
- `worktree-isolation`: spawn parallel file-writing subagents without collisions.
- `verifying-delegated-output`: check claims, git state, and content before you trust returned work.
- `dynamic-workflows`: author a Claude Code dynamic workflow, or decide when a workflow beats manual delegation.

Authoring:

- `meta-skill-creator`: the doctrine and builder for writing portable single-job skills.

## Install

Subscribe through the Claude Code plugin marketplace, so merged updates reach you on the next session:

```
/plugin marketplace add Exploitacious/agent-skills
/plugin install agent-skills@agent-skills
```

Or copy the files into any harness. Clone the repo and drop any `skills/<name>/` folder into the directory your harness scans for skills. You own the copy from then on, updates included.

## License

MIT. See `LICENSE`.

## Credit

The skill shape and progressive-disclosure model follow the agentskills.io spec. The prose follows the unslop writing style vendored from cursor/plugins `pstack/skills`, whose model in turn draws on mattpocock/skills `writing-for-agents`.
