# Software Engineer Roadmap

This is a detailed roadmap to become a senior software developer.

## Object Oriented Programming
- Encapsulation
- Abstraction
- Modularity
- Hierarchy

`` Sources: ``
- [OOP-1]
- [OOP-2]

---
## SOLID Principles
-  Single-responsibility Principle
-  Open-closed Principle
-  Liskov Substitution Principle
-  Interface Segregation Principle
-  Dependency Inversion Principle
  
`` Sources: ``
- [SOLID-1]
- [SOLID-2]
- [SOLID-3]
---
## General Responsibility Assignment Software Principles (GRASP)
- Controller
- Creator
- Indirection
- Information Expert
- Low Coupling
- High Cohesion
- Polymorphism
- Protected Variations
- Pure Fabrication

`` Sources: ``
- [GRASP-1]
- [GRASP-2]

---
## Follow these links to learn markdown files

`` Sources: ``
- [README-1]
- [README-2]
- [README-3]

---
## Types of relationships in OOP
- Composition
- Aggregation
- Association
- Inheritance
- Realization (Implementation)

`` Source: ``
- [OOP-3]

---
## Learning UML
As a developer you must be able to understand the language of diagrams. The beginning of this path is using the UML. 

`` Watch these videos: ``
- [UML-1]
- [UML-2]

---
## Architectural Style

1. Microservices Architecture
`` Sources: ``
  * [MA-1]
  * [MA-2]
  * [MA-3]

2. Event-Driven Architecture
`` Sources: ``
  * [ED-1]
  * [ED-2]
  * [ED-3]

3. Service-Oriented Architecture
`` Sources: ``
  * [SO-1]
  * [SO-2]
  * [SO-3]

4. Component-based Architecture
`` Sources: ``
  * [CB-1]
  * [CB-2]

---
## Architectural Design Principles and Practices
1. The scale cube (x,y,z dimensions)
`` Sources: ``
  * [TSC]

2. CAP Theorem
`` Sources: ``
  * [CAP-1]
  * [CAP-2]

3. Eventual Consistency
`` Sources: ``
  * [EC]

4. Hollywood Principle (Don't call us, we'll call you)
`` Sources: ``
  * [HP]

5. Persistence Ignorance
`` Sources: ``
  * [PI]

6. Composition over Inheritance
`` Sources: ``
  * [COI]

7. Back Pressure
`` Sources: ``
  * [BP]

##Programming Paradigms

- knowing different paradigms of programming

- Can differentiate between the usage of each one

- Can use multi-paradigm ability of programming language


##Memory Management

- Knowing what is GC

- Knowing how garbage collection (GC) works in .NET Core

- Knowing how detect memory issues

- Large objects heap

- Ports exhaustion on HttpClient

- Object pooling


## Domain-Driven Design - Strategic Design

- Bounded context

- Ubiquitous Language

- Context Maps

- Shared Kernel  

- Customer / Supplier

- Conformist

- Partner


## CQRS - Command and Query Responsibility Segregation

- Regular Model

- Transaction Script in the Command stack

- Query stack (LET or SQL)

  
## Communication patterns

- Synchronous messaging

- HTTP

- gRPC

- Asynchronous messaging

- Notifications

- Request/response

- Request/asynchronous response

- Publish/Subscriber

- Publish/asynchronous response


## Messaging

- Exactly-Once, at-least-once, at-most-once Delivery

- Outbox, Inbox patterns and delivery guarantees

- Idempotent Consumer


## Security

- SSL

- TSL

- HTTPS

- Certificates

  
# Concurrency Management

- knowing Concurrency

- Types Pessimistic

- Optimistic

- Last In Wins

- Resolution strategies


### Supplementary references: 
- [SOA VS Microservices]
- [ASP.NET Core Best Practices]

[OOP-1]: <https://www.geeksforgeeks.org/introduction-of-object-oriented-programming/>
[OOP-2]: <https://www.educative.io/blog/object-oriented-programming/>
[OOP-3]: <https://blog.visual-paradigm.com/what-are-the-six-types-of-relationships-in-uml-class-diagrams/>

[SOLID-1]: <https://www.digitalocean.com/community/conceptual-articles/s-o-l-i-d-the-first-five-principles-of-object-oriented-design/>
[SOLID-2]: <https://www.freecodecamp.org/news/solid-principles-explained-in-plain-english/>
[SOLID-3]: <https://www.geeksforgeeks.org/solid-principle-in-programming-understand-with-real-life-examples/>

[GRASP-1]: <https://www.geeksforgeeks.org/grasp-design-principles-in-ooad/>
[GRASP-2]: <https://medium.com/@mehar.chand.cloud/grasp-general-responsibility-assignment-software-patterns-7d1104b0aad5/>

[README-1]: <https://www.freecodecamp.org/news/how-to-write-a-good-readme-file/>
[README-2]: <https://www.youtube.com/watch?v=zKS7mjVvxGc/>
[README-3]: <https://docs.github.com/en/get-started/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax>

[UML-1]: <https://www.youtube.com/watch?v=6XrL5jXmTwM/>
[UML-2]: <https://www.youtube.com/watch?v=WnMQ8HlmeXc/>

[MA-1]: <https://cloud.google.com/learn/what-is-microservices-architecture#:~:text=on%20Google%20Cloud.-,Microservices%20architecture%20defined,architecture%20diagrams%20and%20services%20independently/>
[MA-2]: <https://microservices.io/patterns/microservices.html/>
[MA-3]: <https://medium.com/hashmapinc/the-what-why-and-how-of-a-microservices-architecture-4179579423a9/>

[ED-1]: <https://aws.amazon.com/event-driven-architecture//>
[ED-2]: <https://www.redhat.com/en/topics/integration/what-is-event-driven-architecture/>
[ED-3]: <https://learn.microsoft.com/en-us/azure/architecture/guide/architecture-styles/event-driven/>

[SO-1]: <https://www.redhat.com/en/topics/cloud-native-apps/what-is-service-oriented-architecture//>
[SO-2]: <https://aws.amazon.com/what-is/service-oriented-architecture/#:~:text=you%20implement%20microservices%3F-,What%20is%20service%2Doriented%20architecture%3F,other%20across%20platforms%20and%20languages/>
[SO-3]: <https://www.ibm.com/topics/soa/>

[CB-1]: <https://configr.medium.com/component-based-architecture-building-scalable-maintainable-software-36137d56ff46/>
[CB-2]: <https://marutitech.com/guide-to-component-based-architecture/>

[TSC]: <https://microservices.io/articles/scalecube.html/>

[CAP-1]: <https://medium.com/@ngneha090/understanding-the-cap-theorem-balancing-consistency-availability-and-partition-cb11c2b97e2b/>
[CAP-2]: <https://www.ibm.com/topics/cap-theorem/>

[EC]: <https://medium.com/@abhirup.acharya009/strong-consistency-vs-eventual-consistency-19ce6f87c112/>

[HP]: <https://medium.com/@sandy619g/hollywood-principle-in-software-engineering-5d68f679b524/>

[PI]: <https://learn.microsoft.com/en-us/archive/msdn-magazine/2009/june/the-unit-of-work-pattern-and-persistence-ignorance/>

[COI]: <https://sheldonrcohen.medium.com/favoring-composition-over-inheritance-ff2ece6b7b4e/>

[BP]: <https://medium.com/@jayphelps/backpressure-explained-the-flow-of-data-through-software-2350b3e77ce7/>

[SOA VS Microservices]: <https://www.ibm.com/think/topics/soa-vs-microservices>
[ASP.NET Core Best Practices]: <https://learn.microsoft.com/en-us/aspnet/core/fundamentals/best-practices?view=aspnetcore-8.0>
