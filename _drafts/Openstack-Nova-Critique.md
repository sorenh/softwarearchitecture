---
layout: post
title:  "OpenStack Nova - Critique"
date:   2016-10-13 11:49:23
tags:
  - critique
  - openstack
  - nova
---
> OpenStack Compute (Nova) is a cloud computing fabric controller, which is the
> main part of an IaaS system. It is designed to manage and automate pools of
> computer resources and can work with widely available virtualization
> technologies, as well as bare metal and high-performance computing (HPC)
> configurations.

(Source: <https://en.wikipedia.org/wiki/OpenStack#Compute_.28Nova.29>)

It has many components, typically spread across dozens, hundreds or even thousands of nodes. Clearly, Nova is a highly distributed system. Is it scalable, though?

Let's consider what that means.

From [What does scalable mean]({% post_url 2016-10-03-What-does-scalable-mean %}) we have these questions:

* Is the software able to elegantly handle the volume of data it has acquired or will acquire over time?

  The default datastore for Nova is MySQL. MySQL is known to handle huge volumes of data. I foresee no problems with regards to storing the data, even for very large deployments.

  "Are there operations that become prohibitively slower as the data volume grows?"

  Let's examine a common operation: Running a new virtual machine. See this patch for detailed analysis. In summary, these are the steps:
  1. Fetch capacity information about all nodes in the cluster from the database.
  2. Filter the nodes to remove unusable ones (wrong CPU type, insufficient capacity, etc).
  3. Analyze each node in order to assign a preference (or "weight") to each one.
  4. Sort the list of nodes according to the assigned weights.
  5. Pick the most preferred node.

  [Typical time complexity analysis]({% post_url 2016-10-06-Time-complexity %}) teaches us that step no. 4 is the worst one, since it sorts a list[1], an O(n log n) operation. However, for the sorting algorithm to be more time consuming than fetching the data and calculating the weights, the number of nodes would have to be well into the zillions.

  So, in practice, we're dealing with an operation that scales linearly for any reasonable input size. Let's examine further.

  Fetching the data can be quite expensive. This is a well known problem[1]. With the current (October 2016) schema, a ComputeNode row in the db weighs in at around 160 bytes. With a 15,000 node deployment, this is 2.5MB of data which first has to be assembled on the DB side and then turned into domain objects on the Nova side. To add insult to injury, if the scheduler does not already have capacity information for a given compute node, a new request is made to the database to get this information. That's a DB request for each compute node. This happens if the given compute node has not yet sent capacity information, or if that particular functionality has been disabled. I'm not sure how common this is.

  Once all this data has been collected, the scheduler filters the hosts and assigns a weight to them. Finally, it sorts them and chooses the top one. It sends the request to the chosen compute node. Should the compute node in the mean time have run out of capacity, it may reject the request and the scheduling process starts over. Naturally, if there are more than one scheduler (or multiple threads on the one scheduler), this is fairly likely as they will all choose the same host.

  In an effort to mitigate this race, `scheduler_host_subset_size` may be increased. If set to e.g. `20`, rather than picking the single best compute node, the scheduler will pick a random one of the 20 best ones. The default value, however, is `1`.

  With the default scheduler, all of the above happens for every single request to run new virtual machines and the whole thing (assuming no retries, and no parallel scheduling requests) can take a whole minute for a 10,000 node deployment.

  RightScale estimated the number of instances launched on EC2 per day back in 2009[2] to be above 50,000. It's safe to assume that this number is much higher today, but let's just use that number. 50,000 a day is one every couple of seconds. If each request took a minute to complete, there would be around 30 in progress at any given time. 30 requests that all fetch the same data, apply the same logic, and likely choose the same hosts as their top candidates. Even with `scheduler_host_subset_size = 1000`, there would still be a 35% chance of a collision[3], causing a retry which just adds to the pile of concurrent scheduling requests.

  In conclusion: Yes, there most certainly are operations that become prohibitively slower as the data volume grows.

  Also, if you find it acceptable to run someone's virtual machine on the not just the second or third best, but 1000th best, does it really matter at all where you run it? Why not just pick *any* host that has capacity?

  [1]: "*Querying the DB is the most expensive part of the scheduling process.*" (from <http://specs.openstack.org/openstack/nova-specs/specs/backlog/approved/parallel-scheduler.html>)

  [2]: http://www.rightscale.com/blog/cloud-industry-insights/amazon-usage-estimates

  [3]: https://en.wikipedia.org/wiki/Birthday_problem

  Assigning the weights is relatively cheap. It involves iterating over the nodes and applying simple arithmetic.

* "Is the software able to keep serving requests if a component fails?"

  This is pretty simple: No. The default datastore, MySQL, favors consistency over availability. Should it crash or otherwise go offline, Nova will be unable to serve many types of requests.
