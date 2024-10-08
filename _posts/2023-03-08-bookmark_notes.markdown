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


## [Follow the Denotation](https://reasonablypolymorphic.com/blog/follow-the-denotation/)
What do these do?

    ```haskell
    empathy :: r -> f -> X r f -> X r f
    -- Takes r, f, something with r,f and returns a something. Looks maybe like an update?
    fear    :: e -> X e m -> Either () m
    -- Takes e, something with e,m, maybe gives an m. Possibly a query?
    taste   :: X o i -> X o i -> X o i
    -- Takes two somethings with o, i, returns another something. Possibly an update? Merge?
    zoo     :: X z x
    -- Looks like an empty.
    ```
    These rigorously define the behavior of a Map-like object.

    Whereas the _names_ tell you what the fuck is going on:
    ```haskell
    -- get back what you put in
    lookup k   (insert k v m) = Just v

    -- keys replace one another
    insert k b (insert k a m) = insert k b m

    -- empty is an identity for union
    union empty m = m
    union m empty = m

    -- union is just repeated inserts
    insert k v m = union (insert k v empty) m
    ```
    They are "meaning functors" under either name.

    If we reduce Map to the "purest possible" functions, we shouldn't be able to tell the difference
    between that and any implementation.
    ```
    -- With the meaning of our type nailed down
    μ(Map k v) = k -> Maybe v


    -- Empty map assigns Nothing to everything.
    μ(empty) = \k -> Nothing


    -- Looking up a key in the map is just giving back the value at that key.
    μ(lookup k m) = μ(m) k


    -- Give back the value just inserted.
    μ(insert k' v m) = \k ->
      if k == k'
        then Just v
        else μ(m) k

    -- Lookup on left first.
    μ(union m1 m2) = \k ->
      case μ(m1) k of
        Just v  -> Just v
        Nothing -> μ(m2) k
    ```
    Our map must be _isomorphic_ to this definition. But wait:
    ```
    -- The "obvious" Monoid definition for Map
    instance Monoid (Map k v) where
    mempty  = empty
    mappend = union

    -- What we get from the above "meaning functors" doesn't match:
    instance Monoid v => Monoid (k -> Maybe v) where
    ```


## [Making The World's Fastest Website](https://dev.to/tigt/making-the-worlds-fastest-website-and-other-mistakes-56na)
Consultant for Kroger's bloated jabbascript site.

### Streamed HTML


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



# [Delivering on an Architecture Strategy](https://blog.thepete.net/blog/2019/12/09/delivering-on-an-architecture-strategy/)

Balance between Product and Architecture visions and prioritizations.

- Need to prioritize backlog appropriately between new feature work and the architectural underpinnings of the (product) feature platform.
    - If PMs don't understand the technical vision, arch improvement doesn't happen.
- Assertion that managing tech & product work with a single backlog is better than reserving capacity for "tech debt" or creating teams for new architectural projects.

## Achieving a balanced backlog
- Most natural for teams with long-term ownership of verticals, this gives time to build trust between product & eng.
- Empower autonomous teams with enough context (avoid prescribing solutions, resulting in XY problem, link to [Mission Command Post](https://blog.thepete.net/blog/2019/02/08/mission-command-enabling-autonomous-software-teams/)) to understand the broader goal and decide (pivot) accordingly
  - Empowerment while hiding context will result in misalignment (dinner party example -- "free to bring another dish" without sharing "we need a vegetarian dish")

## Connect technical work to buisiness outcomes
- Strategy ("accelerate deployments") is too broad for engineers to act on, an Architectural Initiative is the in-between.
    - e.g. "fully automated delivery pipeline" and "shared test automation platform" initiatives help achieve the acceleration strategy.
    - Biz goal -> tech strategy -> arch initiative
        - e.g. "broaden TAM by launching 2 products in 18 mo" -> "extract shared service platform from current product" -> "extract user accounts mgmt and UI framework"

## Strategic Architectural Initiatives
- Format: Target State, Current State, Next Steps
    - Duh? This is just Goals / Background / Proposal from D/TDocs
    - This helps utilize both "Doers" and "Dreamers", highlighting the gaps.


# [Uber Up Microservices](https://www.uber.com/blog/up-portable-microservices-ready-for-the-cloud/)
- Start with manual config for services, allow fast inter-service calls within a Zone.
- Transitioning to cloud-first, this doesn't fly.
    - Add linter rules and auto-detection of hardcoded zone info.
    - Perpetual continuous migration for regression testing.

## Up Layers
- Experience: UI direct control, CD/deploy automation, Balancer moving workloads, auto scaling.
- Platform: abstractions used by experience layer like service & service abstractions (constraints on machines, region, etc)
- Federation: integration to compute clusters, organizes clusters by capability and physical location, handles e.g. rebalance requests by moving clusters with canaries and monitoring 
- Regions: physical zones with kubernetes & peloton clusters, external to Up

## Impact
- 2 years of development followed by 2 years of work to migrate 4k services.
- Mostly automated migration allowed focusing on hard use cases (Note: jives with https://lethain.com/migrations/)
- "Returned 10s of MM to biz by autoscaling and efficiency, 10s of K eng hours by manual service updates."
    - One would hope so: team of 10 for 2 years is 36k hours.

## Next
- Move to cloud
- Automate CD with more checks and tests, increase security & library "freshness"
- Deploy Safety (moar automation & automated tests)
    - "Existing data shows that quite a substantial number of incidents we observe are related to code and configuration changes."
        - Candidate for most banal sentence in engineering history.
- Platform
    - Better dependency & interaction modeling, increased observability

## Summary
We moved shit to the cloud, we had to remove hardcoded zone information & assumptions.
Deploys and testing were a heterogeneous clusterfuck, now that's centrally managed.
Microservice interactions and completely independent teams (w/o shared platform) are impossible to monitor and control.


# [Pinecone: What is a Vector Database](https://www.pinecone.io/learn/vector-database/)
Similarity queries on vectors

- Random projection (mult against NxM random matrix => 1xN -> 1xM)
- Product Quantization: split vector into smaller vectors, assign each to nearest k-means center
- Locality-sensitive hashing: bucket by hash and search within bucket
- Hierarchical Navigable Small World
    - Is this just B+ trees for vectors?
    - cluster (e.g. k-means), edges between clusters are generated with some similarity metric

Similarity measures
- Cosine similarity (angle between vectors)
- Euclidean (straight line distance)
- Dot product (product of magnitude and cosine of angle)

Filtering
- vector + metadata indices, apply meta filter to pre- or post-topk search

DB Ops
- perf and fault tolerance, sharding/partitioning, replication/fault tolerance/consistency
- blah, blah




# [Wix: Event Driven Architecture — 5 Pitfalls to Avoid](https://medium.com/wix-engineering/event-driven-architecture-5-pitfalls-to-avoid-b3ebf885bdb1)
1. Atomicity. E.g. write to DB and fire event
    - Resilient producers with retry via [Greyhound, built on Kafka](https://github.com/wix/greyhound#greyhound) ("durable execution" reminds me of Temporal's Durable Execution [SE Radio podcast](https://se-radio.net/2023/12/se-radio-596-maxim-fateev-on-durable-execution-with-temporal/)
        - Doesn't guarantee ordering
    - [Debezium](https://debezium.io/documentation/reference/stable/architecture.html) sends CDC to Kafka
        - CDC log guarantees ordering
2. Event Sourcing is often disadvantageous.
    - Problems:
        - Performance means snapshotting
        - Hard to create global libraries for snapshotting and read optimizations
        - Eventual consistency.
    - CRUD+CDC as an alternative.
        - Each service can expose state via an API
        - Optionally create its own stream of event changes (more refined/less granular than all of its input events)
3. No Context Propagation
    - Tracking events through system is "fun" (no explicit chain of requests)
    - This is basically [Correlation Identifiers](https://www.enterpriseintegrationpatterns.com/ramblings/09_correlation.html) all over again.
4. Large Payloads
    - Big blobs (Images, videos, etc) may spike latency & compute
    - Compression, chunking, or ref passing can mitigate
        - Pulsar has chunking OTS
    - Duh?
5. Duplicate Events
    - Idempotentcy is important
    - Use identifiers in consumers
    - Recommendation of optimistic locking
        - Could be combined with various dedupe strategies (refuse delivery, cache, blacklist)

Final strong recommendation for streaming CDC events to solve #1 and #2



# [Segcache: a memory-efficient and scalable in-memory key-value cache for small objects](https://www.usenix.org/conference/nsdi21/presentation/yang-juncheng)
## tl;dr
Shove objects with similar TTLs in buckets, share metadata within bucket, expire entire buckets; this causes (tolerable) early expiry. Buckets have exponentially wider TTL windows to support broad lifetimes.

Garbage explanation of eviction algorithm, but uses a LFU approximate counter to save memory on frequently used items, caps update frequency to smooth bursts.

In conjunction with reduced metadata bookkeeping, single-writer/multi-reader threading model with update frequency caps dramatically improves thread scalability (update frequency cap reduces locking).

## Paper
1. group objects with similar create/expiration time into segments
2. approximate some metadata, put most into shared segment headers to reduce metadata overhead
3. segment-level bulk expiration and eviction with small critical sections

Prior work largely focuses on miss ratios (eviction algos) or throughtput, this focuses on efficiency for 10-1k B objects; Redis/Memcache have 50B overhead/object. Proactive expiration often has high overhead. Allocation with malloc causes external framentation, slab internal and "slab calcification" (slabs stuck with certain class of objects due to sizing constraints when reallocation isn't allowed).

Log structures reduce fragmentation, memshare uses small log "segments" for partitioning between tenants but migration and per-tenant compute is expensive.

Q: does segcache do multi-tenant? If not, just use memshare with 1 tenant?

Segcache is TTL-indexed, dynamically partitioned, segment structured cache; objects with similar TTL stored together in segments (reducing metadata size, ~5B/object), it evicts by merging segments and retaining the "most important objects". Time sorted, TTL-indexed segment chains allow for (novel!) efficient removal immediately after expiry.

- Single pass, merge-based eviction algorithm.
    - approximate and smoothed freq counter to balance high value retention and eviction
- "macro management strategy", doing batched bookkeeping


### Background
TTLs used variously to limit data inconsistency, prompt recomputation, implicit deletion, and comply with privacy laws.
- Lazy expiration (delete on access) increases memory footprint
- Proactive expiration is often expensive.
    - Memcached checks set number of LRU entries at the tail and removes expired entries, but locking makes this scale poorly, can be delayed given huge LRU entry count.
    - Transient object pools store objects with small TTLs separately, but choosing the TTL threshold is challenging with potential side effects.
    - Full cache scan does what it says on the tin; expensive so usually infrequent.
    - Redis does random sampling, continues sampling if proportion of expired objects in sample is above a threshold; inefficient by nature.


TIL "cuckoo hashing": using multiple hash tables and pushing keys between them upon collisions (hash function generates different positions in each table).

Memory fragmentation
- External allocation (e.g. malloc) => external fragmentation and OOM
- Slab allocation => internal fragmentation at the end of chunks and slabs, calcification if slabs can't get enough memory (especially problematic if slab class distribution changes over time)
    - rebalancing between slabs can be expensive

Throughput and scalability
- Locking around LRU queues, free object queues, and hash table
    - some use simpler eviction to remove locks (less memory efficient)
    - some use opportunictic concurrency control (RIP write throughput)
    - random eviction to avoid concurrency


# Design principles:
1. Proactive expiration: eagerly remove expired objects for mem savings
2. Share metadata to amortize cost.
3. Macro Management: operate on segments for bulk expire/evict with minimal locking


3 Components: hash table for lookup, segmented object store, TTL-indexed buckets.

### TTL buckets
* For t_1 < t_2, all objects in range [t_1, t_2) are "approximated" at t_1 (possible early expiry), objects within each t_i bucket sorted by creation time.
* Range: 1024 buckets, into 4 groups, bucket width grows by factor of 16 with each group.

### Segments (Object Store)
* Configurable size, each is treated like a mini-log and may house varied object types.
    * No updates to objects once written (except incr/decr)
* Each bucket stores pointers to head/tail of time-sorted chain.

### Hash Table
* Bulk-chaining hash table, which allocates one cache line (64B) as 8 slots in each hash bucket.
    * | bucket info | object info x 6 | object OR pointer to next bucket
        * Info slot houses 8b spin lock, 8b usage counter, 16b last-access timestamp, 32b cas value.
        * Item slots have 24b segment ID, 20b segment offset, 8b frequency counter, 12b tag (hash used to reduce comparisons upon hash collisions)

### Object Metadata
* Shared in two places: hash table bucket and segment.
    * segments share creation time, TTL, ref count
        * TTL using oldest object
    * buckets share access timestamp, spinlock, cas value, hash pointer
* No object-level hash chain pointers b/c bulk chaining
* No LRU chain pointers b/c expiry & evict at segment level
* Hash-bucket-level cas value increases false data races within bucket, but neglible empirical impact (shared between a few keys)

### Proactive Expiry
* Within a segment, objects are ordered by creation time and share TTL => bulk removal
    * BG thread scans, removes each expired segment.
        * Metadata is consecutive (TTL array), so scanning is efficient
    * Occasionally early expiry, maximum of bucket width, objects usually less useful near end of TTL in practice.


### Segment Eviction
* Caches must support eviction to make room for new objects, obv affects miss ratio
* Segment-level, merge-based eviction algorithm
* Merge first N consecutive, un-expired segments (Within TTL bucket), inherit the creation time of oldest evicted segment.
    * Created segment inserted at same position as the evicted segments while maintaining time-sorted chain.
    * Round-robin selection of TTL bucket
* While merging N segments, uses a dynamic retention threshold, updated after 1/10 of a segment, tries to retain 1/N bytes of each segment.
* Least Frequently Used eviction strategy is best under some assumptions
    * Use `freq / size`, so needs freq counter with high accuracy for less-popular objects (evicting these affects miss ratio)
    * _Approximate and Smoothed Frequency Counter (ASFC)_
        1. Approximate: If freq < 16 (4b), increase by one on each request. Otherwise similar to Morris counter (order of magnitude / log frequency estimate)
        2. Smooth: the last access timestamp rate-limits freq updates, at most 1/s; absorbs bursts.
            * Traffic bursts can pollute naive LFU.
    * Evictions reset ASFC counter, approximating window-based frequency.

### Threading
* Minimize crit sections, optimistic concurrency control, blah blah
* Segment level reduces object-level bookkeeping (e.g. free-object queues)
    * Reduces lock freq by 4 orders of magnitude(!)
* Read path locksa at most 1/s for frequency update
* Write path uses atomics with append-only store
* Each thread maintains local view of *active* segments, only that thread can write.
    * Segment removal requires locking, but object removal is lock-free

