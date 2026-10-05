# 2. Code design

This track is what many interviews call low-level design. You can shape types, draw the two diagrams people actually read, and choose a small set of patterns by the problem they solve.

GRASP is optional vocabulary. It is in the [GRASP appendix](../appendix/grasp.md).

## Relationships between types

**Level:** Must-have

**You can:** pick association, composition, inheritance, or interface implementation, and say why.

### In one minute

These names come from class diagrams. Use them to describe how types relate, in speech and on a whiteboard.

- **Association.** One object uses another. An order refers to a customer. Both can exist on their own.
- **Composition.** A part has no independent life. A line item belongs to an order. Delete the order and the line items go with it. The part is created and owned by the whole.
- **Aggregation.** A weaker "whole and part" where the part can outlive the whole. It is in the UML spec and it causes more arguments than clarity. Prefer a plain association unless the lifecycle is truly shared and you are ready to defend it.
- **Inheritance (generalization).** A subtype is a kind of the base type and keeps the base type's promises. A `PriorityButton` that still behaves as a `Button`.
- **Realization.** A class fulfills an interface. The interface is the promise. The class is one way to keep it.
- **Dependency.** A type mentions another in a method signature and does not keep it as a field. A controller that accepts a clock argument depends on the clock.

The source that says "six relationships" is counting association, aggregation, composition, inheritance, realization, and dependency. Aggregation is the one people drop.

### When it matters

A design interview that starts with "design a parking lot" or "design a library checkout" is asking for these relationships, plus the behavior.

### What it costs

Inheritance feels fast and becomes rigid. Composition is a bit more wiring and stays flexible. Aggregation-as-a-diamond is a symbol you can spend twenty minutes debating. Spend that time on behavior instead.

### A failure you should recognize

An `AdminUser` class that inherits `User` and then overrides methods to do nothing, or to throw. That is not a subtype. It is a different type wearing a parent's name. Model the difference with a role, or with a separate type.

### Learn

Start here: the [UML specification](https://www.omg.org/spec/UML/) only if you need the official terms. For daily use, the definitions above are enough, and [C4](#diagrams-people-can-read) is the better default for architecture pictures.

### Review

1. When does a line item stop existing relative to its order?
2. Why is aggregation a weak choice in a design conversation?
3. What promise does a subtype have to keep?

### In an interview

Draw the relationship and say the lifecycle in one sentence. "The order owns its lines. The customer does not own the order. Cancelling the customer does not delete history."

### Exercise

Model a calendar event with attendees and a reminder. Write which relationship you used for the attendees and for the reminder, and what happens to each when the event is deleted.

## SOLID

**Level:** Must-have

**You can:** explain each principle with a misuse, because the misuse is how seniors recognize a lecture that was memorized.

### In one minute

SOLID is a set of five guides for object design, popularized by Robert C. Martin. They are guides. A design that satisfies the sentence and fights the next change has missed the point.

- **Single responsibility.** A type has one reason to change. Group the code that changes together, and keep unrelated changes apart. Misuse: a class per function, and a call chain no one can read.
- **Open-closed.** You can add a behavior without editing a stable type. Misuse: a plugin system for a report that changes twice a year. An `if` would have been honest.
- **Liskov substitution.** A subtype must be usable anywhere the base type is usable, with the same promises. Misuse: a `Square` type whose setters break the invariants callers expect from a `Rectangle`.
- **Interface segregation.** Callers should depend on the methods they use. Misuse: forty interfaces of one method each, always injected together. That is a split that helps no caller.
- **Dependency inversion.** High-level policy depends on abstractions it owns. Details (SQL, HTTP, the clock) implement those abstractions. Misuse: an interface that is a photocopy of a vendor SDK, method for method. You do not own that shape, so you did not invert anything. You moved the vendor into your constructor.

### When it matters

A pull request where one class knows the pricing rules, the SQL, and the email template. Or a design interview where you are asked how you would add a new discount without a long chain of edits.

### What it costs

Each split is a file, a name, and a jump. Split when you have a second real variation, or a second reason to change. Do the split before the class is 800 lines, and after you can name the two reasons.

### A failure you should recognize

A "base controller" that every endpoint inherits, with protected helpers for auth, logging, database, and time. The next feature needs one of those and is forced to take the rest. Prefer a small collaborator per job.

### Learn

Start here: Robert C. Martin, *Agile Software Development, Principles, Patterns, and Practices*, the principles chapters.

A shorter read, useful and not the primary source: DigitalOcean's [SOLID introduction](https://www.digitalocean.com/community/conceptual-articles/s-o-l-i-d-the-first-five-principles-of-object-oriented-design/).

### Review

1. What are two different reasons the `Order` type in a typical tutorial might change?
2. State the Liskov rule without using the word "square."
3. When is an interface a photocopy of a vendor, and why does that fail dependency inversion?

### In an interview

They may not say "SOLID." They say "how would you add a second payment provider?" Your answer is an interface you own (`ChargeCustomer`), an adapter per provider, and the existing order code left alone.

### Exercise

Find one class you wrote that has two reasons to change. Split one reason out. Leave a sentence in the pull request description naming the two reasons.

## Composition over inheritance

**Level:** Must-have

**You can:** reuse behavior by holding a collaborator, and use inheritance only when the subtype truly is the base type.

### In one minute

The Gang of Four book says to favor object composition over class inheritance. Inheritance copies a design into the type hierarchy. Every subclass is stuck with the parent's protected details. Composition means the object *has* another object and calls it. You can swap that collaborator in a test, or in the next release, without a new subclass.

Use inheritance when you mean "this is a kind of that" and every promise still holds. Use composition when you mean "this uses that" or when you only wanted to reuse a method.

### When it matters

A hierarchy of `BaseHandler` → `ValidatedHandler` → `LoggedHandler` → `ActualHandler` that exists to stack behaviors. Decorators, or a short pipeline of functions, express the same stack and are easier to reorder.

### What it costs

Composition has more small types and a bit of wiring. Inheritance has fewer types until the first exception, and then it has a tangle.

### A failure you should recognize

`PostgresRepository` inherits `RepositoryBase` inherits `SqlHelper` so it can call `protected Query()`. A test cannot construct it without a connection string. Give it a database port it calls. The SQL helper can sit behind that port.

### Learn

Start here: Gamma, Helm, Johnson, Vlissides, *Design Patterns* (the Gang of Four), the introduction, "inheritance versus composition."

### Review

1. What are you trying to reuse when you inherit, and what are you forced to keep?
2. Give one case where inheritance is the honest model.
3. How do you test a type that inherits a helper which opens a socket?

### In an interview

If you draw a deep tree, be ready for "what if one of these is not actually a kind of the parent?" Prefer a shallow tree and collaborators.

### Exercise

Replace one level of inheritance in a sample with a field that does the same job. The tests should still pass, and the subclass count should drop.

## Patterns you will actually use

**Level:** Must-have

**You can:** reach for six patterns by the problem, and leave the other pattern-book pages alone until you have that problem.

Learn the intent, not a UML stencil.

### Strategy

**Problem.** A decision procedure has several real variants: weekday pricing, weekend pricing, partner pricing.

**Shape.** One interface, several small implementations, one place that picks an implementation. The caller runs the interface.

**Misuse.** A strategy with one implementation, and no second one coming.

### Adapter

**Problem.** A third-party client has a shape you do not want inside your domain.

**Shape.** Your code talks to an interface you named (`ChargeCustomer`). One adapter translates that to Stripe, or to Adyen. The vendor types do not leak past the adapter.

**Misuse.** An adapter that exposes the vendor's method names unchanged. That is a rename, not a boundary.

### Decorator

**Problem.** You want to add a behavior around an object (timing, caching, a retry) and you want to keep that behavior separate from the object.

**Shape.** A wrapper with the same interface. It does its extra work, then calls the inner object.

**Misuse.** A wrapper that changes the meaning of the result. Callers can no longer treat the decorator as the original.

### Factory

**Problem.** Building an object takes a set of steps, or a choice, and you do not want every caller to know the steps.

**Shape.** One function or type whose job is construction. Callers ask for the finished object.

**Misuse.** A factory that only calls one constructor and adds no choice and no steps. Call the constructor.

### Repository

**Problem.** The domain wants to load and save an aggregate as if it were a collection. SQL does not belong in the pricing rule.

**Shape.** An interface such as `Orders` with `Get(id)` and `Save(order)`. One implementation speaks SQL. This pattern is from Eric Evans and from Martin Fowler's *Patterns of Enterprise Application Architecture*, not from the Gang of Four.

**Misuse.** A generic repository with twenty unused methods, or a repository that returns database rows to the domain.

### Unit of Work

**Problem.** One user action changes several objects, and those changes must commit or roll back together.

**Shape.** A unit of work tracks the changes and flushes them in one transaction. Fowler's catalog describes it: [Unit of Work](https://martinfowler.com/eaaCatalog/unitOfWork.html). In many stacks the database context you already use *is* the unit of work. Do not invent a second one beside it.

**Misuse.** A unit of work that outlives the request and then saves "whenever." The boundary of the unit of work is the boundary of the transaction.

### Learn

Start here, for the first four: the Gang of Four book, the chapters for Strategy, Adapter, Decorator, and Factory Method or Abstract Factory. Read for intent.

For the last two: [Repository](https://martinfowler.com/eaaCatalog/repository.html) and [Unit of Work](https://martinfowler.com/eaaCatalog/unitOfWork.html).

### Review

1. Which pattern keeps a vendor SDK out of the order service?
2. When is a factory just a constructor with extra paperwork?
3. What is the transaction boundary of a unit of work?

### In an interview

Name the problem first, then the pattern. "We will have three pricing rules, so a strategy." If you cannot name a second variant, say you would keep a straight method until the second one exists.

### Exercise

Put a fake payment adapter and a real-shaped interface behind a checkout function. The checkout test uses the fake and never mentions HTTP.

## Diagrams people can read

**Level:** Must-have

**You can:** draw a sequence diagram for a request, a small class diagram for a low-level design, and a C4 context or container picture for an architecture discussion.

### In one minute

Diagrams are a language for a conversation. Two of them cover low-level interviews. One model covers architecture.

**Sequence diagram.** Boxes across the top are participants (client, API, database). Arrows down the page are calls in time order. Use this when the question is "what talks to what, and in what order?"

**Class diagram.** Boxes are types. Arrows are the relationships from the section above. Use this when the question is "what are the nouns and who owns them?" Keep it to the types you will speak about. A diagram of every class in the repo is a map no one reads.

**C4 model** (Simon Brown). Four levels of zoom, and you stop at the level the audience needs:

1. **Context.** Your system, the people, and the other systems.
2. **Container.** Applications and data stores you deploy (the API, the worker, the database). "Container" here does not mean Docker.
3. **Component.** The major parts inside one application.
4. **Code.** Classes. Rare in an architecture review.

Use C4 in design discussions and design docs. Use UML sequence and class diagrams in low-level design interviews. You do not need activity diagrams, deployment stereotypes, or a UML certification.

### When it matters

Any interview or design review where a picture would replace five minutes of hand-waving. The picture is there so someone can point at an arrow and ask "what if this times out?"

### What it costs

A pretty diagram that is wrong is worse than a rough one that has the failure on it. Draw the unhappy arrow too: the timeout, the retry, the duplicate.

### A failure you should recognize

A single box labeled "backend" with arrows to "cloud." There is no data store, no queue, and no other system. The picture cannot be wrong because it does not say anything. Add the data store before you add more services.

### Learn

Start here: [C4 model](https://c4model.com/). For the official UML terms, the [UML specification](https://www.omg.org/spec/UML/) is the standard. You rarely need to read it end to end.

### Review

1. Which picture answers "who calls whom, in order?"
2. In C4, what is a container?
3. What extra arrow do you add once the happy path is drawn?

### In an interview

Start with context: users and neighboring systems. Then one container diagram. Only drop to classes if they ask how a module is shaped. Label data stores. Label sync and async differently.

### Exercise

Draw a C4 container diagram of a service you know, on one page. Include the database and one external system. Add one arrow for the failure you fear most.

## Good to have

### Hollywood principle

**Level:** Good-to-have

**You can:** explain inversion of control in one sentence.

A framework calls your code. You do not call the framework's lifecycle from the inside. "Don't call us, we'll call you." Your handler is invoked by the web server. Your test is invoked by the test runner. You hand the framework a piece of work.

This is the same idea as inversion of control. Martin Fowler's note is the clear source: [Inversion of Control](https://martinfowler.com/bliki/InversionOfControl.html).

The misuse is a framework so thick that a one-line job requires three registrations. Invert control where a framework already sits (HTTP, tests, jobs). Do not build a framework for a single call site.

### Dependency injection as a habit

**Level:** Good-to-have

**You can:** pass collaborators in from the outside, usually through the constructor, so a test can pass a fake.

Dependency injection is the everyday mechanics of dependency inversion. The composition root (startup) is the one place that knows the real database, the real clock, and the real payment adapter. Everything else receives interfaces or small concrete values.

A service locator (a global `Get<T>()` bag) hides the dependency list. Prefer a constructor that shows what the type needs.
