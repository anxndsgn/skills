---
name: conventional-commit
description: >-
  Create git commits with conventional commit messages and split broad diffs
  into multiple logical commits when needed. Use when the user asks to commit,
  or when a diff must be inspected, grouped into logical commits, staged
  safely, and committed with messages such as `feat(scope): summary` or
  `fix(scope): summary` instead of creating one unfocused commit.
disable-model-invocation: true
---

# Conventional Commit

## Overview

Inspect the current git worktree, group related changes into the smallest reasonable set of commits, and write conventional commit messages that describe intent clearly. Prefer one commit per logical change; when the diff spans multiple concerns, split it into separate commits by scope or change type.

## Workflow

1. Run the repo's formatter first, if it declares one (package scripts, CI config), so formatting noise doesn't land in the commits.
2. Inspect the worktree with `git status --short`, `git diff --stat`, and targeted `git diff` reads.
3. Build a short commit plan before staging anything. For each planned commit, identify:
   - the files or hunks that belong together
   - the conventional commit `type`
   - the optional `scope`
   - the one-line summary
4. Stage one logical group at a time. Prefer path-based staging such as `git add path/to/file`.
5. Verify the staged diff with `git diff --cached --stat` and `git diff --cached`.
6. Create the commit.
7. Repeat for the remaining groups until the intended changes are committed.

## Grouping Rules

- Keep changes together only when they serve the same intent.
- Split commits by subsystem or concern when the diff mixes unrelated work.
- Keep code and tests together when the tests validate that same behavior change.
- Keep generated files with their source change when they are required for a valid commit.
- Separate formatting-only, dependency-only, docs-only, or cleanup-only changes unless they are mechanically required by the main change.
- Prefer multiple small commits over one large mixed commit.

Use these default grouping heuristics when there are many changes:

- `feat` or `fix` changes in the application code
- `refactor` changes that do not alter behavior
- `test` additions or repairs that are independent from feature work
- `docs` updates
- `chore`, `build`, or `ci` changes
- migrations or schema updates, paired with the code that depends on them

If a single file contains unrelated edits and they cannot be separated safely with non-destructive staging, stop and tell the user which file is mixed instead of guessing.

## Conventional Commit Format

Use:

```text
type(scope): summary
```

Rules:

- Use lowercase commit types.
- Keep the summary in imperative mood.
- Keep the subject line concise and without a trailing period.
- Add `!` only for breaking changes.
- Omit `scope` when it does not add clarity.
- Add a body only when rationale, context, or breaking-change guidance matters.

Preferred types:

- `feat`: user-facing behavior or new capability
- `fix`: bug fix
- `refactor`: internal restructuring without behavior change
- `docs`: documentation only
- `test`: test-only changes
- `chore`: maintenance work
- `build`: build tooling or dependencies
- `ci`: automation or pipeline changes
- `perf`: performance improvement
- `style`: formatting-only changes
- `revert`: revert a previous commit

## Staging Guidance

- Prefer `git add <paths...>` when a full file belongs to one commit.
- Use partial staging only when necessary to separate unrelated edits inside the same file.
- Do not stage unrelated local changes that were not part of the requested work.
- Before committing, confirm the staged snapshot matches the planned message.
- After each commit, re-run `git status --short` so the next commit starts from the remaining diff.

## Output Expectations

When using this skill, report the commit plan before making commits whenever the grouping is non-trivial. For each planned commit, include:

- commit message
- files or areas included
- why the group is logically separate

If the repository is already dirty with unrelated changes, preserve them and commit only the requested scope.

## Examples

- `feat(auth): add email sign-in card`
- `fix(theme): persist selected color mode`
- `refactor(routes): split landing page sections`
- `docs(readme): update local setup steps`
- `chore(repo): add commit skill`

For a large mixed diff, prefer a sequence such as:

1. `feat(auth): add login form`
2. `test(auth): cover login form validation`
3. `docs(auth): document local auth setup`
