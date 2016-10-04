---
layout: post
title:  "Distributed systems - Introduction"
date:   2016-10-04 12:39:41
tags:
  - academy
  - distributed systems
  - introduction
---
A lot of what I'll be covering on this site is going to be about distributed systems. A distributed system is a system that involves more than one server. Some services have multiple components that all run on a single server. I consider those "complex", because they're composed of multiple components, but they're not "distributed".

I think most services and applications today are distributed. Even if it's a service with a single component running on a single server, if you communicate with it over the Internet, then it's distributed.

When you're go from running an application on a single server to running it on several servers, a lot of assumptions become invalid.

**Example**:

> Server A sends three messages to server B, but the middle one is lost in transit. Server B does not know that anything is missing.

This could not happen on a single system. Things don't get lost that way.


**Example**:

> A client sends two updates. One ends up on server A, the other on server B. They have the same timestamp, so there's no way to know which one the client sent last.

or worse:

> A client sends two updates, 2 seconds apart. The first ends up on server A, the second on server B. Server A's clock is 5 seconds ahead of server B, so when B receives its update, it gets rejected, because server A already has an update with a newer timestamp.

One a single system, there are many ways to determine which of two actions happened first. In a distributed system, you can come close, but to be sure you have to have a single point of authority. That's why leader election (or "consensus") protocols like [Paxos](https://en.wikipedia.org/wiki/Paxos_(computer_science)) and [Raft](https://en.wikipedia.org/wiki/Raft_(computer_science)) are a big deal.


**Example**:

> Two servers, running different components of a distributed system. One server catches fire. The other is fine.

If a server running all components of a complex system crashes, everything crashes, so each component does not need to handle partial failures.


**Example**:

> If a component is not reachable, it may still be running.

It's remarkable common for network connections to fail. On a single system, you can know for certain whether a given component is running or not. In a distributed system, you may not be able to reach the other components, even though they're still running... And just because you can't reach them doesn't mean that noone else can.


## This sounds terrible and impossible. Why do you bother?

Since distributing things sounds like a hassle, why not just use larger servers, instead of spreading across multiple servers?

Well, most of us are limited to what is commercially available. You can't (as of October 2016) go to [Dell.com](http://www.dell.com/) and order a server with 28TB of RAM, 4PB of storage and 4096 cores. That puts an upper limit on what scale you can achieve. Not good.

Also, what if your single, huge server fails?

With the right distributed architecture, you can deliver services that are highly available, reliable, and fast. It's not easy, but it's worth it.
