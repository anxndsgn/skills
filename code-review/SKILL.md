---
name: code-review
description: Review code changes for real bugs as a careful senior engineer, then report findings as a structured list. Automatically picks the review scope — working diff, PR, branch, or file — from the user's words and repo state. Use for "review my changes", "check this diff for bugs", "review this PR", "code review this branch".
---

# Code review

`minimal prompt → depth matched to the change → findings with explicit evidence status, once`

You are reviewing code changes for real bugs. Decide the review scope yourself
from the user's words and the repo state — do not ask which scope they meant.

Explicit signals win — a PR number or URL, a branch name, a file path, or what
this conversation has been working on each name their own scope. With no
signal, fall back deterministically: uncommitted changes mean the working tree
plus the branch's committed work; otherwise the branch diff against its merge
base (`@{upstream}`, else `main`). When more than one scope plausibly applies,
include both rather than guessing narrow. Validate the scope before any
fan-out — the ref must resolve and the diff must be non-empty; a bad ref or an
empty diff stops the review here, not inside a sub-agent. When receiving a
handoff, validate its supplied scope and snapshot against the current target;
recover missing context from available artifacts before asking the user.
Open the review by stating scope, depth, and why in one line. Identify the base
and head commits and any included uncommitted snapshot in the report and briefs;
include a retrievable diff or snapshot covering relevant staged, unstaged, and
untracked contents, not just HEAD. Reuse earlier review evidence only where
behavior and dependencies are unchanged; refresh affected evidence and report
unidentified snapshots as limitations.

Review the diff as a careful senior engineer would: read every hunk, open the
surrounding files for context as needed, and hunt for correctness issues —
inverted conditions, missing `await`, dropped guards, broken callers, races,
and their kin. Prefer real failure modes over style; every finding needs a
concrete scenario in which the code misbehaves.

When the review has fully converged (see Convergence and close), report findings
most-severe first. If ReportFindings or an equivalent reporting tool is available,
submit one batch using its actual schema. If that schema supports `level` with
these values, report the depth actually run: light → `low`, standard → `medium`
or `high` per bias, thorough → `xhigh`. Use the tool only for findings its schema
can represent faithfully; report any unsupported evidence status in prose.
Always include a final reply with one entry per finding:
`file:line — evidence status — summary`, adding the missing evidence for a
`PLAUSIBLE` risk. Without a reporting tool, this reply is the deliverable.

## Calibrating review depth

Choose how deep to review from the change itself — do not ask, and do not key
the choice off any global setting. The shapes form a spectrum:

- **Light** — one careful pass: inline when the diff came from elsewhere, in a
  single fresh-context sub-agent when this session wrote the code — a reviewer
  that just wrote the diff reads its own intent instead of what the code does.
  For mechanical renames, formatting, docs-only or config-only changes, and
  other diffs whose failure modes are shallow.
- **Standard** — check relevant failure modes (logic, callers and contracts,
  state and lifecycle, error paths), then dedup and verify candidates. One
  reviewer can cover a compact diff; delegate independent angles when separate
  contexts would improve coverage. For typical bug fixes and small refactors.
- **Thorough** — more finder angles and candidates per angle, plus a final sweep
  over removed code blocks. For complete features, changes touching concurrency,
  auth, migrations, money, or public interfaces, and anything with a wide blast
  radius.

The user's words are the strongest signal and override the rest — "quick look"
means light, "thorough audit" means thorough; an explicitly set effort level
may nudge the choice one step, but the change itself is the primary input.

Bias follows risk the same way: for high-stakes changes prefer **recall** —
report confirmed defects and evidence-backed plausible risks, with their status
explicit; for routine changes prefer **precision** — report confirmed defects.
Use the evidence standards below for both modes. Group long reports by mechanism
rather than truncating them. Run independent work concurrently within the
environment's capacity, batching when needed. If delegation is unavailable,
perform the selected depth's finder angles, verification, and any sweep directly.

Pipeline rules when fanning out (standard and thorough):

- **Phase 1 — find.** Run the finder angles independently. Each finder prompt
  is self-contained — the diff, the scope, and anything it must judge against,
  included explicitly; use fresh context when supported rather than relying on
  inherited conversation. Each returns its candidates without a fixed quota.
  A candidate has four fields — `file`, `line`, a
  one-line `summary`, and a concrete `failure_scenario` — not an essay. Pass
  through every candidate with a nameable failure scenario — finders that
  silently drop half-believed candidates bypass the verify step and are the
  dominant cause of misses.
- **Conventions need a failure mode.** A documented convention or surrounding
  pattern can reveal a broken assumption, but deviation alone is not a finding.
  Cite the source and show how the changed code causes a concrete failure.
- **Phase 2 — verify.** Dedup candidates pointing at the same line and mechanism,
  keeping the one with the most concrete failure scenario. Run one verifier per
  remaining candidate with the diff, the relevant files, and the candidate; it
  returns a verdict, supporting evidence, and any missing evidence. Apply the
  evidence standards below; a candidate surviving a failed attempt to refute it
  is not sufficient evidence on its own.
- **Sweep (thorough).** A final pass focused on removed code blocks. Output
  no findings only after that pass leaves no reportable defect or risk under
  the selected bias.

### Evidence standards

- **CONFIRMED** — a reproduction, targeted test, or traced code path establishes
  the trigger, violated behavior, and consequence. Cite the evidence.
- **PLAUSIBLE** — specific code evidence supports a concrete failure scenario,
  but a named runtime condition or external contract remains unverified. State
  what is missing and how to verify it; report it as an unconfirmed risk only
  in recall mode.
- **REFUTED** — code, contract, or execution evidence disproves the proposed
  failure scenario. Cite the disproof and omit the candidate from findings.

If evidence is insufficient for any verdict, record the candidate as unresolved
with the missing access or check in the review limitations. Tool failures or
unavailable evidence do not turn a candidate into a confirmed or plausible issue.

## Convergence and close

Find everything first, verify everything second, report everything once.

- **One batch, one close.** Findings are delivered as a single batch after the
  entire find → verify pipeline has drained. Never state a verdict — least of
  all "no findings" — while any finder, verifier, or sweep is still
  outstanding.
- **State the tally.** Open the final verdict with the completion count
  (`finders 4/4, verifiers 6/6 completed`) so an undrained pipeline is visible
  rather than silently passing as done. For inline work, report passes and
  candidates checked instead of agent counts. Report blocked checks and
  unresolved candidates as limitations, without implying a clean review.
- **Carry the evidence.** Use the final report as the handoff: include checks
  or inspections and their results, linked to the reviewed snapshot, along
  with unresolved risks and any next action the user authorized. Findings do
  not themselves authorize fixes; link existing evidence rather than copying logs.
- **Fixes get a delta re-review.** If the session goes on to fix the findings,
  finish and report the full batch first. After fixing, re-review only the fix
  hunks and the invariants they touch — do not rescan the whole branch each
  round.
- **Validate affected behavior.** While verifying or fixing a candidate, run
  the tests that confirm or refute it. After fixes, run the required checks
  appropriate to their scope. Reuse results that still apply; repeat or broaden
  checks, including the full suite, when new changes, failures, or unresolved
  concerns warrant it. Passing checks need no repetition without such a reason.
- **Stop when dry.** When a pass yields no new confirmed finding with a
  concrete failure scenario, close. Do not drift into speculative hardening or
  ever-wider test matrices.

## Scope boundaries

- Correctness first. Style, formatting, and anything a linter, typechecker, or
  compiler catches is out of scope — CI runs separately.
- Pre-existing issues on lines the diff did not touch are out of scope.
- Spec conformance is out of scope. Do not walk acceptance criteria against
  the implementation — an unmet criterion, a missing test for it, or a stale
  spec status line belongs to `/spec-verify`, not here. A spec deviation is a
  finding only when it is also a concrete failure the code exhibits: wrong
  output, a broken caller, a self-contradictory public contract.
- Reuse, simplification, efficiency, and placement concerns belong here only
  when they explain a concrete failure in changed code, such as duplicated
  implementations returning inconsistent results. Quality improvements without
  such a failure belong to `/simplify`; avoid repeating its cleanup review.
