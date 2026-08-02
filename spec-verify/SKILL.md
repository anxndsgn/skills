---
name: spec-verify
description: Verify a completed implementation against a spec's Acceptance Criteria and Verification Plan. Use after implementation or any later code change.
disable-model-invocation: true
---

Verify the final implementation against the named spec. Do not edit code or reinterpret requirements.

Read the spec, final diff, and relevant code. For every Acceptance Criterion, inspect or run the evidence required by the Verification Plan; never treat the implementation summary as proof. Also flag missing, partial, and out-of-scope behavior.

Report:

- One row per criterion: `passed`, `failed`, or `unverified`, with concrete evidence.
- Checks run and their results.
- Scope deviations.
- A final verdict. The spec is satisfied only when no criterion is failed or unverified.

Do not perform general code review, refactor, or change the spec.
