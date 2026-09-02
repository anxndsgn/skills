---
name: to-spec
description: Turn the current conversation into a spec — no interview, just synthesis of what you've already discussed.
---

This skill takes the current conversation context and codebase understanding and produces a spec (you may know this document as a PRD). Do not interview the user; synthesize what the conversation already established.

## Process

1. Explore the repo to understand the current state of the codebase, if you haven't already. Use the project's established domain vocabulary throughout the spec, and respect any ADRs in the area you're touching. If the repo has an established spec home and house style (e.g. `docs/spec/` with a frontmatter convention and normative prose), write the spec there in that style — adapt this template's sections to it rather than imposing the template's headings.

2. Sketch out the seams at which you're going to test the feature. Existing seams should be preferred to new ones. Use the highest seam possible. If new seams are needed, propose them at the highest point you can. The fewer seams across the codebase, the better - the ideal number is one.

Check with the user that these seams match their expectations.

<spec-template>

## Problem Statement

The problem that the user is facing, from the user's perspective.

## Solution

The solution to the problem, from the user's perspective.

## Behavioral Coverage

How to fill this section depends on where the spec sits in the workflow:

- **Elicitation mode** — the feature has NOT been through a design discussion (no grilling session, no decision log in the conversation). Write a numbered list of user stories broad enough to surface requirements — every actor and situation, not just the happy path:

  1. As an <actor>, I want a <feature>, so that <benefit>

  <user-story-example>
  1. As a mobile bank customer, I want to see balance on my accounts, so that I can make better informed decisions about my spending
  </user-story-example>

- **Synthesis mode** — the decisions already exist earlier in the conversation. Do not restate them as stories; post-hoc stories are decision projections, not requirements, and they add reading weight without adding constraint. Instead do the one thing the story exercise is genuinely good for: an actor × situation sweep for edges no decision has covered yet — empty states, missing configuration, offline, version skew, races, permission and auth failures, idempotent re-runs. Write the result as a numbered list of acceptance criteria: externally observable behaviors a test could assert. Add a story only when it pins down behavior nothing else has fixed.

In either mode, include a numbered `### Acceptance Criteria` subsection of externally observable behaviors a test could assert.

## Implementation Decisions

A list of implementation decisions that were made. This can include:

- The modules that will be built/modified
- The interfaces of those modules that will be modified
- Technical clarifications from the developer
- Architectural decisions
- Schema changes
- API contracts
- Specific interactions

Do not include specific file paths or code snippets; they go stale quickly.

Exception: if a prototype produced a snippet that encodes a decision more precisely than prose can (state machine, reducer, schema, type shape), inline it within the relevant decision and note briefly that it came from a prototype. Trim to the decision-rich parts — not a working demo, just the important bits.

## Verification Plan

For each acceptance criterion, name the smallest credible evidence that would prove it. Include:

- Relevant tests or other checks
- Runtime or manual verification where automation is insufficient
- Applicable typecheck, lint, build, migration, or compatibility checks

Prefer existing test seams and external behavior. Do not invent exact commands or file names that the repository has not confirmed.

## Out of Scope

A description of the things that are out of scope for this spec.

## Further Notes

Any further notes about the feature.

</spec-template>
