# Appendix: .NET runtime

**Level:** Good-to-have unless you work on .NET. Then it is part of your platform craft.

**You can:** describe the .NET garbage collector at a useful level, avoid the `HttpClient` socket leak, and say which runtime you would start a new service on in October 2026.

The language and runtime track explains stack, heap, and generational collection without a vendor. This page is the worked example.

## Which runtime to target

As of October 2026, **.NET 10** is the current long-term support release. Microsoft's support policy puts its end of support in **November 2028**.

**.NET 8** (the previous LTS) and **.NET 9** (standard-term support) both reach end of support in **November 2026**. They still receive patches until that date. A new service should start on .NET 10. An existing service on 8 or 9 should have an upgrade plan, not a hope.

Support dates move only when Microsoft moves them. Check the [official support policy](https://dotnet.microsoft.com/en-us/platform/support/policy/dotnet-core) before you quote a date in a design doc. This page was checked in October 2026.

ASP.NET Core's [performance best practices](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/best-practices) are worth a pass when you are about to ship a hot path. They change with the framework. Prefer them over a blog post from a single version.

## Garbage collection on .NET

**You can:** say what the collector is doing, which mode your process is in, and which generation a long-lived cache lands in.

.NET uses a generational collector. New small objects go in generation 0. Survivors are promoted. Generation 2 holds long-lived objects. Collecting generation 0 is common and usually cheap. Collecting generation 2 is rarer and more expensive, and it is where a leak shows up as a process that always grows.

The [fundamentals page](https://learn.microsoft.com/en-us/dotnet/standard/garbage-collection/fundamentals) is the map: roots, generations, and the fact that a leak is a root you still hold (a static collection, a cache with no bound, an event subscription, a timer). It does not teach the modes below. Read those pages when the map stops. Allocating on a hot path creates generation 0 pressure. It matters at high QPS. It is rarely the first bug. An unbounded cache is the first bug.

### Workstation GC and server GC

[Workstation GC](https://learn.microsoft.com/en-us/dotnet/standard/garbage-collection/workstation-server-gc) is the default for a standalone app. The collection runs on the thread that triggered it, at that thread's priority, and it competes with the rest of the process for CPU. A machine with one logical CPU always uses workstation GC.

[Server GC](https://learn.microsoft.com/en-us/dotnet/standard/garbage-collection/workstation-server-gc) is built for throughput. It keeps a heap per logical CPU and collects those heaps together on dedicated threads. The same live set is collected faster than workstation GC would collect it, and the process uses more memory to get that throughput. Many server-GC processes on a small machine will fight each other for cores.

The host chooses. The ASP.NET Core web SDK (`Microsoft.NET.Sdk.Web`) requests server GC when the project has not set the property, and the [ASP.NET Core memory page](https://learn.microsoft.com/en-us/aspnet/core/performance/memory?view=aspnetcore-10.0) calls server GC the default for these apps. A console app on the base SDK stays on workstation GC until the project sets `ServerGarbageCollection`. A library does not get to choose. On a single logical CPU the runtime uses workstation GC anyway. Do not flip the mode because a screenshot said so. The [config page](https://learn.microsoft.com/en-us/dotnet/core/runtime-config/garbage-collector) is where the switch lives (`System.GC.Server`).

That same ASP.NET page is the worked comparison. Under one load, server GC held a working set near 500 MB and workstation GC held about 70 MB, with collections much more often. On a machine where CPU is the scarce resource, server GC is the usual choice. In a dense set of small containers, where memory is the scarce resource, workstation GC can be the better fit. Measure before you switch.

### Background collection

[Background GC](https://learn.microsoft.com/en-us/dotnet/standard/garbage-collection/background-gc) is the default. Generation 2 can be collected on dedicated threads while managed threads keep allocating. Generations 0 and 1 are still foreground collections: they suspend managed threads. "Background" means the expensive generation can overlap with your work. It does not mean a request never pauses. During a background generation 2 collection, a generation 0 or 1 collection can still stop the world; the background work waits for that foreground pass and then continues.

### Large object heap

An object of **85,000 bytes or more** is large. The runtime allocates it on the large object heap (the LOH), which is collected only when generation 2 is collected. A `byte[]` that large is the usual case. The threshold is the documented default on the [config page](https://learn.microsoft.com/en-us/dotnet/core/runtime-config/garbage-collector), and a config may only raise it.

Moving a large object is expensive, so a collection of the LOH sweeps dead objects into a free list instead of compacting on every pass. You can ask for compaction on the next full blocking collection with `GCSettings.LargeObjectHeapCompactionMode`. The runtime may also compact on its own. The [ASP.NET Core memory page](https://learn.microsoft.com/en-us/aspnet/core/performance/memory?view=aspnetcore-10.0) adds the container rule: on .NET Core 3.0 and later, the LOH in a container is compacted automatically. The [large object heap](https://learn.microsoft.com/en-us/dotnet/standard/garbage-collection/large-object-heap) page is the write-up of the sweep policy. Its title scopes the long walkthrough to Windows. The 85,000-byte rule is the runtime default either way.

Allocating a large object also clears that memory, so the allocate itself costs. A hot path that builds a new large array per request pays that cost and triggers generation 2 collections. The ASP.NET page shows the cliff with a picture: a `byte[84975]` stays on the small-object heap, and one more byte forces generation 2 collections and about half the throughput. The array's own overhead is why the cliff is not exactly 85,000. Pool the buffer (`ArrayPool<T>.Shared`) and return it in a `finally`.

A pinned object cannot be moved, so a long pin in a compacting generation leaves a hole. Pool the buffers you hand to native I/O. The config page lists a third heap, the pinned object heap, next to the small and large ones.

### DATAS, on by default since .NET 9

Server GC, left alone, grows the heap to protect throughput and behaves as if the process owns the machine. [DATAS](https://learn.microsoft.com/en-us/dotnet/standard/garbage-collection/datas) (dynamic adaptation to application sizes) pulls the heap back toward the size of the long-lived data and changes how many heaps it uses. It was opt-in on .NET 8. The [config page](https://learn.microsoft.com/en-us/dotnet/core/runtime-config/garbage-collector) says it is enabled by default starting in .NET 9, so a new .NET 10 service has it. It starts with one heap. A cold process, or a sudden peak, can stall allocations for a few collections while the heap count grows. Memory graphs after an upgrade to .NET 10 often drop for that reason. That drop is the feature. Turn DATAS off only after a measurement, on a machine where this process is supposed to keep the memory. Maoni Stephens (the GC architect) wrote the upgrade note: [Preparing for the .NET 10 GC](https://devblogs.microsoft.com/dotnet/preparing-for-dotnet-10-gc/).

Older posts that talk about giant GC segments are describing the layout from before .NET 7. On 64-bit Windows and Linux the heap has used regions since then. The config page is the current description.

### Where a longer study goes

- The manual pages linked above, one sitting each. Fundamentals first, then workstation and server, background GC, the large object heap, DATAS.
- Maoni Stephens' [.NET memory performance analysis](https://github.com/Maoni0/mem-doc/blob/master/doc/.NETMemoryPerformanceAnalysis.md), when you have a pause or a heap to explain. This is the note that teaches you to read a collection, which the fundamentals page does not.
- Konrad Kokosa, *Pro .NET Memory Management*, when you want the book-length .NET model. Jones, Hosking, and Moss remain the language-neutral book, named in the runtime track.

### Review

1. Why can a .NET process leak memory?
2. What size of object is associated with the large object heap, and why is pooling the usual fix for a temporary one?
3. What does server GC spend memory on, and who turns it on for an ASP.NET Core app?
4. What does DATAS change about a server-GC process that just moved to .NET 10?

## HttpClient and sockets

**You can:** share clients safely, and explain the old port-exhaustion bug without reintroducing it.

`HttpClient` is safe to use for many calls, and it is the wrong shape to `new` up for every request and then dispose immediately. Disposing it on every call forces the underlying sockets through TIME_WAIT and, under load, the process runs out of ephemeral ports. The symptom looks like random outbound connection failures.

The current guidance is:

- Use [`IHttpClientFactory`](https://learn.microsoft.com/en-us/dotnet/core/extensions/httpclient-factory) in ASP.NET Core and in worker services. You register a named or typed client. The factory manages handler lifetime, including DNS changes, which a single process-long static `HttpClient` can miss.
- For a simple console tool, one long-lived `HttpClient` is fine.
- Set a timeout on the client or on the request. The default is 100 seconds, which is a long time to occupy a thread-pool waiter if you also block.

The [HttpClient guidelines](https://learn.microsoft.com/en-us/dotnet/fundamentals/networking/http/httpclient-guidelines) are the page to trust when a blog and this appendix disagree. Guidelines get revised. The failure mode (a client per request, then dispose) is the part that does not go out of date.

### Review

1. What fails when every request creates and disposes an `HttpClient`?
2. What does `IHttpClientFactory` manage that a careless static client can get wrong?
3. Where do you set a timeout, and why is "the default" not a design?

## Async pitfalls

**You can:** say what `await` does to a thread, and avoid blocking a pool thread on an incomplete task.

`await` compiles to a state machine. The method returns a task, the wait releases the thread, and the continuation runs later when the operation completes. Waiting gets cheaper. CPU work between the awaits still occupies a pool thread for as long as it runs. A CPU-bound loop with `await` sprinkled through it is still a CPU-bound loop.

In application code, do not call `.Result`, `.Wait()`, or `GetAwaiter().GetResult()` on a task that may still be running.

- On classic ASP.NET the request had a synchronization context. The continuation needed that context, and the blocked request thread was holding it, so the call could deadlock.
- ASP.NET Core does not install that context. The same call usually does not deadlock. It still holds a pool thread that other completions need. Under load the pool grows or requests queue. One call in a unit test does not show it. The diagnostic page is [debug thread-pool starvation](https://learn.microsoft.com/en-us/dotnet/core/diagnostics/debug-threadpool-starvation).

`ConfigureAwait(false)` tells a library await not to resume on a captured context. In application code on ASP.NET Core there is no context to avoid, so sprinkling it does not fix starvation. Stephen Toub's [ConfigureAwait FAQ](https://devblogs.microsoft.com/dotnet/configureawait-faq/) is the page that settles the arguments.

`async void` is for event handlers. It swallows a place to observe the exception. Application methods return `Task` or `ValueTask`.

[`ValueTask`](https://devblogs.microsoft.com/dotnet/understanding-the-whys-whats-and-whens-of-valuetask/) avoids an allocation when the result is often already complete, as a cache hit is. Await a given `ValueTask` once. Do not block on it, and do not store it to await later.

The article that matches a long study of the feature is Stephen Toub's [How async/await really works](https://devblogs.microsoft.com/dotnet/how-async-await-really-works/): the state machine, the builder, and where the thread actually goes. [Managed threading basics](https://learn.microsoft.com/en-us/dotnet/standard/threading/managed-threading-basics) does not contain that. David Fowler's [ASP.NET Core diagnostic scenarios](https://github.com/davidfowl/AspNetCoreDiagnosticScenarios) is the catalog of the ways the pitfall shows up in a real host, including sync-over-async and a client timeout that is long enough to pin the pool.

### Review

1. What replaces `.Result` in application code?
2. Why can the same `.Result` deadlock on classic ASP.NET and starve the pool on ASP.NET Core?
3. When is `ValueTask` the right return, and how many times may you await one?

## Object pooling

**You can:** reuse an expensive short-lived object when allocation shows up in a profile, and not before.

`ArrayPool<byte>` is the common case for buffers. `Microsoft.Extensions.ObjectPool` can pool larger objects you reset between uses. A pool without a bound, or a pooled object you forget to return, is a leak with extra steps. Profile first. The GC page above is the context. The [object pool documentation](https://learn.microsoft.com/en-us/aspnet/core/performance/objectpool) is the API reference.

## Exercise

Register a typed `HttpClient` with a 2-second timeout and a named base address. Call it from a handler that is itself async from the controller down. Confirm in a review of your own code that nothing on that path calls `.Result`.

## Interview cue

If the role is .NET, one runtime story is enough: the `HttpClient` lifetime, or a generation 2 leak you tracked to a static collection. Pair it with a heap snapshot or a metric, not with a list of every GC flavor.
