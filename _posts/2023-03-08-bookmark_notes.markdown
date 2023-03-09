---
layout: post
title:  "bookmark commentary"
date:   2023-03-08 18:29:21 -0700
categories: jekyll default
---

## Fun sites
Performance: https://computers-are-fast.github.io/ , https://gist.github.com/jboner/2841832

## [Perforamance Management Maturity Model](https://infrequently.org/2022/05/performance-management-maturity/#fnref-performance-management-maturity-4)
[WPO Stats](https://wpostats.com/), [Web Dev Vitals](https://web.dev/vitals/)

L0: Bliss
- Everything's fine, things OK on high-end machines, seething morass of unused tools

L1: Firefighting
- Things are unacceptably slow, no one really knows why, blind metric chasing, poor understanding of user journey and experience. Risk of talent churn.

L2: Global Metrics
- After repeated incidents, baselines and metrics are adopted via Real User Monitoring. Continual reporting against baselines, sometimes too much data.

L3: P75+, Site-specific Metrics
- Graduate from aggregate metrics to biz-specific user journeys, thinking in percentiles instead of averages, data as a distribution. Integration of metrics into experiments. Building faith in custom metrics.

L4: Variance Control and Regression Prevention
- P75 is every 4th interaction (:surprised:), awareness of innocuous changes adding up to a slow bleed, forecast this as a failure mode to avoid. Performance ship gates around *known important* flows. Alerting tracks over months and quarters, rewarding stopping slow bleeds, latency budgets.
- Staffing a "performance team". Skeptical of, and thoroughly vet, new tools and systems.

L5: Strategic Performance
- Managment and technical structures that further bolster performance and prevent cultural regression (training, advocacy, writing).
- Faster is better *when it serves user needs*. Fast is not free.
- Performance teams can deny slow features.

Maturation
- Level 1 (and maybe 2) are possible via fiat. Surpassing global baseline metrics requires product and market understanding.
- Mutual trust is required to explore and instrument the unknown.
- Adopting technologies isn't enough: improvements require funding and understanding.

- The separation between Level 0 firefighters vs Level 5 execution is context, space, and support. Engineers want to optimize.
- Senior Leadership sending mixed messages (or worse, blame) kills teams.

Questions for management:
- Do we understand how better (*strategic*) performance would improve our business (slow costs money)?
- What constraints/targets have we given the team?
- Have we developed a management fluency wth histograms and distributions over time?
- What support do we give teams that want to improve performance? Do we reward improvements?
- What support do we give mid-level managers who [push back on shiny tech](https://kellanem.com/notes/new-tech) in favour of better performance?

Performance is an aspect of system mastery (like SRE/DevOps/Observability), an important cross-cutting concern which is usually unowned.


## [New Technology Questions](https://kellanem.com/notes/new-tech)
> “We should use this new technology X, it’s faster, it’s better, it’s more elegant, it’s more actively developed, aren’t you committed to people learning and growing here at company Y, look I whipped up a prototype over the weekend and it’s in production, isn’t this technology amazing, huh, well fuck this fascist totalitarian state, I’m out of here.”

1. What problem are we trying to solve? (Tech should never be introduced as an end to itself)
1. How could we solve the problem with our current tech stack? (If the answer is we can’t, then we probably haven’t thought about the problem deeply enough)
1. Are we clear on what new costs we are taking on with the new technology? (monitoring, training, cognitive load, etc)
1. What about our current stack makes solving this problem in a cost-effective manner (in terms of money, people or time) difficult?
    1. If this new tech is a replacement for something we currently do, are we committed to moving everything to this new technology in the future? Or are we proliferating multiple solutions to the same problem? (aka “Will this solution kill and eat the solution that it replaces?”)
    1. Who do we know and trust who uses this tech? Have we talked to them about it? What did they say about it? What don’t they like about it? (if they don’t hate it, they haven’t used it in depth yet)
    1. What’s a low risk way to get started?
    1. Have you gotten a mixed discipline group of senior folks together and thrashed out each of the above points? Where is that documented?






