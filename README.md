# Personal Agent Skills

Small, composable skills for aligning intent, implementing from a stable spec, and proving the result. Each skill has one responsibility; chain only the ones the task needs.

<img alt="skills loop flow" src="https://github.com/user-attachments/assets/5c8f8530-7eaf-4750-a05b-c1638e6d57e1" />

## Discovery Skills

Upstream of any spec — for when the work is still an idea.

| Skill                 | Responsibility                                                                            |
| --------------------- | ----------------------------------------------------------------------------------------- |
| `/talk-about-an-idea` | Develop a fuzzy product idea in conversation until it is crisp — or honestly dead.        |
| `/research`           | Chase a question to wherever the answer is observable first-hand, with cited evidence.    |
| `/debate`             | Two agents build the strongest evidenced case for and against an idea; rule on the whole. |

Use discovery skills for the uncertainty that remains: conversation develops the idea, research settles factual questions, and debate tests a consequential go/no-go decision. Carry settled decisions forward rather than repeating discovery before writing a spec.

## Core Skills

| Skill          | Responsibility                                                                                 |
| -------------- | ---------------------------------------------------------------------------------------------- |
| `/grill-me`    | Surface unresolved human decisions in rounds, one frontier of the design tree at a time.       |
| `/to-spec`     | Turn the agreed context into a task contract with Acceptance Criteria and a Verification Plan. |
| `/spec-review` | Challenge a spec before implementation for necessity, completeness, and feasibility.           |
| `/spec-verify` | Verify the final implementation against every Acceptance Criterion using concrete evidence.    |
| `/code-review` | Find material correctness defects and hidden engineering risks in the changed code.            |
| `/simplify`    | Reduce unnecessary complexity without changing intended behavior. Claude code built-in skill.  |
| `/arch-review` | Survey the codebase for the few architecture changes worth making — state, structure, seams.   |

Implementation is intentionally not a skill:

```text
Implement @spec-file and complete its Verification Plan.
```

## Choosing the Next Step

Choose the step that resolves the remaining uncertainty or missing evidence. Each skill can be used independently; a completed step does not require every later skill.

| What is missing                                                | Next step                             |
| -------------------------------------------------------------- | ------------------------------------- |
| A clear problem or product idea                                | `/talk-about-an-idea`                 |
| A user decision about goals or tradeoffs                       | `/grill-me`, focused on that decision |
| An external fact                                               | `/research`                           |
| Evidence for a consequential go/no-go decision                 | `/debate`                             |
| Evidence of technical feasibility                              | A bounded experiment or prototype     |
| An executable record of agreed decisions                       | `/to-spec`                            |
| Confidence in a spec's necessity, completeness, or feasibility | `/spec-review`                        |
| Simpler changed code without behavior changes                  | `/simplify`                           |
| Confidence in complex or risky code behavior                   | `/code-review`                        |
| Proof that the implementation meets the spec                   | `/spec-verify`                        |
| Evidence about state placement or module boundaries            | `/arch-review`                        |

A technical experiment answers one concrete question with the smallest useful probe, an observable success condition, and a clear stopping point. Record the result, its limitations, and the decision it supports in the existing discussion or spec. Treat prototype code as exploratory until deliberately adopted and validated for production.

## Example Paths

Agreed feature with a useful spec:

```text
/to-spec → commit spec → implement → /spec-verify
```

Feature with one unresolved tradeoff:

```text
/grill-me (open decision) → /to-spec → commit spec → implement → /spec-verify
```

Add `/spec-review` when the spec warrants scrutiny. If it finds one unresolved error state, settle that decision and revise the affected section; reopen other decisions only when new evidence invalidates them. A feasibility concern may need an experiment before further specification.

Commit the spec before implementing: that commit is the diff baseline `/spec-verify` checks against. After implementing, run `/simplify` first (only when the implementation shows clear complexity), since it changes code; then `/code-review` (for substantial or high-risk changes) and `/spec-verify`, in parallel when available capacity permits — both are read-only. After fixes, re-review the changed behavior and re-verify affected acceptance criteria. Reuse evidence that still applies to the final implementation; repeat or broaden checks when new changes, failures, or unresolved concerns invalidate it. Finish when required checks pass and every acceptance criterion has current evidence, or report the specific blocker and unverified criteria.

When `/spec-verify` or `/code-review` finds something the spec missed, distinguish a missed application of an existing rule from a recurring gap in the workflow. Use real task outcomes to decide whether a reusable rule needs changing: late decisions, repeated checks, review findings that led to useful fixes, and unnecessary user interruptions. Keep observations in existing task records; one incident does not automatically require another skill rule.

Tiny changes do not need this pipeline: describe the change, implement it, run the smallest relevant check, and inspect the diff.

## Handoffs

When changing steps, sessions, or agents, use the [handoff contract](references/handoff.md) to carry scope, decisions, code snapshot, and applicable evidence in the existing work record. Parallel checks must identify the snapshot they inspected; edits made afterward require refreshing the affected evidence.

## Utilities

| Skill                  | Responsibility                                                        |
| ---------------------- | --------------------------------------------------------------------- |
| `/conventional-commit` | Group and commit changes with focused conventional commits.           |
| `/create-pr`           | Push the current work through the repository's pull-request workflow. |

## Acknowledgements

Inspired by [Matt Pocock's skills](https://github.com/mattpocock/skills). Thanks for open-sourcing the concise, composable approach that helped shape this workflow.
