---
layout: post
title:  "DZone: Control Downtime With Application Dependency Maps"
date:   2016-10-03 14:25:11
---
[Sridhar Iyengar](https://dzone.com/users/2876920/sridhar-iyengar.html) over at [DZone](https://dzone.com) just posted [an article](https://dzone.com/articles/control-downtime-with-application-dependency-maps) about the value of mapping out the dependencies of your applications.

> Application downtime is an enterprise’s worst nightmare. On top of
> that, it’s fairly common. According to a survey conducted by Dun &
> Bradstreet, 59 percent of Fortune 500 companies experience a minimum
> of 1.6 hours of downtime per week. Each time a business application
> fails, the IT admins have a herculean task ahead of them. They must
> figure out the root cause of the problem, rectify it and get the
> business running—all within a short span of time.

This is very true.

He mentions "automated dependency maps" which sounds questionable. I'm not sure how you'd automatically determine all of an application's dependencies. Some are in configuration files, some are listed in various databases, some depend on data passed in by a client, etc. If I wrote an application myself, how would another system know how to identify my dependencies?

Regardless, the wisdom of creating a map like this holds true. It will help you find root cause as well as rule out alternative diagnoses.
