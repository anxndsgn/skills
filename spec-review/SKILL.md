---
name: spec-review
description: Review a spec through three lenses — is the feature needed, is the spec complete, does the implementation plan hold up. Use when the user asks to review, critique, or sanity-check a spec or draft spec.
---

Review the named spec (or the spec currently under discussion). For a review-only request, deliver the review. If the user has also authorized revisions or implementation, finish the review and continue within that scope; authorization given earlier in the conversation counts.

Read only what the review needs: the target spec, whatever the repo uses to state project intent and constraints (README, CLAUDE.md, ADRs), and the specs the target names or amends. Do not sweep the whole repo — a review that costs more than the spec is a bad trade.

Work through three lenses, in this order (a feature that fails lens 1 makes lenses 2–3 moot):

1. **Necessity.** Does the problem justify the feature? Test it against the project's stated intent and boundaries — where a spec conflicts with them, the spec is defective. Ask: could an existing surface (a verb, a plugin, another spec) absorb this instead? What actually breaks for the user if this is never built?

2. **Completeness.** Would two implementers reading only this spec build the same thing? Hunt the underspecified edges: empty and error states, interactions with the specs it touches, concurrency and idempotency where the domain has them, and whether every Acceptance Criterion is observable and mapped to credible evidence in the Verification Plan. Check that the project's domain terms are used exactly — a term used loosely is a decision left unmade.

3. **Implementation.** Will the plan survive contact with the codebase? Check it against the architecture invariants in the repo CLAUDE.md; flag hidden migrations, hard-to-reverse choices, and machinery out of proportion to the problem it solves.

Report format — keep it short:

- One verdict line per lens: **sound** / **gaps** / **defective**, with a half-sentence reason.
- Findings as one ranked list, most severe first. Each finding: one sentence naming the defect, the spec section it lives in, and a concrete fix or question. Cite sections, not vibes.
- A clean lens gets one line saying so. No praise padding.

When a finding is really a decision only the user can make, phrase it as a question with your recommended answer, rather than reporting it as a defect.

Route follow-up to the affected decision or section. A local gap calls for a local revision; reopen settled decisions only when evidence invalidates them. For a feasibility question that inspection cannot settle, recommend a bounded experiment with a concrete success condition.

When receiving a handoff, confirm the requested scope and spec revision against the current target. In the review report, identify that revision, the evidence behind findings, unresolved decisions, and any authorized next action. Recheck affected findings if the spec or supporting code has changed. Findings alone do not authorize implementation; recover missing context from available artifacts before asking the user.
