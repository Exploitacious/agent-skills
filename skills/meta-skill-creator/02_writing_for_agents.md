# Writing for agents

Read when writing or rewriting any skill body. This distills mattpocock/skills `writing-for-agents` and binds it to the eight rules. For the full treatment, read that upstream skill; the terms below are its terms, kept so our doctrine shares vocabulary with a public one rather than coining its own.

A skill is a document an agent runs. The levers below make each run predictable: the agent takes the same process every time, not the same output.

## The two loads

Every line you add spends one of two budgets.

- Context load is the cost of always-loaded material on the agent's window: a description, an `AGENTS.md` line, anything in context every turn, spending tokens and attention whether or not it fires.
- Cognitive load is the cost on the human: which files exist and when to reach each. Not a cost to minimize to zero. It is the price of human agency; spend it where judgment matters, remove it where it does not.

Material behind a pointer escapes context load for the price of the pointer's own line. Material with no pointer rides entirely on cognitive load.

## Context pointers

A context pointer names out-of-context material and encodes the condition for reaching it. A skill's description is one; a line in a body naming a reference file is the same object. The pointer's wording, not its target, decides when the agent reaches the material and how reliably. A must-have target behind a weak pointer is a variance bug: sharpen the wording first, inline the material only if sharpening fails. Pointer-wording discipline is in `01_skill_spec.md` under description craft; it applies to every in-body pointer too.

## The information hierarchy

A body is built from steps (ordered actions the agent performs) and reference (definitions, rules, facts consulted on demand). They mix freely. The decision is where each piece sits on the ladder, ranked by how immediately the agent needs it.

1. In-file step. The primary tier: what the agent does, in order.
2. In-file reference. Consulted on demand. Often a flat peer-set, every rule on one rung, which is a fine arrangement, not a smell.
3. Disclosed reference. Pushed to a separate file behind a pointer, loaded only when the pointer fires.

Push too little down and the top bloats; push too much and you hide material the agent needs. That tension is the whole decision. Progressive disclosure is the move down the ladder so the top stays legible; it protects the hierarchy, it is not mainly a token trick.

Co-location is the within-file companion: keep a concept's definition, rules, and caveats under one heading rather than scattered, so reading one part brings its neighbors. The body should read like documentation written for the agent.

## Completion criteria

Every step ends on a completion criterion, the condition that tells the agent the work is done. Two properties make it a lever.

- Clarity: can the agent tell done from not-done? A vague bound ("understanding reached") invites premature completion, ending the step while attention slips to being done. Sharpen the bound first; it is local and cheap.
- Demand: how much the criterion requires. "Every modified model accounted for" forces thorough work where "produce a change list" does not. Demand drives the legwork latent in the wording. It binds flat reference too: "every rule applied" holds a body of reference the way "every step done" holds a sequence.

The strongest criteria are both checkable and exhaustive.

## Leading words

A leading word is a compact concept already in the model's pretraining that the agent thinks with while running the body (tight, red, lesson). Repeated as a token, never spelled out as a sentence, it anchors a region of behavior in the fewest tokens by recruiting priors the model already holds. Coining your own works only if you define it clearly, and you pay in definition tokens what a pretrained word gives free; reach for an existing word first. Hunt for restatements a leading word retires: "fast, deterministic, low-overhead" collapses to tight.

Negation is the failure beside this lever. Steering by prohibition drags the forbidden behavior into context and makes it more available, not less. State the positive target ("write one-line comments") so the banned behavior is never spoken. A prohibition earns its place only as a hard guardrail you cannot phrase positively, and even then pair it with the positive.

## Pruning

- Single source of truth. Keep each meaning in one authoritative place, so changing behavior is a one-place edit. Duplication costs maintenance and tokens and inflates a meaning's apparent rank.
- The environment is a source of truth too: config files, directory layout, `--help` output. A body that restates it is a cache, worth its load only when the lookup is expensive. Cache the unwritten convention and the reason behind a choice; leave one-command lookups to the environment where they cannot go stale.
- Relevance. Every line must still bear on what the skill does. A line loses relevance by never bearing on the task, or by going stale as the world it describes changes. Shorter bodies stay relevant more easily; the default fate otherwise is sediment, stale layers that settle because adding feels safe and removing feels risky.
- No-ops. Hunt sentence by sentence for an instruction the model already obeys by default. The test is model-relative: does this change behavior versus the default? Settle a disagreement by running the body, not by debate. When a sentence fails, delete the whole sentence.

## Anti-patterns that survive from the old doctrine

These predate `writing-for-agents` and still hold. They are all no-ops or negations by another name.

- Persona names ("You are Lyra, the optimizer"). The name changes nothing; the instructions do. It also reads inconsistently when the agent uses it sometimes and not others.
- Mode selection ("choose BASIC or DETAIL"). Forces a meta-decision before the work. Set maximum depth as the default and scale by scope control instead.
- Welcome messages ("when activated, display this greeting"). Skills have no activation event; a scripted greeting wastes the first response.
- Default-restating instructions ("be helpful", "be accurate", "think step by step"). The model already does these. If deleting the line changes no output, delete it.
- Capability claims with a shelf life ("in development", "not yet", "can't do X directly"). A loaded body is trusted over the live tool list, so a stale hedge makes the agent refuse a tool it has. This is observed, not theoretical: an agent with working tools planned from memory because a doc told it the tools were not wired up. Write the affirmative capability, and scrub every hedge in the same pass that wires a tool live.
- Cross-platform instructions ("for ChatGPT use X, for Claude use Y"). Write for the harness, not around it.
