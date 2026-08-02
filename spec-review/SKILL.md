---
name: spec-review
description: Review a spec through three lenses — is the feature needed, is the spec complete, does the implementation plan hold up. Use when the user asks to review, critique, or sanity-check a spec or draft spec.
disable-model-invocation: true
---

Review the named spec (or the spec currently under discussion). The deliverable is the review itself — do not edit the spec or write code unless asked afterwards.

Read only what the review needs: the target spec, the charter, CONTEXT.md, and the specs the target names or amends. Do not sweep the whole repo — a review that costs more than the spec is a bad trade.

Work through three lenses, in this order (a feature that fails lens 1 makes lenses 2–3 moot):

1. **Necessity.** Does the problem justify the feature? Test it against the charter's intent and boundaries — where a spec conflicts with the charter, the spec is defective. Ask: could an existing surface (a verb, a plugin, another spec) absorb this instead? What actually breaks for the user if this is never built?

2. **Completeness.** Would two implementers reading only this spec build the same thing? Hunt the underspecified edges: empty and error states, interactions with the specs it touches, concurrency and idempotency where the domain has them, and whether the Verification section pins each promised behavior to something testable. Check that glossary terms are used exactly — a term used loosely is a decision left unmade.

3. **Implementation.** Will the plan survive contact with the codebase? Check it against the architecture invariants in the repo CLAUDE.md; flag hidden migrations, hard-to-reverse choices, and machinery out of proportion to the problem it solves.

Report format — keep it short:

- One verdict line per lens: **sound** / **gaps** / **defective**, with a half-sentence reason.
- Findings as one ranked list, most severe first. Each finding: one sentence naming the defect, the spec section it lives in, and a concrete fix or question. Cite sections, not vibes.
- A clean lens gets one line saying so. No praise padding.

When a finding is really a decision only the user can make, phrase it as a question with your recommended answer, rather than reporting it as a defect.