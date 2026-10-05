# Interview index

Use this for the 10-day interview pass. Sit the prompt for the time in the table. Then answer the review questions on the linked pages.

A strong answer has a number, a rejected option, and a failure. A weak answer has only boxes and product names.

Say your assumptions before you draw. If the interviewer changes a number, change the design on purpose and say what moved.

## How to run a design prompt

45 minutes is the usual slot. Spend it like this:

1. **5 minutes.** Ask what the system must do, and what it does not have to do. Ask for users, read/write shape, and freshness.
2. **5 minutes.** Estimate out loud. Decide whether this is a correctness problem or a volume problem.
3. **20 minutes.** Draw the path. Label sync and async. Name the data store and the partition key if you have one.
4. **10 minutes.** Walk one failure: a timeout, a duplicate message, a dead process. Say what the user sees.
5. **5 minutes.** Say what you would measure (one SLI).

Low-level design prompts are shorter. 20 to 30 minutes. Talk in types and relationships, then one sequence diagram.

## System design prompts

| Prompt | Time | Use these pages | A strong answer includes |
| --- | --- | --- | --- |
| Design a URL shortener | 45 min | [Design](../designs/url-shortener.md), [cache](../tracks/05-architecture.md#caching), [estimate](../tracks/05-architecture.md#capacity-estimation) | Peak reads far above writes, 302 not 301, random code, cache key and TTL, click count off the redirect path |
| Design a home feed | 45 min | [Design](../designs/activity-feed.md), [consistency](../tracks/05-architecture.md#consistency) | Hybrid fan-out, celebrity called out with a number, partition by user, author sees their own post immediately |
| Design chat | 45 min | [Design](../designs/chat.md), [communication](../tracks/05-architecture.md#communication-patterns) | Connections versus history, sequence per conversation, partition by conversation, presence with a TTL, catch-up after reconnect |
| Design a rate limiter | 35 min | [Design](../designs/rate-limiter.md), [back pressure](../tracks/05-architecture.md#back-pressure) | Shared counter, token bucket, 429 and Retry-After, fail-open choice, why a database row per request is the wrong tool |
| Design large file upload | 45 min | [Design](../designs/file-upload.md), [HTTP](../tracks/04-data-and-apis.md#http-apis) | Pre-signed URL, bytes skip the API, status for the scan, short-lived download URL, authorization on the metadata |
| Design checkout | 45 min | [Design](../designs/order-checkout.md), [outbox](../tracks/05-architecture.md#outbox-inbox-and-the-idempotent-consumer), [sagas](../tracks/05-architecture.md#sagas) | This is correctness, not sharding, at the stated volume. Outbox plus payment idempotency key. Timeout is not a decline. Conditional status update |
| Design a seat reservation | 35 min | [Transactions](../tracks/04-data-and-apis.md#transactions-and-isolation), [concurrency](../tracks/04-data-and-apis.md#optimistic-and-pessimistic-concurrency) | One seat cannot be confirmed twice. Version or a single-statement update. What the loser sees (409). Hold expiry |
| Add a cache to a slow read API | 25 min | [Caching](../tracks/05-architecture.md#caching), [consistency](../tracks/05-architecture.md#consistency) | Cache-aside, key, TTL, delete on write, stampede, what you refuse to cache |
| The primary database is the bottleneck | 35 min | [Scale cube](../tracks/05-architecture.md#the-scale-cube), [replication](../tracks/05-architecture.md#replication-and-partitioning), [estimate](../tracks/05-architecture.md#capacity-estimation) | Say whether the pain is reads or writes. Replicas for reads. Partition key taken from the query. More API boxes do not fix write saturation |
| Split a monolith | 35 min | [Choosing a style](../tracks/05-architecture.md#choosing-a-style), [microservices](../tracks/05-architecture.md#microservices), [sagas](../tracks/05-architecture.md#sagas) | One module, one reason, data moves with it. What used to be a transaction becomes a saga. What you leave behind |
| Notifications for several consumers | 30 min | [Events](../tracks/05-architecture.md#event-driven-architecture), [delivery](../tracks/05-architecture.md#delivery-guarantees), [outbox](../tracks/05-architecture.md#outbox-inbox-and-the-idempotent-consumer) | Event versus command. At-least-once. Outbox. Consumer dedupes. Publisher does not list consumers |
| Multi-region users | 30 min | [Multi-region](../tracks/05-architecture.md#multi-region), [CAP](../tracks/05-architecture.md#cap-and-pacelc) | Active-passive first. If writes happen in two regions, give a user a home region or a conflict rule. Do not label a database "AP" and stop |

## API and data prompts

| Prompt | Time | Use these pages | A strong answer includes |
| --- | --- | --- | --- |
| Design `POST /orders` so retries are safe | 20 min | [HTTP](../tracks/04-data-and-apis.md#http-apis), [checkout](../designs/order-checkout.md) | Idempotency key, stored result, same key plus different body is an error, how long you keep the key. The IETF header draft is expired. Stripe's behavior is the concrete reference |
| Two editors save the same ticket | 15 min | [Optimistic concurrency](../tracks/04-data-and-apis.md#optimistic-and-pessimistic-concurrency) | Version column, 409 or 412, the loser reloads. A lock held while a human thinks is the wrong tool |
| A list endpoint got slow | 20 min | [Indexes](../tracks/04-data-and-apis.md#indexes), [N+1](../tracks/04-data-and-apis.md#n1-queries-and-connection-pools) | Look at the queries first. N+1 or a missing left-prefix index. `EXPLAIN`. The new index's write cost |
| Rename a hot column with no downtime | 15 min | [Migrations](../tracks/04-data-and-apis.md#schema-migrations) | Expand, write both, backfill in batches, read new, contract later. Old instances stay up |
| What isolation do you want for a balance update? | 15 min | [Transactions](../tracks/04-data-and-apis.md#transactions-and-isolation) | Name the database. Lost update with numbers. Single-statement increment, or a version, or serializable with a retry |

## Low-level design prompts

| Prompt | Time | Use these pages | A strong answer includes |
| --- | --- | --- | --- |
| Design a parking-lot or library model | 30 min | [Relationships](../tracks/02-code-design.md#relationships-between-types), [diagrams](../tracks/02-code-design.md#diagrams-people-can-read) | A few types, ownership (composition versus association), one sequence for the main action, one invariant |
| Add a second payment provider | 20 min | [Adapter](../tracks/02-code-design.md#patterns-you-will-actually-use), [hexagonal](../tracks/05-architecture.md#hexagonal-architecture-ports-and-adapters) | An interface you own, one adapter each, the vendor types do not leak, a test with a fake |
| Several pricing rules | 15 min | [Strategy](../tracks/02-code-design.md#patterns-you-will-actually-use) | Name the second and third rule. One interface. Say you would not add the pattern for a single rule |
| How do you test a charge? | 15 min | [Tests](../tracks/03-testing-and-change.md#tests) | Pure rule as a unit test. The race or the SQL as an integration test. The mock that only asserts a call is the weak test |

## Production and judgment prompts

| Prompt | Time | Use these pages | A strong answer includes |
| --- | --- | --- | --- |
| Walk me through login and a later API call | 20 min | [Auth](../tracks/06-production.md#authentication-and-authorization) | Authentication versus authorization on the object. Authorization code plus PKCE. No implicit flow. Cookie flags or a short-lived token, and why a JWT in local storage is sharp |
| The dependency is timing out | 20 min | [Resilience](../tracks/05-architecture.md#resilience) | A timeout you chose, retry only if idempotent, backoff and jitter, a breaker, what the user sees |
| How do you know this is failing at 2 a.m.? | 15 min | [Observability](../tracks/06-production.md#observability-and-slos) | One SLI and one SLO. Trace id across the queue. An alert on symptoms, not on a single CPU graph |
| Tell me about an incident | 10 min | [Stories](../tracks/07-senior-judgment.md#stories-for-interviews), [incidents](../tracks/06-production.md#incidents) | User impact first, what you did, the lasting change. No villain |
| Tell me about a design tradeoff | 10 min | [Stories](../tracks/07-senior-judgment.md#stories-for-interviews) | The option you rejected, and the cost of the one you took |
| How long would checkout take to build? | 10 min | [Estimation](../tracks/07-senior-judgment.md#estimation) | A range and the assumption that moves it. Offer a smaller slice |
| How does a change reach production? | 10 min | [CI and containers](../tracks/06-production.md#ci-cd-and-containers) | Pipeline gates, one image promoted, old and new versions overlap, secrets not in the image |
| How do you not lose the orders? | 10 min | [Backups](../tracks/06-production.md#backups-and-recovery) | A replica is not a backup. RPO, RTO, and a restore you have actually done |

## A one-week order

If you have five weekdays, do not try to reread every page.

1. Checkout design, outbox, transactions. Say them out loud.
2. HTTP idempotency, concurrency, migrations.
3. URL shortener and cache. Feed fan-out.
4. Chat ordering. Rate limiter. Resilience.
5. Auth, one incident story, one tradeoff story, one estimate.

## Coding puzzles

This index does not cover algorithm puzzles. Practice those on the set you already use. When a puzzle ends, you should still be able to say the cost of your solution in time and memory, and whether the approach would survive a million rows. That sentence is the bridge back to this roadmap.
