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



## [What Is ChatGPT Doing... and Why Does It Work?](https://writings.stephenwolfram.com/2023/02/what-is-chatgpt-doing-and-why-does-it-work/)
- Probabilistically adds one token (word or part of word) at a time, by attempting to "match in meaning".
  - Not always the "highest-ranked" word, randomness for interest (controlled by empirical temperature parameter).
- Training corpus looks at n-gram word probabilities, but this naive approach doesn't get close enough with existing (100M books) data
  - LLM tries to estimate probabilities of longer sequences.
- Explanation of basic curve fitting and ML models like NNs
  - Neural nets and ML models are empirically derived models
    - There's no way to "prove" they work without understanding the underlying phenomenon (e.g. image recognition or prose writing)
  - NN classification analogy to high-dimensional Voronoi diagram, Wolfram terms these "attractors"
  - NNs may use image "features" at each layer (e.g. edges or outlines) but typically not the ones humans would communicate
  - NN training is largely empirical art with some retrospective explanations
    - Empircally end-to-end applications seem to work better than chaining specialized smaller networks
  - Training ML models takes a lot of data.
  - ChatGPT uses unsupervised learning by occluding text in the source material and predicting the missing text.
- Learning and computational irreducibility
  - Learning involves compressing data by leveraging regularities, irreducibility implies there's a limit to these regularities.
  - The more trainable a system is, the less sophisticated is computation.
  - The fact that we can train computers to do this doesn't mean they've suddenly gotten more powerful, but rather that human behaviors might be less computationally difficult than we thought.
- Embeddings reduce inputs (e.g. words, pictures) to vectors.
- CNNs exploit structure (in e.g. images) to prune connections (e.g. connect to nearby pixel neurons)
- Transformers try to do something similar for sequences of tokens in a piece of text
  - They "pay more attention" to some parts of the sequence
- ChatGPT has three stages
  - Represent previously-generated tokens ("words") as an embedding
  - Run this embedding through its NN model to produce a new embedding
  - Use that embedding to generate probabilities for the next token (~50k probabilities)
  - (All of these are learned inside the NN)
- Step 2 has many "attention blocks" comprised of "attention heads" which operate independently on chunks of values in the embedding vector.
  - Why? It works. (TM)
  - Looking at various chunks allows the model to represent relationships between different areas of the generated text.
- NN has 175B connections! (Human brain may have 100B neurons and ~100T synapses)
  - It's interesting that training data size (words) is comparable to the number of model weights (connections).
- After raw training on text, human feedback was introduced.
  - Another NN was trained on human feedback to predict rating results.
  - This can be used like a loss function to train the original.
- Because we can train a model of this size to produce such good results, human language might be less complicated than we thought.
  - This might imply new "laws of language" or "thought" are yet to be discovered.
  - Unfortunately we can't explain ChatGPT so these aren't obvious/easy to find.
- Grammatical rules define how words can be put together in to "parse trees"
  - ChatGPT implicitly "discovers" these during training.
  - Example with parenthesis language, potentially "too precise" of algorithm to model (irreducible).
  - (Me) This might have implications for the size of model needed for larger composition (e.g. tying together/referencing concepts across longer inputs)
  - Analogies to boolean logic reducibility to NAND
- Projecting ChatGPT vectors to 2D shows groupings of semantically similar words, subgroups correspond to different meanings.
- Is there additional structure?
  - Kind of. If you squint, there may be shapes correspoding to analogies
  - Generative trajectories (at 0 temperature) don't immediately look like patterns/laws. But we might be looking at the wrong variables/coordinate system.
- Syntax is easier than semantics, but even semantically sane sentences can be nonsensical in context ("The elephant traveled to the moon").
  - Eventually need "world-level" context
  - Human language is imprecise due to social contracts and context between users
- Wolfram sees an analogy between semantic grammar and syllogistic logic, is excited about building a computational language framework for the former.
- ChatGPT still fundamentally limited by
   - the architecture of NNs (probably learning is very different in humans)
   - lack of computational capability (e.g. algorithmic computation analogs like "loops" or "recompute")
   - speed of training prevents adding these naively




