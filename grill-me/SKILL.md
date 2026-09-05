---
name: grill-me
description: Grill the user relentlessly about a plan, decision, or idea. Use when the user wants to stress-test their thinking, or uses any 'grill' trigger phrases.
---

Interview the user relentlessly until you reach a shared understanding. Map this as a **design tree**: every decision branches into the decisions that hang off it.

Start from settled decisions in the conversation or supplied handoff when continuing another step. Limit the tree to unresolved decisions that materially affect the current scope; reopen a settled branch only when new evidence or user direction invalidates it.

Work the tree in **rounds**. The **frontier** is every decision whose prerequisites are already settled — the questions you can ask _now_ without guessing at answers you haven't heard yet. Ask the whole frontier in one round: number each question and give your recommended answer. Then wait for the user's answers before the next round. When necessity is unresolved for a nontrivial change, start there — what breaks for the user if this is never built, and whether an existing surface could absorb it. Otherwise, begin at the remaining frontier.

Each question should be formatted like so:

```
❓ **Q1** - **<question title>**: <question body, might be multiple paragraphs, including multiple choices>

➡️ <your recommended answer>
```

Each round the user answers reshapes the tree — settled decisions push the frontier outward and unblock questions that depended on them. Recompute the frontier and ask the next round. A question whose answer depends on another question still open in this round belongs to a _later_ round, not this one.

Finding _facts_ is your job, never the user's. When a frontier question needs a fact from the environment (filesystem, tools, etc.), use available delegation tools to investigate while asking independent questions. If delegation is unavailable, look up the fact directly. A running exploration is an unsettled prerequisite, so only the questions downstream of it wait for the result. The _decisions_ are the user's — put each to them and wait.

When technical feasibility needs execution evidence, use a bounded experiment within the authorized scope: one question, the smallest useful probe, an observable success condition, and a stopping point. Bring the evidence and limitations back to the affected decision. Keep independent branches moving; an inconclusive experiment leaves its prerequisite unresolved.

The interview is done when the frontier is empty or the user explicitly ends it and directs the next step. Treat an instruction such as "use your recommendations and proceed" as settling those recommendations and authorizing that next step, without another confirmation. Summarize the settled decisions and any remaining unknowns; for an interview-only request, end with that summary.
