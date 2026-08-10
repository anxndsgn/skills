---
name: research
description: Investigate a question with cited evidence, going wherever the answer is observable first-hand — market landscape, community sentiment, a specific code repository, or anything else external. Use when the user asks to research, look into, or validate something about a market, competitors, what people are saying, or an external repo.
---

`/research <question or target> → find where the answer is observable → gather in parallel → report with sources`

The deliverable is a short report the user can act on. Its value is entirely in being trustworthy: every claim links to its source, evidence is kept separate from your inference, and findings carry dates — a complaint from 2019 may be long fixed, a competitor from memory may be dead. What you couldn't find is a finding too; report it instead of papering over it.

## Where to look

One question decides the whole approach: **where would this answer leave a first-hand trace?** Go there, as close to the primary source as access allows. There is no fixed menu of research types — a market question leaves traces in what people pay for and switch to; a sentiment question in people's own words where they actually talk (`site:` queries reach forums and reviews well); a question about a codebase in the code itself, which you can shallow-clone into the scratchpad and read.

Whatever the trace, the same rule applies: secondary accounts — an article about the market, a README describing its own code, a post summarizing what users think — are leads, not evidence. Follow them down to the behavior, the verbatim words, or the artifact they describe before citing anything. And weigh patterns over volume: the same complaint recurring across independent voices is signal; one loud post is not.

Fan out independent angles as parallel sub-agents when the question has more than one, so a slow angle doesn't serialize the rest.

## Report

    ## Research: <question>

    **Answer:** <the one-paragraph takeaway>

    ## Findings
    <ranked by how much each changes the user's decision; every claim sourced>

    ## Still unknown
    <what the evidence couldn't settle, and how it could be settled>
