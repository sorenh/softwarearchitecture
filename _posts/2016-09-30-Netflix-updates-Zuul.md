---
layout: post
title:  "Netflix updates Zuul"
date:   2016-09-30 21:13:35
---
Netflix just updated their http gateway, [Zuul](https://github.com/Netflix/zuul).

> The major architectural difference between Zuul 2 and the original
> is that Zuul 2 is running on an asynchronous and non-blocking
> framework [...]

Let's explore what that means.

The original Zuul used a synchronous, blocking, thread based model. When a request came in, a thread would be dedicated to handling this request and nothing else. It would pass on the request to the appropriate backend, wait for the response, and send it back to the client. It's relatively simple, since each thread only knows about the request it's handling.

An asynchronous, non-blocking model handles all connections in a single thread. It's more delicate, because you have to carefully track all client connections, which backend connection they correspond to, be careful never to block, etc.

So why bother? Scalability, of course!

Only one process or thread can run on a CPU core at any given time, so switching from handling one request to handling another involves a context switch.

> Switching from one process to another requires a certain amount of
> time for doing the administration – saving and loading registers
> and memory maps, updating various tables and lists, etc.

(Source: <https://en.wikipedia.org/wiki/Context_switch#Cost>)

If all requests are handled in the same thread, you can avoid context switching entirely. This has the potiential for a massive performance boost.

With the threaded model, if a backend takes longer to respond, the threads will liver longer. If they live longer, but they're still being created at the same rate, the number of threads could explode, leading to service downtime.

It sounds like a no-brainer, right? Well, not entirely.

* In a multi-threaded model, if a thread crashes, it usually does not affect the other threads.
* If you only have one thread you must be very careful not to ever block. This is trickier than it sounds:
  * All kinds of I/O can block.
    * Network I/O is pretty obvious.
    * Filesystem I/O can also block, especially under heavy load!
  * Computations can also "block".

    We don't usually refer to computations as "blocking", but the effect is the same: If the thread is busy calculating checksums, comparing URLs to a long list of regular expressions, etc., it can't do anything else.
  
Read the full story on [the Netflix Engineering blog](http://techblog.netflix.com/2016/09/zuul-2-netflix-journey-to-asynchronous.html). They do a great job explaining the different models, the motivation, the results, etc.
