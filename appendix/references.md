# References

Checked in **October 2026**. Each topic in the tracks names one place to start. This page is the catalog, with the status of the source so a draft is not mistaken for a standard.

When two pages disagree, prefer a dated standard (an RFC, a W3C recommendation, or the project's own current manual) over this roadmap, and then update the topic page.

## How these were chosen

The first version of this repo pointed at a mix of vendor pages, GeeksforGeeks, freeCodeCamp, Educative, YouTube, and Medium posts. Some of those links had a doubled slash. Several sections had no source at all.

This version keeps a source when it is a standard, an official manual, or a canonical article a senior can defend. Summaries that only repeat a definition were dropped. Books are named where a web page would be a thin substitute. Drafts are labeled as drafts.

Two pages were blocked on the first pass in October 2026 and opened again before this list was settled.

- IBM's SOA overview is the start-here page. It explains the contract, the registry, and the enterprise service bus. The .NET architecture guide's SOA chapter is about a page, from the 2018 book, and it only contrasts the bus with microservices. It stays as a companion when you are already in that book.
- Google Cloud's "what is microservices" page is a product introduction. The tradeoff list on microservices.io is the page a design answer can defend, so the Google overview is not a source here.

Pact's documentation host was slow on that same pass. It is the reference. The project homepage is only the front door.

## Standards

| Source | Status | Use it for |
| --- | --- | --- |
| [RFC 9110, HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110.html) | Internet Standard | Methods, safety, idempotency, status codes, conditional requests |
| [RFC 9111, HTTP Caching](https://www.rfc-editor.org/rfc/rfc9111.html) | Internet Standard | Cache headers |
| [RFC 9112](https://www.rfc-editor.org/rfc/rfc9112.html), [RFC 9113](https://www.rfc-editor.org/rfc/rfc9113.html), [RFC 9114](https://www.rfc-editor.org/rfc/rfc9114.html) | Internet Standards | HTTP/1.1, HTTP/2, and HTTP/3 on the wire. You usually design 9110 and let the platform carry it |
| [RFC 9457, Problem Details](https://www.rfc-editor.org/rfc/rfc9457.html) | Proposed Standard | `application/problem+json` error bodies. Obsoletes RFC 7807 |
| [RFC 9562, UUID](https://www.rfc-editor.org/rfc/rfc9562.html) | Proposed Standard | UUID version 7 (time-ordered) and version 4 |
| [RFC 8446, TLS 1.3](https://www.rfc-editor.org/rfc/rfc8446.html) | Internet Standard | The current TLS protocol |
| [RFC 9325](https://www.rfc-editor.org/rfc/rfc9325.html) | Best Current Practice | How to configure TLS. SSL and TLS 1.0/1.1 stay off |
| [RFC 6749, OAuth 2.0](https://www.rfc-editor.org/rfc/rfc6749.html) | Proposed Standard | The base OAuth framework |
| [RFC 7636, PKCE](https://www.rfc-editor.org/rfc/rfc7636.html) | Proposed Standard | Proof Key for Code Exchange |
| [RFC 9700](https://www.rfc-editor.org/rfc/rfc9700.html) | Best Current Practice, January 2025 | OAuth security: code plus PKCE, no implicit flow, no password flow |
| [RFC 7519, JWT](https://www.rfc-editor.org/rfc/rfc7519.html) | Proposed Standard | The token format, not a session design |
| [OpenID Connect Core 1.0](https://openid.net/specs/openid-connect-core-1_0.html) | OpenID specification | Login on top of OAuth. ID token versus access token |
| [OAuth 2.1 draft](https://datatracker.ietf.org/doc/draft-ietf-oauth-v2-1/) | Internet-Draft, not an RFC as of October 2026 | Where the security BCP is heading. Follow RFC 9700 today. Overview: [oauth.net/2.1](https://oauth.net/2.1/) |
| [Idempotency-Key draft](https://datatracker.ietf.org/doc/draft-ietf-httpapi-idempotency-key-header/) | Expired Internet-Draft (revision 07 expired 18 April 2026) | Background only. Not a standard |
| [OpenAPI specification](https://spec.openapis.org/oas/latest.html) | OpenAPI 3.2, patch 3.2.1 as of September 2026 | Describe HTTP APIs. 3.1 documents remain valid |
| [W3C Trace Context](https://www.w3.org/TR/trace-context/) | W3C Recommendation | The `traceparent` header |
| [W3C WebAuthn](https://www.w3.org/TR/webauthn-2/) | W3C Recommendation | Passkeys. Introduction: [passkeys.dev](https://passkeys.dev/) |
| [UML](https://www.omg.org/spec/UML/) | OMG specification | Official relationship names, when you need them |
| [C4 model](https://c4model.com/) | Author's specification (Simon Brown) | Context, container, component, code |
| [OCI image spec](https://github.com/opencontainers/image-spec) | Open Container Initiative | What a container image is |
| [CloudEvents](https://cloudevents.io/) | CNCF specification | Event envelope, when many teams consume events |
| [Semantic Versioning](https://semver.org/) | Specification | Library versions |
| [gRPC core concepts](https://grpc.io/docs/what-is-grpc/core-concepts/) | Project documentation | Service-to-service RPC |
| [Reactive Streams](https://www.reactive-streams.org/) | Specification | The formal definition of back pressure |

## Official manuals

| Source | Use it for |
| --- | --- |
| [PostgreSQL transaction isolation](https://www.postgresql.org/docs/current/transaction-iso.html) | What each isolation level actually does, including where it is stricter than the SQL standard |
| [PostgreSQL explicit locking](https://www.postgresql.org/docs/current/explicit-locking.html) | `FOR UPDATE` and lock levels |
| [PostgreSQL indexes](https://www.postgresql.org/docs/current/indexes.html) | B-trees and when an index matches a query |
| [PostgreSQL high availability](https://www.postgresql.org/docs/current/high-availability.html) | Replication choices |
| [PostgreSQL backup](https://www.postgresql.org/docs/current/backup.html) | Base backups and point-in-time recovery |
| [Npgsql connection string](https://www.npgsql.org/doc/connection-string-parameters.html) | Pool settings on .NET |
| [.NET support policy](https://dotnet.microsoft.com/en-us/platform/support/policy/dotnet-core) | Which runtime is still patched. .NET 10 is the LTS to start on as of October 2026 |
| [Garbage collection fundamentals](https://learn.microsoft.com/en-us/dotnet/standard/garbage-collection/fundamentals) | The map: roots and generations. It does not cover the modes below |
| [Workstation and server GC](https://learn.microsoft.com/en-us/dotnet/standard/garbage-collection/workstation-server-gc) | Which mode, and what server GC spends |
| [Background garbage collection](https://learn.microsoft.com/en-us/dotnet/standard/garbage-collection/background-gc) | Generation 2 overlaps; generations 0 and 1 still pause |
| [Large object heap](https://learn.microsoft.com/en-us/dotnet/standard/garbage-collection/large-object-heap) | The 85,000-byte threshold, sweep versus compact. The long walkthrough is scoped to Windows |
| [DATAS](https://learn.microsoft.com/en-us/dotnet/standard/garbage-collection/datas) | Heap size tracks live data. On by default since .NET 9 |
| [GC config settings](https://learn.microsoft.com/en-us/dotnet/core/runtime-config/garbage-collector) | The knobs, including the LOH threshold and DATAS. Read before you set one |
| [HttpClient guidelines](https://learn.microsoft.com/en-us/dotnet/fundamentals/networking/http/httpclient-guidelines) | Lifetime and sockets |
| [IHttpClientFactory](https://learn.microsoft.com/en-us/dotnet/core/extensions/httpclient-factory) | The ASP.NET Core way to share clients |
| [Object pool](https://learn.microsoft.com/en-us/aspnet/core/performance/objectpool) | Pooling expensive objects. The older `dotnet/core/extensions/object-pool` URL is a 404 |
| [Managed threading](https://learn.microsoft.com/en-us/dotnet/standard/threading/managed-threading-basics) | Threads on .NET. It does not teach `async` |
| [Debug thread-pool starvation](https://learn.microsoft.com/en-us/dotnet/core/diagnostics/debug-threadpool-starvation) | What a blocked pool looks like |
| [EF Core concurrency](https://learn.microsoft.com/en-us/ef/core/saving/concurrency) | A version token in the ORM |
| [ASP.NET Core best practices](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/best-practices) | Hot-path habits. Re-read them for the version you run |
| [ASP.NET Core memory](https://learn.microsoft.com/en-us/aspnet/core/performance/memory?view=aspnetcore-10.0) | The pictures: a static leak, server versus workstation, the 85,000-byte cliff, `HttpClient` ports. .NET 10 view |
| [.NET microservices architecture guide](https://learn.microsoft.com/en-us/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/) | Applied DDD and CQRS |
| [Persistence layer](https://learn.microsoft.com/en-us/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/infrastructure-persistence-layer-design) | Mapping at the edge of the domain |
| [Tactical DDD chapter](https://learn.microsoft.com/en-us/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/ddd-oriented-microservice) | Entity, value object, aggregate |
| [Azure architecture styles](https://learn.microsoft.com/en-us/azure/architecture/guide/architecture-styles/) | A catalog of styles |
| [Azure event-driven style](https://learn.microsoft.com/en-us/azure/architecture/guide/architecture-styles/event-driven) | Events, in the same catalog |
| [Cache-aside](https://learn.microsoft.com/en-us/azure/architecture/patterns/cache-aside) | The cache pattern to implement first |
| [OpenTelemetry documentation](https://opentelemetry.io/docs/) | Logs, metrics, and traces |
| [OWASP Top 10:2025](https://top10.owasp.org/2025/) | Web application risk categories. [A01 Broken Access Control](https://top10.owasp.org/2025/A01_2025-Broken_Access_Control/), [A03 Supply Chain](https://top10.owasp.org/2025/A03_2025-Software_Supply_Chain_Failures/) |
| [OWASP ASVS](https://owasp.org/www-project-application-security-verification-standard/) | A deeper verification bar than the Top 10 |
| [OWASP authentication cheat sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html) | Login |
| [OWASP session cheat sheet](https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html) | Cookie flags |
| [OWASP GenAI LLM Top 10, 2026](https://genai.owasp.org/resource/owasp-genai-llm-top-10-2026/) | Only if you call a model |
| [OWASP agentic Top 10](https://genai.owasp.org/resource/owasp-top-10-for-agentic-applications-for-2026/) | Only if the system takes actions |
| [Kafka delivery semantics](https://kafka.apache.org/documentation/#semantics) | What Kafka means by exactly-once, and what it does not cover |
| [Elastic Load Balancing user guide](https://docs.aws.amazon.com/elasticloadbalancing/latest/userguide/what-is-load-balancing.html) | Health checks and L4/L7 |
| [Docker: what is a container](https://docs.docker.com/get-started/docker-concepts/the-basics/what-is-a-container/) | A plain introduction |
| [SLSA](https://slsa.dev/) | A maturity map for the build path |
| [CycloneDX](https://cyclonedx.org/) | A software bill of materials |
| [DORA metrics](https://dora.dev/guides/dora-metrics/) | Four delivery measures |
| [Conventional Commits](https://www.conventionalcommits.org/) | A commit convention, if the repo uses it |

## Canonical articles and pattern pages

| Source | Use it for |
| --- | --- |
| [The Scale Cube](https://microservices.io/articles/scalecube.html) | X, Y, and Z |
| [Microservices pattern](https://microservices.io/patterns/microservices.html) | Definition and costs |
| [Transactional outbox](https://microservices.io/patterns/data/transactional-outbox.html) | Publishing without a distributed transaction |
| [Idempotent consumer](https://microservices.io/patterns/communication-style/idempotent-consumer.html) | Duplicate delivery |
| [Saga](https://microservices.io/patterns/data/saga.html) | Local transactions and compensations |
| [Please stop calling databases CP or AP](https://martin.kleppmann.com/2015/05/11/please-stop-calling-databases-cp-or-ap.html) | The CAP correction |
| [PACELC](https://en.wikipedia.org/wiki/PACELC_theorem) and Abadi's [paper](https://www.cs.umd.edu/~abadi/papers/abadi-pacelc.pdf) | The normal-operation tradeoff. The paper is the source. The summary is the page that is easier to open |
| [Bounded Context](https://martinfowler.com/bliki/BoundedContext.html) | Strategic DDD in one page |
| [CQRS](https://martinfowler.com/bliki/CQRS.html) | When the split pays, and when it does not |
| [Monolith First](https://martinfowler.com/bliki/MonolithFirst.html) | Earning a distributed system |
| [What do you mean by "Event-Driven"?](https://martinfowler.com/articles/201701-event-driven.html) | The several meanings of the phrase |
| [Inversion of Control](https://martinfowler.com/bliki/InversionOfControl.html) | The Hollywood principle, in Fowler's words |
| [Repository](https://martinfowler.com/eaaCatalog/repository.html) and [Unit of Work](https://martinfowler.com/eaaCatalog/unitOfWork.html) | Persistence patterns |
| [The Practical Test Pyramid](https://martinfowler.com/articles/practical-test-pyramid.html) | How many tests at each layer |
| [Idempotent Receiver](https://www.enterpriseintegrationpatterns.com/patterns/messaging/IdempotentReceiver.html) | The messaging pattern |
| [Enterprise Integration Patterns](https://www.enterpriseintegrationpatterns.com/) (Hohpe, Woolf) | Message shapes |
| [Timeouts, retries, and backoff with jitter](https://aws.amazon.com/builders-library/timeouts-retries-and-backoff-with-jitter/) | Retry storms |
| [Service Level Objectives](https://sre.google/sre-book/service-level-objectives/) | SLI, SLO, error budget |
| [Alerting on SLOs](https://sre.google/workbook/alerting-on-slos/) | Pages that mean something |
| [Postmortem culture](https://sre.google/sre-book/postmortem-culture/) | Blameless write-ups |
| [Code review guide](https://google.github.io/eng-practices/review/) | Review tone and scope |
| [ADR](https://adr.github.io/) | Decision records |
| [Hexagonal architecture](https://alistair.cockburn.us/hexagonal-architecture/) (Alistair Cockburn) | Ports and adapters, from the author |
| [The Testing Trophy](https://kentcdodds.com/blog/the-testing-trophy-and-testing-classifications) | Put tests where the risk is |
| [Stripe idempotent requests](https://docs.stripe.com/api/idempotent_requests) | The concrete idempotency contract to copy. The header name is widespread. The IETF draft has expired |
| [What is event-driven architecture?](https://aws.amazon.com/event-driven-architecture/) | A short industry overview |
| [What is service-oriented architecture?](https://www.ibm.com/think/topics/soa) | The contract, the registry, and the enterprise service bus. Start here for SOA |
| [SOA versus microservices](https://www.ibm.com/think/topics/soa-vs-microservices) | Scope, reuse, and shared data. Skip the product pitches |
| [Service-oriented architecture (.NET guide)](https://learn.microsoft.com/en-us/dotnet/architecture/microservices/architect-microservice-container-applications/service-oriented-architecture) | A one-page contrast from the 2018 book. Companion only |
| [Twelve-factor](https://12factor.net/) | Config, logs, and processes |
| [Testcontainers](https://testcontainers.com/) | Disposable databases in tests |
| [Pact documentation](https://docs.pact.io/) | Consumer-driven contracts. Project home: [pact.io](https://pact.io/) |
| [How async/await really works](https://devblogs.microsoft.com/dotnet/how-async-await-really-works/) (Stephen Toub) | The state machine. This is the async study, not the threading overview |
| [ConfigureAwait FAQ](https://devblogs.microsoft.com/dotnet/configureawait-faq/) (Stephen Toub) | When `ConfigureAwait(false)` matters |
| [ValueTask](https://devblogs.microsoft.com/dotnet/understanding-the-whys-whats-and-whens-of-valuetask/) | When the result is often already complete |
| [ASP.NET Core diagnostic scenarios](https://github.com/davidfowl/AspNetCoreDiagnosticScenarios) (David Fowler) | Sync-over-async, timeouts, and the pool, in a real host |
| [Preparing for the .NET 10 GC](https://devblogs.microsoft.com/dotnet/preparing-for-dotnet-10-gc/) (Maoni Stephens) | Why a .NET 10 memory graph moved, and when to turn DATAS off |
| [.NET memory performance analysis](https://github.com/Maoni0/mem-doc/blob/master/doc/.NETMemoryPerformanceAnalysis.md) (Maoni Stephens) | How to read a collection. The hours-of-study note |
| [SOLID introduction](https://www.digitalocean.com/community/conceptual-articles/s-o-l-i-d-the-first-five-principles-of-object-oriented-design/) | A readable secondary explanation. Martin's book below is the source |

## Books worth owning

These do not go stale the way a blog does. Editions matter. Prefer the latest edition you can get.

- Martin Kleppmann, *Designing Data-Intensive Applications*. [dataintensive.net](https://dataintensive.net/). Data models, storage engines, replication, partitioning, transactions.
- Erich Gamma, Richard Helm, Ralph Johnson, John Vlissides, *Design Patterns*. Strategy, Adapter, Decorator, Factory, and composition.
- Martin Fowler, *Patterns of Enterprise Application Architecture*. Repository and unit of work.
- Eric Evans, *Domain-Driven Design*. Strategic design and the tactical vocabulary.
- Vaughn Vernon, *Implementing Domain-Driven Design*. The practical context map.
- Michael Nygard, *Release It!* Timeouts, circuit breakers, bulkheads.
- Sam Newman, *Building Microservices*. The book-length version of the microservices tradeoff.
- Len Bass, Paul Clements, Rick Kazman, *Software Architecture in Practice*. Component-and-connector views.
- Robert C. Martin, *Agile Software Development, Principles, Patterns, and Practices*. The SOLID principles.
- Craig Larman, *Applying UML and Patterns*. GRASP. Optional.
- Jones, Hosking, and Moss, *The Garbage Collection Handbook*. The language-neutral collector. When the manual is not enough.
- Konrad Kokosa, *Pro .NET Memory Management*. The .NET memory book: generations, the large object heap, and how to read a dump. Pair it with Maoni Stephens' analysis note above.
- Gregor Hohpe and Bobby Woolf, *Enterprise Integration Patterns*.
- Peter Van Roy, [Programming Paradigms for Dummies](https://www.info.ucl.ac.be/~pvr/VanRoyChapter.pdf). A free chapter, not a full book. Enough for the paradigm map.
- Alex Xu, *System Design Interview*. Extra design reps after the six in this repo. Use it as practice, and check its product advice against the standards above.

## Industry practice that is not a standard

- **Idempotency-Key.** Stripe, Adyen, and many others implement it. The IETF draft expired in April 2026. Document the behavior you implement: same key and same payload returns the stored result, same key and different payload is an error, and you publish the retention.
- **Rate-limit response headers** such as `RateLimit-Remaining` are a common convention. `429` and `Retry-After` are the part that is actually in HTTP.
- **OAuth 2.1** is the draft that deletes the unsafe flows. RFC 9700 is the document you can cite today.

## Companion guides

These are separate repos. This roadmap links them where the detail belongs. It does not copy them.

| Source | Use it for |
| --- | --- |
| [dsa-guide](https://github.com/xolghadr/dsa-guide) | Structures, cost, graphs, and which structure fits, in C#. The coding-puzzle practice this roadmap leaves out |
| [ef-core-review](https://github.com/xolghadr/ef-core-review) | EF Core in depth: `DbContext` lifetime, tracking, split queries, concurrency tokens, migrations. Baseline EF Core 8, with newer behavior marked. The PostgreSQL manual stays the database source |

## Dropped from the old outline

These were in the July 2024 README. They are not start-here sources anymore.

- GeeksforGeeks pages on OOP, SOLID, and GRASP. They are definition lists. The books and the topic pages replace them.
- Educative and freeCodeCamp intros, and YouTube "what is UML" videos. Fine as a first exposure, weak as a reference.
- Medium posts on microservices, CAP, eventual consistency, back pressure, the Hollywood principle, and component-based architecture. Several restated a better page you can read directly (microservices.io, Kleppmann, Fowler, Reactive Streams).
- A doubled slash on the Microsoft CQRS printer-friendly URL. Replaced by [Fowler's CQRS article](https://martinfowler.com/bliki/CQRS.html).
- "TSL" as a security topic. The protocol is TLS. HTTPS is HTTP over TLS. See [RFC 9325](https://www.rfc-editor.org/rfc/rfc9325.html).
- A bare "LET or SQL" on the CQRS query side. The query side is SQL or a language query such as LINQ. The topic page says so.
