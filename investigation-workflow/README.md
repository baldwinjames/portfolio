# Investigation workflow

An agentic workflow that runs the full Tier 2 support lifecycle: take a ticket, route it, map the relevant part of the codebase, pull the logs, check the infrastructure, and arrive at a root cause or a directed escalation.

**Where:** a venture-backed SaaS company, 2026. Source is private and owned by a former employer, so this describes architecture and outcomes.

## The problem

A Tier 2 ticket meant an engineer manually mapping code, pulling logs, checking infrastructure, and reasoning to a cause. Thirty minutes a ticket at the low end, and a large share of it escalated to product engineering. The support engineers were capable of the reasoning. What they lacked was the map and the access.

## What it does

Six playbooks in sequence:

```
ticket → route → map codebase → pull logs → check infrastructure → root cause → resolution or directed escalation
```

It reads a companion codebase-architecture reference and live cloud infrastructure, pulling logs through a dedicated read-only account. What comes out is a routed ticket plus a directed investigation, and a fix where the thing is fixable.

## Design decisions that mattered

**Built on a separate codebase-architecture skill rather than one monolith.** The map of the repository estate is its own artifact with its own validation, so each half can be versioned and tested independently. That map covers the full repository estate across more than a dozen domains, with the highest-traffic services deep-read, and it carries zero unverified claims. Four senior platform engineers tested it and filled the gaps they found.

**Permission-aware degradation.** A support engineer without full system access still gets a completed investigation and a directed escalation rather than a failure. The escalation arrives at engineering with the work already done. This was the difference between a tool that helps the most senior person on the team and a tool that helps everyone.

**Declared at ceiling rather than pushed further.** One system's access was confirmed unobtainable. Rather than keep tuning against a wall, I recorded the workflow as at its ceiling and named the reason, which separates an access limit from a skill deficiency. Somebody reading the benchmark a year later needs to know which one they are looking at.

**Read-only agent tools.** The agent can read and cannot act. Prompt injection through a customer ticket can therefore surface bad information and cannot take an action, which a later five-reviewer security audit validated.

## What it measured

State this in two parts and never collapse them, because they measure different things.

**Routing and investigation direction: 70% to 91% match on a 50-ticket benchmark, zero misroutes.**

**Full autonomous resolution: 54% clean, 37% partial.**

The 91% is the number that matters operationally, because the workflow is a force multiplier for a Tier 2 engineer rather than a replacement for one. Quoting the 91% as a resolution rate would be a lie by compression.

The 21-point jump came from one change, and it was not a prompt rewrite. I made the agent tag claims it had grounded in code it had actually read, and watched those tags go from 28 to 68. The gain came from verifying code rather than writing better prose about it.

## Adoption

Distributed company-wide through managed settings to the Tier 2 engineering group. Validated by the VP of Engineering and three senior engineers. One engineer took it as the foundation for an agent of his own, which is the adoption signal I trust most, because nobody builds on a tool they had to be told to use.

## What I would tell you in an interview

The interesting part of this project was not the agent. It was working out that the bottleneck was never the reasoning. Support engineers could reason about these tickets fine. They could not see the code and they could not see the logs, and every hour of the thirty minutes went into assembling context rather than thinking. Once I understood that, the design question stopped being "how do I make an agent that debugs" and became "how do I get the context in front of the person."

That reframe is also why the permission-aware degradation exists, and it is the part of the design I am most confident was right.
