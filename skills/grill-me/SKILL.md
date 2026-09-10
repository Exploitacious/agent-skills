---
name: grill-me
description: Use at intake when a request, plan, or design still has decisions the user has not made, or when the user says grill me, stress-test this, or poke holes in this. Not for work whose plan is already approved. Interviews the user in rounds until nothing is left open.
---

# Grill me

Interview the user until you share one understanding of what they want. Map the work as a decision tree: every decision branches into the decisions that hang off it.

## Rounds and the frontier

Work the tree in rounds. The frontier is every decision whose prerequisites are settled: the questions you can ask now without guessing at answers you have not heard. Ask the whole frontier in one round, then wait. A question whose answer depends on another question still open in this round belongs to a later round.

Each round of answers reshapes the tree: settled decisions push the frontier outward and unblock what depended on them. Recompute and ask the next round. The session is done when the frontier is empty: every branch visited, nothing left silently assumed. Say so, state the shared understanding in a few lines, and wait for the user to confirm it before acting. That confirmation is the go; do not ask for a second one, and if the user already approved the plan in this conversation, treat the last round of answers as the confirmation and proceed.

Do not re-ask what the user already told you, in this conversation or in their standing preferences. Ask about what to build, not whether to build it.

## How to ask

Number each question, give it a short title, spell out the choices, and give your recommended answer with the reason in one line. Where the active tool list offers a structured question tool, use it for a round of discrete-choice questions that fits the tool's own limit (four for AskUserQuestion in Claude Code, three for Codex's request_user_input, which is also mode-restricted), recommended option first. On a harness without one, for a round larger than the tool allows, or for answers that need free text, write the round as numbered prose:

```
Q1. <title>: <question, with the choices spelled out>
Recommended: <choice>, because <reason>.
```

No emojis, no filler. "I don't know" is a valid answer; record it as open and ask what would settle it.

## Facts are your job, decisions are theirs

When a question needs a fact from the environment (a file, a config, a tool's real behavior, a vendor limit), find it yourself: read the file, run the read-only command, or spawn an investigator lane. Never ask the user for something you could look up. A running lookup is an unsettled prerequisite: hold only the questions downstream of it and ask the rest of the frontier now. The decisions are the user's: put each to them and wait.

## Signs it is working

Disagreement, a recommendation the user overturns, a question that changes the shape of the work, a conclusion neither of you started with. If the user agrees round after round, check whether any decision or risk is still unresolved (failure modes, who else is affected, what this looks like in six months, what would make it not worth doing); if nothing is, agreement has completed the intake, so stop asking.

## What it does not do

It writes no files and leaves no workspace behind; the output is a sharper shared picture, usually captured in the plan that follows. It does not replace a written spec or design record for large work; hand the conversation to that step once the frontier is empty.
