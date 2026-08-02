---
name: code-review
description: Review changed code for correctness and hidden engineering risks. Use after substantial, cross-cutting, or high-risk implementation.
disable-model-invocation: true
---

Review the user-named target; otherwise inspect the branch diff plus uncommitted changes. Read relevant repository rules and enough surrounding code to understand the change.

Look only for material defects: incorrect edge cases, error handling, security or permissions, data consistency, concurrency, performance, regressions, missing tests, architecture violations, and unintended scope expansion.

Report findings only; do not edit code. Rank them by severity and give each an exact file and line, concrete impact, supporting evidence, and the smallest credible fix. Omit tool-enforced style issues and speculative preferences. If there are no findings, say so.

Do not verify spec completeness or simplify working code; those belong to `/spec-verify` and `/simplify`.
