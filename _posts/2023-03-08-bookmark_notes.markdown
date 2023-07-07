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




# [How To Do Hard Things (Acceptance and Commitment Therapy)](https://every.to/no-small-plans/how-to-do-hard-things)
- ACT is research backed, but isolated to therapy circles.
- Focuses on 6 core processes to cultivate psychological flexibility in the face of avoidance behaviors (inflexibility)
  - Experiential Avoidance -> Willingness
  - Fusion -> Defusion
  - Past/Future -> Present-Moment Awareness
  - Rigid Stores -> Flexible Perspective-Taking
  - Lack of Direction -> Clear Values
  - Inaction -> Committed Action

## Experiential Avoidance -> Willingness
Avoidance may be a "reverse compass," for identifying growth tasks--within reason.
Play to your strengths, but sometime you have to cultivate new ones.

Practice: sit and allow difficult thoughts and feelings to be present, open up to them as part of the path to your goals.
"What are you unwilling to feel?"

## Fusion -> Defusion
When "fused" to your thoughts, you react to them as if thy were objective, external reality.

- Reasons (I can't do X because Y)
- Judgements (People are only looking out for themselves)
- Rules (I shouldn't feel Z / I have to work harder than others)

These are simply thoughts, we can choose to pay attention to the "workable" ones moving us towards our goals.

E.g. "mistakes are bad" rule, often stems from school where you're taught the correct answers ahead of time and simply have to recall.
Real life doesn't work this way. Things are challenging an uncertain, can be difficult ot enjoy work/life if you think you're always doing something wrong.

Practice: Silly example--wave your hand about while saying "I'm not waving my hand back and forth." Thoughts and statements are fallible.

## Past/Future -> Present-Moment Awareness
Fear of the future (e.g. future problems like running out of money or uncertain outcomes) can consume your thoughts.
Obviously planning and recall are important, but there are diminishing returns.

Rumination and worrying aren't reality: they're cogitation in the present moment. Treat them as such.

Practice: pause and focus only on the information from your five senses, doesn't have to be meditation (but could be). Can eventually observe arising thoughts and emotions.

## Rigid Stores -> Flexible Perspective-Taking
Often interpersonal perceptions: "they are so irrational" or "misinformed". People are complex.

We do this to ourselves too (e.g. imposter syndrome), these narratives are limiting and oversimplifications.

Any story we tell about anyone, good or bad, is just a facet from a particular context. Sense of self should transcend the sum of experiences and history.

Practice: notice when we're caricaturing ourselves or others, put yourself in their shoes.

## Lack of Direction -> Clear Values
Aimlessness in the present may be an indication of avoidance or fusion, so start there.

Reconnect day-to-day with the things that matter most to us. Clarify to yourself what a well-lived-life looks like.
E.g. "love" and "play" for the author.

These aren't discrete goals, rather meaningful _ways of being_ which give meaning to acts in our lives.


Practice: Memento mori--death and mortality can provide focus for finding these. 
What do you want your tombstone to say?
What would it say if you died in your sleep tonight?

## Inaction -> Committed Action
You gotta effect change from the above.

7 "R"s: Reminders, Records, Rewards, Routines, Relationships, Reflecting, Restructuring. Provide a scaffolding for new behaviors until they're self-reinforcing.

There are no "right" or "wrong" behaviors, nor steps too small--anything done in service of your values is a step towards building a life that matters to you.

Practice: What small behavioral shift could I make in service of moving toward what matters in my life? And how might I support it with the scaffolding above?


# [The age of average](https://www.alexmurrell.co.uk/articles/the-age-of-average)
Two artists hire market research firm to survey American art taste, ended up painting some very average and samey pieces.
Author posits that this averaging rules our current world, resulting in convention and cliche.

Examples:

- Airbnb and coffee shop styling
  - Look at a photo and try to say which global city they're in. Spoiler: you can't.
- Architecture
  - A "non-place" is a place defined by transience and anonymity.
  - City skylines are converging, driven by function & efficiency
  - Cheap stick framing and mid-rises.
    - This falls apart for many asian cities?
    - Building codes, rising land costs, consolidation (barriers to entry) and cost-cutting via plan reuse.
  - Cars
    - Aerodynamic tests & sharing platforms (sounds like more consolidation)
    - Colors are converging too, but it's unclear why.
    - Logos
  - People
    - Convergence of "Instagram Faces".
    - Injectable treatments are on the rise, while they're cheaper and more available.
    - Again, this sounds like shared aesthetics from global culture. These are learned to a large extent.
    - Seems to me like it's a capitalism chokepoint: instagram. The algorithms have huge effects here on seeing & driving culture.
  - Media
    - Testing averages everything out: anything interesting is probably polarizing.
    - % of prequels/sequels within top-grossing movies has been rising from 25% in 2000 to ~100% in 2021.
    - Book cover examples: everything is "not giving a fuck" or "fuck yeah".
      - Seems like selection bias.
    - Best-selling video games are nearly 100% franchises since 2005
      - Counterargument: indie games are on the rise (https://www.liquidweb.com/insights/video-game-statistics/#anchor7 , https://www.statista.com/chart/27207/percentage-point-change-in-the-share-of-us-gamers-saying-they-played-selected-game-genre/ )
  - Brands
    - Advertisements look the same, argues brands draw from the same reference images online.
    - Same palette of plain/pastel branding, claims tech has led the way

Conclusion: "people's choice" creates sameness and being bold is an opportunity.


Thoughts:
In short, it sounds like consolidation and global culture due to more rapid information exchange (internet).
Overall left with "maybe?"
- Familiarity for an increasingly globetrotting populace is worth something
- Purely efficiency or shared global culture? We had microcosms previously due to slower information spread.
- Increasing wealth concentration might be driving some of the efficiency / cost-conscious convergence.
  - Architectural plan reuse sounds like a side-effect of consolidation (segue into late-stage capitalism discussion)
- Testing averaging: room for disruption? Seems like the bland sameness should result in novel content being promoted/vaunted?


# 
Beliefs about the state of software systems aren't true unless they're enforced as 
**behaviors**
  * Observed behaviors are in logs, metrics, dashboards, etc.
  * Tracked behaviors inform you when they change after a SW update; e.g. latency increases.
  * Verified behaviors are consistently true because there is a timely mechanism that ensures them before deployment, like (end-to-end integration, performance, and load) tests, fault injection triggering failure modes.
or **properties** of the system.
  * Intended properties are "best effort" through training, docs, etc
  * Tracked properties are reported on but not enforced.
  * Asserted properties are enforced before deploying.

Coming back to your Post-Apocalyptic software: how do you regain the confidence to make changes? List out all the beliefs that you’d need to have in order to be confident in modifying your software, and then for each belief an approach for evolving it up the belief and property ladder.



# [AOSA: ZeroMQ](https://aosabook.org/en/v2/zeromq.html)
- Conceived as messaging for stock trading--lots of benchmarking and optimization.
## Library vs Application
- ZMQ is a library supporting a fully-distributed architecture due to concerns with typical "smart broker, dumb clients" architecture
    - Performance: broker inevitably becomes bottleneck
    - Large-scale deployments: no company wants to give control to a server in another company (secrets & liability), results in a lot of messaging bridges
- Library model has been a boon as users perceive lower adoption & operational costs

## Global State
- Don't do it. Other libraries might themselves use ZMQ, which creates race conditions and havoc.
- ZMQ library requires a "context" object which maintains worker threads, state storage, etc

## Performance
- Throughput and latency are top concerns but the relationship isn't obvious.
    - Recommendation to read up on [Queueing Theory](https://en.wikipedia.org/wiki/Queueing_theory)
- Sender vs receiver throughput are different, averages can be misleading (see e.g. SLI monitoring in The SRE Book)
- Lesson: "Make it fast" isn't a simple charter. Know precisely what you're solving.

## Critical Path
- mallocs, syscalls, and the concurrency model have the biggest impact on performance.
- Lesson: Optimize things that happen often--the "critical path" (here that's _sending the damn messages_)

## Allocating Memory
- Only one allocation during message send: the message itself.
- For small messages, copying is cheaper than allocation, so preallocate.
- For large messages, copying is expensive, so allocate once and pass a pointer! ("zero-copy")
    - Ref [Goglin 2008](https://inria.hal.science/file/index/docid/292831/filename/Open-MX-IOAT.pdf)
- ZMQ message repr is an opaque handle, small messages stored directly in the handle while large messages are reference counted.
- Lesson: there may be several subclasses of a problem you're trying to solve, each with an optimal algorithm.

## Batching
- Syscalls can become bottlenecks with sufficiently high throughput (cost to traverse the stack, context switch, etc)
- Can tradeoff latency for throughput by batching (see also [Nagle's Algorithm](https://en.wikipedia.org/wiki/Nagle%27s_algorithm), [Interrupt coalescing](https://en.wikipedia.org/wiki/Interrupt_coalescing))
- ZMQ dynamically adjusts: no batching when message sending is sparse.
    - As the message rate grows, messages begin to get queued, so batching is switched on.
- Lesson: optimal throughput and response time means no batching. As data arrives faster than it can be processed, start batching.

## Architecture Overview
```
network
============================
worker thread   +---------+
                | engine  |
+----------+    +---------+
| listener |--->| session |
+----------+    +---------+
     |            |   ^|
     |            |   |v
     |            |  Pipes
     |            |   ^|
=====|============|===||====
     |            |   |v
+-------------------------+
|         socket          |
+-------------------------+
            |
           API
application thread
```
- User interacts with ZMQ based on user-thread-local socket abstraction
- Objects are all owned by a socket (sometimes transitively), this ownership tree is used in shutdown (kill the kids first or whatever)
- Two kinds of objects: those involved in message passing (usually connection management, e.g. creating a TCP engine/session fo each new connection) and those not (usually responsible for data transfer, e.g. engines for protocols like TCP, IPC, PGM, etc)

## Concurrency Model
- Classic threading constructs (critical sections, semaphores, etc) can be slower than a single-threaded version due to waits and context switches.
- Actor Model: avoid locking and pass asynchronous messages between threads.
- Limiting to one thread per physical core means no context switching, no critical sections, mutexes, &c. No CPU migration mitigates cache pollution.
- Sharing worker thread among objects means scheduler-based cooperative multitasking
    - No blocking ops, because this would block every object sharing the thread => all objects are state machines.
    - Shutting down a fully async system is tough (30-50% of the reported bugs)!
- Lesson: actor model is almost the only pattern for such performance and scale, but without specialized system it's a lot of hard infrastructure code. Think about shutdown from day 1.

## Lock-Free Algorithms
- "Lock-Free" really means using hardware atomics to push locking down the stack.
    - Atomic operations can be expensive, esp with multi-core systems.
- Queues have a single reader and writer thread to prevent synchronization issues within a queue.
- Batching helps solve the atomic perf penalty: read or write all the available messages at once, use "pre-write" and "pre-read" staging areas.
    - A writer flush or reader fetch operation is implemented with a single pointer move.
- Lesson: don't reinvent the (lock-free algorithm) wheel, and batch intelligently for performance.

## API
- "It's [the user interface] the only part of your program visible to the outside world and if you get it wrong the world will hate you."
- Migration from AMQP model to BSD Socket API (probably) caused adoption to soar.
    - This is a entrenched, stable, and familiar API! Users have a mental model of it.
    - API changed product perception! Went from "complex infrastructure" to "a simple library that helps me send messages from A to B"
- Lesson: Reuse and learn from everything you can--ideas, APIs, conceptual frameworks, etc. Leverage other industry experience and let users leverage theirs.

## Messaging Patterns
- Two approaches to solve the central problem of specifying which messages are routed where
    - 1. "Do one thing and do it well" (Unix philosophy)--restrict the problem domain, something like [MQTT](http://mqtt.org/)
    - 2. Focus on generality to provide a powerful and configurable system (AMQP approach)
- ZMQ does the former, supports patterns like pubsub, request/reply, parallelized pipeline.
- Generally: abstract the stack into layers to deal with problem areas (transport/routing/presentation/etc), and provide multiple non-intersecting layer implementations for each use case.
    - Analogous to TCP / UDP transport layer implementations
- Lesson: monolithic solutions may not be the best path, consider redefining the problem area as a layer and providing implementations for each use case.


# (Handling Eventual Consistency)[https://medium.com/@hugo.oliveira.rocha/handling-eventual-consistency-11324324aec4#69f5]
## Patterns
"Town Crier" events: the naive approach of shouting what happened to everyone else (notify)

"Bee pattern" as events carrying enough context, similar to Town Crier
