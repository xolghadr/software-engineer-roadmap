# 7. Senior judgment

The earlier tracks are the material. This track is how a senior carries it: a written decision, an estimate with assumptions, and a story that shows judgment under pressure.

## Design docs and ADRs

**Level:** Must-have

**You can:** write a short design that states the decision, the option you rejected, and how you will know the decision was wrong.

### In one minute

A **design doc** is a few pages written before a change that is hard to undo. It is a tool for a conversation, not a novel. One sitting should be enough to read it.

Use this outline:

1. **Context.** What is true today, and why this needs a decision now.
2. **Goals.** What will be true if this works, in terms a user or an operator would notice.
3. **Non-goals.** What this effort will not solve. This is the paragraph that keeps the project small.
4. **Options.** Two or three real options, each with a cost. A fake option you would never pick does not count.
5. **Decision.** The option you want, and the reason.
6. **Risks.** What failure looks like, including the migration and the rollback.
7. **Rollout.** How it ships (flag, expand/contract, who is first) and what you will measure.

An **ADR** (architecture decision record) is the short form after you have decided: the decision, the status, and the consequences, kept next to the code. Start a series with Michael Nygard's template: [Documenting architecture decisions](https://adr.github.io/).

A diagram belongs in the doc if it has a data store and a failure arrow. Link the glossary terms to the track pages rather than re-explaining CAP in the doc.

### When it matters

A new service, a new data model, a public API, a migration that touches a hot table, or any change you would be unhappy to discover in a pull request with no warning.

### What it costs

An afternoon. The return is a disagreement that happens in the doc, where changing your mind is cheap, instead of in production.

### A failure you should recognize

A doc that lists only the chosen design, with no rejected option and no rollback. Readers cannot tell what you considered. Six months later no one knows why the queue exists, and someone deletes it.

### Learn

Start here: [ADR templates](https://adr.github.io/).

### Review

1. What does a non-goal protect you from?
2. Why does a design need a rejected option?
3. What is the difference in length and purpose between a design doc and an ADR?

### In an interview

When they ask "how would you design this?", the spoken answer can follow the same outline in miniature: goal, two options, decision, the failure.

### Exercise

Write a one-page design for adding idempotency keys to `POST /payments`. Include the option of "rely on the client not to retry," and say why you reject it. Include how long you keep the key.

## Estimation

**Level:** Must-have

**You can:** give a range, name the assumption that moves it, and update the estimate when the assumption breaks.

### In one minute

An estimate is a forecast with assumptions, not a promise made to sound confident.

Say a **range** ("three to five days") and the **assumption** that sets the ends. "Three days if we only add a version column and a 409. Five days if the mobile client must show a merge UI. About two weeks if we also rewrite the history of the row." The listener can change the scope instead of arguing with the number.

Break the work into slices you have done before. The unknown slice is the wide part of the range. Say so. "I have shipped migrations like the first part. I have not integrated this vendor. That part is one to three days, and I will know more after a spike of half a day."

Update the estimate when you learn. A senior update sounds like: "The vendor's sandbox does not support refunds. The upper bound is now a week, unless we cut refunds from this release." Silence until the deadline is how trust dies.

Do not convert a guess into a single hour to look precise. Precision you do not have is a different kind of wrong.

### When it matters

Planning, interviews ("how long would this take?"), and any time someone treats a first number as a contract.

### What it costs

A spike costs half a day and buys a narrower range. Skipping the spike costs the whole project when the unknown was the real work.

### A failure you should recognize

"It is a small change" for a column rename on a hot table, ignoring the expand/contract deploys and the backfill. The code change is small. The release is not. Estimate the release.

### Learn

Start here: the practice above. Pair it with the migration and rollout pages so the estimate includes the steps that are not typing.

### Review

1. What two things does a useful estimate contain besides a number?
2. How do you treat the slice you have never done?
3. What do you say when an assumption breaks halfway through?

### In an interview

Give the range and one assumption without being asked. Offer a smaller scope that fits a shorter range. That is judgment.

### Exercise

Estimate the checkout saga in the designs folder as if you were adding it to a system that already has orders in one database. Write the range and the three assumptions.

## Stories for interviews

**Level:** Must-have

**You can:** tell two concrete stories: a tradeoff you argued, and a production failure you took part in.

### In one minute

Senior interviews ask for judgment you have already shown. Prepare two stories. Four minutes each is enough. Use this shape:

- **Situation.** The system and the constraint, in two sentences.
- **Your part.** What you personally did. "We" is fine for context. The action needs an "I."
- **Options.** The real alternative, and the cost of each.
- **Outcome.** What shipped, what the user saw, and what you measured if you measured anything.
- **Change.** What you would do differently, or what you changed afterwards.

**Story A, a tradeoff.** A time you pushed for a simpler design, or for a more careful one, against a default. Examples that work: keeping a module inside the monolith, refusing a shared table, adding an outbox instead of publishing inside a transaction, cutting scope to hit a date safely.

**Story B, a failure.** A time production went wrong. Lead with the user impact, then your mitigation, then the lasting fix (a test, an alert, a migration habit). Leave out blame. Include your own miss if you had one. Interviewers trust a story where you learned more than a story where you were perfect.

Write the stories down. Say them once out loud. If you cannot say what you personally did, the story is not ready.

### When it matters

Behavioral rounds, and the last ten minutes of a design round ("tell me about a system you actually shipped").

### What it costs

An hour of preparation. An unprepared story wanders into company history and never reaches a decision.

### A failure you should recognize

A story that is only a list of technologies. "We used Kafka and Kubernetes and it scaled" has no decision in it. Add the alternative you rejected.

### Learn

Start here: your own notes, plus the incident chapter's tone from [Postmortem culture](https://sre.google/sre-book/postmortem-culture/).

### Review

1. Where does "I" have to appear in the story?
2. What does story B lead with?
3. What is missing from a story that only names tools?

### In an interview

Stop at four minutes. Offer to go deeper on the decision. Let them pull the detail.

### Exercise

Write both stories at half a page each. Underline the sentence that states the option you rejected. If you cannot underline one, the story is not finished.

## Good to have

### Mentoring

**Level:** Good-to-have

**You can:** review and delegate in a way that makes the next change easier for someone else.

A senior multiplies a team. In practice that looks like:

- A review comment that explains the failure mode, not only the requested edit.
- A task sliced so a newer engineer owns a real piece, with the risky migration called out in advance.
- A design doc they were invited to disagree with.

You still own the outcome. Delegation without a look at the risky part is abdication. Doing the risky part yourself every time is a team that cannot ship when you are away.

In an interview, one example of someone else's growth is enough. Skip a speech about leadership.
