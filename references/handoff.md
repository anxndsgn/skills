# Workflow handoff

Use this contract when passing work to another skill, session, or agent. Carry
the following in the existing task summary, spec, review report, or agent brief;
a separate document is unnecessary. Include only fields relevant to the next step.

- **Intent and scope:** the current goal, requested next action, and boundaries
  of the user's authorization. Recommendations are not authorization.
- **Decisions:** what is agreed, what is proposed, and which unresolved questions
  could change the next step. Link the relevant spec and identify its revision.
- **Code snapshot:** the reviewed base and head commits and any narrower path
  scope. For uncommitted work, include a retrievable diff or snapshot that also
  accounts for relevant staged, unstaged, and untracked files; HEAD alone does
  not identify those contents.
- **Evidence:** checks or inspections performed, their results, the snapshot and
  criteria they cover, and remaining limitations. Link existing evidence rather
  than copying logs.

The receiving step checks that the supplied scope and snapshot match its target.
Reuse evidence only where subsequent changes leave the checked behavior and its
dependencies unaffected. Refresh affected evidence when they differ; investigate
missing context from available artifacts before asking the user. If a snapshot
cannot be identified, mark its evidence unverified rather than treating it as current.
