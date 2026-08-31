# Context slots

Read when a skill needs operator, org, or environment specifics: a voice, a ticketing binding, a team roster, a tool namespace. Slots are the mechanism that lets one skill in a shared repo serve every user without a fork.

## What a fork costs, and why slots remove it

A skill that hard-codes "log time on a specific PSA ticket" or "match one person's brand voice" cannot ship to another user or the public without editing. Forking one copy per user multiplies maintenance and drifts: a fix in one copy never reaches the others. Slots remove the fork by keeping the local specific out of the skill entirely.

## The mechanism

The skill references a well-known slot file and degrades gracefully when the slot is absent. The live example is mattpocock's `diagnosing-bugs`, which opens with:

> read `CONTEXT.md` (if it exists) to get a clear mental model of the relevant modules

The skill carries the technique. The slot carries the local binding. The same file runs unmodified for a user who has the slot and a user who does not.

Rules:

- Reference the slot by its well-known name, not an absolute machine path. Path plumbing is the distribution's job.
- Always state the degrade: what to do when the slot is missing. A skill that breaks without its slot is not portable.
- Never inline the slot's content into the skill. Inlining is the fork you are avoiding.
- Phrase the reference as a positive with a fallback, not a prohibition. "If `voice.md` exists, match its register; otherwise use plain professional prose."

## The slot registry

Slots live in the user's distribution, in a well-known `CONTEXT/` directory, never in a skill repo. The initial registry:

| Slot | Holds | Degrade when absent |
|---|---|---|
| `voice.md` | operator register and brand voice | plain professional prose |
| `work-tracking.md` | ticketing and billing bindings: which PSA, which fields | skip the tracking step and say it was skipped |
| `team.md` | roster, roles, who owns what | address roles generically, no names |
| `tools.md` | which MCP namespaces exist on this host | use only tools the session actually exposes |
| `doctrine.md` | the project's engineering-principle registry (the P/F numbers skills cite by name) | name the exact required behavior instead of a principle number |

The `work-tracking.md` pattern is proven: a session-close skill that reads its ticketing binding from context rather than hard-coding a PSA is what lets the same close ritual run for a user on a different tracker.

## Adding a slot

A new slot earns its place when more than one skill needs the same class of local specific. Add the row here, give it a degrade, and put the file in the distribution's `CONTEXT/`, not in the skill. One skill needing one specific is a candidate for a private companion skill instead, not a new slot (see `04_decomposition.md`).
