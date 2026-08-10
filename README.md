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

They chain naturally: `/talk-about-an-idea` surfaces open questions, `/research` settles them, `/debate` gives a big idea an adversarial go/no-go, and a surviving idea flows into `/grill-me → /to-spec`.

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

## Common Chains

Clear, small feature:

```text
/to-spec → implement → /spec-verify
```

Feature needing alignment:

```text
/grill-me → /to-spec → implement → /spec-verify
```

Full feature:

```text
/grill-me → /to-spec → /spec-review
→ /grill-me → /to-spec @spec-file
→ implement → /spec-verify
```

Add `/code-review` for substantial or high-risk changes. Add `/simplify` only when the implementation shows clear complexity. After either changes code, run `/spec-verify` again.

Tiny changes do not need this pipeline: describe the change, implement it, run the smallest relevant check, and inspect the diff.

## Utilities

| Skill                  | Responsibility                                                        |
| ---------------------- | --------------------------------------------------------------------- |
| `/conventional-commit` | Group and commit changes with focused conventional commits.           |
| `/create-pr`           | Push the current work through the repository's pull-request workflow. |

## Acknowledgements

Inspired by [Matt Pocock's skills](https://github.com/mattpocock/skills). Thanks for open-sourcing the concise, composable approach that helped shape this workflow.
