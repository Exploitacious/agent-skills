# Decomposition

Read when converting an existing fat skill, or a cloud-only Project, into portable single-job skills.

## The method, in order

1. Split technique from bindings. Separate the portable procedure from the local specifics. The technique goes to the portable skill (public or org). The bindings go to a slot (see `03_context_slots.md`) or, when they are too specific for a slot, a private companion skill the portable one is used alongside.
2. Split multi-job bodies into single-job skills. If the body has sections covering separate workflows, each section is its own skill. A skill named for one job that also does three others is four skills.
3. Rewrite each result in unslop prose. Run the `unslop` skill and the writing doctrine (`02_writing_for_agents.md`) over every new body.

Do the splits before the rewrite. Rewriting a body you are about to cut in half wastes the pass.

## Technique versus bindings

Technique is what any user could run: the steps of a debug loop, the shape of a trade plan, the checklist for a doc review. Bindings are what only this user has: the PSA to log against, the voice to match, the hostnames, the client names.

The test: could a stranger run this line unchanged? If yes, it is technique. If it names a system only you have, it is a binding, and it moves to a slot or a companion.

## Where a split piece lands

| Piece | Destination |
|---|---|
| Portable technique, useful to anyone | a public skills repo |
| Technique useful to your org, org-generic | your org's private skills repo |
| Binding shared by several skills | a context slot |
| Binding specific to one skill | a private companion skill |
| Personal-only technique | your personal skills directory |

## When to split, and when not

Splitting spends one of the two loads (see `02_writing_for_agents.md`), so split only when the cut earns it.

- Split by job. A body doing several jobs is several skills. This is rule 1 and the main reason to decompose.
- Split by sequence. Split a run of steps when the later steps, visible ahead, tempt the agent to rush the one in front of it. Keeping them out of view drives more legwork on the current task. Hiding works only across a real context boundary, a hand-off or a subagent dispatch; an inline call leaves the later steps in context and clears nothing. The reverse warns you too: merging sequences exposes each step to what follows and invites premature completion.
- Split by invocation. Split off a model-invoked skill when a distinct leading word should trigger it on its own, or another skill must reach it. You pay context load for the new always-loaded description, so the independent reach has to be worth it (see `01_skill_spec.md`).

Do not split to look thorough. A split that produces a skill with a body of only references, no steps or technique of its own, went too far; fold it back.

## Keep the shape

Splitting never changes the folder shape or the progressive-disclosure model (see `01_skill_spec.md`). Each resulting skill is a full `SKILLS/<slug>/` folder: metadata, then body, then references.

## Writing craft for each split

As you rewrite each body, apply the patterns that make a body work.

- One identity, imperative, first line. Not two roles stitched together.
- Frame constraints as what the skill does, not a rule list bolted on.
- Present options with trade-offs when the user is the decision-maker; map the decision space, do not prescribe the choice.
- Every instruction changes behavior. If removing a line changes no output, remove it.
