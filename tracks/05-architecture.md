# 5. Architecture

Architecture is the set of decisions that are expensive to change: process boundaries, data ownership, and how failure travels. Start with one deployable and clear modules. Split a process when you have a reason you can say out loud.

Read the six [worked designs](../designs/) after this track. They use these ideas with numbers attached.

## Choosing a style

**Level:** Must-have

**You can:** name the style you are choosing, the pain it removes, and the pain it adds.

### In one minute

A style is a shape, not a rank. Microservices are not a promotion from a monolith.

Use this order when you choose:

1. **Modular monolith**, until a module has a reason to deploy, scale, or fail on its own.
2. **Service boundaries** (a service-oriented split, or microservices) when that reason is real.
3. **Events** when several consumers must react to a fact, and a direct call would couple them to the publisher's timeline.

Component-based structure is how you build the inside of any of these. It is not a rival to them.

Martin Fowler's short argument for earning a distributed system is [Monolith First](https://martinfowler.com/bliki/MonolithFirst.html).

### When it matters

The first architecture question in an interview, and the first year of a new system.

### What it costs

Each network hop adds latency, partial failure, and a deploy pipeline. You pay that cost to buy independent change and independent scaling. Buying it before you need it is how small teams drown in operations.

### A failure you should recognize

Twelve services on day one, three engineers, and no distributed tracing. A bug is a guess across twelve logs. The same system as one process with twelve modules would have been a stack trace.

### Learn

Start here: [Monolith First](https://martinfowler.com/bliki/MonolithFirst.html), then the style pages below.

Azure's [architecture styles guide](https://learn.microsoft.com/en-us/azure/architecture/guide/architecture-styles/) is a clear catalog of the styles vendors talk about (N-tier, microservices, event-driven, web-queue-worker).

### Review

1. What pain does a separate deployable remove, in one sentence?
2. What new failures appear the moment two modules talk over the network?
3. Why is "we might need to scale" a weak reason on its own?

### In an interview

Say the default (modular monolith) and the condition that would split it. Interviewers are listening for the condition.

### Exercise

Pick a system you know. Write the module you would split out first, and the measurement that would justify it (deploy rate, load shape, or a failure domain).

## Modular monolith

**Level:** Must-have

**You can:** draw module boundaries inside one deployable, with data ownership that could later become a service.

### In one minute

A **modular monolith** is one application you deploy as a unit. Inside it, modules do not reach into each other's tables. They call a public method on the module, or they publish an in-process event. Each module owns its schema (its tables, or at least its writes).

You get one pipeline, one process, local transactions, and real boundaries. Later, a module can move to its own process if its boundary was already honest. If every module joins every other module's tables, you do not have a modular monolith. You have a pile of projects.

### When it matters

The default for a new product, and the honest description of many successful systems that were mislabeled "legacy."

### What it costs

You cannot scale one module's CPU without scaling the process. A defect can still take the whole process down. Those are the reasons you eventually split a hot or risky module, not reasons to split all of them.

### A failure you should recognize

A "module" whose repository runs SQL against another module's tables. The boundary is a folder name. A second team will break the first team's invariants.

### Learn

Start here: Fowler, [Monolith First](https://martinfowler.com/bliki/MonolithFirst.html). Pair it with the boundary rules in the tactical notes below.

### Review

1. What is shared in a modular monolith, and what is forbidden to share?
2. How does a modular monolith use a database transaction that microservices cannot?
3. What would you check in review to see that a module boundary is real?

### In an interview

Offer it as the starting design for a new product. Describe one module's public operations and its tables.

### Exercise

List three modules for an online shop (catalog, cart, orders). Write one operation each module exposes, and one table the others must not write.

## Service-oriented architecture

**Level:** Must-have

**You can:** describe SOA as a way to integrate capabilities across a large organization, and tell it apart from "small services."

### In one minute

**Service-oriented architecture (SOA)** organizes a large estate around services that offer a business capability through a stable contract. The historical tools were often an enterprise service bus, shared XML schemas, and services that were not particularly small.

The idea that still matters: a caller depends on a contract, not on another team's database. The part that aged badly is a central bus that hides routing, transformation, and failure in one box no one wants to change.

You will meet SOA in enterprises that grew up with it. Treat the contract and the ownership as the lesson. Do not copy the bus unless you already have one and a reason to stay.

### When it matters

When the prompt is integration across departments, or when someone uses "SOA" and "microservices" as synonyms. They are relatives. They are not the same decade's default shape.

### What it costs

Shared canonical schemas become a meeting. Every team waits on the global model. A published contract per capability, versioned by the owner, ages better.

### A failure you should recognize

A bus rule that "just" transforms customer ids between five systems, with no owner, no test, and no trace. The mapping is wrong for one country and no service logs show it.

### Learn

Start here: the .NET architecture guide's [service-oriented architecture](https://learn.microsoft.com/en-us/dotnet/architecture/microservices/architect-microservice-container-applications/service-oriented-architecture) page. It is short, and it puts SOA next to microservices without treating them as the same thing.

### Review

1. What does a caller depend on, in the part of SOA worth keeping?
2. What problem does a central bus hide?
3. Why is a single global schema a scaling problem for teams?

### In an interview

One sentence of definition, then the contract-over-database point. Skip the product names of old buses unless you ran one.

### Exercise

Write a one-paragraph contract for "get customer summary": who owns it, what it returns, and what it refuses to do (it does not update billing).

## Microservices

**Level:** Must-have

**You can:** define a microservice by its data and its deploy, and list the costs you accept.

### In one minute

A **microservice** is a small service with its own data and its own deploy. Other services talk to its API or to its events. They do not share its tables. The useful test is the data: if two "services" share a database and deploy as a set, they are one system with extra network.

The benefits are independent deploys, a hard team boundary, and the ability to scale or crash one part. The costs are distributed transactions you no longer have, operational load, and latency.

The tradeoff list on [microservices.io](https://microservices.io/patterns/microservices.html) is the standard starting page. Sam Newman's *Building Microservices* is the book-length version.

### When it matters

When a module's deploy cadence, scaling shape, or failure domain is clearly different from the rest. A payments boundary is a common first split. A "user profile helper" is a common premature one.

### What it costs

You need timeouts, retries with idempotency, tracing, and a story for data that used to be one transaction. Without those, you have the costs and not the benefits.

### A failure you should recognize

A service that reads another service's database "just for this report." The schema is now a public API with no version. The owning team cannot rename a column.

### Learn

Start here: [microservices.io — pattern](https://microservices.io/patterns/microservices.html).

### Review

1. What is the practical test that two deployables are really separate services?
2. Which local-transaction habit do you lose?
3. What platform pieces do you want in place before the third service?

### In an interview

Give the data-ownership definition before the list of benefits. Mention one cost without being asked.

### Exercise

Take the shop modules from the monolith exercise. Choose one to extract. Write what data moves with it, and which former SQL join becomes an API call.

## Event-driven architecture

**Level:** Must-have

**You can:** say what an event is, who owns it, and when a direct call is the better tool.

### In one minute

In an **event-driven** design, a component publishes a fact that already happened: `OrderPlaced`, `FileScanned`. Other components subscribe. The publisher does not know their names.

That is different from a **command**, which is a request aimed at one owner: `ChargeCard`. Commands want a result or a clear rejection. Events announce history.

Use events when several consumers must react and you do not want the publisher to call each of them. Use a direct call when the user is waiting and you need the answer in this request.

Events arrive late, arrive twice, and arrive out of order relative to some other stream. Consumers have to tolerate that. The messaging pages below are the tolerance.

Martin Fowler's [What do you mean by "Event-Driven"?](https://martinfowler.com/articles/201701-event-driven.html) separates the meanings people mix up. AWS has a clear product-neutral overview: [What is event-driven architecture?](https://aws.amazon.com/event-driven-architecture/). Azure's style page is a good companion: [Event-driven style](https://learn.microsoft.com/en-us/azure/architecture/guide/architecture-styles/event-driven).

### When it matters

Fan-out (one fact, many reactions), integration between teams, and any user flow you are tempted to chain with five synchronous calls.

### What it costs

You lose a single stack trace. You gain a queue to operate, consumers to replay, and data that is briefly stale. Debugging requires a trace id on the message.

### A failure you should recognize

A chain of synchronous calls (API to inventory to pricing to tax to shipping) where each hop can time out. The user waits on the sum of the timeouts. One event, or one async job with a status the client polls, fits better when the user does not need every answer in the same second.

### Learn

Start here: Fowler, [Event-Driven](https://martinfowler.com/articles/201701-event-driven.html).

### Review

1. What is the difference between an event and a command?
2. What three delivery surprises must a consumer tolerate?
3. When is a synchronous call the right shape for the user?

### In an interview

Draw the event only if more than one consumer exists or will clearly exist. Name the owner of the event schema.

### Exercise

Take `OrderPlaced`. List two consumers and one thing that must stay a synchronous call (payment authorization the user is waiting on). Explain the split in three sentences.

## Component-based structure

**Level:** Must-have

**You can:** describe a component as a replaceable piece with an interface, inside a process or across processes.

### In one minute

A **component** is a piece of software with a stated interface and a hidden inside. The caller depends on the interface. You can replace the inside (a new pricing component, a test double) without rewriting the caller.

This is true inside one process and across a network. Component-based architecture is the habit of building from those pieces. It does not require a network. A modular monolith is component-based. A microservice system is component-based with process boundaries added.

The architecture books call the picture a component-and-connector view: boxes are components, lines are connectors (a method call, a queue, an HTTP request). Bass, Clements, and Kazman, *Software Architecture in Practice*, is the book that teaches that view.

### When it matters

When a design discussion is stuck on products ("should we use Kafka?") before it has named the components and the connectors.

### What it costs

Too many tiny components become a directory of indirection. A component earns its boundary when there is a real interface and a real chance of replacement or reuse.

### A failure you should recognize

A "component framework" that loads plugins for a program with one team and one deployment. The interface changes every sprint, so the plug-in benefit never arrives.

### Learn

Start here: Bass, Clements, Kazman, *Software Architecture in Practice*, the chapter on component-and-connector views.

### Review

1. What does a caller depend on?
2. How can a system be component-based and still be one deployable?
3. What would you want to see before you accept a plugin boundary?

### In an interview

Use the word for the boxes in your picture, and say what crosses the line (a call, a message). That is enough. You do not need a framework.

### Exercise

Draw four components for the file-upload design (API, object store, scanner, status store). Label each connector as sync or async.

## Hexagonal architecture (ports and adapters)

**Level:** Must-have

**You can:** put I/O at the edge and keep a use case testable without a network.

### In one minute

**Hexagonal architecture**, also called **ports and adapters** (Alistair Cockburn), says the application core talks to **ports** (interfaces): "load an order," "charge a card," "clock." **Adapters** implement the ports: a SQL adapter, a Stripe adapter, a system-clock adapter. Incoming drivers (HTTP, a queue consumer, a test) call the core through a driving port.

The core does not import a web framework or a database client. Tests call the same use case the HTTP adapter calls, and they pass in-memory adapters.

This matches [persistence ignorance](04-data-and-apis.md#persistence-ignorance) and the adapter pattern. It is the folder structure version of those ideas.

### When it matters

A service with real rules and several I/O edges (HTTP, a worker, a third-party client).

### What it costs

More types. For a thin CRUD service, a single project with a clear data-access folder is enough. Use the hexagon when the rules deserve a pure test.

### A failure you should recognize

A "domain" project that references the HTTP framework so it can read the current user from a static context. The port is missing. Pass the user id in, or pass a small `CurrentUser` port.

### Learn

Start here: Alistair Cockburn's original article, [Hexagonal architecture](https://alistair.cockburn.us/hexagonal-architecture/).

### Review

1. What is a port, and what is an adapter, in one sentence each?
2. How does a test replace the payment provider?
3. When is this structure more machinery than the service needs?

### In an interview

Mention it when you talk about testability of a use case. Do not reorganize a whole whiteboard diagram into hexagons if the question was about capacity.

### Exercise

Take one use case. Write the two ports it needs (one outgoing). Implement neither with a real network. Test the use case with fakes.

## The scale cube

**Level:** Must-have

**You can:** explain X, Y, and Z scaling, and say which axis a design is using.

### In one minute

The **scale cube**, from *The Art of Scalability* and summarized on microservices.io, names three ways to grow:

- **X.** Run more copies of the same thing behind a load balancer. Simple. Every copy can do every request. You still have one large codebase, and every copy needs the data it reads.
- **Y.** Split by function. Catalog and checkout scale and deploy separately. This is the functional split behind services.
- **Z.** Split by data. One set of copies serves a subset of customers, regions, or tenants. A shard key decides where a request goes.

Most systems do X first. Y and Z are how you respond when one copy of everything no longer fits.

### When it matters

"How does this scale?" is an invitation to pick an axis, not to say "Kubernetes."

### What it costs

X wastes money if one feature is the hot one. Y adds network and data splits. Z adds routing and the pain of a hot shard (one tenant bigger than a shard).

### A failure you should recognize

A design that only scales on X, with a single primary database that cannot take more writes. More application copies queue on the same database. The next axis is Y or Z at the data, or a change to the write path.

### Learn

Start here: [The Scale Cube](https://microservices.io/articles/scalecube.html).

### Review

1. What does X scaling leave unchanged?
2. Give an example of a Z split.
3. Why can more API instances fail to improve a write-heavy system?

### In an interview

Point at the picture and say which axis each box uses. "The API scales on X. Orders are sharded on Z by restaurant id."

### Exercise

For the URL shortener design, write which axis handles reads and which decision handles the id space. One short paragraph.

## CAP and PACELC

**Level:** Must-have

**You can:** state CAP without labeling a whole database "CP" or "AP," and add the normal-operation tradeoff from PACELC.

### In one minute

**CAP** (Eric Brewer) is about a **network partition**, a stretch of time when nodes cannot talk.

While the partition lasts, a distributed system cannot give you both:

- **Consistency** in the strict sense: every read sees the latest write (linearizability).
- **Availability** in the CAP sense: every request to a live node gets a non-error response.

You choose, for that failure mode, whether a live node should answer with something that might be stale (or refuse), or wait and maybe not answer.

Outside a partition, CAP does not rank your database. Calling a product "a CP database" or "an AP database" as a personality is misleading. Martin Kleppmann's note [Please stop calling databases CP or AP](https://martin.kleppmann.com/2015/05/11/please-stop-calling-databases-cp-or-ap.html) is the correction senior engineers are expected to know.

**PACELC** (Daniel Abadi) adds the rest of the day. If there is a **Partition**, choose **Availability** or **Consistency**. **Else** (normal operation), choose **Latency** or **Consistency**. A system can answer faster by returning a local copy that is not the latest write. That tradeoff exists on a healthy afternoon, with no partition in sight.

### When it matters

Any design with more than one copy of data: replicas, caches, regions.

### What it costs

Choosing availability can show a user stale data or allow two conflicting writes. Choosing consistency can make a request fail or wait while a replica is unreachable. Say which user-visible behavior you picked.

### A failure you should recognize

A design review that ends with "we'll use an AP database, so we're fine." No one has said what a user sees after a failed primary, or how fresh a read is.

### Learn

Start here: Kleppmann, [Please stop calling databases CP or AP](https://martin.kleppmann.com/2015/05/11/please-stop-calling-databases-cp-or-ap.html).

Then PACELC: the readable summary is the [PACELC theorem](https://en.wikipedia.org/wiki/PACELC_theorem) page, and Abadi's paper is the source ([PDF](https://www.cs.umd.edu/~abadi/papers/abadi-pacelc.pdf)).

### Review

1. CAP is a statement about which situation?
2. Why is "our database is AP" a weak architecture sentence?
3. What does the "else" in PACELC ask you to choose?

### In an interview

Define the partition case in two sentences. Then say what your design returns when a replica is behind. That is the whole point of the question.

### Exercise

Take a read-your-writes profile page. Write what the user sees if the write went to the primary and the read hit a replica that is two seconds behind. Say whether that is acceptable.

## Consistency

**Level:** Must-have

**You can:** tell strong consistency, read-your-writes, and eventual consistency apart, and tie each one to a user experience.

### In one minute

**Strong consistency** here means a read sees the latest successful write. It is what people expect from a single primary database when they read from that primary.

**Eventual consistency** means replicas will agree if you stop writing, but a read right now may be behind. The lag is usually small and is not a promise of "eventually" in the casual sense. You design the user experience for the lag.

**Read-your-writes** is a session guarantee: the user who just wrote will see their own write, even if other users might not yet. It is often the real requirement behind "it has to be consistent." A comment form that loses the comment you just posted feels broken. A search index that takes two seconds to show someone else's comment often does not.

A cache is a replica with a time-to-live. It is eventually consistent with the database by construction.

### When it matters

Caches, read replicas, search indexes, and any async flow where the UI returns before the worker finishes.

### What it costs

Stronger guarantees add waits, primary reads, or version checks. Weaker guarantees add "why is it not there yet?" support tickets unless the UI tells the truth ("we're sending your receipt").

### A failure you should recognize

A write to the primary, then an immediate read from a replica or a cache that was filled a second ago. The API returns 200 on the write and 404 on the following read. The client shows an error for a successful action. Fix: read from the primary for that session, bypass the cache for the key you just wrote, or return the new resource in the write response so the client does not have to read it back.

### Learn

Start here: Kleppmann, *Designing Data-Intensive Applications*, chapter 5 (replication) and chapter 7 (transactions). The book is the source. [dataintensive.net](https://dataintensive.net/).

### Review

1. What does a user with read-your-writes need to see?
2. Why is a cache eventually consistent even when the database is not?
3. How can a successful write followed by a read still show the old value?

### In an interview

Attach the consistency word to a screen. "The payment status the user just triggered is read-your-writes. The public feed is eventual, lag under a few seconds."

### Exercise

Write the sequence of calls for "user updates their display name and lands on their profile." Mark which read must see the new name, and which store is allowed to be behind.

## Back pressure

**Level:** Must-have

**You can:** explain what happens when a consumer is slower than a producer, and put a limit somewhere on purpose.

### In one minute

**Back pressure** is a signal that the consumer cannot keep up, passed toward the producer so the producer slows down. TCP does this with the receive window. A bounded queue does this when a push waits or fails because the queue is full.

Without back pressure, the producer keeps going. The buffer grows without a limit, memory grows, and the process dies. The work is lost anyway, later and more violently.

Places to apply it:

- A **bounded queue** between a web server and a worker. When it is full, reject new work with 429 or 503 instead of queueing forever.
- A **prefetch limit** on a message consumer, so one slow consumer does not pull a million messages into memory.
- A **timeout** so a slow dependency does not occupy every request thread.

The [Reactive Streams](https://www.reactive-streams.org/) specification is the formal version used by a few libraries. You do not need those libraries to apply the idea. A bounded channel is enough.

### When it matters

Any spike: a sale, a retry storm, a consumer that started failing and is now slow.

### What it costs

Callers see rejection under load. That is better than a dead process and a pile of half-done work. Decide the error you prefer.

### A failure you should recognize

An unbounded in-memory queue in a request path "so we never drop." After a downstream slowdown the queue holds millions of closures, each closing over a request body. The process is killed for memory. Clients retry. The new process dies faster.

### Learn

Start here: [Reactive Streams](https://www.reactive-streams.org/), the opening definition of back pressure. Then look at the bounded channel or queue in your own platform.

### Review

1. What is the signal back pressure sends, and in which direction?
2. What happens to memory if the buffer is unbounded?
3. What status code can you return when the queue is full?

### In an interview

Point at a queue in your diagram and say its bound, and what the caller sees when the bound is hit.

### Exercise

Name the bound on one queue in a system you run or can imagine. Write the caller-visible behavior when that bound is reached.

## Communication patterns

**Level:** Must-have

**You can:** choose a synchronous call or a message, and name the message shape.

### In one minute

**Synchronous.** The caller waits. The usual protocols are HTTP (see the [HTTP API](04-data-and-apis.md#http-apis) page) and [gRPC](https://grpc.io/docs/what-is-grpc/core-concepts/). gRPC uses HTTP/2, binary protobuf contracts, and generates clients. It fits service-to-service calls where you own both sides and want a strict contract. Public browsers still speak HTTP and JSON more naturally. gRPC-Web exists and is a good-to-have detail, not the default for a public API.

**Asynchronous.** The caller does not wait for the whole job. A broker stores a message until a consumer takes it.

Shapes worth naming:

- **Notification.** A fire-and-forget fact. The sender does not expect a reply message.
- **Request/response.** A reply comes back on the same channel or a reply channel. The user, or the caller, is still conceptually waiting.
- **Request/async response.** The server returns 202 and a status URL. The client polls or subscribes. The work finishes later.
- **Publish/subscribe.** Many consumers receive the event. The publisher does not list them.
- **Publish/async response.** A consumer does the work and later emits a new event (`PaymentCaptured`). The original publisher, or another service, reacts to that event.

**WebSockets** (and similar long-lived connections) fit chat and other live streams. The server can push. The connection is stateful and likes sticky routing or a shared pub/sub layer so any API instance can reach the user. The [chat design](../designs/chat.md) uses this.

**Webhooks** are "call my URL when it happens." Sign the body, retry with backoff, and make the receiver dedupe. They are good-to-have in the sense that you can learn them with the messaging rules you already have.

### When it matters

Every arrow in a design diagram. Label it sync or async before you pick a product.

### What it costs

Sync is easy to reason about and ties up a caller on someone else's latency. Async survives a short outage of the consumer and makes the user experience about status, not about a single request.

### A failure you should recognize

A synchronous chain of six services on the user-facing path, each with a 2-second timeout. The user can wait 12 seconds for a failure. Collapse the chain, or move the slow steps off the path.

### Learn

Start here: [gRPC core concepts](https://grpc.io/docs/what-is-grpc/core-concepts/) for the sync alternative to HTTP, and the event-driven page above for async.

Enterprise Integration Patterns (Hohpe and Woolf) is the book for message shapes: [enterpriseintegrationpatterns.com](https://www.enterpriseintegrationpatterns.com/).

### Review

1. When would you pick gRPC over HTTP and JSON?
2. What does the client hold after a 202?
3. What must a webhook receiver do, given that the sender retries?

### In an interview

Label every arrow. If an arrow is sync, say the timeout. If it is async, say the message name.

### Exercise

Redraw a feature you know with every arrow marked sync or async. Convert one sync arrow to a 202 plus a status check, and write what the UI shows while it waits.

## Delivery guarantees

**Level:** Must-have

**You can:** define at-most-once, at-least-once, and the practical meaning of exactly-once.

### In one minute

Networks and processes fail. A sender often cannot tell "the consumer never saw it" from "the consumer saw it and the acknowledgement was lost."

- **At-most-once.** The message is delivered zero or one time. There is no retry. You can lose messages. You do not get duplicates from the transport.
- **At-least-once.** The sender retries until it gets an acknowledgement. The consumer can see the same message twice. This is the default you should design for.
- **Exactly-once effect.** The outcome is applied once, even though the transport may deliver twice. You get this by combining at-least-once delivery with an **idempotent consumer**, or by committing the state change and the "I have seen this message" record in the same transaction.

A broker can offer a narrower feature with the same name. [Kafka's exactly-once semantics](https://kafka.apache.org/documentation/#semantics) cover a read-process-write path *inside Kafka* when you use transactions correctly. They do not, by themselves, make a write to an external database happen exactly once. The outbox and the inbox below are how you cross that boundary.

Say "exactly-once effect" when you mean the business outcome. Say "Kafka transactions" when you mean Kafka.

### When it matters

Every consumer that moves money, sends email, or reserves stock.

### What it costs

At-most-once is simple and lossy. At-least-once requires idempotent handlers and a store of ids you have already applied. Broker transactions add latency and operational rules.

### A failure you should recognize

A consumer that sends a receipt email and then crashes before it acknowledges. The message is redelivered. The customer gets two emails. The handler should record "receipt sent for order 18" in the same transaction as the state change, and do nothing when that row already exists.

### Learn

Start here: [Kafka delivery semantics](https://kafka.apache.org/documentation/#semantics). Read it for the scope of the guarantee, even if you use a different broker. The scope lesson is the point.

### Review

1. Why can a retry create a duplicate?
2. What does "exactly-once" mean inside Kafka, and what does it leave out?
3. How do you get a single business effect if the transport is at-least-once?

### In an interview

Default to at-least-once plus an idempotent consumer. Mention the Kafka nuance only if the design uses Kafka. Do not promise exactly-once as a checkbox on the broker.

### Exercise

Write the steps of a consumer that applies "add 10 loyalty points" so that a second delivery does not add 20. Name the key you store.

## Outbox, inbox, and the idempotent consumer

**Level:** Must-have

**You can:** publish a message and update a database without a distributed transaction, and ignore a duplicate on the way in.

### In one minute

**Transactional outbox.** You need to save an order and announce `OrderPlaced`. A database commit and a broker publish cannot be one atomic commit in any portable way. If you publish and then fail to commit, you announced a lie. If you commit and then fail to publish, you lost the announcement.

The outbox writes the business row and a message row in the **same database transaction**. A separate relay reads the outbox and publishes to the broker. If the relay publishes and crashes before marking the row sent, it publishes again. Consumers must tolerate that duplicate. The pattern is documented on [microservices.io](https://microservices.io/patterns/data/transactional-outbox.html).

**Inbox (deduplication).** The consumer stores the message id in the same transaction as the business write. If the id is already there, the consumer acknowledges and does no work. That stored id is the inbox.

**Idempotent consumer.** The handler can run twice and leave the same state. An inbox is the usual way to build one. Sometimes the business write is naturally idempotent (`SET status = 'paid'` for a given order). [Enterprise Integration Patterns](https://www.enterpriseintegrationpatterns.com/patterns/messaging/IdempotentReceiver.html) calls this the idempotent receiver. microservices.io describes the [idempotent consumer](https://microservices.io/patterns/communication-style/idempotent-consumer.html).

### When it matters

Any time a state change in your database must cause work elsewhere: a charge, an email, a projection, another service.

### What it costs

An extra table, a relay to operate, and consumers that are safe to retry. That is cheaper than two-phase commit across a database and a broker.

### A failure you should recognize

Code that calls the broker inside the database transaction, before commit. The publish is visible to consumers. The transaction then rolls back. Consumers act on an order that does not exist. The outbox moves the publish to after the commit, via the relay.

### Learn

Start here: [Transactional outbox](https://microservices.io/patterns/data/transactional-outbox.html).

### Review

1. Why is "commit, then publish" able to lose a message?
2. Why is "publish, then commit" able to publish a lie?
3. What do you store to make the consumer idempotent, and in which transaction?

### In an interview

Draw the outbox row in the same box as the order row. Draw the relay as a separate loop. Say that the consumer dedupes. That picture is the senior answer.

### Exercise

Sketch the columns of an outbox table and of an inbox table. Walk a crash of the relay after a successful publish. What does the consumer do on the second copy?

## Strategic domain-driven design

**Level:** Must-have

**You can:** draw a bounded context, and name the relationship between two contexts.

### In one minute

**Domain-driven design (DDD)** is Eric Evans's approach: model the software on the business, in the business's language, and be explicit about boundaries.

**Strategic design** is the boundary part. You need it even if you never write an aggregate.

- **Ubiquitous language.** Inside a boundary, a word means one thing. "Order" in sales and "order" in the warehouse are allowed to differ only if they live in different contexts.
- **Bounded context.** A boundary around a model and a language. One team, one schema, one meaning. A context map shows how contexts relate.
- **Shared kernel.** Two contexts share a small piece of model. They change it together. Keep it tiny. It is a coupling you are choosing.
- **Customer / supplier.** The downstream team needs things from the upstream team, and the upstream team takes the downstream team's needs seriously and plans for them.
- **Conformist.** The downstream team takes the upstream model as it is and adapts. They have little influence. This is common with a large vendor or a legacy system.
- **Partnership.** Two teams succeed or fail together and coordinate changes.
- **Anticorruption layer.** The downstream team translates the upstream model into its own. The foreign model does not leak inside. This is the adapter pattern at context scale.
- **Open host service.** The upstream team offers a protocol for many consumers, not a private integration per caller.
- **Published language.** A documented schema (often JSON, or a [CloudEvents](https://cloudevents.io/) envelope plus your payload) that consumers can depend on.
- **Separate ways.** You decide not to integrate. Sometimes the cheapest relationship.
- **Big ball of mud.** The context has no model. You isolate it so it does not spread. You do not "just query it" from a clean context.

[CloudEvents](https://cloudevents.io/) is a CNCF specification for the envelope (id, source, type, time). The payload is still yours. Use it when many teams consume events. It is optional inside one team.

### When it matters

Any system with more than one team, or more than one meaning for the same noun.

### What it costs

Boundaries add translation. A single model for a whole company sounds efficient and becomes a meeting about the word "customer." Translation at the boundary is cheaper than that meeting.

### A failure you should recognize

A sales screen and a shipping screen sharing one `Order` table with fifty nullable columns, because "it's the same order." The statuses contradict. Split the models. Integrate with an event or an API, not with a shared table.

### Learn

Start here: Martin Fowler, [Bounded Context](https://martinfowler.com/bliki/BoundedContext.html).

The book is Evans, *Domain-Driven Design*, part 4 (strategic design). Vaughn Vernon, *Implementing Domain-Driven Design*, has the most readable context-map chapters.

Microsoft's guide is a solid applied version: [DDD and CQRS in microservices](https://learn.microsoft.com/en-us/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/).

### Review

1. What is allowed to happen to a word inside one bounded context?
2. When do you build an anticorruption layer instead of conforming?
3. What are you protecting when you refuse a shared table between two contexts?

### In an interview

Draw two contexts for the prompt. Name the relationship (conformist, anticorruption layer, open host). Mention the noun they do *not* share.

### Exercise

Take "account" in a product you know. Write two meanings it has. Put them in two contexts. Name the relationship and the one field that must not leak across.

## CQRS

**Level:** Must-have

**You can:** separate a write model from a read model when their shapes diverge, and leave them together when they do not.

### In one minute

**CQRS** (Command Query Responsibility Segregation) means the model that handles commands (changes) is not the model that handles queries (reads). Greg Young promoted the idea. Martin Fowler's [CQRS](https://martinfowler.com/bliki/CQRS.html) is the careful overview, including when not to bother.

A **command** changes state and returns little: an id, or an acceptance. A **query** returns a shape built for a screen and changes nothing.

On the write side, a rich domain model is sometimes worth it. Often a **transaction script** is enough: a straight procedure that validates, writes, and commits. Use the rich model when the rules are rich.

On the read side, a query can be SQL or a LINQ query aimed at a table shaped for the screen. The old notes that said "LET or SQL" meant a query language or SQL. Say LINQ if you mean LINQ.

The read model can be the same database, with different tables updated in the same transaction. That is the modest, useful form. A separate database updated by events is the strong form. It can be stale, and you must operate the projection.

CQRS is not event sourcing. You can separate models and still store the current state as rows.

### When it matters

A write model that is a state machine, and a screen that is a join across many of those machines. Forcing one object model to serve both gets awkward.

### What it costs

Two models to change when a field is added. Staleness, if the read side is async. Do not pay this for a form that edits the same fields it shows.

### A failure you should recognize

CQRS, event sourcing, and a message bus introduced for a settings page. Every new checkbox is a projection change. A single table would have shipped in a day.

### Learn

Start here: Fowler, [CQRS](https://martinfowler.com/bliki/CQRS.html).

### Review

1. What problem is CQRS solving when it is earning its cost?
2. Why can the modest form stay inside one database transaction?
3. Why is CQRS not the same decision as event sourcing?

### In an interview

Say "same model" first if the screen matches the write. Offer CQRS when the read shape is clearly different. Mention staleness if the update is async.

### Exercise

Pick a screen that is a report over several write aggregates. Sketch the write command and the read row. Say whether the read row is updated in the same transaction or by a consumer.

## Resilience

**Level:** Must-have

**You can:** put a timeout, a limited retry, and a circuit breaker on an outbound call, and say which errors are safe to retry.

### In one minute

Remote calls fail partway. You will not "handle it later" as a mood. You will pick numbers.

**Timeout.** Every outbound call has a deadline. The caller's deadline is shorter than the user's patience and long enough for a healthy dependency. Deeper calls in a chain get shorter deadlines, so a child does not outlive the parent. A call with no timeout can hold a thread until the process is out of threads.

**Retry.** Retry when a later try might work: a timeout, a connection reset, a 429, a 503. Do not retry a 400, or a payment call that has no idempotency key. Use **exponential backoff** and **jitter** (a random extra wait) so all callers do not retry on the same millisecond. Cap the attempts. AWS's builder's library article is the standard explanation: [Timeouts, retries, and backoff with jitter](https://aws.amazon.com/builders-library/timeouts-retries-and-backoff-with-jitter/).

**Circuit breaker.** After a run of failures, stop calling for a short open period, then allow one trial call. This protects the caller and gives the dependency room. A breaker that opens and is never noticed is an outage you built. Log the state change and alert on it. Michael Nygard's *Release It!* is the book that put this pattern into common practice.

**Bulkhead.** Isolate resources so one dependency cannot use every thread or connection. A separate small pool for a flaky vendor is a bulkhead. The rest of the API stays up when that vendor hangs.

**Rate limit.** Protect yourself and your neighbors. The [rate limiter design](../designs/rate-limiter.md) goes into algorithms.

### When it matters

Every box in the diagram that is not in your process.

### What it costs

Retries add load exactly when the dependency is sick. A breaker adds a behavior you must explain ("we fail fast for 30 seconds"). Timeouts that are tighter than the real latency turn a slow dependency into a down one.

### A failure you should recognize

A retry with no backoff, on a dependency that is timing out, from 50 instances. The dependency gets worse. This is a **retry storm**. The fix is a small cap, backoff with jitter, and a breaker.

### Learn

Start here: [Timeouts, retries, and backoff with jitter](https://aws.amazon.com/builders-library/timeouts-retries-and-backoff-with-jitter/).

Book: Michael Nygard, *Release It!*

### Review

1. Which status codes are worth a retry, and which are not?
2. Why does jitter exist?
3. What does a circuit breaker do to the caller's threads while it is open?
4. What is a bulkhead in terms of a connection pool?

### In an interview

Put a timeout on the arrow as you draw it. Mention idempotency before you mention retrying a write.

### Exercise

Configure an HTTP client with a 500 ms timeout, at most two retries, backoff, and jitter, aimed at a local port that sometimes sleeps. Log each attempt. This can be a twenty-line program.

## Caching

**Level:** Must-have

**You can:** choose cache-aside, set a lifetime, and name cache stampede.

### In one minute

A **cache** holds a copy closer to the reader so you do not recompute or reread the slow source.

**Cache-aside** is the pattern to know. The application reads the cache. On a miss, it reads the database, stores the value with a **time to live (TTL)**, and returns it. On a write, it updates the database and deletes the cache key (or updates it). The next read fills the cache. Azure's description is clear: [cache-aside pattern](https://learn.microsoft.com/en-us/azure/architecture/patterns/cache-aside).

**Stampede.** A popular key expires, and a thousand requests miss at once and all hit the database. Mitigations: a short lock so only one request refills, a soft TTL that serves the old value while one request refreshes, or jitter on the TTL so keys do not expire together.

Do not cache what you cannot afford to serve stale: an authorization decision you are unwilling to revoke, a payment result, a one-time code.

A **CDN** (content delivery network) is someone else's cache at the edge, for public bytes. It is a good-to-have, below.

### When it matters

Read-heavy designs, the URL shortener, product pages, and any computed result that is requested far more often than it changes.

### What it costs

Stale reads, memory, and invalidation bugs. "Invalidate on write" is only easy if there is one writer and you know every key that contains the value.

### A failure you should recognize

A cache of the full user object, including roles, with a TTL of one hour, and no delete on role change. An admin revokes access. The user keeps the old role until the hour ends. Cache the role for seconds, or delete that key in the same action that changes the role.

### Learn

Start here: [Cache-aside](https://learn.microsoft.com/en-us/azure/architecture/patterns/cache-aside).

HTTP response caches, which are a different layer: [RFC 9111](https://www.rfc-editor.org/rfc/rfc9111.html).

### Review

1. Walk a cache-aside read, including a miss.
2. What is a stampede, and name one mitigation?
3. What kind of data do you refuse to cache for a long TTL?

### In an interview

Say the key, the TTL, and the invalidation. "Key is the short code, TTL is a day, and we delete it if the owner changes the target."

### Exercise

Add a process-local cache in front of a function that counts calls to the "database." Show that a hundred reads become one database call, and that an update is visible after you delete the key.

## Replication and partitioning

**Level:** Must-have

**You can:** explain a primary with replicas, and choose a partition key from the access path.

### In one minute

**Replication** copies the same data to more than one node.

- A **primary** (leader) takes the writes.
- **Replicas** (followers) apply the changes.
- **Synchronous** replication waits for a replica before the write succeeds. You lose less data if the primary dies. The write is slower and fails if the replica is down.
- **Asynchronous** replication acknowledges once the primary has the write. It is faster. A crash can lose the latest writes that never reached a replica.
- Read replicas scale reads. They can be behind the primary. That lag is the consistency topic above.

PostgreSQL's [high availability](https://www.postgresql.org/docs/current/high-availability.html) chapter is a concrete version of these choices.

**Partitioning** (sharding) splits rows across nodes by a **key**. All rows for a key live together. You pick the key from how you query: by restaurant, by tenant, by conversation. A query that always filters by that key hits one partition. A query that needs "all tenants, sorted" fans out to every partition. Fan-out is the feature you give up.

A **hot partition** is a key with far more traffic than the rest. One celebrity, one giant tenant. More shards do not help that key. You isolate it, or you split by a finer key.

Moving data to a new shard scheme is a project. Spend the first choice carefully.

### When it matters

When one database's writes or disk are no longer enough, and when you draw anything that says "shard."

### What it costs

Replication costs disk and lag. Partitioning costs operational pain and the loss of easy global queries and easy cross-key transactions.

### A failure you should recognize

Sharding users by `user_id` while the hot path is "all messages in a conversation." Every conversation read hits many shards. Shard messages by `conversation_id`.

### Learn

Start here: Kleppmann, chapters 5 and 6. PostgreSQL's [high availability](https://www.postgresql.org/docs/current/high-availability.html) page for a real system.

### Review

1. What can asynchronous replication lose?
2. How do you pick a partition key?
3. Why does a hot key ignore the fact that you have many shards?

### In an interview

Say what is replicated and what is partitioned. They are different axes. A common design is a partition of primaries, each with its own replicas.

### Exercise

Pick a partition key for chat messages and for a user's inbox. Explain a query that becomes easy and a query that becomes a fan-out.

## Load balancing

**Level:** Must-have

**You can:** say what a load balancer checks, and why sticky sessions are a smell.

### In one minute

A **load balancer** spreads requests across healthy copies of a service. It runs health checks and stops sending traffic to a copy that fails them.

- **Layer 4** balances TCP connections. It does not read the HTTP path.
- **Layer 7** understands HTTP. It can route `/api` and `/static` differently and can retry an idempotent `GET` in some products. Know your product's retry behavior so you do not double-apply a `POST`.

**Sticky sessions** send one user to one instance, because that instance holds the session in memory. That sticks capacity planning to one node and makes deploys awkward. Prefer a stateless application process and a shared session store, or a signed cookie that any instance can read.

### When it matters

Any design with more than one instance, which is every design you should draw for a service that must stay up through a deploy.

### What it costs

A load balancer is another thing that can be misconfigured (health check too aggressive, timeout shorter than the app's timeout). It is still the right tool. One instance is a single point of failure.

### A failure you should recognize

A health check that hits a path which always returns 200, even when the database is down. The balancer keeps sending traffic. The health check should fail when the instance cannot do the job.

### Learn

Start here: the docs of the balancer you use. AWS documents the ideas in the [Elastic Load Balancing features](https://docs.aws.amazon.com/elasticloadbalancing/latest/userguide/what-is-load-balancing.html) guide (the "what is" chapter). The feature list page moves; the user guide is the stable door.

### Review

1. What does a health check decide?
2. Why do sticky sessions fight an even scale-out?
3. Why is a balancer retry of `POST` dangerous if you have not designed for it?

### In an interview

Draw the balancer in front of stateless instances. Mention the health check in one clause.

### Exercise

Write the health-check rule for an API that depends on a database. Include one condition that must fail the check, and one that must not (a downstream analytics system being down should not fail the user-facing health check if the API can still serve).

## Capacity estimation

**Level:** Must-have

**You can:** do a back-of-the-envelope estimate and show the assumptions.

### In one minute

You are checking orders of magnitude, not buying hardware in the interview.

1. **Requests per second.** Daily active users × actions per user / seconds in the busy period. State whether you used a full day or a 4-hour peak.
2. **Peak.** Multiply the average by a factor you state. 2× to 10× is a normal range. A product launch can be more. Say the number.
3. **Storage.** Writes per second × bytes per write × retention seconds.
4. **Bandwidth.** Requests per second × bytes on the wire. Remember that reads often dominate.
5. **Cache memory.** Fraction of data that is hot × size of each entry. The 80/20 guess is fine if you call it a guess.

Write the units. Convert at the end. A "billion" that is really a million changes the design.

Worked miniature: 5 million daily active users, each reads a feed twice a day.

- Average read QPS ≈ (5e6 × 2) / 86400 ≈ 115 requests/second.
- Peak at 10× ≈ 1,200 requests/second.
- If each response is 20 KB, peak bandwidth ≈ 24 MB/s.

That is one API tier, not a special architecture. The storage and the fan-out may still be special. The estimate tells you which part is special.

### When it matters

The start of every system design, so you do not shard a problem that fits on one database.

### What it costs

Estimates take a few minutes. Skipping them takes the rest of the interview, because you designed for the wrong size.

### A failure you should recognize

"We'll shard from day one" for 50 requests per second of small JSON. The complexity will dominate the project. One primary and a replica are enough until the estimate says otherwise.

### Learn

Start here: the method above. Kleppmann's back-of-the-envelope examples in chapter 1 of *Designing Data-Intensive Applications* are the same habit.

### Review

1. What do you state before the arithmetic?
2. Roughly what read QPS is 5 million users × 2 reads per day, on average?
3. Which result would push you to partition, and which would not?

### In an interview

Speak the assumptions. Round so the interviewer can follow. Tie the result to a decision: "About a thousand read QPS at peak, so one primary and a cache, and I would not shard yet."

### Exercise

Estimate the checkout design: 2 million orders a day, peak 10×, 2 KB stored per order, keep 7 years. Compute peak write QPS and total storage. Write the decision the numbers support.

## Sagas

**Level:** Must-have

**You can:** split a business transaction across services into local steps with a compensation, and say why two-phase commit is not the default.

### In one minute

A **saga** is a sequence of local transactions. Each step commits in its own service. If a later step fails, earlier steps run a **compensation** (refund, release the seat, cancel the shipment). There is no single lock across all the services.

Two styles:

- **Choreography.** Each service listens for events and does the next local step. Simple when there are few steps. Hard to follow when there are many.
- **Orchestration.** One process manager tells each service what to do and stores the saga state. Easier to read. The orchestrator is a component you operate.

**Two-phase commit** (2PC) asks every participant to prepare, then commit. It works inside some databases and transaction monitors. It is a poor fit across a payment provider you do not control, and it stalls if a participant is down. Prefer a saga plus idempotent steps.

The [checkout design](../designs/order-checkout.md) is the worked saga.

### When it matters

Any flow that used to be one database transaction and now crosses a service or a third party: order, payment, inventory, shipping.

### What it costs

The user can observe intermediate states ("paid, not yet shipped"). Compensations can fail too, so you need an operator view and a retry. This is more code than a transaction. It is the code distributed systems require.

### A failure you should recognize

A compensation that is not idempotent. The refund message is delivered twice and the customer is refunded twice. The payment step needs an idempotency key, and so does the refund.

### Learn

Start here: microservices.io, [Saga pattern](https://microservices.io/patterns/data/saga.html).

### Review

1. What is a compensation?
2. When does choreography become hard to understand?
3. Why is two-phase commit a weak plan for a call to an external payment API?

### In an interview

List the steps and the compensation for each. Mention the state a user can see in the middle.

### Exercise

Write a three-step saga for "reserve ticket, take payment, email ticket." For each step, write the compensation and the idempotency key.

## Good to have

### Event sourcing

**Level:** Good-to-have

**You can:** explain that state is derived from an append-only log of events, and name the cost.

Instead of storing only the current row, you store the events that led to it (`ItemAdded`, `ItemRemoved`, `OrderPlaced`). The current state is a fold over those events. You can rebuild a projection, audit every change, and feed other read models.

You also take on schema evolution of events, snapshots for long streams, and a more complex mental model. Use it when the history *is* the business (ledgers, audit). Leave it when a row is the business. Fowler's CQRS article discusses the pairing. Event sourcing does not require CQRS, and CQRS does not require event sourcing.

### Tactical domain-driven design

**Level:** Good-to-have

**You can:** use aggregate, entity, and value object correctly inside one bounded context.

- An **entity** has an identity over time. Two customers with the same name are different customers.
- A **value object** is defined by its values. Two addresses with the same fields are the same address. They are immutable.
- An **aggregate** is a cluster of entities and value objects with one root. Outside code may hold a reference only to the root. A transaction changes one aggregate. The aggregate is the consistency boundary. An order and its lines are a typical aggregate. An order and a customer are not: they change for different reasons.
- A **domain event** is a fact the aggregate raises (`OrderPlaced`). It can update other aggregates after the commit, through the outbox.

The source is Evans, *Domain-Driven Design*, and Vernon for a more practical pass. The Microsoft [tactical DDD chapter](https://learn.microsoft.com/en-us/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/ddd-oriented-microservice) matches this vocabulary.

### API gateway

**Level:** Good-to-have

**You can:** say what belongs at the edge and what must stay in the service.

A gateway terminates TLS, authenticates, rate-limits, and routes to services. It is a place for concerns every service would otherwise copy. It is a bad place for a business rule that only one service understands. That rule belongs in the service, where its tests are.

### CDN

**Level:** Good-to-have

**You can:** put a CDN in front of public, cacheable bytes, and keep private responses out of it.

A CDN caches at locations near users. It fits images, downloads, and public `GET` responses with cache headers. A response that depends on the caller's identity needs a cache key that includes that identity, or it should not be cached at the edge. A leaked cached profile is an authorization bug.

### Consistent hashing

**Level:** Good-to-have

**You can:** explain why adding a cache node should not move every key.

With `hash(key) mod N`, changing N moves almost every key. **Consistent hashing** places nodes on a ring. A key belongs to the next node clockwise. Adding a node moves only the keys that fall on its new slice. Caches and some partitioners use this so growth is not a full reshuffle. The idea is from Karger and others (1997). Kleppmann, chapter 6, is the readable account. Virtual nodes keep the slices even when real nodes are few.

### Search

**Level:** Good-to-have

**You can:** put full-text search in a search index and keep the database as the source of truth.

A relational `LIKE '%shoe%'` scan does not scale as a product search. A search engine (OpenSearch, Elasticsearch, or a managed equivalent) indexes tokens. The index is fed by the outbox or a change stream. It lags. The product page itself should read the database, or you accept that search is slightly stale and the detail page is not.

### GraphQL

**Level:** Good-to-have

**You can:** say when a graph query helps, and the two costs that come with it.

GraphQL lets a client ask for the shape it wants from one endpoint. It helps when many clients need different slices of the same graph (a web app and several mobile screens).

It makes HTTP caching harder, and a single query can fan out into expensive work. You need limits on depth and cost, and you need to solve N+1 inside resolvers (batch the loads). A JSON HTTP API is still the default for a service with a few known clients.

### Webhooks

**Level:** Good-to-have

**You can:** receive a signed, retried callback without applying it twice.

Verify a signature over the raw body, using a secret you rotate. Respond quickly with 2xx after you have stored the event, and do the work after. Retries will come. Dedupe on the event id. Reject timestamps that are too old if the provider signs the time, so a captured request cannot be replayed forever.

### Feature flags

**Level:** Good-to-have

**You can:** separate "the code is deployed" from "the behavior is on."

A flag lets you merge to main, deploy dark, and turn the behavior on for one tenant or one percent of traffic. Remove the flag after the launch. A codebase full of stale flags is a second, undocumented product. Flags are configuration, and the dangerous ones are the ones that change a write path. Test both sides.

### Multi-region

**Level:** Good-to-have

**You can:** distinguish active-passive from active-active.

**Active-passive.** One region serves. The other is a replica. Failover has a time and a possible loss of the last async writes. This is the pattern to describe first.

**Active-active.** Two regions take writes. You need a conflict rule (last write wins, or a merge, or partitioning users so each user has a home region and conflicts do not arise). Home-region users are the usual way to avoid inventing a global strongly consistent database.

### Consensus

**Level:** Good-to-have

**You can:** say what a consensus algorithm is for, and that you use it through a system you did not write.

**Consensus** lets a small group of nodes agree on a value even if some fail. **Raft** is the algorithm you can learn: a leader, a log, elections. etcd and similar systems implement it. You use them for configuration and leader election. You do not implement Raft for an application feature. If an interview asks, explain leader election at a high level and put the feature's data in a database.
