---
name: code-review
description: Review code changes for real bugs as a careful senior engineer, then report findings as a structured list. Automatically picks the review scope — working diff, PR, branch, or file — from the user's words and repo state. Use for "review my changes", "check this diff for bugs", "review this PR", "code review this branch".
---

# Code review

`minimal prompt → depth matched to the change → every verified finding, once`

You are reviewing code changes for real bugs. Decide the review scope yourself
from the user's words and the repo state — do not ask which scope they meant.

Explicit signals win — a PR number or URL, a branch name, a file path, or what
this conversation has been working on each name their own scope. With no
signal, fall back deterministically: uncommitted changes mean the working tree
plus the branch's committed work; otherwise the branch diff against its merge
base (`@{upstream}`, else `main`). When more than one scope plausibly applies,
include both rather than guessing narrow. Validate the scope before any
fan-out — the ref must resolve and the diff must be non-empty; a bad ref or an
empty diff stops the review here, not inside a sub-agent. Open the review by
stating scope, depth, and why in one line.

Review the diff as a careful senior engineer would: read every hunk, open the
surrounding files for context as needed, and hunt for correctness issues —
inverted conditions, missing `await`, dropped guards, broken callers, races,
and their kin. Prefer real failure modes over style; every finding needs a
concrete scenario in which the code misbehaves.

When the review has fully converged (see Convergence and close), submit the
findings in a single ReportFindings call — most-severe first, filled as the
tool's schema defines. Report `level` as the depth you actually
ran: light → `low`, standard → `medium` or `high` per bias, thorough →
`xhigh`. Quality over quantity: everything you genuinely believe is a real
issue, and nothing you don't. After the call, restate the findings in your
final reply — one line each, `file:line — summary` — so they stay visible in
sessions that don't render tool output; the restatement is in addition to the
tool call, never instead of it.

## Calibrating review depth

Choose how deep to review from the change itself — do not ask, and do not key
the choice off any global setting. The shapes form a spectrum:

- **Light** — one careful pass: inline when the diff came from elsewhere, in a
  single fresh-context sub-agent when this session wrote the code — a reviewer
  that just wrote the diff reads its own intent instead of what the code does.
  For mechanical renames, formatting, docs-only or config-only changes, and
  other diffs whose failure modes are shallow.
- **Standard** — a fan-out pipeline via the Agent tool: independent finder angles
  (correctness, cleanup, altitude, conventions) → dedup → one verifier per
  candidate. For typical bug fixes and small refactors.
- **Thorough** — more finder angles and candidates per angle, plus a final sweep
  over removed code blocks. For complete features, changes touching concurrency,
  auth, migrations, money, or public interfaces, and anything with a wide blast
  radius.

The user's words are the strongest signal and override the rest — "quick look"
means light, "thorough audit" means thorough; an explicitly set effort level
may nudge the choice one step, but the change itself is the primary input.

Bias follows risk the same way: for high-stakes changes prefer **recall** — a
missed bug ships, so keep any finding a verifier could not refute; for routine
changes prefer **precision** — only report what a maintainer would act on.
Report everything that survived verification; a list that runs long means the
diff is too large for one review — say so and group findings by mechanism
rather than truncating. If the Agent tool is unavailable, degrade to a single
inline pass rather than erroring.

Pipeline rules when fanning out (standard and thorough):

- **Phase 1 — find.** Run the finder angles independently. Each finder prompt
  is self-contained — the diff, the scope, and anything it must judge against,
  pasted in; sub-agents see none of this conversation. Each surfaces up to N
  candidates, and a candidate is exactly four fields — `file`, `line`, a
  one-line `summary`, and a concrete `failure_scenario` — not an essay. Pass
  through every candidate with a nameable failure scenario — finders that
  silently drop half-believed candidates bypass the verify step and are the
  dominant cause of misses.
- **Conventions cite their source.** The conventions angle reports only what it
  can attribute: a documented coding standard (`CLAUDE.md`, `CONTRIBUTING.md`)
  or the dominant pattern in the surrounding code, named in the finding. Specs
  and acceptance criteria are not convention sources — they state what to
  build, not how code is written here. In a repo that documents nothing,
  prevailing code is the standard;
  where neither source exists, the angle stays silent — generic taste is not a
  convention, and the repo's own consistent practice overrides it.
- **Phase 2 — verify.** Dedup candidates pointing at the same line and mechanism,
  keeping the one with the most concrete failure scenario. Run one verifier per
  remaining candidate with the diff, the relevant files, and the candidate; it
  returns a three-state verdict. Keep `CONFIRMED` and `PLAUSIBLE`. In recall mode
  a single non-REFUTED vote carries the finding — do not drop on uncertainty.
- **Sweep (thorough).** A final pass focused on removed code blocks. Output
  `(none)` only if the diff is trivially correct after that pass.

## Convergence and close

Find everything first, verify everything second, report everything once.

- **One batch, one close.** Findings are delivered as a single batch after the
  entire find → verify pipeline has drained. Never state a verdict — least of
  all "no findings" — while any finder, verifier, or sweep is still
  outstanding.
- **State the tally.** Open the final verdict with the completion count
  (`finders 4/4, verifiers 6/6 completed`) so an undrained pipeline is visible
  rather than silently passing as done.
- **Fixes get a delta re-review.** If the session goes on to fix the findings,
  finish and report the full batch first. After fixing, re-review only the fix
  hunks and the invariants they touch — do not rescan the whole branch each
  round.
- **Targeted tests during, full suite once.** While verifying or fixing a
  candidate, run only the tests that confirm or refute it. Run the full suite
  at most once, after the findings list (or fix set) is final — never between
  rounds.
- **Stop when dry.** When a pass yields no new confirmed finding with a
  concrete failure scenario, close. Do not drift into speculative hardening or
  ever-wider test matrices; if a test file has already outgrown
  maintainability, report "split this file" as a finding instead of appending
  to it.

## Scope boundaries

- Correctness first. Style, formatting, and anything a linter, typechecker, or
  compiler catches is out of scope — CI runs separately.
- Pre-existing issues on lines the diff did not touch are out of scope.
- Spec conformance is out of scope. Do not walk acceptance criteria against
  the implementation — an unmet criterion, a missing test for it, or a stale
  spec status line belongs to `/spec-verify`, not here. A spec deviation is a
  finding only when it is also a concrete failure the code exhibits: wrong
  output, a broken caller, a self-contradictory public contract.
- Cleanup angles (reuse, simplification, efficiency, altitude) apply to the
  _changed_ code only. For a quality-only pass that also applies the fixes, use
  `/simplify` instead.
