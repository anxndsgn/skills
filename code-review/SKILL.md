---
name: code-review
description: Review code changes for real bugs as a careful senior engineer, then report findings as a structured list. Automatically picks the review scope — working diff, PR, branch, or file — from the user's words and repo state. Use for "review my changes", "check this diff for bugs", "review this PR", "code review this branch".
---

# Code review

`minimal prompt → depth matched to the change → ≤15 findings`

You are reviewing code changes for real bugs. Decide the review scope yourself
from the user's words and the repo state — do not ask which scope they meant.

Explicit signals win: a PR number or URL means review that PR (`gh pr diff`); a
branch name means that branch against its merge base; a file or directory path
means the changes touching that target. The conversation counts as a signal too —
"review my changes" after editing files in this session means those changes;
"review this PR" when the conversation has been about a specific PR means that PR.

With no explicit signal, fall back deterministically: if there are uncommitted
changes, review the working tree (`git diff HEAD`) plus the branch's committed
work; otherwise review the branch diff `git diff @{upstream}...HEAD` (or
`git diff main...HEAD` / `git diff HEAD~1` if there's no upstream). When more than
one scope plausibly applies, include both rather than guessing narrow, and open
the review by stating in one line which scope you chose and why. Treat this diff
as the review scope.

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

Top level: `level` (`low` | `medium` | `high` | `xhigh` | `max`) — report the
depth you actually ran at (light → `low`, standard → `medium` or `high` per
bias, thorough → `xhigh`) — and `findings` (max 32, most-severe first, empty
array if nothing survived verification).

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

## Calibrating review depth

Choose how deep to review from the change itself — do not ask, and do not key
the choice off any global setting. The shapes form a spectrum:

- **Light** — one careful inline pass, no sub-agents. For mechanical renames,
  formatting, docs-only or config-only changes, and other diffs whose failure
  modes are shallow.
- **Standard** — a fan-out pipeline via the Agent tool: independent finder angles
  (correctness, cleanup, altitude, conventions) → dedup → one verifier per
  candidate. For typical bug fixes and small refactors.
- **Thorough** — more finder angles and candidates per angle, plus a final sweep
  over removed code blocks. For complete features, changes touching concurrency,
  auth, migrations, money, or public interfaces, and anything with a wide blast
  radius.

Signals for the choice: what kind of change it is (feature vs. fix vs.
mechanical), diff size, how many callers the touched code has, and how costly a
missed bug would be. The user's words are the strongest signal and override the
rest — "quick look" means light, "thorough audit" means thorough — and an
explicitly set effort level may nudge the choice one step, but the change itself
is the primary input. State the chosen depth in one line when opening the review.

Bias follows risk the same way: for high-stakes changes prefer **recall** — a
missed bug ships, so keep any finding a verifier could not refute; for routine
changes prefer **precision** — only report what a maintainer would act on. The
cap is 15 findings regardless of depth; it is output discipline, not a depth
knob. If the Agent tool is unavailable, degrade to a single inline pass at the
same cap rather than erroring.

Pipeline rules when fanning out (standard and thorough):

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
- **Sweep (thorough).** A final pass focused on removed code blocks. Output
  `(none)` only if the diff is trivially correct after that pass.

## Scope boundaries

- Correctness first. Style, formatting, and anything a linter, typechecker, or
  compiler catches is out of scope — CI runs separately.
- Pre-existing issues on lines the diff did not touch are out of scope.
- Cleanup angles (reuse, simplification, efficiency, altitude) apply to the
  _changed_ code only. For a quality-only pass that also applies the fixes, use
  `/simplify` instead.
