---
name: meta-skill-creator
description: "Apply when creating, splitting, or auditing an agent skill, or converting a fat skill or cloud Project into portable single-job skills. Produces a SKILLS/<slug>/ folder that loads unmodified across Claude Code, Codex, Hermes, OpenCode, and deepseek-harness, plus the optional GUI-Project mirror."
---

# Meta skill creator

Author portable agent skills. One skill does one job; this skill's job is authoring skills. Anything bigger splits into more skills.

The primary doctrine is mattpocock/skills `writing-for-agents`, whose writing model this file adopts: context pointers, the two loads, the information hierarchy. The secondary references are cursor/plugins `pstack/skills` (the prose exemplar, vendored here as the `unslop` skill) and the agentskills.io spec (folder shape and progressive disclosure).

## The eight rules

Every skill you write or split obeys these. They are the doctrine your team and public adopters inherit.

1. One skill, one job. Prefer many small skills over one fat one. A body whose sections cover five workflows is five skills. Target under 100 lines of body; the hard ceiling is 500. Past that you are holding multiple jobs.
2. The description is a context pointer. Its wording, not its target, decides when the skill fires. First sentence states when to apply, plainly: "Apply when debugging." / "Must always apply." / "Use when the user says X." Front-load the leading word; give one trigger per branch, never synonyms for the same branch. The name is a lowercase-hyphen slug matching the directory; org skills carry their org's prefix. See `01_skill_spec.md`.
3. Spec-minimal frontmatter. `name` and `description` only, in shared repos. Fields only some consumers honor (`allowed-tools`, `disable-model-invocation`, body substitutions) belong only in personal or distribution skills where the consumer is known, one comment line documenting each. Unknown fields degrade silently rather than erroring, which is the danger, not the safety. See `01_skill_spec.md`.
4. Unslop prose. Every body follows the `unslop` skill: plain words, numbered patterns, active voice, sentence-case headings, no em dashes, no ceremony, no bold spam. Prompt the positive; a prohibition drags the banned behavior into context and half-reads as an instruction to do it. See `02_writing_for_agents.md`.
5. Context slots, not forks. A skill never embeds operator or org specifics (names, IDs, hosts, voices). It reads a well-known slot file and degrades when the slot is absent, the way `diagnosing-bugs` reads `CONTEXT.md` if it exists. This is what lets one skill serve every user unforked. See `03_context_slots.md`.
6. Decompose technique from bindings. Converting a fat skill: split the portable technique from the local bindings, then split multi-job bodies into single-job skills, then rewrite each in unslop prose. See `04_decomposition.md`.
7. Portability is name plus description. A skill loads unmodified in Claude Code, Codex, Hermes, OpenCode, and deepseek-harness. The only load-bearing contract is `name` + `description`. Which directory each harness scans is the distribution's job, not the skill's.
8. GUI dual-track stays personal. The `00_System_Prompt.md` claude.ai Project files are a personal-repo convention for skills that also serve GUI Projects. They are not part of the portable shape and never ship to shared repos.

## Build flow

1. Deconstruct. Find the job the input actually needs, not what was said: the entities (tools, domains, workflows) and the platform artifacts to strip (ChatGPT-isms, Gemini-isms).
2. Diagnose. Name what is wrong before building: competing identities, no-op lines that change no behavior, embedded specifics that belong in a slot, multi-job bodies that should split. Tell the user the diagnosis and why the restructure is better; they maintain it.
3. Develop. Build `SKILLS/<slug>/` per `01_skill_spec.md`: a lean `SKILL.md` plus reference files, one job each, local specifics routed to slots. If converting, run the decomposition method.
4. Deliver. Hand back the folder, run the build checklist in `01_skill_spec.md`, then ask whether to walk a file or test it.

## Clarifying questions

Ask at most 3-5, each with a default the user can confirm rather than author. Never ask what the input already answers. When confidence is reasonable, state the assumption and move on.

## Refuse

- Persona names ("You are Lyra"). The instructions carry the behavior; the name does not.
- Mode selection ("BASIC or DETAIL"). Set maximum depth as the default.
- Capability claims with a shelf life ("in development", "can't do X yet"). A loaded doc is trusted over the live tool list, so a stale hedge makes the agent refuse a tool it has. Write the affirmative capability; scrub hedges when tools go live.
- Fluff files to look thorough. One SKILL.md and one reference is a complete skill.
- Operator or org specifics in a shared skill. That is what slots are for.

## Reference files

Load one when its trigger fires. Do not read them all up front.

- `01_skill_spec.md`. The container: folder shape, spec-minimal frontmatter, invocation choice, budgets, portability, the build checklist.
- `02_writing_for_agents.md`. The writing doctrine: pointers, the two loads, the information hierarchy, completion criteria, leading words, pruning.
- `03_context_slots.md`. The slot mechanism and the slot registry.
- `04_decomposition.md`. Converting a fat or cloud-only skill into single-job portable skills.
