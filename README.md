# Software Engineer Roadmap

A roadmap for a **senior backend engineer**. Use it to review what you already know, to learn a topic properly, or to prepare for an interview.

This is the path from working on features inside one service to owning the design, the failure modes, and the tradeoff. The examples are language-neutral. Where a platform detail matters, the worked example is **.NET 10**, the current long-term support release as of October 2026.

Standards named in this repo were checked in **October 2026**. Each topic has one place to start. The full list, with the status of each source, is in [appendix/references.md](appendix/references.md).

## How to use this roadmap

**Review, about two weeks.** Read the outcome and the review questions for each must-have topic. Answer out loud. If you stall, open the "Start here" source, then come back the next day.

**Learn, about 8 to 12 weeks.** Take one topic at a time. Read the page, read the start-here source, do the exercise, and answer the review questions the next day. Then do one worked design from [designs/](designs/).

**Interview, about 10 days.** Use the [interview index](interview/index.md). On day A, sit one timed prompt (35 to 45 minutes for a design, 15 minutes for a spoken review). On day B, answer the review questions for the topics that prompt used. Finish with the two stories in [Senior judgment](tracks/07-senior-judgment.md).

A topic page always has the same shape: what you should be able to do, a short explanation, when it matters, what it costs, a failure, a source, review questions, an interview cue, and an exercise.

## Scope

In scope:

- Code design inside a service
- Data, HTTP APIs, and consistency
- Architecture and distributed messaging
- Production: security, observability, delivery
- The judgment interviews look for in a senior backend candidate

Out of scope, on purpose:

- Frontend, mobile UI, and visual design
- A catalog of coding-puzzle solutions. Practice data structures and algorithms on a set you already use. This roadmap covers the engineering around them: saying how much work a path does, choosing a structure, and reading a hot path in a running service.
- Vendor certification dumps

The .NET runtime notes (garbage collection, `HttpClient`, the large object heap) live in [appendix/dotnet-runtime.md](appendix/dotnet-runtime.md). Read them if you work on .NET. The rest of the roadmap stands without them.

## Must-have and good-to-have

**Must-have** topics are the ones a senior backend interview, or the first year of senior work, will actually touch. Learn these.

**Good-to-have** topics are real, and they come up often enough to be worth a page. Learn them after the must-haves in that track. Skipping one of them does not leave a hole in the core. Skipping a must-have does.

| Track | Must-have | Good-to-have |
| --- | --- | --- |
| [1. Language and runtime](tracks/01-language-and-runtime.md) | Paradigms, memory, in-process concurrency | — |
| [2. Code design](tracks/02-code-design.md) | Relationships, SOLID, composition, six patterns, diagrams | Hollywood principle, dependency injection as a habit |
| [3. Testing and change](tracks/03-testing-and-change.md) | Tests, Git and review, debugging | Contract tests, property-based tests |
| [4. Data and APIs](tracks/04-data-and-apis.md) | Schema, indexes, transactions, concurrency, migrations, N+1 and pools, persistence ignorance, HTTP APIs | — |
| [5. Architecture](tracks/05-architecture.md) | Styles in order, boundaries, scale cube, CAP and PACELC, consistency, back pressure, communication, delivery, outbox and inbox, DDD context maps, CQRS, resilience, caching, replication and partitioning, load balancing, capacity, sagas | Event sourcing, tactical DDD, API gateway, CDN, consistent hashing, search, GraphQL, webhooks, feature flags, multi-region, consensus |
| [6. Production](tracks/06-production.md) | TLS, authentication and authorization, secrets and supply chain, observability and SLOs, incidents, CI/CD and containers, backups | Passkeys, canary releases, privacy habits, signing images, model-call risks |
| [7. Senior judgment](tracks/07-senior-judgment.md) | Design docs, estimation, two stories | Mentoring |
| [Worked designs](designs/) | All six | — |
| [Interview index](interview/index.md) | The whole table | — |

GRASP, the responsibility-assignment vocabulary from Craig Larman, is in [appendix/grasp.md](appendix/grasp.md). It is optional. Interviews usually say SOLID and the pattern names above.

## The six designs

Read these after track 5. Each one asks for numbers, a picture, the option you reject, and the failure you expect.

1. [URL shortener](designs/url-shortener.md) — read-heavy cache and id generation
2. [Activity feed](designs/activity-feed.md) — fan-out and a celebrity user
3. [Chat](designs/chat.md) — connections, order, presence
4. [Rate limiter](designs/rate-limiter.md) — where the limit lives
5. [File upload](designs/file-upload.md) — object storage and a slow scan
6. [Order checkout](designs/order-checkout.md) — one charge, an outbox, a saga

## A week of review

If you only have one week before an interview, walk this list and stop to study wherever the answer is fuzzy.

1. Transactions, isolation, and lost updates
2. HTTP status codes, idempotency, and pagination
3. Timeouts, retries, and idempotent consumers
4. Outbox, inbox, and the checkout design
5. Caching and the URL shortener
6. Load, replication, and a hot partition
7. Authentication versus authorization
8. One design doc out loud, plus one incident story

## Old outline

The first version of this repo was a list of topic names and links. Those topics are still here. They now have an explanation, a current source, and a way to practice. References that were blog summaries, or links with a broken shape, are replaced in [appendix/references.md](appendix/references.md).
