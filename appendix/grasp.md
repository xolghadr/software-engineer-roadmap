# Appendix: GRASP

**Level:** Good-to-have. Optional for interviews.

**You can:** use GRASP words when you talk about which object should own a responsibility, and translate them into the pattern names interviews already use.

GRASP (General Responsibility Assignment Software Patterns) is Craig Larman's set of nine guides, from *Applying UML and Patterns*. They are about **where a method should live**. They are older than most interview loops. SOLID and the six patterns in the [code design track](../tracks/02-code-design.md) cover the same ground in the words people use now. Learn GRASP if you want the responsibility vocabulary. Do not lead an interview with the acronym unless the interviewer uses it first.

## The nine

**Information expert.** Give the job to the object that already has the data. A total belongs on the order if the order has the lines. The misuse is a "manager" that pulls fields out of every object and does the work beside them.

**Creator.** Give object B the job of creating object A when B contains A, or records A, or has the data you need to build A. An order creates its own line items. This is composition from the [relationships](../tracks/02-code-design.md#relationships-between-types) page.

**Controller.** The first object after the UI that receives a system operation. It coordinates. It does not contain every rule. A use-case handler is a controller in this sense. A "god" controller that also knows SQL is the misuse.

**Low coupling.** A type should depend on as few other types as it can, and on stable ones. An adapter exists so the domain is not coupled to a vendor SDK.

**High cohesion.** The methods on a type should be about one purpose. This is the same itch as the single-responsibility principle.

**Polymorphism.** When behavior varies by type, put the variation behind a common operation instead of a chain of type checks. Strategy and interface implementation are the everyday forms.

**Pure fabrication.** Invent a type that is not a business noun when you need it to keep coupling low or cohesion high. A repository is a fabrication. It is not a thing in the shop. It earns its place by keeping SQL out of the order.

**Indirection.** Put a stable middle in front of something that changes. An interface you own, in front of a provider, is indirection. Too much of it is a file with one line that calls another file with one line.

**Protected variations.** Wrap the point you expect to change (a vendor, a policy, a tax rule) so the rest of the system does not see the change. This is the goal that adapters, strategy, and ports are reaching for.

## How this maps to the rest of the roadmap

| GRASP idea | Where it shows up here |
| --- | --- |
| Information expert, creator | Relationships and the domain type that owns a rule |
| Controller | A use case at the edge of [hexagonal architecture](../tracks/05-architecture.md#hexagonal-architecture-ports-and-adapters) |
| Low coupling, indirection, protected variations | Adapter, ports, anticorruption layer |
| High cohesion | Single responsibility, and the warning not to split until there are two reasons to change |
| Polymorphism | Strategy |
| Pure fabrication | Repository, unit of work |

## Review

1. Who should calculate an order total, and why?
2. What GRASP idea is a repository an example of?
3. Which interview-facing pattern matches "protected variations" around a payment vendor?

## Learn

Start here: Craig Larman, *Applying UML and Patterns*, the GRASP chapters. Use this page as the map, and the book as the source. Secondary blog posts tend to list the nine names and stop. The book has the examples.
