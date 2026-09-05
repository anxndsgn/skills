---
name: spec-verify
description: Verify a completed implementation against a spec's Acceptance Criteria and Verification Plan. Use after implementation or any later code change.
disable-model-invocation: true
---

Verify the final implementation against the named spec. Do not edit code or reinterpret requirements.

Identify the spec revision and inspected code snapshot in the report: base and head commits plus any relevant staged, unstaged, or untracked contents. When receiving a handoff, confirm that its evidence covers this target. Reuse evidence for unchanged behavior and dependencies; refresh affected evidence after changes. An unidentified snapshot leaves its evidence unverified.

Read the spec, final diff, and relevant code. For every Acceptance Criterion, inspect or run the evidence required by the Verification Plan; never treat the implementation summary as proof. Also flag missing, partial, and out-of-scope behavior.

Judge whether the evidence demonstrates the criterion, not merely whether a listed check passed. Mark a criterion `unverified` when available evidence establishes neither compliance nor failure. Report implementation mismatches against the agreed requirement; propose any requirement change separately with its rationale rather than changing the criterion to fit the code.

Report:

- One row per criterion: `passed`, `failed`, or `unverified`, with concrete evidence.
- Checks run and their results.
- Scope deviations.
- A final verdict. The spec is satisfied only when no criterion is failed or unverified.

Do not perform general code review, refactor, or change the spec.
