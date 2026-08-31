# Skill spec

Read when writing or editing a `SKILL.md`. The folder shape and the progressive-disclosure model follow the agentskills.io spec; the frontmatter and invocation model follow mattpocock/skills `writing-for-agents` and its `SKILL-MECHANICS.md`; the prose follows the `unslop` skill.

## Folder shape

A skill is a directory whose name is the skill's slug.

```
SKILLS/<slug>/
  SKILL.md          required: frontmatter + body
  01_*.md           optional: reference files, disclosed on demand
  scripts/          optional: helpers, invoked not read
```

The directory name, the frontmatter `name`, and the slug a reader sees all match. Lowercase, hyphen-separated. Org skills carry their org's prefix. A name that must be globally unique across every mounted source is why the prefix exists.

## Frontmatter

Shared-repo skills carry two fields, nothing else.

```yaml
---
name: fix-root-causes
description: "Apply when debugging. Trace each symptom to its root cause and fix it there."
---
```

Why only two: unknown frontmatter fields are ignored by every major harness (Claude Code, Codex, Hermes, OpenCode, deepseek-harness, verified Aug 2026). An extra field does not error, it degrades silently. A skill that leans on a field one harness drops misbehaves on that harness with no warning. The portable contract is `name` + `description` and nothing a consumer can quietly discard.

Consumer-specific fields are allowed only where the consumer is known and named, one comment line per field saying why.

```yaml
# allowed-tools: honored by Claude Code only; keeps this audit skill read-only
allowed-tools: [Read, Grep, Glob]
```

Fields in this class: `allowed-tools`, `disable-model-invocation`, `user-invocable`, the Claude Code fork keys (`context`, `agent`), and any body substitution (`$ARGUMENTS`, a backtick shell injection). They live in personal or distribution skills, never in shared-repo skills.

## Invocation

The description is the skill's top-level context pointer, and its presence is a choice with a cost.

- Model-invoked keeps a `description`, so the agent fires the skill on its own and other skills can reach it. The price is permanent context load: the description sits in the window every turn. Typing the name still works; model-invocation only adds agent discovery, it never removes the human's reach. Mechanics: omit `disable-model-invocation`, write a model-facing description carrying the trigger branches.
- User-invoked strips the description from the agent's reach: only the human typing the name fires it, and no other skill can. Zero context load, paid for in cognitive load, since the human is now the index that must remember the skill exists. Mechanics: set `disable-model-invocation: true` (a consumer-specific field, comment it); the description becomes a human-facing one-line summary with the trigger list stripped.

Pick model-invocation only when the agent, or another skill, must reach the skill unprompted. If it only ever fires by hand, make it user-invoked and pay no context load.

Router skills: when user-invoked skills pile up past what a human can remember, one user-invoked router names the others and when to reach for each. It can only hint, never fire them, because user-invoked skills have no description for anything but the human to match.

## Description craft

The description is the only text a harness reads to decide relevance; the body does not load until then. Write it for retrieval.

- First sentence states when to apply, plainly. "Apply when debugging." / "Must always apply." / "Use when the user says X."
- Front-load the leading word. The pointer does its triggering work at the start.
- One trigger per branch. A branch is a distinct case the skill handles. Synonyms renaming one branch are one branch written twice; keep only genuinely distinct branches.
- Cut identity the body already carries. Every word of an always-loaded pointer costs on every turn.
- Then, if useful, one sentence on what the skill produces or constrains. Three sentences total.

Bad: "Helps with documentation." Good: "Apply when creating or editing API reference pages. Covers the section template, the code-sample rules, and the publish checklist."

## Progressive disclosure

Three tiers of the information hierarchy, each loaded later than the last.

1. name + description. Read at session start, always.
2. body. Read when the description matches.
3. reference files and scripts. Read only when the body points to them.

Keep the body lean so activation is cheap, and disclose depth into reference files. Branching is the test: inline what every run needs, push behind a pointer what only some branches reach. Route to a reference by its trigger, never "read all references to begin."

## Budgets

| Part | Target | Ceiling |
|---|---|---|
| description | 1-3 sentences | 3 sentences |
| body | under 100 lines | 500 lines |
| reference file | as long as its one topic needs | one topic each |

Exceeding a ceiling does not break loading. It raises per-activation cost and signals a skill doing too much.

## Portability and distribution

The load-bearing contract is `name` + `description`. Hold to it and the same folder loads unmodified in Claude Code, Codex, Hermes, OpenCode, and deepseek-harness. Which directory each harness scans, and how a repo of skills reaches a user, is the distribution layer's job, not the skill's; a skill that assumes its own install path has leaked distribution concern into portable content.

Two distribution modes sit above the skill, both demonstrated by mattpocock/skills. A managed plugin is a subscription: mount the repo once and merged version bumps propagate on their own, so the user tracks a source of truth they do not own. A `skills.sh`-style copy is a fork: the user pulls the files into their own repo and owns them from then on, updates included. Author the skill the same way for both; the choice between them belongs to the distribution, and it is why bindings must live in slots (see `03_context_slots.md`) rather than in the skill a stranger might copy.

## Build checklist

Before delivering a skill:

- Directory name, `name`, and the reader-facing slug match, lowercase-hyphen.
- Description first sentence states when to apply; one trigger per branch.
- Frontmatter is `name` + `description` only, or every extra field has a one-line comment and a known consumer.
- Body is one job, under 100 lines, unslop prose.
- No operator or org specifics in the body; local needs read a slot (see `03_context_slots.md`).
- Every non-obvious rule has its why next to it.
- References are routed by trigger, not loaded eagerly.
- A reader outside this conversation could drop the folder in and get useful output with no edits.
