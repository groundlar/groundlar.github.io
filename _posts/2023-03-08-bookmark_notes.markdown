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


# [AOSA nginx](https://aosabook.org/en/v2/nginx.html)
# High Concurrency / Motivating Examples
Apache example: connection << page size
- have to hold connection context open (>1MB)
- doesn't scale--architected to spawn a copy for each connection

nginx aimed at C10K
- event based
- originally deployed alongside apache for static content
- nginx abstractions allow pushing features to edge server (concurrency, latency processing, SSL, static content, compression, caching, connection and throttling)
- 2004, BSD

# Overview of nginx
process- or thread-based connection concurrency with blocking IO can use a lot of RAM & CPU
- each thread/process requires a new runtime env (heap, stack, exec context), this requires CPU and context switching (--> thrashing)
- nginx prefers single-threaded, non-blocking eventing
- lots of mux and event notifications, delegates tasks to separate dedicated processes, each "worker" process can process many K's of concurrent connections

## Code Structure
modular structure allows extension
- core/event modules, phase handlers, protocols, variable handlers, filters, upstreams, load balancers (all build-time as of writing ca 2012)

### Workers Model
master processs spawns workers which communicate to the backend (web/app server, memcached/redis)
- workers execute a "core run-loop" using asynchronous task handling
- workers share a "listen" socket
- each worker relies on kevent/epoll/select and sendfile/AIO/mmap
- workers are reused, no create-destroy pattern, each handling K's of connections
- worker instance count is adjustable (e.g. # of CPU cores, higher for IO bound, etc)
- challenges: disk RW optimizations ongoing, limited support for embedded scripting (can't block or exit unexpectedly)

### nginx Process Roles
Master, workers, cache loader, cache manager (all single-threaded in v1.x), SHM for IPC, only master runs as root.

Master
- reads & validates config
- creates, binds, closes sockets
- starts, maintains, terminates desired # of worker procs
- reconfigures service w/o interruption
- controlls non-stop binary upgrades
- opens/re-opens log files
- compiles embedded perl scripts

Workers accept, handle, process connections, provide rev proxy & filtering, plus "most other tasks".

Cache loader/manager handle population/expiration, respectively.



### Caching overview
Hierarchical data storage in fs
- Configurable cache keys, request-specific params can control cache content
- Keys and metadata are stored in SHM segments accessed by loader, manager, and workers
- (v1.x) no in-memory caching above OS level
  - Each cached response lives in a separate file (derived from MD5 of proxy URL)
- processed/finished request is moved to a cache directory, can purge via deletes


# nginx Config
plain text at (/usr/local)/etc/nginx.conf, can modularize into files, master reads and sends RO compiled form to workers.
- organized into non-overlapping main/http/server/upstream/location/mail context directives
- variables allow runtime config, evaluated on-demand and valid for a request's lifetime
- `try_files` used to match URI->content mappings as a replacement for `if` conditionals

# nginx Internals
Most protocol- and application-specific features handled by modules, not core
- Connections flow through a module pipeline, usually w/ FastCGI, uwsgi or memcached
- Types: event modules, phase handlers, output filters, variable handlers, protocols, upstreams, load balancers
  - event modules provide OS-dependent notifications (kqueue/epoll)
  - protocols allow for HTTPS, TLS/SSL, SMTP, POP3, IMAP

Typical request lifetime:
1. Client sends HTTP request
2. nginx core chooses phase handler based on configured location match for request
3. LB picks server, if applicable
4. Phase handler passes output buffer to first filter
5. Chained filters do their thing
6. Final response is sent.

WIP: Extreme customizability places large burden on module programmers.
- modules can attach in tons of places
  - before conf read, upon conf init, server host init, server conf & main conf merge, location conf init or merge with parent server, master process start/exit, worker process start/exit, request handling, ...

### NOTE(me) this section begins to get confused. There is too much detail without structure/connections between concepts

Inside a worker (run-loop is step 5 & 6)
1. Begin `ngx_worker_process_cycle()`
2. Process events (epoll/kqueue)
3. Accept events, dispatch actions
4. Process/proxy header and body
5. Generate response header & body, stream to client
6. Finalize request
7. Re-init timers and events

Processing a request involves processing the header and body, then calling a handler which runs through processing phases.
- Each phase has handlers which process a request and produce relevant output
- Handlers attached at conf-specified locations
- Generally phase handlers either get location configuration, generate a response, send the header, or send the body.
  - Each takes a single "request" structure arg

If a virtual server is used, there are six phases
1. server rewrite
2. location
3. location rewrite (may go back to #2)
4. access control
5. `try_files`
6. log

Depending on location conf, nginx tries a variety of content handlers (perl, proxy_pass, flv, mp4, random index, index, autoindex, gzip_static, static).
- Filter chains run after content handlers (possibly many filters for a single location)
  - header filters and body filters
  - body filters transform the generated content
    - e.g. server-side includes, xslt, image filtering/resizing, charset modifications, gzip, chunked encoding, ...

After filter chain is the writer.
- this has `copy` and `postpone` filters for caching and subrequests
- filters can coalesce subrequests into a single response, these can be nested and hierarchical, each maps to a file on disk, other handlers, or upstream servers.
  - commonly used to insert additonal content based on data in response (e.g. include->content, appending)

Upstreams implement content handlers as a reverse proxy (`proxy_pass` handler).
- Prepare request to be sent to an upstream server ("backend"), receive response
- no calls to output filters, uses a set of callbacks
  - e..g create request buffer, reinit connection to upstream, processing received payload, aborting requests, finalizing requests, trimming responses, ...

LBs attach to `proxy_pass` handler, allow choosing upstream server (round-robin or ip-hash)
- LBs and upstream handlers include upstream failure detection & re-routing

`geo` and `map` modules allow IP tracking and variable->variable mapping (e.g. flexible hostname and other runtime variables)

Memory management
- Resources allocated for lifetime of connection, prefer pointers to memcpy
- A module's response is emplaced in buffer added to a "buffer chain link," subsequent processing uses this same link.
- "Buffer chains are complicated in nginx"
  - e.g. body filter can only operate on one buffer (=="chain link") at a time, must decide overwrite/replace/insert.
  - Sometimes module will receive incomplete buffer chains (what? why?)
- Pool allocator manages memory.
  - SHM for mutex, cache metadata, SSL session cache, bandwidth limits.
  - Slab allocator for SHM allocation
  - Mutex, semaphore, R-B trees provided.
    - Docs lacking, most reverse-engineered.

# Lessons Learned
"Always room for improvement": most of Internet backbone existed prior to nginx

"Development should be focused": probably shouldn't have written a Windows version.

Modules and extensions crucial to adoption/popularity.  

# [Handling Eventual Consistency](https://medium.com/@hugo.oliveira.rocha/handling-eventual-consistency-11324324aec4#69f5)
## Patterns
"Town Crier" events: the naive approach of shouting what happened to everyone else (notify)

"Bee pattern" as events carrying enough context, similar to Town Crier


# [Figma scaling to multiple DBs](https://www.figma.com/blog/how-figma-scaled-to-multiple-databases/)
2020 bottleneck (moar features & 3x users, 2nd product), using single L AWS RDS instance,  >65% CPU usage on peak
Infra team solves problems before they cause outages, need to maintain velocity & uptime.
  - Not-so-subtle cultural note here: performance is a pillar of Figma (realtime use cases are existential features).
## Maximize runway
Plan:
  - upgrade to biggest instance
  - add read replicas
  - add new DBs to limit growth of old DB
  - Add PgBouncer connection pool (currently 1k+ connections)

## Other options
- NoSQL or Vitess migration complicated (read & write migration) with potential app changes
- NewSQL too new (early adopter tax)
- Decided to vertically partition (move tables or columns out)

### Partitioning
- Prioritization via Impact (AAS/workload change) and Isolation (not coupled to other tables)
    - Isolation is hard to measure, have to analyze queries and determine where you can sacrifice the ability to execute atomically.
    - ActiveRecord doesn't help with static analysis, so added runtime validators to emit prod query & transaction data
        - Use this info to identify clusters of tables

### Managing Migration
1. Limit potential availability impact to <1 minute
1. Automate the procedure so it is easily repeatable
1. Have the ability to undo a recent partition

### Bespoke Solution
1. Prepare client applications to query from multiple database partitions
1. Replicate tables from original database to a new database until replication lag is near 0
1. Pause activity on original database
1. Wait for databases to synchronize
1. Reroute query traffic to the new database
1. Resume activity

How does PgBouncer handle read/write replicas?
    
### The critical steps
- Not sure I follow why 2 PgBouncer instances are necessary given that they're already relying on the ability to dynamically swap the backing DB? Maybe extra layer of protection?
- Reverse replication is interesting.



# Data Oriented Programming
I originally ran across Yehonathan and the idea on The Changelog podcast, and disagreed pretty wholeheartedly.
However, they seemed passionate and enamored with the idea, respectively, so I decided to give it a deeper look, with the expectation of either becoming a convert or galvanizing my convictions.

This led me to [Expression Problem](https://eli.thegreenplace.net/2016/the-expression-problem-and-its-solutions/), which shows the difficulty in OOP--adding a new type is easy, adding new operations to a base class is hard (reimplement for all subclasses) or FP--adding a new type is hard (modify all existing functions to handle), adding new operations is easy.
  - Visitor pattern in OOP flips this (adding new types is hard since you have to update a base Visitor iface and implement everywhere), by representing operations as classes (e.g. `class Evaluator : ExprVisitor` or `class Stringifier : ExprVisitor`) .

- [ ] TODO: finish reading article for more generic solutions.

## [Principle 2: Generic Data Structures](https://blog.klipse.tech/databook/2022/06/22/generic-data-structures.html)
- Let you leverage more functionality (usu built-in) to operate on data class
- "flexible" data model: you don't have to have strict types for each new perturbation of data shape
    - Sounds like a big ball of mud to me.
    - Strawman of "AuthorData" vs "AuthorDataWithFullName"
        - Reeks of the duck typing python "fun": you end up having to verify presence everywhere because you can make no assumptions, this is schema-on-read to the extreme
    - Called out in Cost #2: No Data Schema and Cost #3: No compile-time check that the data is valid
    - "impose the need to manually document the shape of data because the compiler cannot validate it statically"
        - This seems like a bad trade: you're adding a human problem by removing a computer problem--you need to remember to add validators everywhere (maybe AOP helps?) by throwing away type systems.

## [Principle 3: Data is Immutable](https://blog.klipse.tech/databook/2022/06/22/immutable-data.html)
- In many languages, mutating the field of a map referenced in multiple places updates all of them (pointer-like behavior). Naive cloning is inefficient, efficient immutability often requires 3p libraries.
- Gives you consistent read behavior, easy equality checks (reference comparison). Concurrency safety?
- "Data access to all with confidence" already exists with static values / RO refs. Sure, these exist for a reason.
- Predictable code behavior
    - Contrived example of capture-by-reference in a callback printing a value modified elsewhere.
    - Alternative models include ownership, pass-by-copy, (&c?)
- Fast equality checks (compare ref)
- Free concurrency safety.
    - You don't need locks or mutexes if everyone can only copy. I don't know that this is "free" -- the cost *is  adopting the immutability pattern*!
- Costs: performance, extra library for persisted data (without native uspport it's dificult to enforce availability)


## [Principle 4: Separate Data Schema from Data Representation](https://blog.klipse.tech/databook/2022/06/22/data-validation.html)

Request example:
- `(firstName: str, lastName: str, optional books: int)`
    - Use JSON schma to represent this with a map, e.g. `"firstName": {"type": "string"}`
    - Validate on the fly (e.g. Ajv)

Touted benefits:
- "Allows developers to decide which pieces of data should have a schema and which pieces of data should not."
    - Prototyping use case makes sense: want a way to send data quickly and iterate.
        - I'm not convinced this is a net benefit for anything larger than toy projects. I've never seen a team *actually* redress data interfaces once a working prototype is built.
        - "Being forced to update the class definition each time the model changes slows us down" is strictly true, but in practice this is a neglible amount of time, so a very minor velocity boost in the short term (and arguably never in the long term.
    - Code refactoring
        - Claim you don't have to validate in inner functions, because it's been validated at an outer function.
            - This example breaks the touted benefit of reusable functions applicable on generic data types: you're assuming what structure the data has in the inner function. This function is either not reusable, or needs validation.
            - This results in repeated validation code, but things like AOP might save you here.
                - Or encapsulating your data validations into an object (sacrilege!)
- Optional fields
    - This isn't unique to DOP: any sane IDL provides this (Thrift, Protobuf, Avro...)
- Advanced data validation conditions
    - This section is just shilling JSON Schema.
        - To be fair, it *is* a nice library.
    - We can define arbitrarily "advanced" runtime validations for any data.
- Automatic generation of data model visualization.
    - This is a benefit of _having a schema_!
        - The notion that generating UML diagrams from a schema is somehow unique to this set of technologies (JSON Schema) or DOP is absurd. The graphic for this section is a literal class diagram.


## Criticisms
- Any trade of a build-time/static check for a runtime check is inherently suspect. The only time runtime checks are unavoidable is when dealing with external (or user-provided) data.
    - This necessitates a robust testing culture, so we end up trading a machine problem for a human one.
    - It's my view that the sort of culture which obsesses over having to modify a data definition while prototyping will struggle to justify time spent on exhaustive test coverage--this inarguably slows down prototyping.
        - Add in organizational friction (it works, so why spend more time "hardening" something that's "just fine") and it's very likely you'll end up with untested, vulnerable code.
- Lots of the content is either thinly-veiled JSON Schema advertisemtents or presenting ubiquitous capabilities as somehow unique to DOP (e.g. generating UML diagrams, runtime validations).
- It's safe to say that the "more flexible typing" experiment conducted by javascript and Python has led to the creation of things like Typescript and Python 3 type annotations.
    - Have you ever tried to refactor a method without type annotations? The only way to figure out what data is available is to inspect at runtime, or hope that the last maintainer put enough validations in to allow inference.

