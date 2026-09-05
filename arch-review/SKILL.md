---
name: arch-review
description: Survey the codebase for high-leverage architecture improvements — misplaced state, awkward file structure, blurry module boundaries — and report the few changes worth making. Use when the user asks whether the architecture or file organization could be better, where some state should live (local vs global/store), or wants refactoring direction beyond a single diff.
disable-model-invocation: true
---

`/arch-review [path or focus] → 3 survey angles → ranked shortlist → authorized next step`

You are reviewing placement, not prose: where state, code, and responsibility
live, and who depends on whom.

**Scope test** — friction is architectural only if it would survive a perfect
in-place rewrite of every file involved: hand-synced state still needs
syncing, a drilled value still crosses layers that never read it, an import
cycle still cycles. Friction that a good local rewrite would dissolve belongs
to `/simplify` (or `/code-review`, if it's a bug) — not a finding here.

The deliverable is a short ranked report. For a review-only request, end with
a recommendation. If the user has also authorized implementation, continue
with the changes covered by that authorization; an earlier instruction counts
without another selection step.

A single concrete placement question ("should X live in the store?") needs an
answer, not a survey — judge it directly against the scope test and ground
rules and skip the phases below; they are for open-ended reviews.

## Ground rules

- **Evidence over taste.** Every finding must point at concrete friction: two
  copies of the same state synced by hand, a value threaded through five
  layers that never read it, one concept whose every change touches four
  directories. If the only justification is "cleaner" or a naming preference,
  drop it — restructuring churns git history and breaks open branches, so it
  has to buy something real.
- **Respect house conventions.** Judge against how this codebase already
  organizes things, not a textbook ideal. Flag inconsistency between areas,
  not deviation from your favorite layout.
- **Small and separable.** Each finding must be adoptable on its own. Never
  propose a big-bang restructure; if a change is genuinely large, report only
  its first standalone step.

## Phase 0 — Scope

Scope is the whole repo, or the path passed as argument. An argument naming a
concern rather than a path (a subsystem, a pain point) keeps repo-wide scope —
run the churn command over the whole repo and carry the focus into every
agent's brief. Weight attention by churn — architecture debt only costs where
code actually changes:

    git log --since=6.months --format= --name-only -- <scope> | sort | uniq -c | sort -rn | head -30

Skim that list and the directory tree to orient, then fan out.

## Phase 1 — Survey (3 independent angles)

Use available delegation tools for **3 read-only survey agents**, one per
angle, running concurrently within the environment's capacity or in batches
when needed. If delegation is unavailable, cover all three angles directly.
Give each agent the scope, the churn list, the scope test
and ground rules quoted verbatim (an agent can't apply a rule it never sees),
and one angle below. Each returns findings as: where (files), friction (the
concrete evidence), proposed change, and effort (S: one file · M: one module ·
L: crosses modules).

### State placement

Misplaced state, in both directions:

- **Lift up** when the same state is duplicated and hand-synced across
  siblings, or a value is drilled through layers that never read it — a
  shared store/context/module is the fix.
- **Push down** when a global store or context entry has exactly one
  consumer — that "global" is a local with extra ceremony.
- **Derive, don't store** when stored state is recomputable from other state
  and code exists only to keep the two in sync.

### File & module structure

- One concept scattered so a routine change touches several directories.
- Two organizing schemes coexisting (half by-feature, half by-layer) — flag
  the inconsistency, not whichever scheme you'd prefer.
- Dumping grounds (`utils`, `helpers`, `common`) that accrete unrelated code
  and are imported from everywhere.
- A file doing several unrelated jobs, or a hub file everyone edits and
  conflicts in.

### Boundaries & dependencies

- Import cycles, and modules that import — or are imported by — everything.
- Callers reaching through a module into its internals, so its representation
  can't change without a repo-wide sweep.
- Pass-through layers as wide as what they wrap: an interface that adds a hop
  but hides nothing.
- Modules testable only by mocking their neighbors — the test file's setup
  work is the evidence that the boundary exposes wiring, not behavior.

## Phase 2 — Report and recommend

Complete all three angles and wait for any delegated work, then dedup findings
that point at the same mechanism. Keep only the few worth acting on — a
shortlist, not an inventory.
Evidence you cannot reproduce is not evidence: open each survivor's cited
files, confirm the friction is really there, drop what does not reproduce, and
re-rate effort from what the files show. Rank the rest by
friction removed per unit of effort, where high-churn locations outrank
clean-but-frozen corners. Report in conversation:

    ## Architecture review — <scope>

    1. **<title>** (effort S/M/L)
       - Where: <files/dirs>
       - Friction: <the evidence>
       - Change: <the proposal and its first concrete step>

Close the report with which finding you would start with and why. For an
authorized L-effort change, capture the agreed scope with `/to-spec` before
implementing, unless an existing spec already covers it.
