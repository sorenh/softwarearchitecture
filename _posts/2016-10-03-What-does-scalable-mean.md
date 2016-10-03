---
layout: post
title:  "What does \"scalable\" mean anyway?"
date:   2016-10-03 12:18:02
tags:   academy scalable definition
---
If we're going to talk about **scalable software architecture**, we need to agree on what that even means.

Rather than a formal definition, here are three questions that I think cover it pretty well:

* Is the software able to elegantly handle the volume of data it has acquired or will acquire over time?  As your data set grows, can you still respond to requests quickly enough?

* Is the software able to handle the rate of change?  As more more users send you more and more data, can you still process it quickly enough?

* Is the software able to keep serving requests if a component fails?  As your service grows, you'll add more nodes. The more nodes, the more likely something fails. Will this cause downtime?


I find that discussions on "scalability" often focus on one or two of these aspects, but rarely all three.

[MySQL](https://www.mysql.com/), for instance, handles [mind boggling large data sets](https://gigaom.com/2011/12/06/facebook-shares-some-secrets-on-making-mysql-scale/) with relative ease. If that aspect of scale is your only concern, a large MySQL server is a fantastic choice. If serving requests even if a component (like e.g. your MySQL server) fails is your primary concern, a large MySQL server is not a very good choice.

So is MySQL scalable? It depends.
