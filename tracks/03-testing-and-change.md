# 3. Testing and change

A senior ships a change that can be checked, reviewed, and undone. The tests, the pull request, and the way you debug are one skill.

## Tests

**Level:** Must-have

**You can:** choose the cheapest test that would have caught the bug, and say what a weak test looks like.

### In one minute

- A **unit test** checks one behavior in memory. No network, no real database. It is fast and it points at the line that broke.
- An **integration test** checks your code with a real collaborator you do not fully control, usually a database or a queue. Run it against a disposable database, not against a shared dev server. [Testcontainers](https://testcontainers.com/) is a current way to start that database in the test run.
- An **end-to-end test** walks a user journey through the running system. Keep a few of these. They are slow and they fail for many reasons at once.

The **test pyramid** (Mike Cohn, explained by Martin Fowler) says: many unit tests, fewer integration tests, very few end-to-end tests. The shape is about cost and feedback time.

There is a useful correction. If the bugs you ship live in SQL, in a transaction, or in a query plan, a mocked database will not catch them. For a service, a smaller set of **integration tests around the database** is often worth more than a thousand tests that only check a mock was called. Kent C. Dodds argues this from the front-end side as the testing trophy. The lesson for backends is the same: test at the boundary where the risk is.

A strong test names the behavior: "rejects a second reservation of the same seat." A weak test asserts that a method on a mock was called with a particular argument and never checks the outcome.

### When it matters

Every pull request. Also the interview question "how would you test this?" which is really "where is this likely to break?"

### What it costs

Tests are code. A suite that takes forty minutes will be skipped. A suite that mocks every collaborator will stay green while the SQL is wrong. Spend the slow tests on the boundaries that move money, inventory, or permissions.

### A failure you should recognize

A production bug where two requests sold the last seat. The unit tests passed because the repository was a fake dictionary that could not represent a race. The missing test is an integration test with two transactions against a real database.

### Learn

Start here: Martin Fowler, [The Practical Test Pyramid](https://martinfowler.com/articles/practical-test-pyramid.html).

Also: Kent C. Dodds, [The Testing Trophy](https://kentcdodds.com/blog/the-testing-trophy-and-testing-classifications), for the argument about where tests pay off.

### Review

1. What bug does a unit test catch that an end-to-end test catches more slowly?
2. When is a mock of the database the wrong double?
3. What does the name of a strong test describe?

### In an interview

Say the risk, then the layer. "The race is in the transaction, so I want an integration test with two concurrent updates. The pricing rule is pure, so I want a unit test with a table of examples."

### Exercise

Add one test that fails if a business rule breaks, using real inputs and an output you can see. If you already have only mock-call tests around a repository, add one test that runs the query.

## Git and code review

**Level:** Must-have

**You can:** open a small pull request that explains why, and review someone else's change for behavior, risk, and operability.

### In one minute

`main` (or `trunk`) stays deployable. Work lands in small pull requests. A review happens before merge. The pipeline must be green.

A pull request description answers three questions:

- What changed for the user or the operator?
- Why this approach?
- How did you check it?

The diff should be readable in one sitting. A 2,000-line change hides the risky twenty lines. Split mechanical moves (rename, format) from behavior changes.

**Trunk-based development** means short-lived branches, often less than a day, merged to one main line. Long-lived feature branches drift and then merge in a panic. A feature flag can hide unfinished behavior on the main line if the merge has to happen early. Flags are a good-to-have in the architecture track.

[Conventional Commits](https://www.conventionalcommits.org/) (`feat:`, `fix:`, `docs:`) are a clear convention. They are not a moral rule. Use them if the repo uses them. Write the subject in the imperative, and say why in the body when the subject cannot.

Review looks for:

- The behavior matches the intent.
- Tests cover the risk, including the failure path.
- Authorization is checked on the server for the object being touched.
- Migrations are safe to run while the old code is still live (see [migrations](04-data-and-apis.md#schema-migrations)).
- Logs and errors will be enough to debug this at 2 a.m.
- The public API change is backward compatible, or it is explicitly a breaking change.

Google's review guide is the best short manual of tone and scope: [Engineering Practices: Code Review](https://google.github.io/eng-practices/review/).

As a reviewer, be specific. "What happens if the payment call times out?" is a review. "I don't like this" is a mood. As an author, answer by changing the code or by explaining the constraint. Resolve the thread in the diff, not in a side chat that leaves no record.

### When it matters

Every change. Also the interview story "tell me about a review you gave that prevented an incident."

### What it costs

Review latency is real. Small requests keep it short. A review culture that nitpicks style and misses the missing transaction is expensive in the other direction. Automate formatting. Spend human attention on behavior.

### A failure you should recognize

A migration that drops a column in the same deploy as the code that stops reading it. During a rolling deploy the old instances still read the column. The safe shape is expand, then switch, then contract. That sequence is in the data track. A reviewer who asks for it is doing the job.

### Learn

Start here: [Google's code review guide](https://google.github.io/eng-practices/review/).

### Review

1. What three questions does the pull request description answer?
2. Why do you split a rename from a behavior change?
3. Name two things you check that a formatter cannot check.

### In an interview

Describe a review comment that changed the design, not the whitespace. Mention the failure you were preventing.

### Exercise

Take a large diff you wrote recently and mark a seam where it could have been two pull requests. Write the two descriptions.

## Debugging

**Level:** Must-have

**You can:** move from a symptom to a cause with evidence, one hypothesis at a time.

### In one minute

1. Write the symptom in one sentence, including who is affected and since when.
2. Reproduce it, or gather the trace, log, and inputs that are as good as a reproduction.
3. State a hypothesis before you change anything. "The new index is not used when the parameter is null."
4. Change one thing, or add one observation. Check the hypothesis.
5. Fix the cause. Add a test or an alert that would have caught it.

Locally, a debugger shows you state. In production, you have logs, metrics, and traces. A correlation id (a trace id) is what lets you follow one request across services. That id is part of [observability](06-production.md#observability-and-slos).

Bisect when you do not have a hypothesis. When did it last work? Which deploy, flag, or data change sits in between?

"It works on my machine" usually means the data, the clock, the config, or the race is different. Make the difference explicit.

### When it matters

Incidents, flaky tests, and any bug report that says "sometimes."

### What it costs

Guessing without a hypothesis creates extra changes that hide the cause. A two-hour fix with three unrelated refactors is harder to trust than a twenty-line fix with a test.

### A failure you should recognize

A null reference in production that no one can reproduce. The log line does not include the request id or the input id. You cannot tell which order it was. The fix for the next one is to log the id at the boundary. The fix for this one is to find a request that failed at that timestamp and walk backward.

### Learn

Start here: the loop above. Pair it with the postmortem chapter when the bug is an incident: [Postmortem culture](https://sre.google/sre-book/postmortem-culture/) in the Google SRE book.

### Review

1. What do you write down before you edit code?
2. What replaces a debugger in production?
3. What does a correlation id let you do?

### In an interview

Walk a real bug with the loop: symptom, hypothesis, evidence, fix, test. Skip the part where you were heroic. Include the wrong hypothesis you discarded.

### Exercise

Take a flaky or confusing bug you remember. Write the hypothesis you wish you had started with, and the single log line that would have confirmed it.

## Good to have

### Contract tests

**Level:** Good-to-have

**You can:** explain a consumer-driven contract in two sentences.

A consumer-driven contract is a test that records the requests a client makes and the responses it needs. The provider runs that contract in its pipeline. The client's assumptions break in the provider's build, before production.

[Pact](https://docs.pact.io/) is the tool people mean. Use it when several teams share an API and integration environments are scarce. Skip it when one team owns both sides and already runs an integration test against the real API. The documentation is the reference. The project home is [pact.io](https://pact.io/).

### Property-based tests

**Level:** Good-to-have

**You can:** say what a property is, as opposed to an example.

An example test checks one input. A property test checks a rule over many generated inputs: "sorting the list twice equals sorting it once," or "decode(encode(x)) equals x." The library (FsCheck on .NET, QuickCheck in Haskell, Hypothesis in Python) shrinks a failure down to a small input.

Use a property when the rule is clear and the input space is large. Keep example tests for the business cases a reader should see at a glance.
