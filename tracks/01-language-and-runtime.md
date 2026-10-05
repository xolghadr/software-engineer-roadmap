# 1. Language and runtime

You can write correct code in more than one style, you can explain where memory goes, and you can see a race before it becomes a late-night bug.

Platform-specific garbage collection, the large object heap, and `HttpClient` live in the [.NET runtime appendix](../appendix/dotnet-runtime.md).

## Paradigms

**Level:** Must-have

**You can:** name the paradigm you are using, and switch when another one makes the change easier to test.

### In one minute

A paradigm is a way of organizing a program.

- **Imperative.** You list the steps. "Open the file, loop, write the row."
- **Object-oriented.** You put state and the operations on that state together, and you hide the state behind those operations.
- **Functional.** You pass values into functions and you prefer functions that do not change hidden state. The same inputs give the same output.
- **Declarative.** You describe the result you want. SQL is the everyday example: you say which rows you want, and the engine chooses the steps.

Most languages you will be paid to use are multi-paradigm. C# can be imperative, object-oriented, and functional in the same solution. A senior uses that on purpose. A pure style is not a badge.

### When it matters

- A pipeline of transformations (parse, filter, map, reduce) is easier to test as functions that return values.
- A long-lived business rule with rules about who may change it fits an object with a small public surface.
- A query belongs in SQL, or in a query language, more often than in a hand-written loop over a million rows.

### What it costs

A functional style can allocate more short-lived objects. An object style can hide a mutable field that two callers both change. A declarative query can hide a terrible plan until you read it.

### A failure you should recognize

A 400-line method that mixes I/O, business rules, and formatting. No one can test the rule without a database, a clock, and a network. Split the rule into a function that takes values. Keep the I/O at the edge.

### Learn

Start here: Peter Van Roy, [Programming Paradigms for Dummies](https://www.info.ucl.ac.be/~pvr/VanRoyChapter.pdf) (chapter, free). Read the opening taxonomy. You do not need every historical paradigm.

### Review

1. Where does SQL sit, and why is a row-by-row loop a different paradigm?
2. What do you gain by making a pricing function return a new value instead of changing a shared object?
3. Name one place in your current language where you already mix two paradigms.

### In an interview

"Talk me through how you would structure this rule." A senior answer picks a style and says what that choice makes easy to test.

### Exercise

Take a method that both calculates a total and saves it. Split it into a pure calculation and a save. Test the calculation with no database.

## Memory

**Level:** Must-have

**You can:** explain stack, heap, and garbage collection, and name the usual cause of a "leak" in a managed language.

### In one minute

When a function runs, the runtime keeps a small frame for that call. Locals that live only for the call can sit with that frame. People call this the **stack**.

Objects that must outlive the function are allocated on the **heap**. Something has to notice when no one can reach a heap object anymore.

**Garbage collection (GC)** is that something. The collector finds objects that are still reachable from roots (statics, running methods, globals) and reclaims the rest. You do not call `free`. You also do not get to ignore memory.

Most collectors are **generational**. Most objects die young, so the runtime collects the young space often and the old space rarely. A promotion to the old space is fine for a cache you meant to keep. It is expensive when you accidentally keep everything.

A **leak in a GC language** is almost always a reference you still hold: an unbounded static list, a cache with no eviction, an event handler that was never removed, or a closure that captured a large object.

### When it matters

- A process climbs in memory for days and then restarts.
- A request allocates a large buffer per call under load.
- You are choosing between a pooled buffer and a new array on a hot path.

### What it costs

GC pauses and extra CPU are the price of not managing every free by hand. The price of manual memory is use-after-free and leaks of a sharper kind. For application backends, a managed runtime is the default, and you still design so the live set stays bounded.

### A failure you should recognize

A dictionary keyed by request id, used for "a moment" of debugging, with no removal. Under load the process grows until the orchestrator kills it. The heap dump shows the dictionary as the root.

### Learn

Start here: the concept above, then the collector you actually run. For .NET, [Garbage collection fundamentals](https://learn.microsoft.com/en-us/dotnet/standard/garbage-collection/fundamentals).

Book, when you want the real model: Jones, Hosking, Moss, *The Garbage Collection Handbook*.

### Review

1. Why can a C# or Java service leak memory even though it has a GC?
2. What is a generational collector betting on?
3. What would you look at first if memory climbs only after you add a cache?

### In an interview

"The service is restarted every night because memory is high. Where do you look?" A senior answer says: live references, unbounded caches, and a heap snapshot. A junior answer says "the GC is broken."

### Exercise

Write a tiny program that appends to a static list on every call and never removes. Watch the process memory. Then cap the list. Write down the difference in one sentence.

## In-process concurrency

**Level:** Must-have

**You can:** tell a race, a lock, and async I/O apart, and pick a design that shares less mutable state.

Data concurrency (two transactions, one row) is a different topic. It is in [Data and APIs](04-data-and-apis.md).

### In one minute

A **thread** is one path of execution. A process can have many.

**Async** code starts some I/O (a query, an HTTP call) and releases the thread while it waits. Async makes waiting cheaper. It does not make a CPU-heavy loop faster. If the work is CPU, you need fewer heavier tasks, or more processes, not more `await` keywords.

A **race** happens when two threads read and write the same mutable state and the result depends on timing. The bug is rare in tests and common in production.

A **lock** lets one thread enter a block at a time. Locks are honest and local. They also wait, and they deadlock.

A **deadlock** is a cycle of waits. Thread A holds lock 1 and wants lock 2. Thread B holds lock 2 and wants lock 1. Neither proceeds.

The design that ages well is: do not share mutable state. Hand work to a single owner through a queue, or pass immutable values.

### When it matters

- A counter of "requests in progress" is sometimes wrong.
- A cache dictionary throws or loses entries under load.
- A request handler blocks on a lock that another request holds while it calls the network.

### What it costs

Locks are simple and can serialize your throughput. Lock-free structures are faster in narrow cases and much easier to get wrong. Queues add latency and a place for backlog. Say which one you are choosing.

### A failure you should recognize

Two requests update a shared `Dictionary` with no synchronization. Most of the time it works. On a busy hour it throws, or it loses an entry. The fix is a concurrent dictionary, a lock around a private dictionary, or (better) no shared mutable cache on the request path.

Blocking a thread-pool thread on async work can starve the pool: the blocked thread is the one that was supposed to finish the async operation. On .NET this shows up when someone calls `.Result` or `.Wait()` on a task in application code. The appendix has the platform note.

### Learn

Start here: your runtime's threading basics. For .NET, [Managed threading basics](https://learn.microsoft.com/en-us/dotnet/standard/threading/managed-threading-basics).

### Review

1. What does async change about threads, and what does it leave unchanged about CPU work?
2. Draw a two-lock deadlock in words.
3. Why is "it passed the tests" a weak answer when the bug is a race?

### In an interview

"This handler updates a shared cache. Is that safe?" A senior answer asks who else writes it, and then either confines the writes or protects them. They also mention the lock hold time if a lock is the fix.

### Exercise

Increment a shared integer from two threads, a million times each, with no lock. Print the result. Then protect the increment. Explain why the first number is short.
