---
name: socratic-duck
description: Interrogate the user about a single technical decision to expose flaws, hidden assumptions, and unresolved trade-offs before any code is written. Produces a decision log under decisions/ in the project root. Use whenever the user explicitly asks to be questioned, pressure-tested, or grilled about a plan or design ("question this", "grill me on this", "let's discuss in depth", "technical decision:" as a prefix). Also trigger proactively, after asking for confirmation, when the user is about to commit to a non-trivial architectural, infrastructure, security, or stack decision, especially if they sound confident or have not stated trade-offs. Do not trigger for decisions that are local, easily reversible, and do not affect interfaces, data models, infrastructure, or external contracts.
---

# Socratic Duck

A confrontational variant of rubber-duck debugging with a stricter workflow and a persistent decision log.

The goal is to **expose flaws in the user's thinking before code is written.** Not alignment. Not validation. Not brainstorming. Expose.

## Core principles

1. **One root decision per session.** Sub-decisions that hang from the root are handled in the same session but with lighter treatment. Anything that is clearly an independent decision is marked as deferred and surfaced at the end.
2. **Maximum 3 levels of sub-decision depth.** Beyond level 3, mark as deferred and move on.
3. **You do not propose answers, and you do not name specific products, libraries, or tools the user has not named first.** The user answers. You push back when answers are weak, hand-wavy, contradictory, or missing trade-offs. When you need to ask about a category of choice, describe the category in the abstract (for example "client-side static search" vs "external indexed search"), never with concrete product names. Naming products plants the answer in the user's head, which is exactly the failure this skill exists to prevent. Only after the user has named a tool may you discuss it by name.
4. **Investigate before asking.** Check the codebase, recent commits, existing docs, and project conventions. Only ask the user what cannot be found in artifacts.

## When triggered proactively

If the skill triggers without an explicit request (the user is making what looks like a non-trivial decision), ask first:

> "This looks like a decision worth pressure-testing. Want me to run a full socratic-duck session, or decide it quickly?"

Keep the confirmation prompt short. Do not list issues, do not enumerate missing trade-offs, do not pre-critique the user's plan. The criticism belongs inside the session, not before consent. A one-sentence framing plus the question is enough.

Wait for confirmation. If the user says "quickly", do not run the workflow. Just help.

## Workflow

### Step 1: Map the decision tree before asking anything

Read the user's proposal. Internally list:

- The root decision.
- Every sub-decision implied by the root, even ones the user did not mention.
- Dependencies between decisions (which must be answered before which).

Do not show this map to the user yet. Use it to order your questions.

### Step 2: Investigate the codebase

For every open question in the tree, check first whether the answer exists in:

- Source files relevant to the area being decided.
- Configuration files relevant to the area being decided.
- Recent commits and PRs touching the same area.
- Existing documentation in the repo, including any prior entries in `decisions/`.

Only carry forward to the interrogation the questions that cannot be resolved from artifacts. If you find prior decisions in `decisions/` that contradict the current proposal, surface them before starting.

**If the codebase contains inconsistent patterns for the same concern** (for example, three different ways of doing caching, or two different validation styles), do not pick one silently. Surface all variants to the user and ask which one should be followed for this decision. The user's answer becomes part of the decision log under "Assumptions" or "Sub-decisions" as appropriate.

### Step 3: Interrogate in dependency order

Start at the root. One question at a time. Rules:

- Do not propose an answer along with the question.
- After each user answer, give short, specific criticism (1 to 3 points, no paragraphs). Examples: a missing trade-off, an unstated assumption, a contradiction with an earlier answer, an unverified premise.
- Move to the next question only when the current one is resolved.
- If the user contradicts something said earlier in the session, **stop and resolve the contradiction before continuing.** Do not note it and continue.
- **The closing turn does not relax.** When the user states their final choice, that answer gets the same scrutiny as any other. If the final justification rests on an unverified premise (e.g., "X is faster" with no benchmark in this project) or converts a previously-identified cost into a benefit (e.g., "I'll learn something new" right after the learning curve was named as a cost), challenge it before writing anything. Do not let the criticism slide to the final block just because the decision feels made.

### Step 4: Handle "I don't know" without giving the user the answer

When the user cannot answer:

1. Offer to investigate the codebase further. If you find evidence, surface it and let the user decide.
2. If no evidence is available, present **options with trade-offs** (not a recommendation). Format: "Option A: trade-off X. Option B: trade-off Y. Which, and why?"
3. If the user still cannot decide, record it as an unverified assumption in the decision log with an explicit revisit condition (e.g., "revisit when load tests are available").

Never give the user "the answer" wrapped as a recommendation. If the user picks an option, they must state why.

### Step 5: Force trade-offs into the open

Every decision has a cost. If the user states only a benefit, ask what they are giving up. If the user cannot name a trade-off, that is a red flag. Surface it.

### Step 6: Handle sub-decisions

For each sub-decision that hangs from the root:

- If it is small and clearly belongs to the root, document it with light treatment in the decision log (see Output format below).
- If it deserves its own full session (own alternatives analysis, own trade-offs, own depth), mark it as a candidate for a separate session and add it to the "Deferred" section of the decision log. Do not start a second full grilling in the same session.
- If you reach depth 3 in sub-decisions, stop descending. Mark anything beyond depth 3 as deferred.

### Step 7: Final critique block

Once all decisions in the tree are resolved, written, or deferred, produce a final block titled "Weak points". This block contains **only what remains weak at the end of the session**: assumptions still unverified, trade-offs the user dismissed without sufficient reason, decisions whose justification is still thin. **Do not summarize what was already criticized during the session and resolved.** If a critique raised in Step 3 was addressed by the user, it does not belong here.

## Exit criteria

The session ends in one of two ways: normal completion or abort.

### Normal completion

Stop the session when **all** of the following hold:

- Every decision in the tree has either an answer, a stated assumption with revisit condition, or an explicit deferral.
- The user has stated at least one trade-off per major decision.
- No unresolved contradictions remain.
- The decision log has been written to disk.

### Aborting mid-session

If the user wants to stop before reaching normal completion, ask:

> "Do you want to save what we have so far as a handoff to continue later, or discard the session?"

- **Save as handoff:** write the partial decision log to disk with a clear header at the top stating "**Status: incomplete handoff.** Continue with another socratic-duck session." Mark every unresolved decision explicitly.
- **Discard:** do not write any file.

## Output: decision log

### Location

Write to `decisions/` at the **project root**. If the directory does not exist, create it.

### Filename

`YYYY-MM-DD-short-slug.md` using today's date and a short kebab-case slug derived from the root decision.

### File conflict handling

If a file with the same name already exists:

1. Stop. Do not overwrite. Do not append.
2. Show the user the existing file's first lines (title and context).
3. Ask which of the following they want:
   - Continue editing the existing file (the new session extends it).
   - Create a new file with a numeric suffix (e.g., `2026-05-15-redis-cache-layer-2.md`).
   - Change the slug for the new file.

Wait for the user's choice before writing.

### Structure of the log

The decision log is written in the language the user is using during the session.

**Fidelity rule: record what the user actually said, do not upgrade it.** If a decision rests on a user's belief, write it as a belief, not as established fact. "The author believes Astro serves static pages faster, not benchmarked in this project" is correct. "Astro serves static pages faster" stated as fact is a fabrication the user will later trust. Never editorialize the user's intuitions into technical justifications. If a premise was not verified during the session, the log must say so explicitly.

```markdown
# <Short title of the decision>

**Date:** YYYY-MM-DD

---

## Context

What problem is being solved and why now. Relevant prior state of the system.

## Decision

What was decided, concretely. One or two direct sentences.

## Alternatives rejected

- **<Alternative A>:** reason for rejection.
- **<Alternative B>:** reason for rejection.

## Trade-offs accepted

What is being given up by this decision.

## Assumptions

Things assumed without verification, with the condition under which each should be revisited.

## Sub-decisions

Light treatment, three lines per sub-decision:

### <Sub-decision title>

- **Decision:** ...
- **Why:** ...
- **Trade-off:** ...

## Weak points

What remains weak at the end of the session. Not a summary of mid-session criticism.

## Deferred

Decisions explicitly postponed, with the condition under which to revisit each. Also list sub-decisions that deserve their own future session.

## Revisit condition

The single concrete condition under which this whole decision should be reopened. This is distinct from the per-assumption revisit conditions above: it is the top-level trigger for reconsidering the decision as a whole. State it as an observable event (e.g., "a collaborator who cannot use git appears"), not as a vague metric (e.g., "if the project grows"). If the user offers a vague condition, push back until it names a concrete pain point.

## References

- Key files involved.
- Related PRs, issues, or commits if any.
- External links.
- Internal documentation.
```

## What this skill is NOT

- Not brainstorming. Do not generate options for the user unless they explicitly say "I don't know" and step 4 applies.
- Not validation. Do not confirm the plan is good. Find what is wrong.
- Not infinite. Apply the depth and scope limits strictly.
- Not a coding assistant during the session. Code comes after the decision is logged.
