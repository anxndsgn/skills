---
name: debate
description: Adversarially validate a product idea against reality — two parallel agents build the strongest evidenced case for and against it, then the main thread rules on evidence quality. Use when the user asks whether an idea is actually worth building, wants it validated with real evidence rather than conversation, or a big idea emerges from /talk-about-an-idea with stakes high enough to justify two research agents.
---

`/debate <idea> → two agents with opposing briefs in parallel → verdict on evidence quality`

A single researcher investigating an idea mostly finds evidence for their first impression — motivated search is invisible from the inside. Splitting the search into two opposing briefs makes the bias work for you: each side hunts hard, and only evidence that survives the other side's case earns a place in the verdict.

The input is a crisp idea — problem, person, wedge. If it's still fuzzy, clarify it first (that's `/talk-about-an-idea`'s job); debating a vague idea just produces two piles of unrelated links.

## The two briefs

Launch both agents in a single message so they run concurrently. Neither gets a persona or a prior — just an opposing brief. Each brief includes the idea verbatim, the research discipline quoted verbatim (an agent can't follow rules it never sees) — every claim links to its source, evidence stays separate from inference, findings carry dates — and `/research`'s method as the toolbox: go wherever the answer leaves a first-hand trace, and treat secondary accounts as leads, not evidence.

- **For** — the strongest honest case that this is worth building: demand behavior, people paying for inferior substitutes, recurring complaints about the status quo, gaps competitors leave open.
- **Against** — the strongest honest case that it isn't: dead or struggling competitors who tried, users content with the workaround, structural moats, evidence the pain is mild or shrinking.

"Strongest honest case" means advocate, don't fabricate — evidence each side searched for and couldn't find is part of its case, and each returns that too.

## The verdict

Both sides will always find _something_, so never count evidence — weigh it. Observed behavior (paying, switching, building workarounds) outranks complaints; complaints outrank opinion pieces; recent outranks stale; independent sources corroborating outrank one loud voice. The verdict rests on which case's best evidence survives the other case.

    ## Debate: <idea>

    **Verdict:** <worth building / not worth building / hinges on <the one unresolved question>>

    ## The case for
    <its surviving evidence, sourced>

    ## The case against
    <its surviving evidence, sourced>

    ## What would change the verdict
    <the concrete evidence that would flip it, and where to look>

Close with the verdict and stop — whether to proceed is the user's call. A "worth building" flows into `/grill-me → /to-spec`; a "hinges on" names the experiment to run first.
