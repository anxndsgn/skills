---
name: code-review
description: Review code changes as a careful senior engineer for correctness defects, quality cost (reuse, simplification, efficiency, altitude), and design cost (API contracts, component responsibility, readability), then report findings as a structured list; with `fix`, apply the safe ones; `simplify`, not `/simplify` command, runs the quality angle alone and fixes. Automatically picks the review scope — working diff, PR, branch, or file — from the user's words and repo state. Use for "review my changes", "check this diff for bugs", "review this PR", "code review this branch", "simplify my changes", "clean this up".
---

# Code review

`minimal prompt → depth matched to the change → findings with explicit evidence status, once → fixes on request`

You are reviewing code changes for the cost they add: defects, quality debt,
and design debt. Decide the review scope yourself from the user's words and the
repo state — do not ask which scope they meant.

Explicit signals win — a PR number or URL, a branch name, a file path, or what
this conversation has been working on each name their own scope. With no
signal, fall back deterministically: uncommitted changes mean the working tree
plus the branch's committed work; otherwise the branch diff against its merge
base (`@{upstream}`, else `main`). When more than one scope plausibly applies,
include both rather than guessing narrow. Validate the scope before any
fan-out — the ref must resolve and the diff must be non-empty; a bad ref or an
empty diff stops the review here, not inside a sub-agent. Open the review by
stating scope, depth, mode (report or fix), and why in one line.

Arguments select angles and mode. Naming one or more finder angles
(`correctness`, `quality`, `design`, `conventions`) runs only those; `fix`
turns on fix mode; `simplify` is shorthand for `quality fix`. Unnamed, all
four angles run. The scope signals above still apply alongside them.

Review the diff as a careful senior engineer would: read every hunk, open the
surrounding files for context as needed. **Every finding names a concrete
cost and who pays it.** For correctness that is a failure scenario — inputs
and state under which the code misbehaves. For quality and design it is the
helper re-implemented, the work repeated, the caller that must do what the
API should, the reader that must trace three files to follow one behavior.
"Cleaner" with no payer is not a finding.

When the review has fully converged (see Convergence and close), report findings
most-severe first, defects before quality and design. If ReportFindings or an
equivalent reporting tool is available, submit one batch using its actual
schema, with `category` naming the angle. If that schema supports `level` with
these values, report the depth actually run: light → `low`, standard → `medium`
or `high` per bias, thorough → `xhigh`. Use the tool only for findings its schema
can represent faithfully; report any unsupported evidence status in prose.
Always include a final reply with one entry per finding:
`file:line — angle — evidence status — summary`, adding the missing evidence
for a `PLAUSIBLE` risk. Without a reporting tool, this reply is the deliverable.

## Finder angles

- **Correctness** — inverted conditions, missing `await`, dropped guards,
  broken callers, races, and their kin. Prefer real failure modes over style.
- **Quality** — four lenses on the changed code, each naming the cheaper or
  simpler form. _Reuse_: code that re-implements something the codebase
  already has; grep shared and utility modules and files adjacent to the
  change, and name the existing helper. _Simplification_: redundant or
  derivable state, copy-paste with slight variation, deep nesting, dead code
  left behind. _Efficiency_: repeated computation or I/O, independent
  operations run sequentially, blocking work added to startup or hot paths,
  long-lived objects built from closures that keep a large enclosing scope
  alive. _Altitude_: special cases layered on shared infrastructure where
  generalizing the underlying mechanism would do.
- **Design** — the shape of what the diff adds: the contract a caller sees
  (parameters, return, errors, naming), the responsibility a component
  carries, and what a reader must hold in their head to follow it. Judge
  against how the surrounding code already shapes such things, not a
  textbook. Scope test: a design finding here must be fixable by rewriting
  the files the diff touches; friction that would survive a perfect in-place
  rewrite of those files is architecture and belongs to `/arch-review`.
- **Conventions** — reports only what it can attribute: a documented coding
  standard (`CLAUDE.md`, `CONTRIBUTING.md`) or the dominant pattern in the
  surrounding code, named in the finding. Specs and acceptance criteria are
  not convention sources — they state what to build, not how code is written
  here. In a repo that documents nothing, prevailing code is the standard;
  where neither source exists, the angle stays silent — generic taste is not
  a convention, and the repo's own consistent practice overrides it.

## Calibrating review depth

Choose how deep to review from the change itself — do not ask, and do not key
the choice off any global setting. The shapes form a spectrum:

- **Light** — one careful correctness pass: inline when the diff came from
  elsewhere, in a single fresh-context sub-agent when this session wrote the
  code — a reviewer that just wrote the diff reads its own intent instead of
  what the code does. For mechanical renames, formatting, docs-only or
  config-only changes, and other diffs whose failure modes are shallow.
- **Standard** — a fan-out pipeline via available delegation tools: the
  selected finder angles independently → dedup → one verifier per candidate.
  For typical bug fixes, small refactors, and any request that names an
  angle, `fix`, or `simplify`.
- **Thorough** — more candidates per angle, plus a final sweep over removed
  code blocks. For complete features, changes touching concurrency, auth,
  migrations, money, or public interfaces, and anything with a wide blast
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
  is self-contained — the diff, the scope, its angle's definition and the
  cost rule quoted verbatim, and anything it must judge against, included
  explicitly; use fresh context when supported rather than relying on
  inherited conversation. Each returns its candidates without a fixed quota.
  A candidate has four fields — `file`, `line`, a one-line `summary`, and the
  concrete `cost` (a failure scenario, or who pays and what) — not an essay.
  Pass through every candidate with a nameable cost — finders that silently
  drop half-believed candidates bypass the verify step and are the dominant
  cause of misses.
- **Phase 2 — verify.** Dedup candidates pointing at the same line and mechanism,
  keeping the one with the most concrete cost. Run one verifier per
  remaining candidate with the diff, the relevant files, and the candidate; it
  returns a verdict, supporting evidence, and any missing evidence. Apply the
  evidence standards below; a candidate surviving a failed attempt to refute it
  is not sufficient evidence on its own.
- **Sweep (thorough).** A final pass focused on removed code blocks. Output
  no findings only after that pass leaves no reportable defect or risk under
  the selected bias.

### Evidence standards

- **CONFIRMED** — for a defect: a reproduction, targeted test, or traced code
  path establishes the trigger, violated behavior, and consequence. For a
  quality or design cost: the payer and the cost are observable in the code
  as it stands — the named helper exists, the caller does do the work, the
  duplicate is there. Cite the evidence.
- **PLAUSIBLE** — specific code evidence supports a concrete cost, but a
  named runtime condition or external contract remains unverified. State
  what is missing and how to verify it; report it as an unconfirmed risk only
  in recall mode.
- **REFUTED** — code, contract, or execution evidence disproves the proposed
  cost. Cite the disproof and omit the candidate from findings.

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
  concrete cost, close. Do not drift into speculative hardening or
  ever-wider test matrices; if a test file has already outgrown
  maintainability, report "split this file" as a finding instead of appending
  to it.

## Fix mode

`fix` as an argument, or the user asking to apply, simplify, or clean up,
turns the report into a starting point rather than the deliverable. After the
full batch is reported, apply each finding whose fix stays inside the reviewed
diff and preserves intended behavior; skip the rest — a fix that would change
what the code is meant to do, reach well outside the diff, or that you judge a
false positive — and note each skip rather than arguing with it. Then run the
delta re-review and checks above, and close with what was fixed and what was
skipped.

## Scope boundaries

- Anything a linter, typechecker, or compiler catches is out of scope — CI
  runs separately.
- Pre-existing issues on lines the diff did not touch are out of scope. All
  four angles apply to the _changed_ code only; the reuse lens may name an
  existing helper elsewhere, but the finding lands on the new duplicate.
- Spec conformance is out of scope. Do not walk acceptance criteria against
  the implementation — an unmet criterion, a missing test for it, or a stale
  spec status line belongs to `/spec-verify`, not here. A spec deviation is a
  finding only when it is also a concrete failure the code exhibits: wrong
  output, a broken caller, a self-contradictory public contract.
- Placement across the repo — where state should live, module boundaries,
  file structure — belongs to `/arch-review`; see the design angle's scope
  test.
