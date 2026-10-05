# 4. Data and APIs

Most senior backend work is a transaction, a query, or an HTTP contract. This track is the part candidates skip, and the part incidents come from.

Examples use PostgreSQL because its isolation behavior is documented and slightly different from the textbook table. Say which database you mean in an interview. MySQL and SQL Server do not match PostgreSQL line for line.

## Schema and modeling

**Level:** Must-have

**You can:** design a small schema from the access path, and say what is unique, what is required, and what is history.

### In one minute

Start from the questions the application asks, not from a list of nouns.

- A **primary key** identifies a row. Prefer a key that does not change. A surrogate key (a number, or a UUID) is fine. A natural key (an email) often changes, so it is a better unique constraint than a primary key.
- **UUID version 7** (RFC 9562) is time-ordered. As a primary key it clusters better in a B-tree than random UUID version 4. Use version 4 when you do not want the id to leak creation time, for example an unguessable token. An id is not a secret capability by itself. Check authorization on every read.
- A **foreign key** is a constraint the database enforces. Use it unless you have a measured reason not to (some high-volume append-only tables skip it and enforce the rule in a careful write path).
- **Normalization** means a fact lives in one place. A user's name lives on the user, not copied onto every order, unless you need the name *as it was* when the order was placed. That copy is history, and it is deliberate.
- **Null** means "no value," and it complicates every query. Use it for a value that is genuinely optional. Do not use it for "unknown status." Use an explicit status.

### When it matters

Every new feature that stores something. Also the interview prompt "design the tables for this."

### What it costs

A fully normalized model can require joins you did not budget for. A denormalized copy can go stale. Pick the copy when you are recording a fact at a moment in time (the price the customer paid). Keep one source of truth when the value must move everywhere together (the customer's current email).

### A failure you should recognize

An `orders` table with `item1`, `item2`, `item3` columns. The fourth item requires a migration and a code change. The items are a child table.

### Learn

Start here: a database book you will actually finish. *Designing Data-Intensive Applications* by Martin Kleppmann, chapter 2, is the right chapter for data models. The book's site is [dataintensive.net](https://dataintensive.net/).

For identifiers: [RFC 9562](https://www.rfc-editor.org/rfc/rfc9562.html) (UUID).

### Review

1. When do you copy a customer's name onto an order, and when do you join it?
2. Why is UUID version 7 kinder to a B-tree than version 4?
3. What does a foreign key give you that application code does not, the moment a second writer appears?

### In an interview

Talk through one read path and one write path before you draw tables. Then add the unique constraint you are afraid to forget.

### Exercise

Model a seat reservation: show, seat, user, status, expiry. Write the unique rule that stops two confirmed reservations for the same seat.

## Indexes

**Level:** Must-have

**You can:** add an index for a real query, and explain why an extra index slows writes.

### In one minute

An **index** is a structure the database searches instead of reading the whole table. The default in PostgreSQL and in most relational databases is a **B-tree**. It supports equality and range (`=`, `<`, `BETWEEN`, `ORDER BY` on that key).

A **composite index** on `(account_id, created_at)` serves filters that start with `account_id`. It does not efficiently serve a filter on `created_at` alone. The left prefix is the part you can use.

A **covering index** holds every column the query needs, so the database can answer from the index alone.

**Selectivity** is how many rows a condition throws away. An index on a boolean column rarely helps. An index on a high-cardinality id often does.

Every index speeds some reads and slows writes, because every insert and update must maintain it. Index the queries you run, not every column you have.

Read the plan (`EXPLAIN` in PostgreSQL) before and after. A guess about indexes is a guess.

### When it matters

A page that was fine at 10,000 rows and times out at 10 million. Also any unique constraint, which is an index plus a rule.

### What it costs

Write amplification, disk, and the chance that the planner picks a bad index if statistics are stale. The fix for a slow query is sometimes a rewrite (a more selective predicate, a different join), not a new index.

### A failure you should recognize

`WHERE lower(email) = lower($1)` while the index is on `email`. The expression does not match the index. Index the expression you actually query, or store a normalized email.

### Learn

Start here: the indexing chapter of your database manual. For PostgreSQL, the [indexes introduction](https://www.postgresql.org/docs/current/indexes.html).

Kleppmann, chapter 3, explains B-trees and LSM-trees if you want the storage-engine view.

### Review

1. Which queries can use an index on `(account_id, created_at)`?
2. Why can indexes make a write slower?
3. What do you read before you declare that an index fixed the query?

### In an interview

Say the query, then the index, then the write cost. Mention `EXPLAIN` if you have used it.

### Exercise

Take a slow query you know. Write the index you would add, the queries it would *not* help, and one sentence on the write cost.

## Transactions and isolation

**Level:** Must-have

**You can:** say what a transaction guarantees, pick an isolation level, and name the anomaly you still accept.

### In one minute

A **transaction** is a group of reads and writes that commit together or roll back together. If the process dies in the middle, the database does not keep half of the group.

**ACID** is the classic label:

- **Atomicity.** All of the group, or none.
- **Consistency.** The database moves from one state that satisfies its rules to another. The application also has rules the database cannot see. You still have to write those.
- **Isolation.** Concurrent transactions do not see a tangled mix of each other's work. How tangled is allowed depends on the level.
- **Durability.** A committed transaction survives a crash.

**Isolation levels** in the SQL standard, and what PostgreSQL actually does (see the manual; this is the important part):

| Level in PostgreSQL | What you still have to think about |
| --- | --- |
| **Read committed** (the default) | A row another transaction has committed can change under you inside one transaction. Two transactions can read the same balance and both write "balance + 1", and one update is lost. Dirty reads do not happen. |
| **Repeatable read** | Your transaction sees a snapshot. Plain lost updates of one row are caught if you use the snapshot's expected row. **Write skew** can still happen: two transactions read overlapping data, write different rows, and together break a rule (two doctors on call, both going off). |
| **Serializable** | The database aborts one transaction if the result could not have happened in some serial order. In PostgreSQL this is Serializable Snapshot Isolation. You must retry aborted transactions. |

Other databases use the same words for different behavior. MySQL's repeatable read uses gap locks and prevents some phantoms that PostgreSQL's repeatable read allows. In an interview, name the database.

A **lost update** is the anomaly to be able to explain with numbers. Balance is 100. Two transactions read 100. Both add 20. Both write 120. The balance should be 140.

### When it matters

Inventory, balances, seats, any read-modify-write. Also any job that claims a row and must not claim it twice.

### What it costs

Stricter isolation aborts more transactions, and you write retry logic. Weaker isolation is faster and can lose an update. The cost of a lost seat is usually higher than the cost of a retry.

### A failure you should recognize

Application code that reads a counter, adds one in memory, and writes it back, on the default isolation level, with no single-statement `UPDATE ... SET balance = balance + 1` and no version check. Under concurrency the counter drifts.

### Learn

Start here: [PostgreSQL transaction isolation](https://www.postgresql.org/docs/current/transaction-iso.html). Read the section that compares levels to the SQL standard. PostgreSQL is stricter than the standard in some places and the page says so.

### Review

1. What does a transaction give you when the application crashes after the first of two writes?
2. Describe a lost update with a number.
3. What is write skew, and which PostgreSQL level still allows it?

### In an interview

"Two customers buy the last item." Walk through one transaction strategy, name the lost update, and say what the client does when the database aborts your serializable transaction (retry, or tell the user the item is gone).

### Exercise

Implement a counter increment both as a single `UPDATE balance = balance + 1` and as read-then-write. Run two sessions at once against the read-then-write version. Record the result.

## Optimistic and pessimistic concurrency

**Level:** Must-have

**You can:** choose a version check or a row lock, and map the failure onto an HTTP response.

### In one minute

This is concurrency of **data**, not of threads.

**Pessimistic concurrency** locks the row before changing it. In PostgreSQL, `SELECT ... FOR UPDATE` inside a transaction. Other transactions that want that row wait or fail. Use it when conflicts are common and the critical section is short. Hold the lock across a user think-time and you will stall the system. Hold it across a call to a payment provider and you will create an outage.

**Optimistic concurrency** assumes conflicts are rare. You remember a version (a number or an updated timestamp). The update says `WHERE id = $1 AND version = $2`, and it increments the version. If no row was updated, someone else wrote first. The loser reloads and tries again, or gives up.

**Last write wins** is what you get with no version and no lock: the later commit replaces the earlier one, silently. It is acceptable for a single-writer setting, a presence ping, or a cache. It is a bug for a bank balance or a shared document with two editors.

On HTTP, a failed optimistic check is often **409 Conflict** (the resource changed) or **412 Precondition Failed** when the client sent `If-Match` with an ETag. Pick one, document it, and use [Problem Details](https://www.rfc-editor.org/rfc/rfc9457.html) for the body. Tell the client to reload.

### When it matters

Any edit form that two people can open. Any worker that claims jobs. Any shopping cart line.

### What it costs

Pessimistic locking blocks and can deadlock. Optimistic locking wastes the work of the loser and needs a clear message in the UI. Last-write-wins is cheapest and sometimes wrong.

### A failure you should recognize

Two support agents open the same ticket, both add a note in a "notes" text column, both save. The second save erases the first note. A version column, or storing notes as child rows, keeps both.

### Learn

Start here: [PostgreSQL explicit locking](https://www.postgresql.org/docs/current/explicit-locking.html), and, if you use Entity Framework, [handling concurrency conflicts](https://learn.microsoft.com/en-us/ef/core/saving/concurrency).

HTTP preconditions are in [RFC 9110, section 13](https://www.rfc-editor.org/rfc/rfc9110.html).

### Review

1. Why is a row lock across a third-party HTTP call dangerous?
2. What does `WHERE version = $expected` return when you lose the race?
3. When is last-write-wins an acceptable product decision?

### In an interview

Say how often you expect conflicts. Rare conflicts point to a version column. A short, hot, correctness-critical section can take a lock. Name the status code the loser receives.

### Exercise

Add a `version` integer to a sample row. Write an update that changes the row only if the version matches, and returns a conflict result when it does not.

## Schema migrations

**Level:** Must-have

**You can:** change a schema while old and new application instances are both running.

### In one minute

Deploys are rolling. For a while, old code and new code share one database. A migration that only works with the new code will break the old instances, or the other way around.

Use **expand, then migrate, then contract**:

1. **Expand.** Add the new column or table as optional. Old code ignores it.
2. **Write both.** New code writes the old and the new shape.
3. **Backfill.** Copy existing rows, in batches, not in one giant transaction that locks the table.
4. **Read new.** Switch reads to the new shape once every writer is on new code.
5. **Contract.** Drop the old column in a later deploy, after nothing reads it.

Renames are an expand (add the new name) plus a contract (drop the old name), not a single rename, if old code is still live.

A lock that rewrites a huge table can stall production. Prefer adding a nullable column, or a column with a default that your database can store without rewriting every row. Check the database version's behavior before you assume a default is cheap. PostgreSQL 11 and later can add a constant default without rewriting the table. Confirm for the version you run.

### When it matters

Every change to a table that is in production.

### What it costs

Expand/contract takes more than one deploy. The cost of skipping it is an outage during release.

### A failure you should recognize

`ALTER TABLE ... ALTER COLUMN ... SET NOT NULL` on a large table while the site is up, with no backfill and no lock timeout. Writes queue behind the lock. The site looks "down" and the database is busy.

### Learn

Start here: the expand/contract sequence above. For PostgreSQL locking specifics, the [explicit locking](https://www.postgresql.org/docs/current/explicit-locking.html) page includes `ALTER TABLE` lock levels in the broader locking docs; also read your migrator's notes for the version you run.

### Review

1. Why is a column rename dangerous during a rolling deploy?
2. What is the order of expand, backfill, and drop?
3. Why do you backfill in batches?

### In an interview

If the design includes a migration, say that old code keeps working after the first step. That sentence is the senior signal.

### Exercise

Write the three deploys required to rename `full_name` to `display_name` with no downtime. List what each deploy's code reads and writes.

## N+1 queries and connection pools

**Level:** Must-have

**You can:** spot a query-per-row loop, and size a connection pool so the database stays the bottleneck you intended.

### In one minute

**N+1.** You load 100 orders with one query, then load the customer for each order with another query. That is 101 round trips. The fix is a join, or one batched query (`WHERE id IN (...)`), or a dataloader-style batcher. An ORM will do this cheerfully if you touch `order.Customer.Name` inside a loop.

**Connection pool.** Opening a database connection is slow and the database allows only so many. The application keeps a pool and borrows one per unit of work. The pool has a maximum.

If you run 20 instances with a pool of 100, you can open 2,000 connections. The database falls over before your app does. Size the pool from the database's max connections, divided across instances, with headroom for admin and migrations. A request that holds a connection while it calls a third-party HTTP API is holding a scarce permit for no reason. Fetch the rows, return the connection, then call the network.

### When it matters

A page that gets slower as the list gets longer, in a straight line. Or an incident where PostgreSQL reports too many connections while CPU is idle.

### What it costs

A join can return a wide result. A pool that is too small queues requests. A pool that is too large melts the database. Measure both.

### A failure you should recognize

A timeout spike after a scale-out. Each new instance added 100 connections. The database started refusing logins. The application logs "timeout," which hides "the pool never got a connection."

### Learn

Start here: your ORM's documentation on eager loading, plus the pool settings of the driver you use. For Npgsql, the [connection string pool options](https://www.npgsql.org/doc/connection-string-parameters.html) are the reference.

### Review

1. What does N+1 look like in a log of SQL statements?
2. How do you compute a safe pool size across many instances?
3. Why is it harmful to hold a pooled connection during an outbound HTTP call?

### In an interview

Mention both when you talk about a list endpoint. "I would load the children in one query, and I would not hold a connection while I call the payment service."

### Exercise

Enable SQL logging for a list page. Count the statements. If you see one per row, rewrite it as one query and count again.

## Persistence ignorance

**Level:** Must-have

**You can:** keep domain rules free of the database client, so you can test the rules without a server.

### In one minute

**Persistence ignorance** means the domain model does not know it is stored. An `Order` does not inherit a base class from the ORM, does not open a connection, and does not contain SQL. It holds data and rules. Something at the edge, often a repository implementation, maps between the object and the rows.

You can test "an order over the credit limit is rejected" by constructing an order. You do not need a database for that sentence.

This is a guide, not a law of physics. A simple CRUD screen can use a data record and a query with no domain model in between. Build the ignorant model when the rules are real and you are tired of testing them through the database.

### When it matters

Pricing, eligibility, state machines, anything with branches you regret testing only by clicking.

### What it costs

A mapping layer. If the mapping is as complex as the rules, you may have added a layer to a CRUD form. Stop and let the form be a form.

### A failure you should recognize

A domain method that calls `SaveChanges()` halfway through a rule. You cannot test the rule without a database, and a failure after the call has already committed a half-finished order.

### Learn

Start here: Martin Fowler's [Repository](https://martinfowler.com/eaaCatalog/repository.html), and the Microsoft architecture guide's [infrastructure persistence chapter](https://learn.microsoft.com/en-us/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/infrastructure-persistence-layer-design).

### Review

1. What is allowed inside a domain method, and what stays at the edge?
2. When is a domain model too much machinery for the feature?
3. How do you test a rule if the entity inherits an ORM base class that needs a context?

### In an interview

Show a small domain type with a rule, and a separate mapper or repository. Say that the unit test covers the rule and the integration test covers the mapping.

### Exercise

Move one `if` that encodes a business rule out of a controller or a repository and into a type that has no database import. Test that `if` directly.

## HTTP APIs

**Level:** Must-have

**You can:** design a small HTTP API with honest status codes, safe retries, a pagination story, and an error body other programs can read.

### In one minute

HTTP is the contract. [RFC 9110](https://www.rfc-editor.org/rfc/rfc9110.html) is the semantics standard. HTTP/1.1, HTTP/2, and HTTP/3 are ways to carry those semantics on the wire ([RFC 9112](https://www.rfc-editor.org/rfc/rfc9112.html), [RFC 9113](https://www.rfc-editor.org/rfc/rfc9113.html), [RFC 9114](https://www.rfc-editor.org/rfc/rfc9114.html)). You design the semantics. The platform picks the version.

**Methods.**

- **Safe** methods do not change server state: `GET`, `HEAD`, `OPTIONS`, `TRACE`.
- **Idempotent** methods can be repeated with the same intended effect as one call: the safe methods, plus `PUT` and `DELETE`.
- `POST` is neither safe nor idempotent. A retry can create a second resource.
- `PATCH` is not idempotent in general. A patch like "set status to cancelled" can be. A patch like "increment by one" is not. Say which one you implemented.

**Status codes you should actually use.**

| Code | Use it when |
| --- | --- |
| 200 | The request worked and there is a body. |
| 201 | A resource was created. Send `Location`. |
| 202 | The work was accepted and is not finished. |
| 204 | It worked and there is no body. |
| 400 | The request is malformed or fails validation. |
| 401 | The caller did not authenticate. |
| 403 | The caller is known and is not allowed. |
| 404 | There is no such resource, or you are hiding its existence. |
| 409 | The request conflicts with current state (a version mismatch, a duplicate). |
| 412 | A precondition such as `If-Match` failed. |
| 422 | The body is well-formed and the instruction cannot be carried out. Use 400 if you do not want the distinction. |
| 429 | The caller is over the limit. Send `Retry-After` when you can. |
| 500 | You failed and the caller cannot fix the request. |
| 502, 503, 504 | A dependency failed, you are unavailable, or a dependency timed out. |

**Errors.** Return [RFC 9457 Problem Details](https://www.rfc-editor.org/rfc/rfc9457.html) (`application/problem+json`) with a `type`, a `title`, and a `status`. Add a stable application code in an extension field if clients must branch. Do not make clients parse an English sentence.

**Pagination.** Offset (`?page=4`) is fine for a short, stable admin list. For a large or changing feed, use a **cursor**: the server returns an opaque token that means "after this row." Offset pages skip and repeat rows when new items are inserted at the top.

**Versioning.** Add optional fields without a version bump. Removing a field, renaming it, or changing its type is a breaking change. A new URL prefix (`/v2/`) is the blunt instrument and it is easy to explain. Whatever you choose, document the compatibility rule. Do not version by deploying on Tuesday and hoping.

**Idempotency keys.** Clients retry. For `POST` (create a charge, place an order), accept a client-generated key and store the result of the first request. The same key and the same payload returns the stored result. The same key and a different payload is an error. Publish how long you keep the key. Stripe's documentation is the clearest industrial description: [idempotent requests](https://docs.stripe.com/api/idempotent_requests).

The header name `Idempotency-Key` was drafted by the IETF HTTPAPI working group (`draft-ietf-httpapi-idempotency-key-header-07`). That draft **expired on 18 April 2026** and is not an RFC. Many APIs use the header anyway. Treat Stripe's behavior and your own docs as the contract, and treat the draft as background, not as a standard.

**Descriptions.** Publish an [OpenAPI](https://spec.openapis.org/oas/latest.html) document and review it like code. As of October 2026 the current specification is **3.2** (patch 3.2.1). It is additive over 3.1. A 3.1 document is still a valid description. Prefer 3.2 for a new API, especially if you need to describe streaming responses. The 3.1 line remains fine while your tools catch up.

**Caching.** `GET` responses can carry cache headers. [RFC 9111](https://www.rfc-editor.org/rfc/rfc9111.html) is the caching standard. Do not tell a shared cache to store a response that contains one user's private data.

### When it matters

Every endpoint another team or a mobile app will call. The API outlives the first client.

### What it costs

A careful contract feels slow on day one. A careless one becomes a compatibility project six months later, which is much slower.

### A failure you should recognize

A create endpoint that runs the charge on `POST` with no idempotency key. The client's timeout fires, the client retries, and the customer is charged twice. The payment succeeded both times. The bug is the missing key, not the timeout.

### Learn

Start here: [RFC 9110](https://www.rfc-editor.org/rfc/rfc9110.html), sections 9 (methods) and 15 (status codes). Skim, do not memorize every code.

Then [RFC 9457](https://www.rfc-editor.org/rfc/rfc9457.html) and [Stripe idempotent requests](https://docs.stripe.com/api/idempotent_requests).

### Review

1. Which methods may a client retry without an application-level key, and why?
2. What is the difference between 401 and 403?
3. Why does a feed use a cursor?
4. What is the status of the `Idempotency-Key` IETF draft as of 2026, and what do you follow instead?

### In an interview

Design one write endpoint and one list endpoint. Mention retries, the status code on conflict, and the error body. That is enough. A senior also mentions who is allowed to call it.

### Exercise

Specify `POST /orders` on one page: headers, body, success code, conflict code, problem-details example, and the idempotency rule including how long you keep the key.
