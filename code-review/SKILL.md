---
name: code-review
description: Review the working diff (or a named PR / branch / file) for real bugs as a careful senior engineer, then report findings as a structured list. Use for "review my changes", "check this diff for bugs", "code review this branch".
---

# Code review

`minimal prompt → single careful diff pass → ≤15 findings`

You are reviewing a pull request for real bugs. Run `git diff @{upstream}...HEAD`
(or `git diff main...HEAD` / `git diff HEAD~1` if there's no upstream) to get the
unified diff under review. If there are uncommitted changes, or the range diff is
empty, also run `git diff HEAD` and include the working-tree changes in scope —
the review often runs before the commit. If a PR number, branch name, or file path
was passed as an argument, review that target instead. Treat this diff as the
review scope.

Review the diff as a careful senior engineer would: read every hunk, open the
surrounding files for context as needed (Read, Grep, git log/blame/show), and hunt
for correctness issues — wrong or inverted conditions, off-by-one, null/undefined
dereference, missing `await`, dropped error handling, removed guards or
validations, broken callers of changed functions, races. Prefer real failure modes
over style; every finding needs a concrete scenario in which the code misbehaves.

When you are done, submit at most 15 findings via the ReportFindings tool, filling
its fields as defined — for each: the file path and start line, a severity, and a
comment that states the issue and the concrete scenario in which the code
misbehaves. Quality over quantity: include everything you genuinely believe is a
real issue, and nothing you don't.

After the tool call, also restate the findings in your final reply — one line each,
`file:line — summary` — so they stay visible in sessions that do not render tool
output.

## ReportFindings payload

Top level: `level` (`low` | `medium` | `high` | `xhigh` | `max`), `findings`
(max 32, most-severe first, empty array if nothing survived verification).

Each finding:

| field              | required | meaning                                                                                      |
| ------------------ | -------- | -------------------------------------------------------------------------------------------- |
| `file`             | yes      | repo-relative path                                                                           |
| `line`             | no       | 1-indexed anchor line                                                                        |
| `summary`          | yes      | one-sentence statement of the defect                                                         |
| `short_summary`    | no       | ≤60 chars, the claim alone — no rationale or consequence                                     |
| `failure_scenario` | yes      | concrete inputs/state → wrong output/crash                                                   |
| `category`         | no       | kebab-case slug, e.g. `correctness`, `simplification`, `efficiency`, `test-coverage`         |
| `verdict`          | no       | `CONFIRMED` \| `PLAUSIBLE` — set only when a verify pass ran                                 |
| `outcome`          | no       | `fixed` \| `skipped` \| `no_change_needed` — set only when re-reporting after applying fixes |

Call the tool once. Do not also print the findings as a text list in place of the
tool call (the one-line restatement above is in addition to it, not instead).

## Effort scaling

The prompt above is the **low-effort** shape: one careful inline pass, no
sub-agents. At higher reasoning effort the same review runs as a fan-out pipeline
via the Agent tool. If the Agent tool is unavailable, degrade to a single inline
pass at the same finding cap rather than erroring.

| effort      | shape                                                                                               | bias                                                           | cap |
| ----------- | --------------------------------------------------------------------------------------------------- | -------------------------------------------------------------- | --- |
| low         | single careful diff pass, inline                                                                    | balanced                                                       | 15  |
| medium      | 3 correctness + 3 cleanup + 1 altitude + 1 conventions angle × 6 candidates → 1-vote verify         | **precision** — every finding is one a maintainer would act on | 8   |
| high        | same 8 angles × 6 candidates → 1-vote verify, recall-biased                                         | **recall** — catching real bugs beats avoiding false positives | 10  |
| xhigh / max | 5 correctness + 3 cleanup + 1 altitude + 1 conventions angle × 8 candidates → 1-vote verify → sweep | **recall** — a missed bug ships                                | 15  |

Pipeline rules at medium and above:

- **Phase 1 — find.** Run the finder angles independently. Each surfaces up to N
  candidates with `file`, `line`, a one-line `summary`, and a concrete
  `failure_scenario`. Pass through every candidate with a nameable failure
  scenario — finders that silently drop half-believed candidates bypass the verify
  step and are the dominant cause of misses. Don't let one angle's conclusions
  suppress another's; if two angles flag the same line for different reasons, keep
  both into verification.
- **Phase 2 — verify.** Dedup candidates pointing at the same line and mechanism,
  keeping the one with the most concrete failure scenario. Run one verifier per
  remaining candidate with the diff, the relevant files, and the candidate; it
  returns a three-state verdict. Keep `CONFIRMED` and `PLAUSIBLE`. In recall mode
  a single non-REFUTED vote carries the finding — do not drop on uncertainty.
- **Sweep (xhigh/max).** A final pass focused on removed code blocks. Output
  `(none)` only if the diff is trivially correct after that pass.

## Scope boundaries

- Correctness first. Style, formatting, and anything a linter, typechecker, or
  compiler catches is out of scope — CI runs separately.
- Pre-existing issues on lines the diff did not touch are out of scope.
- Cleanup angles (reuse, simplification, efficiency, altitude) apply to the
  _changed_ code only. For a quality-only pass that also applies the fixes, use
  `/simplify` instead.
