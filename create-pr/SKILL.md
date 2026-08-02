---
name: create-pr
description: >-
  Create a GitHub pull request for the current branch with a conventional-commit
  title and this repo's PR body template (What changed / Why / User impact /
  Validation). Use whenever the user asks to open, create, submit, or send a PR,
  or to "push this up for review" — including right after finishing a feature or
  fix, even if they don't say "pull request" explicitly.
---

# Create PR

Open a pull request against `main` using `gh`, matching the style of this
repo's recent PRs. Look at `gh pr list --state merged --limit 5` if you want
concrete examples.

## Workflow

1. **Get off `main` if needed.** If the work sits on `main`, create a branch
   named `type/kebab-topic` (e.g. `feat/email-sign-in`) and move the
   commits there.
2. **Commit anything uncommitted** using the conventional-commit skill's rules:
   `type(scope): summary`, one commit per logical change.
3. **Review the full branch diff** — `git log` and `git diff main...HEAD` — so
   the body describes everything the PR contains, not just the last commit.
4. **Run validation** before pushing: the checks this repo declares (CLAUDE.md,
   package scripts, CI config). If `/spec-verify` already produced evidence for
   this work, cite that instead of re-running the same checks. Report real
   results in the body; if something fails, stop and say so instead of opening
   the PR.
5. **Push** with `git push -u origin <branch>`.
6. **Create the PR** with `gh pr create`, passing the body via a heredoc or
   `--body-file` so markdown survives intact.

## Title

Conventional-commit style, matching the branch's main commit:
`feat(auth): add email sign-in`. Omit the scope when the change spans the
whole package (`feat: implement skill enablement`).

## Body template

```markdown
## What changed

- bullet list of the concrete changes, grouped by concern

## Why

One or two short paragraphs: the problem before this change, and how this
change resolves it. If the PR implements a spec, link it here — e.g.
"Implements [`docs/spec/<name>.md`](link)" — and the ADR if one exists.

## User impact

- what users/teams can now do (omit this section for pure refactors)

## Validation

- one bullet per check run, with its real result (e.g. `<test command>` — <N> passed)
- or cite the `/spec-verify` report when one covers this work
```

Keep the body honest and concrete: bullets describe what actually changed,
"Why" explains intent rather than restating the diff, and the Validation
numbers come from runs you actually performed. For refactor-only PRs, replace
"User impact" with a note that there are no behavior changes. Don't pad — a
small PR deserves a short body.
