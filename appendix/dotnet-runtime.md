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

**You can:** say what the collector is doing, and which generation a long-lived cache lands in.

.NET uses a generational, compacting garbage collector. New objects go in generation 0. Survivors are promoted. Generation 2 holds long-lived objects. Collecting generation 0 is common and usually cheap. Collecting generation 2 is rarer and more expensive, and it is where a leak shows up as a process that always grows.

The [fundamentals page](https://learn.microsoft.com/en-us/dotnet/standard/garbage-collection/fundamentals) is the source. A few facts that are easy to get wrong:

- The collector reclaims objects that are not reachable from roots. A "leak" is a root you still hold: a static collection, a cache with no bound, an event subscription, a timer.
- Server GC and workstation GC are different modes. Server processes usually want server GC, which is the default for ASP.NET Core. Do not switch modes because a screenshot said so. Read the current doc.
- Allocating on a hot path creates generation 0 pressure. It matters at high QPS. It is rarely the first bug. An unbounded cache is the first bug.

### Large object heap

Objects of **85,000 bytes or more** go on the large object heap (the LOH), with the generation 2 lifetime. A `byte[]` that large is a typical resident. The LOH is collected with generation 2. Large arrays that you allocate per request will hurt.

Prefer pooled buffers (`ArrayPool<T>.Shared`) for big temporary buffers, and return them in a `finally`. The [large object heap](https://learn.microsoft.com/en-us/dotnet/standard/garbage-collection/large-object-heap) page is the source for the threshold. Confirm the threshold in the docs if you are on a new runtime. It has been 85,000 bytes for a long time, and it is still the documented threshold.

### Review

1. Why can a .NET process leak memory?
2. What size of object is associated with the large object heap?
3. What do you pool, instead of allocating, on a hot path that needs a big buffer?

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

**You can:** avoid blocking a thread-pool thread on an incomplete task.

`await` the task. In application code, do not call `.Result`, `.Wait()`, or `GetAwaiter().GetResult()` on a task that may still be running. On the classic ASP.NET request context this could deadlock. On ASP.NET Core the deadlock is less common and the starvation is still real: you block a pool thread that the completion needs, and under load the pool grows or requests queue.

`async void` is for event handlers. It swallows a place to observe the exception. Application methods return `Task` or `ValueTask`.

### Review

1. What replaces `.Result` in application code?
2. Why is blocking a pool thread worse under load than it looks in a unit test?

## Object pooling

**You can:** reuse an expensive short-lived object when allocation shows up in a profile, and not before.

`ArrayPool<byte>` is the common case for buffers. `Microsoft.Extensions.ObjectPool` can pool larger objects you reset between uses. A pool without a bound, or a pooled object you forget to return, is a leak with extra steps. Profile first. The GC page above is the context. The [object pool documentation](https://learn.microsoft.com/en-us/aspnet/core/performance/objectpool) is the API reference.

## Exercise

Register a typed `HttpClient` with a 2-second timeout and a named base address. Call it from a handler that is itself async from the controller down. Confirm in a review of your own code that nothing on that path calls `.Result`.

## Interview cue

If the role is .NET, one runtime story is enough: the `HttpClient` lifetime, or a generation 2 leak you tracked to a static collection. Pair it with a heap snapshot or a metric, not with a list of every GC flavor.
