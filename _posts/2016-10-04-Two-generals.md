---
layout: post
title:  "Two generals' problem"
date:   2016-10-04 22:28:53
tags:
  - academy
  - distributed systems
  - two generals
---
The two generals problem helps illustrate why agreeing on anything at all in a distributed system is really, really hard.

Imagine a city with an enemy army on either side. The armies want to conquer the city, but they're outnumbered. To win the battle, they must attack at the same time. If they don't, it's certain death.

The story is set in the middle ages, so there are no phones. The only way the two generals can communicate is by sending a messenger to the other side through enemy territory.

Somehow, they must agree on a time to attack. How?

* Have general A decide and simply send the designated time by messenger to the other side?

  **No**. General A could not be sure that the message was received by the other army. If they didn't get the messages, they would not show up and General A would have sent his army to a battle they would certainly lose.

* Ok, so just have general A decide, send the message across and return a confirmation message?

  Again, **no**. General B could not be certain that his confirmation made it across. If general A does not receive the confirmation, he would not turn up for the battle.

* Hm. Ok, so just send one more confirmation?

  **No**... Even if you send a thousand confirmations and reconfirmations, there will always be uncertainty about the final one... The only one that actually matters.

It turns out that there actually is no way to solve this problem, and this is basically why we can't have nice things.
