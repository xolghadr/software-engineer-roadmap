# 6. Production

A feature is finished when it can be secured, watched, released, and explained after it breaks.

Standards on this page were checked in October 2026.

## Transport security

**Level:** Must-have

**You can:** explain TLS, certificates, and HTTPS without mixing the three names up.

### In one minute

**TLS** (Transport Layer Security) is the protocol that gives you a private, intact channel between two parties. **TLS 1.3** is the current version, specified in [RFC 8446](https://www.rfc-editor.org/rfc/rfc8446.html). **TLS 1.2** is still acceptable when it is configured tightly. [RFC 9325](https://www.rfc-editor.org/rfc/rfc9325.html) is the IETF's best-current-practice for using TLS. SSL 2.0, SSL 3.0, TLS 1.0, and TLS 1.1 are obsolete. Do not enable them.

**HTTPS** is HTTP over TLS. It is not a third protocol beside TLS. It is HTTP using TLS as the transport.

A **certificate** binds a name (a DNS name) to a public key, and it is signed by an issuer the client already trusts. The client checks the name, the dates, and the signature chain. The **private key** stays on the server, or in a secret store, and never goes in git or in an image layer.

Terminate TLS at a load balancer you trust, or in the application. Between your own services, TLS (sometimes with client certificates, called mutual TLS) stops a random process on the network from reading the traffic. Inside a small trust zone some teams skip it. Be able to say which choice you made.

The word "SSL" survives in names (`sslmode`, "SSL certificate"). In new speech say TLS. If a screen still offers an SSL 3 or TLS 1.0 checkbox, leave it off.

### When it matters

Every public endpoint, every certificate renewal, and any design question about "how is this encrypted."

### What it costs

Certificate renewal is a job. Automate it. A manual renewal that depends on one person is an outage with a date on it. Encryption also costs a little CPU. That cost is not a reason to run plain HTTP on a public network.

### A failure you should recognize

A service that disables certificate validation to "fix" a call in development, and the setting ships. The channel still encrypts and no longer authenticates. Anyone who can intercept the connection can serve a certificate and read the traffic. Fix the trust store. Do not disable the check.

### Learn

Start here: [RFC 9325](https://www.rfc-editor.org/rfc/rfc9325.html) (recommendations for TLS). [RFC 8446](https://www.rfc-editor.org/rfc/rfc8446.html) is TLS 1.3 itself, for when you need the protocol.

### Review

1. What is the relationship between HTTPS and TLS?
2. What does a client check on a certificate?
3. Why is "turn off certificate validation" a production incident waiting to ship?

### In an interview

Say TLS 1.3, certificates that match the name, private keys out of git, and renewal as an automated task. Skip the handshake byte layout unless they ask.

### Exercise

Look at one deployed endpoint. Record the TLS version it negotiates and the certificate expiry date. Write down who gets paged before that date.

## Authentication and authorization

**Level:** Must-have

**You can:** separate "who are you" from "what may you do," and describe a current OAuth setup without the flows that were retired.

### In one minute

**Authentication** answers "who is calling?" **Authorization** answers "may this caller do this to this object?"

Broken access control is **A01:2025** in the [OWASP Top 10:2025](https://top10.owasp.org/2025/A01_2025-Broken_Access_Control/). It is the category that most often goes wrong. Check authorization on the server, on every request, against the object id in that request. A hidden field in the client is not a check. A 403 on the button and a working API is not a check.

**Browser session on your own site.** After login, set a session cookie: `Secure` (HTTPS only), `HttpOnly` (not readable from JavaScript), and `SameSite` (limits when the browser will send it on a cross-site request). The server stores the session, or the cookie holds a signed, short-lived token you can validate anywhere. OWASP's [session management cheat sheet](https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html) is the checklist.

**OAuth 2.0** ([RFC 6749](https://www.rfc-editor.org/rfc/rfc6749.html)) is delegated authorization: a user lets an application call an API without giving that application their password. **OpenID Connect** adds login on top, and an **ID token** about the user. An **access token** is for calling an API. They are different objects. The OpenID Connect core spec is the standard: [OpenID Connect Core 1.0](https://openid.net/specs/openid-connect-core-1_0.html).

**What to build in 2026.**

- Use the **authorization code** flow with **PKCE** ([RFC 7636](https://www.rfc-editor.org/rfc/rfc7636.html)). The client proves it started the login when it redeems the code.
- [RFC 9700](https://www.rfc-editor.org/rfc/rfc9700.html) (January 2025) is the OAuth security best current practice. It requires PKCE for public clients and recommends it for confidential clients. It tells you not to use the implicit flow or the resource-owner password flow.
- **OAuth 2.1** is a draft that bundles these rules (`draft-ietf-oauth-v2-1`). As of October 2026 it is **not** an RFC. The overview is at [oauth.net/2.1](https://oauth.net/2.1/). Follow RFC 9700 now. New code should use PKCE for every client, which is where the draft is headed.
- Keep access tokens short-lived. Prefer refresh tokens that rotate, or tokens bound to the client (mTLS or DPoP), as RFC 9700 describes.
- Validate the signature, the issuer (`iss`), the audience (`aud`), and the expiry (`exp`). A token meant for another API is not a token for yours.
- [JWT](https://www.rfc-editor.org/rfc/rfc7519.html) (RFC 7519) is a token format. It is not a session strategy. A JWT in `localStorage` is readable by any script that runs on your page, so an XSS bug becomes a stolen token. For a browser app on your own site, a `Secure` `HttpOnly` cookie is the simpler default.

**Service to service.** A short-lived token, or mutual TLS, with a narrow audience. A long-lived shared secret in a config file will leak into a log or a ticket eventually.

### When it matters

Every endpoint, and every design that says "the user is logged in."

### What it costs

A real identity provider is less work than a homegrown password store, and it is easier to get wrong if you copy a tutorial from 2014. Budget time for token validation and for the "log out of all sessions" path.

### A failure you should recognize

`GET /invoices/881` checks that the caller is logged in and does not check that invoice 881 belongs to that caller. Anyone who can guess or enumerate ids can read other customers' invoices. This is **broken object-level authorization**, and it sits under broken access control. Authorize the object.

### Learn

Start here: [RFC 9700](https://www.rfc-editor.org/rfc/rfc9700.html) and the [OWASP authentication cheat sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html).

The Top 10 itself: [OWASP Top 10:2025](https://top10.owasp.org/2025/).

### Review

1. What question does authentication answer, and what question does authorization answer?
2. Which OAuth flows does RFC 9700 tell you not to use?
3. Is OAuth 2.1 an RFC? What do you follow today?
4. Why is a JWT in local storage a sharp edge?

### In an interview

Say who the caller is, how you know, and how you check the object. Mention one retired OAuth flow so they know your knowledge has a date on it.

### Exercise

Pick one endpoint that takes an id. Write the authorization rule in one sentence ("the invoice's account must be an account the caller administers"). If the code only checks "is logged in," write the missing check as a test you wish existed.

## Secrets and supply chain

**Level:** Must-have

**You can:** keep secrets out of source, and treat dependencies as part of the attack surface.

### In one minute

A **secret** is anything that grants power: a password, an API key, a private key, a connection string with a password. Put secrets in a secret store or in the platform's secret injector. Inject them at runtime. Do not commit them, and do not bake them into an image. If a secret lands in git, rotate it. Deleting the commit does not remove it from every clone and every CI log.

**Supply chain** is the code you did not write: libraries, base images, build plugins, and the path from a commit to a deploy. In the [OWASP Top 10:2025](https://top10.owasp.org/2025/A03_2025-Software_Supply_Chain_Failures/) this is **A03:2025 Software Supply Chain Failures**. It grew out of the older "vulnerable components" category and includes the pipeline, not only the dependency list.

Minimum practice:

- Commit a **lockfile** so a build today and a build tomorrow resolve the same versions.
- Run a dependency scanner in CI and fail, or at least page, on a known exploited issue in code you ship.
- Pin base images tightly enough that "latest" is not what production runs.
- Review a new dependency as you would review code. It is code.

The rest of the OWASP Top 10:2025 belongs in your review habit:

1. Broken access control
2. Security misconfiguration
3. Software supply chain failures
4. Cryptographic failures
5. Injection
6. Insecure design
7. Authentication failures
8. Software or data integrity failures
9. Security logging and alerting failures
10. Mishandling of exceptional conditions

Injection still includes SQL injection. Use parameters. Do not concatenate a request string into SQL. Cryptographic failures include homegrown encryption and storing passwords with a fast hash. Use a password hash meant for passwords (Argon2, bcrypt, or scrypt) if you store passwords at all. Prefer an identity provider.

[OWASP ASVS](https://owasp.org/www-project-application-security-verification-standard/) is the deeper checklist when a system needs a real bar, not only the Top 10 awareness list.

**SLSA** is a useful map of supply-chain maturity ([slsa.dev](https://slsa.dev/)). You do not need the highest level to start. A lockfile and a protected CI pipeline are the start. A **CycloneDX** SBOM ([cyclonedx.org](https://cyclonedx.org/)) is a software bill of materials: a list of what you shipped. Good to have once the scanner is in place.

### When it matters

The first day of a project, and every pull request that adds a package or a config file.

### What it costs

Scanners produce noise. Tune them. An ignored red build teaches the team to ignore red builds. A secret store is another system to learn, and it is cheaper than a leaked cloud key.

### A failure you should recognize

A connection string in `appsettings.json`, committed, with a production password, because the file was "only an example" and then it was edited in place. The fix is a placeholder in git and a secret at runtime, plus a rotation of the password that already leaked.

### Learn

Start here: [OWASP Top 10:2025](https://top10.owasp.org/2025/), especially A01, A02, A03, and A07.

### Review

1. What do you do after a secret has been pushed, besides deleting the file?
2. What changed in spirit between "vulnerable components" and OWASP's 2025 supply-chain category?
3. Why does a lockfile matter for a rebuild next month?

### In an interview

Mention secrets at runtime, parameterized queries, and a scanner in CI. Name A01 as object-level checks, not as a buzzword.

### Exercise

Search a repo you work on for a connection string or an API key pattern. If you find a live secret, rotate it and move it out of git. If you find none, write down where the real secrets are injected.

## Observability and SLOs

**Level:** Must-have

**You can:** define an SLO, and say how logs, metrics, and traces meet in one request.

### In one minute

**Observability** is being able to explain a new failure from the outside, using the signals you exported. Three signals:

- **Logs.** A record of an event, with ids, not a novel. Structured logs (JSON fields) can be filtered. A paragraph cannot.
- **Metrics.** Numbers over time: request rate, error rate, latency, saturation. Cheap to keep. Weak at explaining one user's failure.
- **Traces.** A tree of spans for one request, across services, tied together by a **trace id**. This is how you see which hop was slow.

[OpenTelemetry](https://opentelemetry.io/docs/) is the vendor-neutral standard for producing all three. Use it. Export to whatever backend you have. [W3C Trace Context](https://www.w3.org/TR/trace-context/) is the header standard for passing a trace id across HTTP.

Google's four **golden signals** are latency, traffic, errors, and saturation. The same ideas show up as **RED** (rate, errors, duration) for a service and **USE** (utilization, saturation, errors) for a resource such as a CPU or a pool.

An **SLI** is the measurement (the fraction of requests faster than 300 ms, the fraction that returned success). An **SLO** is the target (99.9% of requests succeed in a month). The **error budget** is what is left: 100% minus the SLO. For 99.9% over about 30 days, the budget is about 43 minutes of full downtime, or the equivalent in failed requests.

Alert on symptoms that burn the budget, not on every CPU wiggle. The Google SRE book is the source: [Service Level Objectives](https://sre.google/sre-book/service-level-objectives/). The workbook chapter [Alerting on SLOs](https://sre.google/workbook/alerting-on-slos/) is the practical follow-up.

Logging and alerting failures are **A09:2025** in the OWASP Top 10. An auth failure you cannot see is a security bug, not only an operations bug.

### When it matters

Before the first on-call shift, and in every design: "how will we know this is failing?"

### What it costs

Cardinality. A metric labeled with a raw user id will explode in cost and can leak personal data. Label with the operation, the status class, and the dependency. Put the user id in the trace, where it belongs to one request.

### A failure you should recognize

A 500 rate that alerts the team, with no trace id in the log line, and no span for the database call. You know something is wrong and you cannot tell whether it is the new index or the payment provider. Add the trace id at the edge and a span around each outbound call.

### Learn

Start here: [OpenTelemetry docs](https://opentelemetry.io/docs/) and [Service Level Objectives](https://sre.google/sre-book/service-level-objectives/).

### Review

1. What question does a trace answer that a metric does not?
2. What is an error budget for a 99.9% monthly SLO, in minutes?
3. Why is a user id a bad metric label?

### In an interview

Give one SLI and one SLO for the design you just drew. Mention the trace id crossing the queue. That is a complete answer.

### Exercise

Write an SLI and an SLO for a login endpoint. Write the alert in one sentence: who is paged, and after what burn, and what they open first.

## Incidents

**Level:** Must-have

**You can:** run a short incident, and write a blameless postmortem that changes something.

### In one minute

An incident is a named stretch of time when the user experience is worse than the SLO, or when you are one mistake away from that.

During it:

- One person coordinates. Everyone else works a task or is free to take one.
- Say the user impact in a sentence, and update it.
- Prefer a mitigation you can do now (turn off a flag, scale, roll back, shed load) over a root-cause fix you will understand later.
- Keep a timeline while it is happening. Memory will rewrite it.

After it, write a **postmortem**: what happened, user impact, timeline, contributing factors, what worked, and action items with owners. **Blameless** means you look for the system that let a human err, not for a person to punish. People hide facts in a blame culture, and the next incident gets worse. Google's chapter is the tone to copy: [Postmortem culture](https://sre.google/sre-book/postmortem-culture/).

Action items should include a test, an alert, or a guardrail. "Be more careful" is not an action item.

### When it matters

The first time you are on call, and the behavioral interview story about a production failure.

### What it costs

A postmortem takes an afternoon. Skipping it means you will spend the same afternoon again next month, with the same timeline.

### A failure you should recognize

A rollback that was never practiced. The previous build artifact is gone, or the database migration from the bad deploy cannot be rolled forward cleanly. Mitigations you have not rehearsed are stories, not tools. This is why expand/contract migrations and immutable artifacts matter.

### Learn

Start here: [Postmortem culture](https://sre.google/sre-book/postmortem-culture/).

### Review

1. What does the coordinator optimize for in the first minutes?
2. What does blameless mean in practice?
3. Why is "be more careful" not an action item?

### In an interview

Use the story structure in [Senior judgment](07-senior-judgment.md). Include the mitigation and the lasting change.

### Exercise

Write a one-page postmortem of an incident you know, even a small one. Include a timeline and one action item that is a change to code, config, or an alert.

## CI, CD, and containers

**Level:** Must-have

**You can:** describe a pipeline that protects `main`, and a container image that is safe to run.

### In one minute

**Continuous integration** means every change is built and tested with the main line, often, in an automated pipeline. A pull request runs the build, unit tests, integration tests, and a dependency scan. A red pipeline does not merge.

**Continuous delivery** means `main` is always in a state you can release. **Continuous deployment** means you actually release it automatically. Many teams deliver continuously and still gate production with a manual approval. Say which one you do.

[DORA](https://dora.dev/guides/dora-metrics/) names four measures of this system: how often you deploy, how long a change takes to reach production, how often a change fails, and how long you take to restore. Use them as a conversation, not as a target to game. A daily deploy of an empty commit improves a number and not the product.

**Containers.** A container image packages your process and its userspace so it runs the same way on a laptop and in production. The [OCI image spec](https://github.com/opencontainers/image-spec) is the standard format. Docker's [what is a container](https://docs.docker.com/get-started/docker-concepts/the-basics/what-is-a-container/) is a fine introduction.

Habits that match how platforms expect an app to run (the [twelve-factor](https://12factor.net/) notes are still a good checklist):

- One main process per container.
- Config and secrets come from the environment or a mounted secret, not from a file baked into the image.
- The process is stateless. Files it writes locally can disappear. Put durable state in a database or object storage.
- Log to standard output. Let the platform collect it.
- Run as a user who is not root.
- A rolling deploy can have two versions live at once. That is why [migrations](04-data-and-apis.md#schema-migrations) expand first.

**Build once.** Promote the same image from test to production. Rebuilding "the same" source in production is how the builds stop being the same.

[Semantic Versioning](https://semver.org/) (`MAJOR.MINOR.PATCH`) is the version language for libraries you publish. Breaking changes bump the major number. Services often use the git revision or the image digest instead, because there is one deployed version, not a matrix of dependents.

### When it matters

The path from a merge to a running process, which is the path every design assumes and interviews rarely draw. Be able to draw it in two minutes.

### What it costs

A slow pipeline gets bypassed. Keep the pull-request signal fast enough to wait for, and run the heavier suite on main or on a schedule if you must. Write down that choice.

### A failure you should recognize

A production image built from `latest`, as root, with a secret in an `ENV` layer. Anyone with the image has the secret. The next build of `latest` is a different operating system. Pin the base image, pass secrets at runtime, and run as a non-root user.

### Learn

Start here: [DORA's four metrics](https://dora.dev/guides/dora-metrics/) and [twelve-factor](https://12factor.net/).

### Review

1. What is the difference between continuous delivery and continuous deployment?
2. Why do you promote one image instead of rebuilding it?
3. Where do secrets sit relative to the image layers?

### In an interview

"How does this reach production?" Pipeline, image, rolling deploy, old and new versions overlapping, migration already expanded. That answer sits well next to a system design.

### Exercise

Draw the pipeline for a service you work on, from pull request to production. Mark which steps block a merge. Mark where a secret enters. If a secret enters in the Dockerfile, that is the fix to write down.

## Backups and recovery

**Level:** Must-have

**You can:** say how much data you can lose and how long you can be down, and name the last time a restore was tried.

### In one minute

A replica is not a backup. A replica copies mistakes, including a dropped table, as happily as it copies good writes.

- **RPO** (recovery point objective) is how much history you can afford to lose. "Five minutes" means the backup or the log shipping must be that fresh.
- **RTO** (recovery time objective) is how long you can be down while you restore.
- Take backups that are **off** the database's disks, and keep more than one generation.
- A backup you have not restored is a rumor. Restore into a scratch database on a schedule, and run one query that would fail if the backup were empty.

For PostgreSQL, base backups plus the write-ahead log let you recover to a point in time. The [backup chapter](https://www.postgresql.org/docs/current/backup.html) is the source. Managed databases often do this for you. You still need to know the RPO they actually give you, and you still need a restore drill.

### When it matters

Before the first production write, and in any design that says "we won't lose orders."

### What it costs

Storage, and an afternoon for a drill. The cost of skipping the drill shows up as an RTO you missed.

### A failure you should recognize

A team that kept replicas in the same account, with the same credentials, and called that a backup. An accidental `DELETE` without a `WHERE` landed on the primary and then on the replica. The last real backup was a month old and had never been restored.

### Learn

Start here: [PostgreSQL backup and restore](https://www.postgresql.org/docs/current/backup.html).

### Review

1. Why is a replica not a backup?
2. What is the difference between RPO and RTO?
3. What makes a backup real?

### In an interview

Give a number for each. "RPO of about a minute from log shipping, RTO of an hour because we have restored this quarter." If you have not restored, say that the drill is the gap, not a fantasy number.

### Exercise

Write the RPO and RTO you would promise for the checkout orders table, and the one command or job that proves the backup can be read.

## Good to have

### Canary releases

**Level:** Good-to-have

**You can:** ship a new version to a small slice of traffic before it takes everyone.

A **rolling** deploy replaces instances a few at a time. Old and new code run together, which is why migrations expand first. A **canary** sends a small percentage of users to the new version and watches the error rate. A **blue-green** deploy keeps two full environments and switches the balancer. Blue-green is simple to roll back and costs two environments. A canary finds a bad change with less capacity and needs a metric you trust.

Feature flags and canaries solve different problems. A flag chooses a behavior. A canary chooses a build.

### Privacy in logs and design

**Level:** Good-to-have

**You can:** keep personal data out of places you did not mean to store it.

Collect what the feature needs. Do not log a full card number, a password, a session token, or a raw government id. A trace can carry a user id. It does not need the user's address repeated on every span. Retention applies to logs and backups too: a deleted account that lives forever in a log archive is still stored. If you operate in a place with a privacy law, read that law's own text or your counsel's summary. This roadmap is not legal advice. The engineering habit is data minimization, which also makes incidents smaller.

### Passkeys

**Level:** Good-to-have

**You can:** say why a passkey resists phishing better than a password.

A **passkey** is a WebAuthn credential. The private key stays on the device. The site holds the public key. A phishing site on a look-alike domain cannot use the credential, because the authenticator checks the real site identity. [passkeys.dev](https://passkeys.dev/) is the industry introduction. The standard is [WebAuthn](https://www.w3.org/TR/webauthn-2/).

Prefer an identity provider that offers passkeys over building WebAuthn yourself, unless authentication is your product.

### Signed images

**Level:** Good-to-have

**You can:** say what signing an image proves.

Signing (for example with Sigstore's cosign) lets a cluster refuse an image that was not built by your pipeline. It pairs with SLSA. It does not replace scanning. A signed vulnerable library is still vulnerable. Signing tells you who built it.

### Risks when you call a model

**Level:** Good-to-have

**You can:** name the first risks if your service sends user content to a large language model or runs an agent.

This is outside the core backend path. Read it if you are adding that feature.

The [OWASP GenAI LLM Top 10 for 2026](https://genai.owasp.org/resource/owasp-genai-llm-top-10-2026/) is the current awareness list. The top of that list is prompt injection, sensitive information disclosure, and excessive agency (the model is allowed to act, and it acts on untrusted text). If the model can call your tools, treat the tool call as a new trust boundary: authenticate the user, authorize the action, and do not let the model decide its own privileges.

There is a separate [Top 10 for Agentic Applications](https://genai.owasp.org/resource/owasp-top-10-for-agentic-applications-for-2026/). Use it when the system plans and takes actions, not only when it completes text.

Do not send secrets or raw personal data to a third-party model because a prompt was convenient. Log the trace id and the tool that was called. Do not log the full prompt if the prompt contains personal data you would not put in an ordinary application log.
