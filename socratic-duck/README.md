# socratic-duck

A Claude skill for stress-testing a single technical decision through structured interrogation, producing a persistent decision log under `decisions/` in your project root.

## What it does

When triggered, the skill runs a one-question-at-a-time interrogation over a single root decision and its sub-decisions (up to 3 levels deep). It investigates the codebase first, refuses to propose answers, criticizes weak reasoning after each user response, and produces a markdown decision log when the session ends.

## Warning: this skill is deliberately confrontational

This is not a friendly assistant mode. The skill is designed to:

- Refuse to validate decisions until trade-offs are stated explicitly.
- Push back on weak, hand-wavy, or unverified answers.
- Stop the session and force resolution when the user contradicts themselves.
- Refuse to propose "the answer" when the user does not know, offering options with trade-offs instead.

**If you want encouragement, supportive feedback, or quick alignment, do not install this skill.** Use a different one or interact with Claude normally. This skill assumes you want your thinking challenged, not confirmed.

## When it triggers

Explicit triggers:

- "Question this", "grill me on this", "pressure-test this".
- "Let's discuss in depth", "let's go deep on this".
- "Technical decision:" used as a prefix to introduce a decision.

Proactive trigger (asks for confirmation before running):

- The user appears to be committing to a non-trivial architectural, infrastructure, security, or stack decision without stating trade-offs.

The skill does not trigger for decisions that are local, easily reversible, and do not affect interfaces, data models, infrastructure, or external contracts.

## Output

A markdown file at `<project-root>/decisions/YYYY-MM-DD-<slug>.md` containing context, the decision, alternatives rejected, trade-offs accepted, assumptions with revisit conditions, sub-decisions (light treatment), weak points flagged at the end of the session, deferred items, and references.

If a file with the same name already exists, the skill stops and asks how to proceed.

## Credit and prior art

This skill is a variant of [Matt Pocock's `grill-me` skill](https://github.com/mattpocock/skills). The differences:

- Single root decision per session, with bounded depth (max 3 sub-levels).
- Mandatory codebase investigation before asking the user.
- The skill never proposes answers; only offers options with trade-offs when the user is stuck.
- Hard stop on user contradictions until they are resolved.
- Persistent output: every session produces a markdown decision log under `decisions/` so the reasoning survives the session.

If you want the lighter, more open-ended version, use Pocock's original.

## License and authorship

This skill is released under the MIT License (see the LICENSE file at the repository root). Original concept by Matt Pocock, whose `grill-me` skill is also MIT-licensed; this is an independent reimplementation of the idea, not a fork of his code.
