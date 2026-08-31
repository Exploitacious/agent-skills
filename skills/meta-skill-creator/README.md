# meta-skill-creator

The doctrine and builder for authoring portable agent skills. One skill does one job; this entry's job is authoring skills. Use it when:

- Creating a new skill.
- Splitting a fat skill into single-job skills.
- Converting a cloud-only claude.ai Project into a portable skill.
- Auditing an existing skill against the eight rules.

## When this activates

`SKILL.md`'s description triggers on: create a skill, new skill, split this skill, audit my skill, convert this Project to a skill, migrate a skill. It is model-invoked; typing the name works too.

## The eight rules

The doctrine is eight ruled points, stated in `SKILL.md` and inherited by every skill your team and public adopters write: one skill one job; description-as-context-pointer; spec-minimal frontmatter; unslop prose; context slots not forks; decompose technique from bindings; portability is name plus description; GUI dual-track stays personal.

## External exemplars

- mattpocock/skills `writing-for-agents` (primary): the writing model this doctrine adopts, so the vocabulary matches a public one.
- cursor/plugins `pstack/skills` (secondary): the prose exemplar, vendored as the `unslop` skill.
- agentskills.io: the folder shape and progressive-disclosure spec.

## Files in this entry

| File | Purpose |
|---|---|
| `SKILL.md` | The doctrine: eight rules, build flow, reference routing. Body under 100 lines. |
| `01_skill_spec.md` | The container: folder shape, spec-minimal frontmatter, invocation choice, budgets, portability, build checklist. |
| `02_writing_for_agents.md` | The writing doctrine distilled from mattpocock, plus the surviving anti-patterns. |
| `03_context_slots.md` | The slot mechanism and the slot registry, the fork-avoidance mechanism. |
| `04_decomposition.md` | Converting a fat or cloud-only skill into single-job portable skills. |

## Deployment

Mount the folder wherever the harness scans for skills. In Claude Code, place it under a skills directory the session loads and the `SKILL.md` picks up next session. The load-bearing contract is `name` + `description`, so no per-harness edits are needed.
